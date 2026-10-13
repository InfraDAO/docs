---
description: 'Authors: [Vince | Nodeify]'
---

# 🐳 Docker

## System Requirements

<table data-full-width="false"><thead><tr><th align="center">CPU</th><th width="140" align="center">OS</th><th width="180" align="center">RAM</th><th align="center">DISK</th></tr></thead><tbody><tr><td align="center">4c/8t</td><td align="center">Ubuntu 22.04</td><td align="center">>= 16GB</td><td align="center">>= 13TB SSD/NVME</td></tr></tbody></table>

## Fantom 👻

Official Docs \
[https://docs.fantom.foundation/node/run-an-api-node#run-transaction-tracing-api-node](https://docs.fantom.foundation/node/run-an-api-node#run-transaction-tracing-api-node)

### Pre-requisites

Update, upgrade, and clean the system, and then firewall management (ufw), Docker, and the Git version control system.

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt install docker.io docker-compose git ufw -y
```

Set explicit default UFW rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Allow SSH, HTTP and HTTPS

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80
sudo ufw allow 443
```

Get the IP address of the host machine, you can use the following command in a terminal or command prompt

```bash
curl ifconfig.me
```

Set an A record for a domain, you need to access the domain's DNS settings and create an A record that points to the IP address of the host machine. This configuration allows users to reach your domain by resolving the domain name to the specific IP address associated with your host machine.

{% embed url="https://www.youtube.com/watch?v=QcNBLSSn8Vg" %}

### Create Fantom directory

The first command, `mkdir fantom`, will create a new directory named fantom in the current location. The second command, `cd fantom`, will change your current working directory to the newly created fantom directory. Now you are inside the fantom directory and can start storing docker-compose and related files in it.

```bash
mkdir fantom
cd fantom
```

### Create .env file

```bash
sudo nano .env
```

Paste the following into the file.

```bash
EMAIL={YOUR_EMAIL} #Your email to receive SSL renewal emails
DOMAIN={YOUR_DOMAIN} #Domain of your reth node you set earlier, reth.indexerdao.com
WHITELIST={YOUR_REMOTE_MACHINE_IP} # Remote IP's allowed to connect to RPC
```

{% hint style="info" %}
ctrl + x and y to save file
{% endhint %}

### Create docker-compose.yml

```bash
sudo nano docker-compose.yml
```

Paste the following into the docker-compose.yml

```docker
version: '3.8'

networks:
  monitor-net:
    driver: bridge

volumes:
    fantom_data: {}
    genesis_data: {}
    traefik_letsencrypt: {}
    

services:

######################################################################################
#####################         TRAEFIK PROXY CONTAINER          #######################
######################################################################################     

  traefik:
    image: traefik:latest
    container_name: traefik
    restart: always
    ports:
      - "443:443"
    networks:
      - monitor-net
    command:
      - "--api=true"
      - "--api.insecure=true"
      - "--api.dashboard=true"
      - "--log.level=DEBUG"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.myresolver.acme.tlschallenge=true"
      - "--certificatesresolvers.myresolver.acme.email=$EMAIL"
      - "--certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json"
    volumes:
      - "traefik_letsencrypt:/letsencrypt"
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
    labels:
      - "traefik.enable=true"
      - "traefik.http.middlewares.ipwhitelist.ipwhitelist.sourcerange=$WHITELIST"

######################################################################################
#####################            FANTOM ARCHIVE CONTAINER      #######################
###################################################################################### 

  fantom:
    image: kw1k/opera:1.1.2-rc.6
    container_name: fantom
    restart: unless-stopped
    expose:
      - "18545" # RPC
      - "18546" # WS
    ports:
      - "5050:5050" # P2P
    networks:
      - monitor-net
    command:
      --genesis=/genesis/mainnet-5577-full-mpt.g
      --config=/usr/local/bin/config.toml
      --port=5050
      --maxpeers=200
      --datadir=/root/data
      --http
      --http.addr=0.0.0.0
      --http.port=18545
      --http.corsdomain="*"
      --http.vhosts="*"
      --ws
      --ws.addr=0.0.0.0
      --ws.port=18546
      --ws.origins="*"
      --nousb
      --db.preset=pbl-1
      --tracenode
      --http.api=eth,web3,net,ftm,trace
    volumes:
      - fantom_data:/root/data
      - genesis_data:/genesis
    labels:
      - "traefik.enable=true"
      - "traefik.http.services.fantom.loadbalancer.server.port=18545"
      - "traefik.http.routers.fantom.entrypoints=websecure"
      - "traefik.http.routers.fantom.tls.certresolver=myresolver"
      - "traefik.http.routers.fantom.rule=Host(`$DOMAIN`)"
      - "traefik.http.routers.fantom.middlewares=ipwhitelist"
```

{% hint style="info" %}
ctrl + x and y to save file
{% endhint %}

### Run Fantom Node

```bash
docker-compose up -d
```

### Monitor Logs

Use `docker logs` to monitor your fantom node. The `-f` flag ensures you are following the log output

```
docker logs fantom -f
```
