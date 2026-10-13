---
description: 'Authors: [Vince | Nodeify]'
---

# 🐳 Docker

## System Requirements

<table data-full-width="false"><thead><tr><th align="center">CPU</th><th width="140" align="center">OS</th><th width="180" align="center">RAM</th><th align="center">DISK</th></tr></thead><tbody><tr><td align="center">4+ core CPU</td><td align="center">Ubuntu 22.04</td><td align="center">16GB+ </td><td align="center">>= 1 TB (1/27/24)</td></tr></tbody></table>

## zkLend

### [https://github.com/eqlabs/pathfinder](https://github.com/eqlabs/pathfinder)

### [https://github.com/starknet-graph/firehose-starknet](https://github.com/starknet-graph/firehose-starknet)

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

### Create Starknet directory

The first command, `mkdir starknet`, will create a new directory named starknet in the current location. The second command, `cd starknet`, will change your current working directory to the newly created starknet directory. Now you are inside the starknet directory and can start storing docker-compose and related files in it.

```bash
mkdir starknet
cd starknet
```

### Create .env file

```bash
sudo nano .env
```

Paste the following into the file.

```bash
EMAIL={YOUR_EMAIL} #Your email to receive SSL renewal emails
DOMAIN={YOUR_DOMAIN} #Domain of your reth node you set earlier, starknet.infradao.com
WHITELIST={YOUR_REMOTE_MACHINE_IP} # Remote IP's allowed to connect to firehose
```

{% hint style="info" %}
ctrl + x and y to save file
{% endhint %}

### Create docker-compose.yml

```bash
sudo nano docker-compose.yml
```

Paste the following into the docker-compose.yml

```yaml
version: '3.8'

networks:
  monitor-net:
    driver: bridge

volumes:
  firehose_data: {}
  pathfinder_data: {}
  traefik_letsencrypt: {}

services:

######################################################################################
#####################         ARBITRUM CLASSIC CONTAINER          ####################
###################################################################################### 

  firestark:
    image: starknet/firestark:0.2.1-pathfinder-0.10.2-amd64
    container_name: firestark
    user: root
    restart: unless-stopped
    stop_grace_period: 30s
    networks:
      - monitor-net
    volumes:
      - firehose_data:/var/lib/firehose
      - pathfinder_data:/var/lib/pathfinder
      - ./firehose.yml:/root/config/firehose.yml
    expose:
      - 9545  # PF RPC
      - 9103  # PF METRICS
      - 10015 # FH
      - 9102  # FH METRICS
    command: start -c /root/config/firehose.yml
    labels:
      - "traefik.enable=true"
      - "traefik.http.services.arbitrum.loadbalancer.server.port=10015"
      - "traefik.http.routers.arbitrum.entrypoints=websecure"
      - "traefik.http.routers.arbitrum.tls.certresolver=myresolver"
      - "traefik.http.routers.arbitrum.rule=Host(`$DOMAIN`)"
      - "traefik.http.routers.arbitrum.middlewares=ipwhitelist"

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

In this step, we will create a firehose config yaml in the same directory to be loaded into the docker container.

```bash
sudo nano firehose.yml
```

Paste the following into the file. Edit the `--ethereum.url` to your own Ethereum RPC.&#x20;

```yaml
start:
  args:
  - merger
  - firehose
  - reader-node
  - relayer
  flags:
    data-dir: /var/lib/firehose
    reader-node-path: /usr/local/bin/pathfinder
    reader-node-arguments: "--data-directory /var/lib/pathfinder --network mainnet --http-rpc 0.0.0.0:9545 --ethereum.url {YOUR-ETHEREUM-RPC} --monitor-address 0.0.0.0:9103"
```

{% hint style="info" %}
ctrl + x and y to save file
{% endhint %}

### Run Starknet Firehose

```bash
docker-compose up -d
```

### Monitor Logs

Use `docker logs` to monitor your node. The `-f` flag ensures you are following the log output

```
docker logs firestark -f
```
