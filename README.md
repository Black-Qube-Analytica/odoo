# Introduction

This repo helps you to quickly host your own cloud instance of Odoo. Odoo is a suite of business management software which includes: CRM, Invoicing and more. Find the complete list of apps on Odoo's official [website](https://www.odoo.com/).

## Prerequisites

1. Ubuntu 20.04 server
2. A domain and three A records, crm.your_domain and monitor.your_domain. Each should point to the IP address of your server
3. Docker installed on the server
4. Docker Compose installed

## Step-by-step guide

1; Create apps directory that will have two directories inside - odoo and net

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

4; The set-up uses Traefik as a reverse proxy. Before you get the Traefik container up and running, set up an encrypted password for a monitoring dashboard. Read more about the Traefik dashboard [here](https://doc.traefik.io/traefik/operations/dashboard/). We can use htpasswd for this. First, install the utility, which is included in the apache2-utils package:

```bash
sudo apt-get install apache2-utils
```

5; Generate the password with htpasswd

```bash
htpasswd -nb admin some_secure_password_here
```

6; The output will look something like below:

```bash
admin:$apr1$ruca84Hq$mbjdMZBAG.KWn7vfN/SNK/
```

7; Open traefik_dynamic.toml and past it there:

```bash
cd ../net && nano traefik_dynamic.toml
```

traefik_dynamic.toml will look like below with the password encrypted

```bash
[http.middlewares.simpleAuth.basicAuth]
  users = [
    "admin:$apr1$ruca84Hq$mbjdMZBAG.KWn7vfN/SNK/"
  ]

[http.routers.api]
  rule = "Host(`monitor.domain.com`)"
  entrypoints = ["websecure"]
  middlewares = ["simpleAuth"]
  service = "api@internal"
  [http.routers.api.tls]
    certResolver = "lets-encrypt"
```

8; Create an empty file that will hold Let’s Encrypt information in the net dir. You’ll share this into the container so Traefik can use it:

```bash
touch acme.json
```

9; Traefik will only be able to use this file if the root user inside of the container has unique read and write access to it. To do this, lock down the permissions on acme.json so that only the owner of the file has read and write permission.

```bash
chmod 600 acme.json
```

10; Create a Docker network for the proxy to share with containers.

```bash
docker network create web
```

11; Create the Traefik container.

```bash
docker run -d \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v $PWD/traefik.toml:/traefik.toml \
  -v $PWD/traefik_dynamic.toml:/traefik_dynamic.toml \
  -v $PWD/acme.json:/acme.json \
  -p 80:80 \
  -p 443:443 \
  --network web \
  --name traefik \
  traefik:v2.2
```

12; Go to odoo directory

```bash
cd ../odoo
```

13; Provide your environment variables in the .env

```bash
mv .env.example .env && nano .env
```

14; Change your PostgreSQL database password

```bash
nano odoo_pg_pass
```

15; Spin up the servers

```bash
docker-compose up -d