## CapRover setup

### Obtain a domain name

In order to deploy the CODA platform sandbox using CapRover, you will need a registered domain name and access to DNS settings. Throughout this guide, we will use `coda-platform.com` as the example base domain.

### Provision virtual machines

The first step in deployment is to create the virtual machines (VMs) that will host the hub and site nodes. Throughout this guide, we will assume two VMs:

- One VM for the hub components (`hub`)
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
