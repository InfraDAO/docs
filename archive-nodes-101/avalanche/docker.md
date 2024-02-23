---
description: 'Authors: [Vince | Nodeify]'
---

# 🐳 Docker

## System Requirements

<table data-full-width="false"><thead><tr><th align="center">CPU</th><th width="140" align="center">OS</th><th width="180" align="center">RAM</th><th align="center">DISK</th></tr></thead><tbody><tr><td align="center">4c/8t CPU</td><td align="center">Ubuntu 22.04</td><td align="center">16GB+</td><td align="center">>= 5TB SSD/NVME</td></tr></tbody></table>

## Avalanche Go ⛰

Official Docs&#x20;

[https://docs.avax.network/nodes](https://docs.avax.network/nodes)

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

### Create Avalanche directory

The first command, `mkdir avalanche`, will create a new directory named avalanche in the current location. The second command, `cd avalanche`, will change your current working directory to the newly created avalanche directory. Now you are inside the avalanche directory and can start storing docker-compose and related files in it.

```bash
mkdir avalanche
cd avalanche
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
  avax_data: {}
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
#####################               AVAX CONTAINER             #######################
######################################################################################     

  avax:
    image: avaplatform/avalanchego:v1.10.7
    container_name: avax
    restart: unless-stopped
    expose:
      - "9650" # RPC
      - "9650" #/ext/metrics
    ports:
      - "9651:9651/tcp"
      - "9651:9651/udp"
    networks:
      - monitor-net
    command:
      - "/avalanchego/build/avalanchego --http-host="
    volumes:
      - avax_data:/root/.avalanchego
      - ./archive-config.json:/root/.avalanchego/configs/chains/C/config.json
    labels:
      - "traefik.enable=true"
      - "traefik.http.middlewares.avax-replacepath.replacepath.path=/ext/bc/C/rpc"
      - "traefik.http.services.avalanche.loadbalancer.server.port=9650"
      - "traefik.http.routers.avax.entrypoints=websecure"
      - "traefik.http.routers.avax.tls.certresolver=myresolver"
      - "traefik.http.routers.avax.rule=Host(`$DOMAIN`)"
      - "traefik.http.routers.avax.middlewares=ipwhitelist"
```

{% hint style="info" %}
ctrl + x and y to save file
{% endhint %}

Download archive-config.json

```bash
wget https://github.com/kw1knode/node-vault/blob/main/avalanche/docker/archive-config.json
```

### Run Avalanche Node

```bash
docker-compose up -d
```

### Monitor Logs

Use `docker logs` to monitor your Avalanche node. The `-f` flag ensures you are following the log output

```
docker logs avax -f
```

## Test Avalanche RPC 🧪

{% code overflow="wrap" %}
```bash
curl --data '{"method":"eth_syncing","params":[],"id":1,"jsonrpc":"2.0"}' -H "Content-Type: application/json" -X POST https://{DOMAIN}
```
{% endcode %}

{% hint style="warning" %}
You should receive this result when synced.
{% endhint %}

{% code overflow="wrap" %}
```
{"jsonrpc":"2.0","id":1,"result":false}
```
{% endcode %}
