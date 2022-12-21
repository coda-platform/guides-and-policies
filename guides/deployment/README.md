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

Obtain the IP address for the `hub` and `site1` VMs. Point your browser to port `3000` of the respective machines, in order to access the CapRover control panel, e.g. `https://XXX.XXX.XXX.XX:3000`. Use the default password `captain42` to login to the CapRover control panel. Ensure you can access the CapRover control panel on both machines. You should see the following screen:

<img src="https://i.postimg.cc/QNn6JycW/caprover-login.png" alt="CapRover Login" width="500"/>

> If CapRover is still deploying, you may see the following sequence of error messages: "Firewall passed", "Captain is not ready yet", "502 error", followed by a successful login screen. You may need to refresh several times, as the setup process may take ~5 minutes.

### Setup domain names on CapRover
