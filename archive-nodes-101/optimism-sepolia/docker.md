---
description: 'Author: [Godwin]'
---

# 🐳 Docker

## System Requirements

| CPU       | OS           | RAM | DISK |
| --------- | ------------ | --- | ---- |
| 4-8 cores | Ubuntu 24.04 | 16  | 5TB  |

{% hint style="info" %}
The Optimism Sepolia Archive Node has a size of 2.3TB as of 3/10/2025.
{% endhint %}

## Pre-Requisites

#### Update System

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y
sudo apt install -y git gcc make --fix-missing
```

#### Installl Docker

```bash
# Update and upgrade packages
sudo apt-get update
sudo apt-get upgrade -y

### Docker and docker compose prerequisites
sudo apt-get install -y curl
sudo apt-get install -y gnupg
sudo apt-get install -y ca-certificates
sudo apt-get install -y lsb-release

### Download the docker gpg file to Ubuntu
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

### Add Docker and docker compose support to the Ubuntu's packages list
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
 
### Install docker and docker compose on Ubuntu
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

sudo usermod -aG docker $(whoami)
 
### Verify the Docker and docker compose install on Ubuntu
sudo docker run hello-world
```

## Firewall Configuration

#### Set Explicit Firewall Configuration

```bash
sudo ufw default deny incoming && sudo ufw default allow outgoing
```

#### Allow SSH

```bash
sudo ufw allow 22/tcp
```

#### Allow Connections for OP-NODE & OP-GETH

```bash
sudo ufw allow 8546
sudo ufw allow 9545
sudo ufw allow 8545
sudo ufw allow 3000
```

#### Enable Firewall Rules

```bash
sudo ufw enable
```

#### Check Status of Firewall Rules (UFW)

```bash
sudo ufw status verbose
```

## Clone the Optimism Docker Setup Directory

### Create Directories

```bash
git clone https://github.com/smartcontracts/simple-optimism-node.git
cd simple-optimism-node
```

#### Copy .env.example to .env

```bash
cp .env.example .env
```

## Configure the .env file&#x20;

#### This is a sample of how to configure the .env for op-sepolia node

{% hint style="info" %}
If port 8545 isn't working, you can connect to the default port for the l2 geth execution node port - 9993
{% endhint %}

```bash
###############################################################################
#                                ↓ REQUIRED ↓                                 #
###############################################################################

# Network to run the node on ("op-mainnet" or "op-sepolia")
NETWORK_NAME=op-sepolia

# Type of node to run ("full" or "archive"), note that "archive" is 10x bigger
NODE_TYPE=archive

###############################################################################
#                            ↓ REQUIRED (BEDROCK) ↓                           #
###############################################################################

# L1 node that the op-node (Bedrock) will get chain data from
OP_NODE__RPC_ENDPOINT=<l1-endpoint>

# L1 beacon endpoint, you can setup your own or use Quicknode
OP_NODE__L1_BEACON=<l1-beacon-endpoint>

# Type of RPC that op-node is connected to, see README
OP_NODE__RPC_TYPE=basic

# Reference L2 node to run healthcheck against
HEALTHCHECK__REFERENCE_RPC_PROVIDER=https://sepolia.optimism.io

###############################################################################
#                            ↓ OPTIONAL (BEDROCK) ↓                           #
###############################################################################

# Optional provider to serve legacy RPC requests, see README
OP_GETH__HISTORICAL_RPC=https://mainnet.optimism.io

# Set to "full" to force op-geth to use --syncmode=full
OP_GETH__SYNCMODE=

###############################################################################
#                                ↓ OPTIONAL ↓                                 #
###############################################################################

# Feel free to customize your image tag if you want, uses "latest" by default
# See here for all available images: https://hub.docker.com/u/ethereumoptimism
IMAGE_TAG__L2GETH=
IMAGE_TAG__DTL=
IMAGE_TAG__HEALTCHECK=
IMAGE_TAG__PROMETHEUS=
IMAGE_TAG__GRAFANA=
IMAGE_TAG__INFLUXDB=
IMAGE_TAG__OP_GETH=
IMAGE_TAG__OP_NODE=

# Exposed server ports (must be unique)
# See docker-compose.yml for default values
PORT__L2GETH_HTTP=8545
PORT__L2GETH_WS=
PORT__DTL=
PORT__HEALTHCHECK_METRICS=
PORT__PROMETHEUS=
PORT__GRAFANA=
PORT__INFLUXDB=
PORT__TORRENT_UI=
PORT__TORRENT=
PORT__OP_GETH_HTTP=
PORT__OP_GETH_WS=
PORT__OP_GETH_P2P=
PORT__OP_NODE_P2P=
PORT__OP_NODE_HTTP=
```

You can get more info on the env config in this link - [https://github.com/smartcontracts/simple-optimism-node#mandatory-configurations](https://github.com/smartcontracts/simple-optimism-node#mandatory-configurations)

The docker-compose.yml file can be found here

[https://github.com/smartcontracts/simple-optimism-node/blob/main/docker-compose.yml](https://github.com/smartcontracts/simple-optimism-node/blob/main/docker-compose.yml)

### Operating the Node

```bash
docker compose up -d --build
```

## View the logs

```bash
docker compose logs <CONTAINER_NAME> -f --tail 10
```

### Monitoring

Run progress.sh to estimate remaining sync time and speed.

```bash
./progress.sh
```

```
Chain ID: 11155420
Sampling, please wait
Blocks per minute: 30
Hours until sync completed: ...
```

#### Grafana dashboard

Grafana is exposed at [http://localhost:3000](http://localhost:3000/) and comes with one pre-loaded dashboard ("Simple Node Dashboard").&#x20;

Use the following login details to access the dashboard:

* Username: `admin`
* Password: `optimism`

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

## Query Node

#### Check Sync Status

```bash
curl -H "Content-Type: application/json" \
-X POST --data '{"jsonrpc":"2.0", "method":"eth_syncing", "params":[], "id":1}' \
http://localhost:8545 or 9993

# If node is done syncing - the response should resemble the below.
{"jsonrpc":"2.0","id":1,"result":false}
```

#### Check optimism sync status

```
curl -X POST -H "Content-Type: application/json" --data \
    '{"jsonrpc":"2.0","method":"optimism_syncStatus","params":[],"id":1}'  \
    http://localhost:9545
```

```bash
{"jsonrpc":"2.0","id":1,"result":{"current_l1":{"hash":"0xda1a28e8d035386478138aa1941d324bf39ac9ff57f4fbcf72ff98450f2c2590","number":7876040,"parentHash":"0x3a3a56dd92718df3b55ad48d3b8adfcf2a1741f7ec1478b9ec9bcf152a4dbc26","timestamp":1741642332},"current_l1_finalized"....
```

#### Check Block Number

```bash
curl -H "Content-Type: application/json" \
-X POST --data '{"jsonrpc":"2.0", "method":"eth_blockNumber", "params":[], "id":1}' \
http://localhost:8545 or 9993

# Response should resemble the below.
{"jsonrpc":"2.0","id":1,"result":"0x17c3e4e"}
```
