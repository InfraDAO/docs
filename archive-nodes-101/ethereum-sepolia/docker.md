# 🐳 Docker

Author: \[ jLeopoldA ]

## System Requirements

| CPU      | OS                  | RAM       | DISK      |
| -------- | ------------------- | --------- | --------- |
| 4+ Cores | Ubunutu 24.04.1 LTS | 16GB+ RAM | 3.5TB SSD |

{% hint style="info" %}
The Ethereum Sepolia Archive Node has a size of 3.2TB as of 2/11/2025
{% endhint %}

## Pre-Requisites

{% hint style="info" %}
This method of setting up an Ethereum Sepolia Archive Node uses Docker, Docker-Compose, Geth (Execution Layer) and Prysm (Consensus Layer / Beacon Node).
{% endhint %}

### Update System

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y
```

### Set Up Firewall

#### Set Explicit Default Firewall Rules

```bash
sudo ufw default deny incoming 
sudo ufw default allow outgoing
```

#### Allow SSH

```bash
sudo ufw allow 22/tcp
```

#### Allow RPC Connections with Geth / Sepolia

```bash
sudo ufw allow 8545
sudo ufw allow 8546
```

#### Allow P2P Connections for Geth and Prysm

```bash
# Required Ports for Geth (Execution Layer)
# Ethereum P2P networking
sudo ufw allow 30303/tcp && sudo ufw allow 30303/udp
sudo ufw allow 8551/tcp # Authentication RPC

# Required Prysm (Consensus Layer)
sudo ufw allow 13000/tcp # P2P networking
sudo ufw allow 12000/udp # Discovery protocol
sudo ufw allow 4000
```

#### Enable Firewall

```bash
sudo ufw enable
```

#### Check Status / Current Rules of UFW

```bash
sudo ufw status verbose
```

### Install Docker & Docker-Compose

#### Install Docker

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Install Docker Packages including Docker Compose
 sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
 
# Verify Docker Engine Installation
sudo docker run hello-world
```

## Build Ethereum Sepolia Archive Node

### Create Directory

```bash
mkdir -p /root/sepolia
cd /root/sepolia
```

### Create jwt.hex

```bash
openssl rand -hex 32 | tr -d "\n" > jwt.hex
```

### Create docker-compose.yml

```bash
nano docker-compose.yml

# Copy and paste the below
services:
  beacon-node:
    image: gcr.io/prysmaticlabs/prysm/beacon-chain:stable
    container_name: beacon-node
    restart: unless-stopped
    volumes:
      - $HOME/.eth2:/data
      - /root/sepolia/jwt.hex:/root/sepolia/jwt.hex:ro
    ports:
      - "4000:4000"
      - "13000:13000"
      - "12000:12000/udp"
    command:
      - --datadir=/data
      - --jwt-secret=/root/sepolia/jwt.hex
      - --rpc-host=0.0.0.0
      - --http-host=0.0.0.0
      - --monitoring-host=0.0.0.0
      - --execution-endpoint=http://geth:8551
      - --sepolia
      - --checkpoint-sync-url=https://sepolia.beaconstate.info
      - --genesis-beacon-api-url=https://beaconstate.info
    networks:
      -  blockchain-network

  geth:
    image: ethereum/client-go:stable
    restart: unless-stopped
    volumes:
      - ./data:/root/.ethereum
      - /root/sepolia/jwt.hex:/root/sepolia/jwt.hex:ro
    ports:
      - "8545:8545"
      - "8546:8546"
      - "8551:8551"
      - "30303:30303"
    command: [
      "--sepolia",
      "--syncmode=full",
      "--gcmode=archive",
      "--authrpc.addr=0.0.0.0",
      "--authrpc.port=8551",
      "--authrpc.vhosts=*",
      "--authrpc.jwtsecret=/root/sepolia/jwt.hex",
      "--http",
      "--http.addr=0.0.0.0",
      "--http.port=8545",
      "--http.api=eth,net,engine,admin",
      "--ws",
      "--ws.addr=0.0.0.0",
      "--ws.port=8546",
      "--ws.api=eth,net,web3"
    ]
    networks:
      -  blockchain-network

networks:
  blockchain-network:
    driver: bridge
```

Press "Ctrl + X". Press "y" when prompted and then "Enter".

### Run Archive Node with Ethereum Sepolia

To run your node - enter the below:

```bash
# Run this from within /root/sepolia
docker compose up -d
```

## Interact with Sepolia Archive Node

### Check Logs

#### Check Logs of Geth / Sepolia

```bash
docker compose logs sepolia-geth-1
```

Logs will slightly resemble the image below.

<figure><img src="../../.gitbook/assets/Screenshot from 2025-02-11 14-23-16.png" alt=""><figcaption></figcaption></figure>

#### Check logs of Prysm

```bash
docker logs beacon-node
```

Logs will look similar to the image below.

<figure><img src="../../.gitbook/assets/spaces_DHv0OHzHXecvl5JZaSK1_uploads_WkjlHSAJcAPxVyNUCXHu_Screenshot from 2025-02-11 14-26-06.webp" alt=""><figcaption></figcaption></figure>

#### Stop Node

```bash
# To stop Geth / Sepolia Archive Node
docker stop sepolia-geth-1

# To stop Prysm (Beacon Node)
docker stop beacon-node
```

## Query Sepolia Archive Node

{% hint style="info" %}
The Ethereum Sepolia Archive Node has a sync time of about 4 days.
{% endhint %}

### Check Sync Status

```bash
curl -H "Content-Type: application/json" \
-X POST --data '{"jsonrpc":"2.0", "method":"eth_syncing", "params":[], "id":1}' http://localhost:8545

```

When node is finished syncing the response from the above command should resemble the below.

```bash
{"jsonrpc":"2.0","id":1,"result":false}
```

### Check Block Number

```bash
curl -H "Content-Type: application/json" \
-X POST --data '{"jsonrpc":"2.0", "method":"eth_blockNumber", "params":[], "id":1}' http://localhost:8545

# Response should have a different "result" value.
{"jsonrpc":"2.0","id":1,"result":"0x754ee0"}
```
