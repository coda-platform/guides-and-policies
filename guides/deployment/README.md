# CODA Deployment Guide

## Provision machines

### Recommended minimal specifications

**Hub VM recommended minimal specs**

- 4 cores and 8GB RAM
- Disks: OS - 100 GB, data - 512 GB
- Data: OS - 100 GB, data - 512 GB<sup>*</sup>

**Site node recommended minimal specs"

- 8 cores and 16 GB RAM
- GPU: RTX 2080 or better NVIDIA-compatible GPU
- Disks: OS - 100 GB, data - 512 GB<sup>*</sup>

> * It is recommended to dedicate a separate block device (virtual disk) for data. Simply because it is best practice and it ensures an easier and more homogenous deployment across sites.

## Networking requirements

The hub node must be accessible from your site node by HTTPS protocol. To reach the hub, your site must permit outbound HTTPS
(TCP/443) connections from the site nodes to the correct CIDR blocks:

- 132.203.231.8/29
- 132.203.189.10/32

> Some institutional proxies and firewalls may not manage this type of communication correctly with default settings. If the site cannot communicate with the hub and maintain a connection, check with your site networking team to permit this kind of communication.

## Proxy address whitelist 

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
