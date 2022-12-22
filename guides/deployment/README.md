# CODA Deployment Guide

## Provision machines

### Recommended minimal specifications

**Hub VM recommended minimal specs**

- 4 cores and 8GB RAM
- Disks: OS - 100 GB, data - 512 GB
- Data: OS - 100 GB, data - 512 GB<sup>*</sup>

**Site node recommended minimal specs**

- 8 cores and 16 GB RAM
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

### Obtain a domain name

In order to deploy the CODA platform, you will need a registered domain name and access to DNS settings. Throughout this guide, we will use `coda-platform.com` as the example base domain.

### Provision virtual machines

The first step in deployment is to create the virtual machines (VMs) that will host the hub and site nodes. Throughout this guide, we will assume two VMs:

- One VM for the hub compoennts (`hub`)
- One VM for an example site node (`site1`)

VMs should run a *NIX distribution (the platform is developed and tested on Ubuntu 18.04). 

### Install CapRover on each VM

Once the VMs are created, follow [instructions](https://caprover.com/docs/get-started.html) to install CapRover on each machine. As an alternative, you can deploy a [VM with CapRover pre-installed](https://cloud.digitalocean.com/droplets/new?image=caprover-18-04&app=caprover&onboarding_origin=marketplace&appId=93379849&refcode=6410aa23d3f3) on DigitalOcean.

> Note: While it is possible to deploy CapRover without a domain name, the setup procedure is more complex. Please follow [these instructions](https://caprover.com/docs/run-locally.html).

### Verify access to CapRover is working

Obtain the IP address for the `hub` and `site1` VMs. Point your browser to port `3000` of the respective machines, in order to access the CapRover control panel, e.g. `https://XXX.XXX.XXX.XX:3000`. You should see the following screen:

<img src="https://i.postimg.cc/QNn6JycW/caprover-login.png" alt="CapRover Login" width="500"/>

Use the **default password** `captain42` to login to the CapRover control panel. Ensure you can access the CapRover control panel on both machines. You should see the following screen after logging in:

<img src="https://i.postimg.cc/VkwHJbj7/caprover-panel.png" alt="CapRover Panel" width="500"/>

> If CapRover is still deploying, you may see the following sequence of error messages: "Firewall passed", "Captain is not ready yet", "502 error", followed by a successful login screen. You may need to refresh several times, as the setup process may take ~5 minutes.

### Setup domain names on DNS

The next step is to setup your domain name DNS to pôint to the `hub` and `site1` VMs. For the following example, we will assume the root domain name is `coda-platform.com`, with the hub located at `hub.coda-platform.com` and the site located at `site1.coda-platform.com`.

Add the following records to your DNS settings, replacing `XXX.XX.XXX.XX` with the IP addresses of the newly created `hub` and `site1` VMs:

```
Host: *.hub
Type: A
TTL: 3600
IP: XXX.XX.XXX.XX
```

```
Host: *.site1
Type: A
TTL: 3600
IP: XXX.XX.XXX.XX
```

> Do not remove the asterisk in the host name.

### Setup domain names on CapRover

#### Hub VM

Go to the CapRover control panel for the `hub` VM (`http://<hub VM IP address>:3000`) and point the "CapRover root domain" to `hub.coda-platform.com`, replacing `coda-platform.com` with your domain name:

<img src="https://i.postimg.cc/VkwHJbj7/caprover-panel.png" alt="CapRover Hub Domain Setup" width="500"/>

You should see the following message:

<img src="https://i.postimg.cc/Kc60Ws8k/caprover-domain-success.png" alt="CapRover Hub Domain Setup" width="500"/>

<br>

> If you encounter an error message stating "Verification failed", the DNS settings are incorrect or have not had the time to propagate yet. DNS settings may take several hours to propagate.

Afer re-logging in to the control panel, click "Enable HTTPS" and follow the steps on screen:

<img src="https://i.postimg.cc/wB1JggW6/caprover-enable-https.png" alt="CapRover Hub HTTPS Setup" width="500"/>

Once HTTPS is enabled, click "Force HTTPS":

<img src="https://i.postimg.cc/X7nCXw21/caprover-force-https.png" alt="CapRover Hub HTTPS Setup" width="500"/>

#### Site VM

Finally, go to the CapRover control panel for the `site1` VM (`http://<site1 VM IP address>:3000`) and point the "CapRover root domain" to `site1.coda-platform.com`, replacing `coda-platform.com` with your domain name:

<img src="https://i.postimg.cc/nhkZrxQD/caprover-domain2.png" alt="CapRover Site Domain Setup" width="500"/>

Follow the same steps as for the hub: enable HTTPS, then force HTTPS.

### Deploy the orchestration hub

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

<img src="https://i.postimg.cc/NFDB5Dm7/auth-service-client-config.png" alt="CapRover CLI Login" width="500"/>

3. Create user called `test-user` in the `hub` realm as follows:

<img src="https://i.postimg.cc/Y9js1x5z/auth-service-user-config-0.png" alt="CapRover CLI Login" width="500"/>
<img src="https://i.postimg.cc/nrDSWhcs/auth-service-user-config-1.png" alt="CapRover CLI Login" width="500"/>
