---
description: 'Authors: [Vince | Nodeify]'
---

# 🐳 Docker

## System Requirements

<table data-full-width="false"><thead><tr><th align="center">CPU</th><th width="140" align="center">OS</th><th width="133" align="center">RAM</th><th align="center">DISK</th></tr></thead><tbody><tr><td align="center">Higher clock speed over core count</td><td align="center">Ubuntu 22.04</td><td align="center">8GB+</td><td align="center">At least 2.1TB (TLC NVMe recommended)</td></tr></tbody></table>

## Reth 🦀 + Lighthouse 🕯️

Official Docs [https://paradigmxyz.github.io/reth/intro.html](https://paradigmxyz.github.io/reth/intro.html)

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

### Create Reth directory

The first command, `mkdir reth`, will create a new directory named reth in the current location. The second command, `cd reth`, will change your current working directory to the newly created reth directory. Now you are inside the "reth" directory and can start storing docker-compose and related files in it.

```bash
mkdir reth
cd reth
```

### Generate JWT token

Generate a 32-byte random hexadecimal value using OpenSSL, removes any newline characters, and then save the result into the file "jwt.hex" located in the reth directory.

```bash
openssl rand -hex 32 | sudo tee ~/reth/jwt.hex > /dev/null
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
    reth_data: {}
    reth_logs: {}
    lighthouse_data: {}

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
#####################             RETH CONTAINER               #######################
###################################################################################### 

  reth:
    image: ghcr.io/paradigmxyz/reth:v0.1.0-alpha.4
    container_name: reth
    restart: unless-stopped
    expose:
      - "8545" #rpc
      - "8551" #auth
      - "9001" #metrics
    ports:
      - "30303:30303" # peering
    command: >
      node
      --chain mainnet
      --metrics 0.0.0.0:9091
      --log.persistent
      --log.directory /root/rethlogs
      --authrpc.addr 0.0.0.0
      --authrpc.port 8551
      --authrpc.jwtsecret /root/jwt/jwt.hex
      --http --http.addr 0.0.0.0 --http.port 8545
      --http.api "eth,net,web3"
    volumes:
      - reth_data:/root/.local/share/reth/mainnet/db
      - reth_logs:/root/rethlogs
      - ./jwt.hex:/root/jwt/jwt.hex:ro
    labels:
      - "traefik.enable=true"
      - "traefik.http.middlewares.reth-stripprefix.stripprefix.prefixes=/eth-mainnet"
      - "traefik.http.services.reth.loadbalancer.server.port=8545"
      - "traefik.http.routers.reth.entrypoints=websecure"
      - "traefik.http.routers.reth.tls.certresolver=myresolver"
      - "traefik.http.routers.reth.rule=Host(`$DOMAIN`) && PathPrefix(`/eth-mainnet`)"
      - "traefik.http.routers.reth.middlewares=arbitrum-stripprefix, ipwhitelist"

######################################################################################
#####################            LIGHTHOUSE CONTAINER               ##################
###################################################################################### 

  lighthouse:
    image: sigp/lighthouse
    container_name: lighthouse
    restart: unless-stopped
    expose:
      - "5052/tcp" #rpc
      - "5053/tcp"
      - "5054/tcp" #metrics
    ports:
     - "9000:9000/tcp" # peering
     - "9000:9000/udp"
    command: >
      lighthouse bn
      --http --http-address 0.0.0.0
      --execution-endpoint http://reth:8551
      --metrics --metrics-address 0.0.0.0
      --execution-jwt /root/jwt/jwt.hex
      --checkpoint-sync-url https://mainnet.checkpoint.sigp.io
    volumes:
      - lighthouse_data:/root/.lighthouse
      - ./jwt.hex:/root/jwt/jwt.hex:ro
```

{% hint style="info" %}
ctrl + x and y to save file
{% endhint %}

### Run Reth

```bash
docker-compose up -d
```

### Monitor Logs

Use `docker logs` to monitor your reth and lighthouse nodes. The `-f` flag ensures you are following the log output

```
docker logs reth -f
docker logs lighthouse -f
```

## Test Reth RPC 🧪

{% hint style="danger" %}
Reth and Consensus client must be synced before testing.
{% endhint %}

{% code overflow="wrap" %}
```bash
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["0x1", true],"id":1}' http://{DOMAIN}/eth-mainnet
```
{% endcode %}

{% hint style="warning" %}
This should be your result.
{% endhint %}

{% code overflow="wrap" %}
```bash
{"jsonrpc":"2.0","id":1,"result":{"difficulty":"0x3ff800000","extraData":"0x476574682f76312e302e302f6c696e75782f676f312e342e32","gasLimit":"0x1388","gasUsed":"0x0","hash":"0x88e96d4537bea4d9c05d12549907b32561d3bf31f45aae734cdc119f13406cb6","logsBloom":"0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000","miner":"0x05a56e2d52c817161883f50c441c3228cfe54d9f","mixHash":"0x969b900de27b6ac6a67742365dd65f55a0526c41fd18e1b16f1a1215c2e66f59","nonce":"0x539bd4979fef1ec4","number":"0x1","parentHash":"0xd4e56740f876aef8c010b86a40d5f56745a118d0906a34e69aec8c0db1cb8fa3","receiptsRoot":"0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421","sha3Uncles":"0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347","size":"0x219","stateRoot":"0xd67e4d450343046425ae4271474353857ab860dbc0a1dde64b41b5cd3a532bf3","timestamp":"0x55ba4224","totalDifficulty":"0x7ff800000","transactions":[],"transactionsRoot":"0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421","uncles":[]}}
```
{% endcode %}
