# Setup Production Environment

## Topics

- [Setup Production Environment](#setup-production-environment)
  - [Topics](#topics)
    - [Install Prerequisites](#install-prerequisites)
    - [Setup Docker Swarm](#setup-docker-swarm)
    - [Setup Traefik](#setup-traefik)
    - [Setup Portainer](#setup-portainer)
    - [Setup MariaDB](#setup-mariadb)
    - [Setup Swarm CRON](#setup-swarm-cron)
    - [Setup ERPNext](#setup-erpnext)

### Install Prerequisites

These steps are required for production server.
Use files from `compose` directory.
Setup assumes you are using Ubuntu 24.04 based Linux distribution.

Update packages
```shell
apt-get update && apt-get dist-upgrade -y
```

Setup unattended upgrades
```shell
dpkg-reconfigure --priority=medium unattended-upgrades
```

Add non-root sudo user
```shell
adduser -D ubuntu
usermod -aG sudo ubuntu
curl -fsSL https://get.docker.com | bash
usermod -aG docker ubuntu
su - ubuntu
```

Clone this repo
```shell
git clone https://github.com/sumedha-niroshan/frappe-on-docker-with-gitops.git
cd frappe-on-docker-with-gitops
```

### Setup Docker Swarm

Initialize swarm
```shell
docker swarm init --advertise-addr=X.X.X.X
```

Note: Make sure the advertise-address does not change if you wish to add multiple nodes to this manager.

### Setup Traefik

Label the master node to install Traefik
```shell
docker node update --label-add traefik-public.traefik-public-certificates=true $(docker info -f '{{.Swarm.NodeID}}')
```

Set email and traefik domain
```shell
export EMAIL=larrydevops@gmail.com
export TRAEFIK_DOMAIN=tra.commquest.co.za
```

Set `HASHED_PASSWORD`
```shell
export HASHED_PASSWORD=$(openssl passwd -apr1)
Password: $ Enter your password here
Verifying - Password: $ Re Enter your password
```

Install Traefik
```shell
docker stack deploy -c compose/traefik.yml traefik
```

### Setup Portainer

Label the master node to install portainer
```shell
docker node update --label-add portainer.portainer-data=true $(docker info -f '{{.Swarm.NodeID}}')
```

Set portainer domain
```shell
export PORTAINER_DOMAIN=po.commquest.co.za
```

Install Portainer
```shell
docker stack deploy -c compose/portainer.yml portainer
```

### Setup MariaDB

- Go to Stacks > Add and create stack called `mariadb`
- Set `DB_PASSWORD` environment variable to set mariadb root password. Defaults to `admin`
- Use `compose/mariadb.yml` to create the stack

### Setup Swarm CRON

In case of docker setup there is no CRON scheduler running. It is needed to take periodic backups.

- Go to Stacks > Add and create stack called `swarm-cron`
- Use `compose/swarm-cron.yml` to create the stack
- Change the `TZ` environment variable as per your timezone

### Setup ERPNext

- `compose/erpnext.yml`: Use to create the `erpnext` stack. Set `VERSION` to version of choice. e.g. `v15.53.1`. Set `SITES` variable as list of sites quoted in back tick  (`` ` ``) and separated by comma (`,`). Example ``SITES=`one.example.com`,`two.example.com` ``. Set `BENCH_NAME` optionally in case of multiple benches, defaults to `erpnext`.
- `compose/configure-erpnext.yml`: Use to setup `sites/common_site_config.json`. Set `VERSION` and optionally `BENCH_NAME` environment variables.
- `compose/create-site.yml`: Use to create a site. Set `VERSION` and optionally `BENCH_NAME` environment variables. Change the command for site name, apps to be installed, admin password and db root password.
- `compose/backup-erpnext.yml`: Use to backup and push snapshots. Set environment variables mentioned in comments in the file.
