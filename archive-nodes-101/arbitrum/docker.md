---
description: 'Authors: [Vince | Nodeify]'
---

# 🐳 Docker

## System Requirements

<table data-full-width="false"><thead><tr><th align="center">CPU</th><th width="140" align="center">OS</th><th width="180" align="center">RAM</th><th align="center">DISK</th></tr></thead><tbody><tr><td align="center">4+ core CPU</td><td align="center">Ubuntu 22.04</td><td align="center">16GB+ for Nitro and 32GB+ for Classic</td><td align="center">>= 8TB SSD/NVME</td></tr></tbody></table>

## Offchain Labs ⛓️

Official Docs [https://docs.arbitrum.io/node-running/how-tos/running-an-archive-node](https://docs.arbitrum.io/node-running/how-tos/running-an-archive-node)

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

### Create Arbitrum directory

The first command, `mkdir arbitrum`, will create a new directory named arbitrum in the current location. The second command, `cd arbitrum`, will change your current working directory to the newly created arbitrum directory. Now you are inside the arbitrum directory and can start storing docker-compose and related files in it.

```bash
mkdir arbitrum
cd arbitrum
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
ARBITRUM_L1_URL={YOUR_L1_RPC} #RPC endpoint of Ethereum L1 RPC
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

x-logging:
  &default-logging
  options:
    max-file: '1'
    compress: 'false'
  driver: local

networks:
  monitor-net:
    driver: bridge

volumes:
  arbitrum-nitro: {}
  arbitrum-classic: {}
  traefik_letsencrypt: {}

services:

######################################################################################
#####################         ARBITRUM NITRO CONTAINER          ######################
###################################################################################### 

  arbitrum-nitro:
    image: 'offchainlabs/nitro-node:v2.0.14-2baa834-slim-amd64'
    container_name: nitro
    restart: unless-stopped
    stop_grace_period: 30s
    user: root
    volumes:
      - 'arbitrum-nitro:/arbitrum-node'
    expose:
      - 8547
      - 8548
    command:
      - --init.url=https://snapshot.arbitrum.foundation/arb1/nitro-archive.tar
      - --node.caching.archive
      - --persistent.chain=/arbitrum-node/data/
      - --persistent.global-config=/arbitrum-node/
      - --node.rpc.classic-redirect=http://arbitrum-classic:8547/
      - --l1.url=${ARBITRUM_L1_URL}
      - --l2.chain-id=42161
      - --http.api=net,web3,eth,debug
      - --http.corsdomain=*
      - --http.addr=0.0.0.0
      - --http.vhosts=*
    labels:
      - "traefik.enable=true"
      - "traefik.http.services.arbitrum.loadbalancer.server.port=8547"
      - "traefik.http.routers.arbitrum.entrypoints=websecure"
      - "traefik.http.routers.arbitrum.tls.certresolver=myresolver"
      - "traefik.http.routers.arbitrum.rule=Host(`$DOMAIN`)"
      - "traefik.http.routers.arbitrum.middlewares=ipwhitelist"

######################################################################################
#####################         ARBITRUM CLASSIC CONTAINER          ####################
###################################################################################### 

  arbitrum-classic:
    image: 'kw1k/arbnode:latest'
    container_name: classic
    restart: always
    stop_grace_period: 30s
    user: root
    volumes:
    - 'arbitrum-classic:/root/.arbitrum/mainnet'
    expose:
      - 8547
      - 8548
    command:
      - --l1.url=${ARBITRUM_L1_URL}
      - --l2.disable-upstream
      - --node.chain-id=42161
      - --node.rpc.tracing.enable
      - --node.rpc.tracing.namespace=trace
      - --core.checkpoint-pruning-mode=off
      - --node.cache.allow-slow-lookup
      - --core.checkpoint-gas-frequency=156250000
      - --node.rpc.addr=0.0.0.0
      - --l2.final-classic-block=22207816

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
```

{% hint style="info" %}
ctrl + x and y to save file
{% endhint %}

In this step, we will create a helpful script named `arbitrum-classic-entrypoint.sh` to automatically download the initial snapshot for your Arbitrum Classic node when it starts for the first time.

```bash
sudo nano arbitrum-classic-entrypoint.sh
```

Paste the following into the file.

```
#!/bin/bash

if [ -f /root/.arbitrum/mainnet/INITIALIZED ]; then
    echo "The data directory has already been initialized."
else
    echo "lemme download the database quickly"
    rm -rf /root/.arbitrum/mainnet/db
    curl https://snapshot.arbitrum.foundation/arb1/classic-archive.tar | tar -xv -C /root/.arbitrum/mainnet/ && touch /root/.arbitrum/mainnet/INITIALIZED    
fi

echo "LFG!!!"

/home/user/go/bin/arb-node $@
```

### Run Arbitrum Node

```bash
docker-compose up -d
```

### Monitor Logs

Use `docker logs` to monitor your nitro and classic nodes. The `-f` flag ensures you are following the log output

```
docker logs nitro -f
docker logs classic -f
```

## Test Arbitrum RPC 🧪

{% code overflow="wrap" %}
```bash
curl --data '{"method":"eth_syncing","params":[],"id":1,"jsonrpc":"2.0"}' -H "Content-Type: application/json" -X POST https://{DOMAIN}
```
{% endcode %}

{% hint style="warning" %}
You should receive result.
{% endhint %}

{% code overflow="wrap" %}
```
{"jsonrpc":"2.0","id":1,"result":{"batchProcessed":308668,"batchSeen":308668,"blockNum":96352870,"broadcasterQueuedMessagesPos":0,"lastL1BlockNum":17885784,"lastl1BlockHash":"0x758b86452273e12b74cdfdc1fb11c373e25589676c09b9b42fd917015fcdeccd","messageOfLastBlock":74145055,"messageOfProcessedBatch":97883962,"msgCount":97884349}}
```
{% endcode %}
