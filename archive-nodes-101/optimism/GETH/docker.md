---
description: 'Authors: [Vince | Nodeify]'
---

# 🐳 Docker

## System Requirements

<table data-full-width="false"><thead><tr><th align="center">CPU</th><th width="140" align="center">OS</th><th width="180" align="center">RAM</th><th align="center">DISK</th></tr></thead><tbody><tr><td align="center">4c/8T</td><td align="center">Ubuntu 22.04</td><td align="center">>= 16GB</td><td align="center">>= 8TB</td></tr></tbody></table>

## 🔴 Optimism

Official Docs [https://community.optimism.io/docs/developers/bedrock/node-operator-guide/#](https://community.optimism.io/docs/developers/bedrock/node-operator-guide/)

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

### Create base directory

The first command, `mkdir optimism`, will create a new directory named optimism in the current location. The second command, `cd optimism`, will change your current working directory to the newly created base directory. Now you are inside the base directory and can start storing docker-compose and related files in it.

```bash
mkdir optimism
cd optimism
```

### Create .env file

```bash
sudo nano .env
```

Paste the following into the file.

{% code overflow="wrap" fullWidth="false" %}
```bash
EMAIL={YOUR_EMAIL} #Your email to receive SSL renewal emails
DOMAIN={YOUR_DOMAIN}
WHITELIST={YOUR_REMOTE_MACHINE_IP}
DATA_TRANSPORT_LAYER__RPC_ENDPOINT={RPC_ENDPOINT_OF_ETHEREUM_L1_NODE} # L1 RPC
NETWORK_NAME={MAINNET}
OP_NODE__RPC_TYPE={BASIC} #default is basic (other options are erigon, alchemy, quicknode, infura, parity, nethermind, debug_geth, any)
```
{% endcode %}

{% hint style="info" %}
ctrl + x and y to save file
{% endhint %}

### Make scripts directory

```bash
cd ~/optimism
mkdir scripts
cd scripts #Goto scripts directory
```

### Download all the helper scripts into scripts directory

{% code overflow="wrap" %}
```bash
cat << EOF | wget -i -
https://raw.githubusercontent.com/InfraDAO/scripts/main/optimism/Dockerfile.bedrock-init
https://raw.githubusercontent.com/InfraDAO/scripts/main/optimism/init-bedrock.sh
https://raw.githubusercontent.com/InfraDAO/scripts/main/optimism/init-l2geth.sh
https://raw.githubusercontent.com/InfraDAO/scripts/main/optimism/start-dtl.sh
https://raw.githubusercontent.com/InfraDAO/scripts/main/optimism/start-l2geth.sh
https://raw.githubusercontent.com/InfraDAO/scripts/main/optimism/start-op-geth.sh
https://raw.githubusercontent.com/InfraDAO/scripts/main/optimism/start-op-node.sh
https://raw.githubusercontent.com/InfraDAO/scripts/main/optimism/utils.sh
EOF
```
{% endcode %}

### Make Scripts Executable

```bash
chmod +x *.sh
```

### Create docker-compose.yml

```bash
cd ~/optimism
sudo nano docker-compose.yml
```

Paste the following into the docker-compose.yml

```docker
version: '3.8'

networks:
  monitor-net:
    driver: bridge

volumes:
  dtl_data: {}
  geth_data: {}
  op-geth_data: {}
  shared: {}
  torrent_downloads: {}
  traefik_letsencrypt: {}

services:

######################################################################################
#####################               TRAEFIK CONTAINER           ######################
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
#####################               OP-GETH CONTAINER           ######################
######################################################################################

  op-geth:
    image: us-docker.pkg.dev/oplabs-tools-artifacts/images/op-geth:v1.101304.0
    container_name: opgeth
    restart: unless-stopped
    expose:
      - 8545 #http
      - 8546 #ws
    networks:
      - monitor-net
    stop_grace_period: 5m
    entrypoint: /scripts/start-op-geth.sh
    environment:
      - BEDROCK_SEQUENCER_HTTP=https://mainnet-sequencer.optimism.io
      - BEDROCK_DATADIR=/geth
    volumes:
      - ./scripts/:/scripts
      - shared:/shared
      - op-geth_data:/geth
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.optimism.service=optimism" #https
      - "traefik.http.services.optimism.loadbalancer.server.port=8545"
      - "traefik.http.routers.optimism.entrypoints=websecure"
      - "traefik.http.routers.optimism.tls.certresolver=myresolver"
      - "traefik.http.routers.optimism.rule=Host(`$DOMAIN`)"
      - "traefik.http.routers.optimism.middlewares=ipwhitelist"

######################################################################################
#####################               OP-NODE CONTAINER           ######################
######################################################################################

  op-node:
    image: us-docker.pkg.dev/oplabs-tools-artifacts/images/op-node:v1.3.0
    container_name: opnode
    restart: unless-stopped
    expose:
      - 9003/tcp
      - 9003/udp
    networks:
      - monitor-net
    stop_grace_period: 5m
    entrypoint: /scripts/start-op-node.sh
    env_file:
      - .env
    volumes:
      - ./scripts/:/scripts
      - shared:/shared

######################################################################################
#####################               DTL CONTAINER               ######################
######################################################################################

  dtl:
    image: ethereumoptimism/data-transport-layer:${IMAGE_TAG__DTL:-latest}
    container_name: dtl
    restart: unless-stopped
    expose:
      - 7878
    networks:
      - monitor-net
    entrypoint: /scripts/start-dtl.sh
    environment:
      - "NODE_TYPE=archive"
      - "DATA_TRANSPORT_LAYER__ADDRESS_MANAGER=0xdE1FCfB0851916CA5101820A69b13a4E276bd81F"
      - "DATA_TRANSPORT_LAYER__L1_START_HEIGHT=13596466"
      - "DATA_TRANSPORT_LAYER__CONFIRMATIONS=12"
      - "DATA_TRANSPORT_LAYER__DANGEROUSLY_CATCH_ALL_ERRORS=true"
      - "DATA_TRANSPORT_LAYER__DB_PATH=/db"
      - "DATA_TRANSPORT_LAYER__ENABLE_METRICS=true"
      - "DATA_TRANSPORT_LAYER__ETH_NETWORK_NAME=${NETWORK_NAME:-mainnet}"
      - "DATA_TRANSPORT_LAYER__L2_CHAIN_ID=10"
      - "DATA_TRANSPORT_LAYER__LOGS_PER_POLLING_INTERVAL=2000"
      - "DATA_TRANSPORT_LAYER__NODE_ENV=production"
      - "DATA_TRANSPORT_LAYER__POLLING_INTERVAL=500"
      - "DATA_TRANSPORT_LAYER__SENTRY_TRACE_RATE=0.05"
      - "DATA_TRANSPORT_LAYER__SERVER_HOSTNAME=0.0.0.0"
      - "DATA_TRANSPORT_LAYER__SERVER_PORT=7878"
      - "DATA_TRANSPORT_LAYER__TRANSACTIONS_PER_POLLING_INTERVAL=1000"
      - "DATA_TRANSPORT_LAYER__DEFAULT_BACKEND=l1"
      - "DATA_TRANSPORT_LAYER__L1_GAS_PRICE_BACKEND=l1"
      - "DATA_TRANSPORT_LAYER__SYNC_FROM_L1=true"
      - "DATA_TRANSPORT_LAYER__SYNC_FROM_L2=false"
      - "DATA_TRANSPORT_LAYER__RPC_ENDPOINT=${DATA_TRANSPORT_LAYER__RPC_ENDPOINT}"
    volumes:
      - dtl_data:/db
      - ./scripts/:/scripts/

######################################################################################
#####################               L2GETH CONTAINER            ######################
######################################################################################

  l2geth:
    image: ethereumoptimism/l2geth:${IMAGE_TAG__L2GETH:-latest}
    container_name: l2geth
    restart: unless-stopped
    expose:
      - 8545 #http
      - 8546 #ws
    networks:
      - monitor-net
    stop_grace_period: 5m
    entrypoint: 
      - /bin/sh
      - -c
      - "/scripts/init-l2geth.sh && /scripts/start-l2geth.sh"
    environment:
      - NODE_TYPE=archive
      - USING_OVM=true
      - SEQUENCER_CLIENT_HTTP=https://mainnet.optimism.io
      - BLOCK_SIGNER_ADDRESS=0x00000398232E2064F896018496b4b44b3D62751F
      - BLOCK_SIGNER_PRIVATE_KEY=6587ae678cf4fc9a33000cdbf9f35226b71dcc6a4684a31203241f9bcfd55d27
      - BLOCK_SIGNER_PRIVATE_KEY_PASSWORD=pwd
      - ETH1_CTC_DEPLOYMENT_HEIGHT=13596466
      - ETH1_SYNC_SERVICE_ENABLE=true
      - L2GETH_GENESIS_URL=https://storage.googleapis.com/optimism/mainnet/genesis-berlin.json
      - L2GETH_GENESIS_HASH=0x106b0a3247ca54714381b1109e82cc6b7e32fd79ae56fbcc2e7b1541122f84ea
      - ROLLUP_CLIENT_HTTP=http://dtl:7878
      - ROLLUP_MAX_CALLDATA_SIZE=40000
      - ROLLUP_POLL_INTERVAL_FLAG=1s
      - ROLLUP_VERIFIER_ENABLE=true
      - ROLLUP_BACKEND=l1
      - L2GETH_STATE_DUMP_PATH=/geth/l2geth-state
      - DATADIR=/geth
      - CHAIN_ID=10
      - NETWORK_ID=10
      - NO_DISCOVER=true
      - NO_USB=true
      - GASPRICE=0
      - TARGET_GAS_LIMIT=15000000
      - RPC_ADDR=0.0.0.0
      - RPC_API=eth,rollup,net,web3,debug
      - RPC_CORS_DOMAIN=*
      - RPC_ENABLE=true
      - RPC_PORT=8545
      - RPC_VHOSTS=*
      - WS_ADDR=0.0.0.0
      - WS_API=eth,rollup,net,web3,debug
      - WS_ORIGINS=*
      - WS=true
    volumes:
      - geth_data:/geth
      - ./scripts/:/scripts/

######################################################################################
#####################               SNAPSHOT CONTAINER          ######################
######################################################################################

  bedrock-init:
    build:
      context: ./scripts
      dockerfile: Dockerfile.bedrock-init
    entrypoint: /scripts/init-bedrock.sh
    env_file:
      - .env
    networks:
      - monitor-net
    volumes:
      - ./scripts/:/scripts
      - shared:/shared
      - op-geth_data:/geth
      - geth_data:/legacy-geth
      - torrent_downloads:/downloads
```

{% hint style="info" %}
ctrl + x and y to save file
{% endhint %}

### Run Optimism Node

```bash
docker-compose up -d
```

### Monitor Logs

Use `docker logs` to monitor your geth and op nodes. The `-f` flag ensures you are following the log output

```
docker logs opgeth -f --tail 50
docker logs opnode -f --tail 50
```

## Test Optimism RPC 🧪

{% code overflow="wrap" %}
```bash
curl --data '{"method":"eth_syncing","params":[],"id":1,"jsonrpc":"2.0"}' -H "Content-Type: application/json" -X POST https://{DOMAIN}
```
{% endcode %}

{% hint style="warning" %}
Sync speed will be highly dependent on your Layer 1 RPC
{% endhint %}
