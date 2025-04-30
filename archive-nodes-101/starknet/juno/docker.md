---
description: 'Author: Godwin'
---

# 🐳 Docker

## System Requirements

<table><thead><tr><th align="center">CPU</th><th align="center">OS</th><th width="254" align="center">RAM</th><th align="center">DISK</th></tr></thead><tbody><tr><td align="center">4-Core CPU</td><td align="center">Debian 12/Ubuntu 22.04</td><td align="center">=> 8 GB RAM</td><td align="center"><p>500 GB+</p><p> (SSD or NVMe)</p></td></tr></tbody></table>

{% hint style="info" %}
_Starknet Juno full node has a size of 397GB on April 19th, 2025_
{% endhint %}

{% hint style="success" %}
Juno is a Go implementation of a Starknet full-node client created by Nethermind to allow node operators to easily and reliably support the network and advance its decentralisation goals. Juno supports various node setups, from casual to production-grade indexers.
{% endhint %}

{% hint style="warning" %}
## Before you start, make sure that you have your own synced Ethereum mainnet L1 RPC URL ready with WS port enabled
{% endhint %}

### Pre-Requisites <a href="#pre-requisties" id="pre-requisties"></a>

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y
​
sudo apt install -y wget curl screen git ufw
```

### Setting up Firewall <a href="#setting-up-firewall" id="setting-up-firewall"></a>

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw allow 22/tcp
sudo ufw allow 80
sudo ufw allow 443
```

### Enable Firewall

```bash
sudo ufw enable
```

## Install Docker

#### Run this command to remove any conflicting docker

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

#### Add Docker's official GPG key:

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

#### Add the repository to ppt sources:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
sudo apt-get update
```

#### Install docker

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Test docker is working
sudo docker run hello-world

#Install docker compose

sudo apt-get update
sudo apt-get install docker-compose-plugin

# Test the docker version
docker compose version
```

**Create Starknet directory**

```bash
mkdir starkent && cd starknet
```

{% hint style="danger" %}
**Juno** only recognizes an endpoint through WebSocket protocol, so make sure to specify it using the `wss://` scheme (e.g., `wss://your-endpoint`)
{% endhint %}

### Launch Starknet full node

```bash
sudo nano docker-compose.yml
```

Paste the following into the `docker-compose.yml:`&#x20;

```bash
version: '3.9'

networks:
  monitor-net:
    driver: bridge

volumes:
  juno_data: {}

services:
  juno:
    image: nethermind/juno:v0.14.2
    user: root
    container_name: juno
    volumes:
      - "/var/lib/juno-data:/data"
    restart: unless-stopped
    command:
      - --db-path=/data
      - --network=mainnet
      - --http 
      - --http-port=6060 
      - --http-host=0.0.0.0
      - --metrics
      - --metrics-host=0.0.0.0 
      - --metrics-port=9090
      - --rpc-cors-enable
      - --ws 
      - --ws-port=6061
      - --ws-host=0.0.0.0
      - --log-level=trace
      - --eth-node=wss://<l1-endpoint>
    expose:
      - 6060 
      - 5050
      - 9090 
      - 8545 
      - 6061 
    ports:
      - "5050:5050"   # P2P Port
      - "6060:6060"   # HTTP RPC Port
      - "9090:9090"   # Metrics Port
      - "8545:8545"   # JSON-RPC (Ethereum compatible)
      - "6061:6061"   # WebSocket
    networks:
      - monitor-net
```

```bash
sudo docker compose up -d
```

### Monitor Logs

Use `docker logs` to monitor your starknet node. The `-f` flag ensures you are following the log output

```bash
docker logs juno -f --tail 100
```

Once your Juno Starknet node starts syncing, the logs are expected to look like this:

```log
09:28:47.686 02/04/2025 +00:00  INFO    migration/migration.go:110      Applying database migration     {"stage": "17/18"}
10:17:08.292 02/04/2025 +00:00  INFO    migration/migration.go:110      Applying database migration     {"stage": "18/18"}
10:17:08.293 02/04/2025 +00:00  INFO    l1/l1.go:112    Subscribing to L1 updates...
10:17:08.294 02/04/2025 +00:00  INFO    l1/l1.go:121    Subscribed to L1 updates
10:17:08.494 02/04/2025 +00:00  DEBUG   upgrader/upgrader.go:81 Application is up-to-date.

....

16:42:03.829 19/04/2025 +00:00  TRACE   jsonrpc/server.go:451   Received request        {"req": {"jsonrpc":"2.0","method":"starknet_blockNumber","id":3268045}}
16:42:03.829 19/04/2025 +00:00  TRACE   jsonrpc/server.go:451   Received request        {"req": {"jsonrpc":"2.0","method":"starknet_getBlockWithReceipts","params":[{"block_number":1330197}],"id":3268046}}
16:42:03.965 19/04/2025 +00:00  TRACE   jsonrpc/server.go:451   Received request        {"req": {"jsonrpc":"2.0","method":"starknet_getStateUpdate","params":[{"block_number":1330197}],"id":3268047}}
16:42:04.022 19/04/2025 +00:00  TRACE   jsonrpc/server.go:451   Received request        {"req": {"jsonrpc":"2.0","method":"starknet_blockNumber","id":3268048}}
```

#### 1. Block Number

This confirms which Starknet network the node is connected to:

```bash
curl --location 'http://localhost:6060'  --header 'Content-Type: application/json'  --data '{ "jsonrpc": "2.0", "method": "starknet_blockNumber", "params": [],"id": 1}'
```

**Expected Response:**

```json
{"jsonrpc":"2.0","result":1330203,"id":1}
```

2. Sync Status

This ensures that your node is syncing correctly and producing up-to-date data.

```bash
curl --location 'http://localhost:6060'  --header 'Content-Type: application/json'  --data '{ "jsonrpc": "2.0", "method": "starknet_syncing", "params": [],"id": 1}'
```

**Expected Response:**

**You get the below result if the node has caught up to the latest block.**

```json
{"jsonrpc":"2.0","result":false,"id":1}
```

{% hint style="success" %}
**Juno starkent node** takes **approximately 5 days** to fully catch up to the latest chain head when syncing from Genesis
{% endhint %}

### References <a href="#references" id="references"></a>

{% embed url="https://juno.nethermind.io/hardware-requirements/" %}
