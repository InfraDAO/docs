---
description: 'Authors: [Vince | Nodeify]'
---

# 🐳 Docker

## System Requirements

<table data-full-width="false"><thead><tr><th align="center">CPU</th><th width="140" align="center">OS</th><th width="180" align="center">RAM</th><th align="center">DISK</th></tr></thead><tbody><tr><td align="center">4c/8t</td><td align="center">Ubuntu 22.04</td><td align="center">>= 16GB</td><td align="center">>= 2TB SSD/NVME</td></tr></tbody></table>

## Celo 🟨

Official Docs [https://docs.celo.org/network/mainnet/run-full-node#running-an-archive-node](https://docs.celo.org/network/mainnet/run-full-node#running-an-archive-node)

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

### Create Celo directory

The first command, `mkdir celo`, will create a new directory named celo in the current location. The second command, `cd celo`, will change your current working directory to the newly created celo directory. Now you are inside the celo directory and can start storing docker-compose and related files in it.

```bash
mkdir celo
cd celo
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
    traefik_letsencrypt: {}
    celo_data: {}

services:

######################################################################################
#####################         TRAEFIK PROXY CONTAINER          #######################
######################################################################################     

  traefik:
    image: traefik:latest
    container_name: traefik
    restart: always
    expose:
      - "8082:8082" # METRICS
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
#####################            CELO ARCHIVE CONTAINER        #######################
###################################################################################### 

  celo:
    image: us.gcr.io/celo-org/geth:mainnet
    container_name: celo
    restart: unless-stopped
    expose:
      - "8545" # RPC
    ports:
      - 31303:30303 # P2P
      - 31303:30303/udp # P2P
    networks:
      - monitor-net
    command: >
      --datadir=/root/.celo
      --syncmode=full
      --gcmode=archive
      --txlookuplimit=0
      --cache.preimages
      --port=31303
      --http
      --http.addr=0.0.0.0
      --http.vhosts="*"
      --verbosity=3
      --ws
      --ws.addr=0.0.0.0
      --ws.port=8545
      --http.api=eth,net,web3,debug,admin,personal
    volumes:
      - celo_data:/root/.celo
    labels:
      - "traefik.enable=true"
      - "traefik.http.services.erigon.loadbalancer.server.port=8545"
      - "traefik.http.routers.erigon.entrypoints=websecure"
      - "traefik.http.routers.erigon.tls.certresolver=myresolver"
      - "traefik.http.routers.erigon.rule=Host(`$DOMAIN`)"
      - "traefik.http.routers.erigon.middlewares=ipwhitelist"
```

{% hint style="info" %}
ctrl + x and y to save file
{% endhint %}

### Run Celo Node

```bash
docker-compose up -d
```

### Monitor Logs

Use `docker logs` to monitor your celo node. The `-f` flag ensures you are following the log output

```
docker logs celo -f
```

## Test Celo RPC 🧪

{% code overflow="wrap" %}
```bash
curl --data '{"method":"eth_syncing","params":[],"id":1,"jsonrpc":"2.0"}' -H "Content-Type: application/json" -X POST https://{DOMAIN}
```
{% endcode %}

{% hint style="warning" %}
You should receive a result. i.e.
{% endhint %}

{% code overflow="wrap" %}
```
{"jsonrpc":"2.0","id":1,"result":{"batchProcessed":308668,"batchSeen":308668,"blockNum":96352870,"broadcasterQueuedMessagesPos":0,"lastL1BlockNum":17885784,"lastl1BlockHash":"0x758b86452273e12b74cdfdc1fb11c373e25589676c09b9b42fd917015fcdeccd","messageOfLastBlock":74145055,"messageOfProcessedBatch":97883962,"msgCount":97884349}}
```
{% endcode %}
