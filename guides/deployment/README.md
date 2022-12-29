# CODA Deployment Guide

## Provision machines

### Minimal specifications

The following specifications represent the "bare minimum" resources required to run the CODA platform in sandbox mode. 

**Orchestration hub**

- 8 cores/vCPUs and 16 GB RAM
- Disks: OS - 100 GB, data - 512 GB
- Data: OS - 100 GB, data - 512 GB<sup>*</sup>

**Site nodes**

- 8 cores/vCPUs and 16 GB RAM
- GPU: RTX 2080 or better NVIDIA-compatible GPU
- Disks: OS - 100 GB, data - 512 GB<sup>*</sup>

**Disk partitioning**

It is recommended to dedicate a separate block device (virtual disk) for data, to ensure more homogenous deployments across sites. Execute `lsblk` on each virtual machine and check the output to get the list of blocks. If `lsblk^` utility is not found install it first by doing `yum install -y util-linux`.

## Networking requirements

## Basic networking requirements

The hub node must be accessible from your site node by HTTPS protocol. You can check whether this is the case using `wget https://google.com`. To reach the hub, your site must permit outbound HTTPS (TCP/443) connections from the site nodes to the correct CIDR blocks.

You will need to find the address of your **NTP servers** (to keep time in sync<sup>*</sup>) and **SMTP servers** (to enable automated system e-mail notifications). Both are optional, but recommended.

> * Necessary to keep all servers time in sync and provides accuracy while doing distributed analysis. Using more than one source is a best practice. If none is provided, deployment scripts will configure default sources outside your organization, outbound NTP traffic must be permitted for this to work correctly.

### Restricted network environments. 

For deployment in restricted networking environments, you may need to obtain the address of your **proxy server**, if Internet access is not permitted.

## Proxy address whitelist 

If you wish to manage proxy accesses with a strict whitelist, ensure the following ports are whitelisted:

<table
<tr><th>URL</th><th>Purpose</th></tr>

<tr><td>

http://mirror.centos.org
<br>
http://centos.mirror.iweb.ca/
<br>
http://fedora-epel.mirror.iweb.com/
</td><td> Installing software packages (yum)</td></tr>

<tr><td>
https://github.com
<br>
https://github-releases.githubusercontent.com
</td><td> Pulling deployment code.<br>
Pulling various artifacts (ie: node_exporter)</td></tr>

<tr><td>
https://license.aidbox.io
</td><td>Validating Aidbox Licence
</td></tr>

<tr><td>https://dl.min.io</td><td> Downloading Min.IO artifacts.</td></tr>
<tr><td>https://hub.docker.com
<br>
https://production.cloudflare.docker.com
<br>
https://registry-1.docker.io
<br>
https://auth.docker.io
</td> <td> Pulling Docker images for Aidbox software</td></tr>
<tr>
<td>
https://heartbeat.hub.coda19.com </td>
<td>Sending heartbeat to control plane (hub)</td>
</tr>
</table>

## Deployment methods

### A. Sandbox environment with Caprover

https://docs.google.com/document/d/1ab6548GrpJStVCa2X9lyzSRJLG71A09aw2KiJloyYcA/edit#

### B. Production environment with Ansible

[In progress]

## Deployment steps

### Prerequisites

#### Provision the hub and site VMs

The first step is to create the hub and site VMs, install CapRover on both, and configure DNS settings. Follow the [CapRover setup](https://github.com/coda-platform/guides-and-policies/blob/main/guides/deployment/caprover-setup.md) instructions.

#### Install Node, NPM, and CapRover CLI on your local machine

First, ensure you have NodeJS and NPM installed - if not, follow [these instructions](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm). Verify your installation by opening your terminal and running:

```
node -v
npm -v
```

Node version ≥ 16.13 and NPM version ≥ 8.19 are recommended. Next, run the following command to install the CapRover command-line utility tool on your local machine:

```
npm install -g caprover
```

### Deploy the orchestration hub

#### Login to hub VM using the CapRover CLI

Run the following command:

```
caprover login
```

Enter the following information:

- CapRover machine URL address: `https://captain.hub.your-domain-name.com`
- CapRover machine password: `captain42`
- CapRover machine name: `hub`

The result should look as follows:

<img src="https://i.postimg.cc/FRFfWv2b/caprover-cli-login.png" alt="CapRover CLI Login" width="500"/>

#### Clone and configure hub deployment scripts

1. Run the following commands to clone the hub deployment scripts, and install dependencies:

```
git clone git@github.com:coda-platform/hub-deployer.git
cd hub-deployer
npm install
```

2. Locally edit the configuration files (`.env` and files in subfolder `env/`) to suit your needs. For a default "sandbox" (insecure) install, you can simply modify `.env` and leave all the other configuration files as is.

3. Run the following command to setup and deploy the CapRover apps:


```
npm run deploy
```

### Deploy a site node

This will create a single hospital site. Creating additional sites follows the same procedure; new sites will automatically notify the hub of their presencer.

#### Login to the site using the CapRover CLI

Run the following command:

```
caprover login
```

Enter the following information:

- CapRover machine URL address: `https://captain.site1.your-domain-name.com`
- CapRover machine password: `captain42`
- CapRover machine name: `site1`

#### Clone and configure site deployment scripts

1. Run the following commands to clone the site deployment scripts, and install dependencies:

```
git clone git@github.com:coda-platform/site-deployer.git
cd site-deployer
npm install
```

2. Locally edit the configuration files (`.env` and files in subfolder `env/`) to suit your needs. For a default "sandbox" (insecure) install, you can simply modify `.env` and leave all the other configuration files as is.

3. Run the following command to setup and deploy the CapRover apps:

```
npm run deploy
```

#### Configure authentication service

1. Create a realm called `hub` configured as follows:

<img src="https://i.postimg.cc/HWwLq4cL/auth-service-hub-config.png" alt="CapRover CLI Login" width="500"/>

2. Create a client called `dashboard` configured as follows:

<img src="https://i.postimg.cc/44cTqJ0b/auth-service-client-config.png" alt="CapRover CLI Login" width="500"/>

3. Create user called `test-user` in the `hub` realm as follows:

<img src="https://i.postimg.cc/Y9js1x5z/auth-service-user-config-0.png" alt="CapRover CLI Login" width="500"/>
<img src="https://i.postimg.cc/nrDSWhcs/auth-service-user-config-1.png" alt="CapRover CLI Login" width="500"/>
