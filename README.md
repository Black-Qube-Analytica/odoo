# Introduction

This repo helps you to quickly host your own cloud instance of Odoo. Odoo is a suite of business management software which includes: CRM, Invoicing and more. Find the complete list of apps on Odoo's official [website](https://www.odoo.com/).

## Prerequisites

1. Ubuntu 20.04 server
2. A domain and three A records, crm.your_domain and monitor.your_domain. Each should point to the IP address of your server
3. Docker installed on the server
4. Docker Compose installed

### Step-by-step guide

1; Create apps directory that will have two dirs inside - odoo and net

```bash
mkdir -p apps/{odoo,net}
```

2; Clone the repo inside the odoo directory

```bash
cd apps/odoo && git clone https://github.com/Black-Qube-Analytica/odoo.git .
```

3; Move the net directory contents (found in the odoo dir) to net dir created above

```bash
mv net/* ../net/
```

4; Odoo uses Traefik as a reverse proxy. Before you get the Traefik container up and running, set up an encrypted password for the monitoring dashboard. We can use htpasswd for this. First, install the utility, which is included in the apache2-utils package:

```bash
sudo apt-get install apache2-utils
```
