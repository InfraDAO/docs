# 🐳 Docker

Authors: \[Vikash Choubey | Dapplooker]

### System Requirements

| CPU    | OS        | RAM  | DISK  |
| ------ | --------- | ---- | ----- |
| 8 vCPU | Ubuntu 22 | 16GB | 52GB+ |

&#x20;_The Boba Mainnet archival node has a size of_ 52GB _on September 19th, 2024_

## Pre-requisite

Before starting, clean the setup then update and upgrade. Install following:

* Docker & Docker Compose
* Git

### **Commands:**

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt install docker.io docker-compose git ufw -y
```

### Set explicit default UFW rules

```
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### Allow SSH, HTTP and HTTPS

```
sudo ufw allow 22/tcp
sudo ufw allow 80
sudo ufw allow 443
```

### Setup process:

Before starting on root directory, create a direct for boba network with command `mkdir boba-archive`, then `cd boba`.

```bash
mkdir boba
cd boba
```

### Clone boba network

```bash
git clone <https://github.com/bobanetwork/boba.git>
cd boba
cd boba-community
```

### **Create an .env file**

The repository includes a sample environment variable file located at `.env.example` that you can copy and modify to get started. Make a copy of this file and name it `.env`.

```bash
cp .env.example .env
```

### **Configuration**

Download **boba mainnet** snapshot and extract

```bash
curl -o boba-mainnet-erigon-db-1149019.tgz -sL <https://boba-db.s3.us-east-2.amazonaws.com/mainnet/boba-mainnet-erigon-db-1149019.tgz>
tar xvf boba-mainnet-erigon-db-1149019.tgz
```

Download **boba l2Geth** snapshot and extract

```bash
curl -o boba-mainnet-geth-db-114909.tgz -sL https://boba-db.s3.us-east-2.amazonaws.com/mainnet/boba-mainnet-geth-db-114909.tgz
tar xvf boba-mainnet-geth-db-114909.tgz// Some code
```

Create a Shared Secret (JWT Token) using:

```bash
openssl rand -hex 32 > jwt-secret.txt
```

Modify Volume Locations

```yaml
l2:
  volumes:
  - ./jwt-secret.txt:/config/jwt-secret.txt
  - DATA_DIR:/db
op-node:
  volumes:
  - ./jwt-secret.txt:/config/jwt-secret.txt
```

### Example docker-compose file:

```yaml
version: '3.4'

services:
  op-erigon:
    image:  us-docker.pkg.dev/boba-392114/bobanetwork-tools-artifacts/images/op-erigon:v1.1.5
    container_name: op-erigon
    command: |
      --datadir=/db
      --chain=boba-mainnet
      --http.addr=0.0.0.0
      --http.port=9545
      --http.corsdomain=*
      --http.vhosts=*
      --authrpc.addr=0.0.0.0
      --authrpc.port=8551
      --authrpc.vhosts=*
      --authrpc.jwtsecret=/config/jwt-secret.txt
      --http.api=eth,debug,net,engine,web3
      --txpool.gossip.disable=true
      --rollup.sequencerhttp=https://mainnet.boba.network
      --db.size.limit=8TB
      --rollup.historicalrpc=https://mainnet.boba.network
    ports:
      - "9545:9545"
      - "8551:8551"
    user: root
    volumes:
      - /mnt/boba-data/config:/config
      - /mnt/boba-data/boba-mainnet-erigon-db-1149019:/db
  boba-legacy:
    image: us-docker.pkg.dev/oplabs-tools-artifacts/images/op-geth:latest
    container_name: boba-legacy
    command: >
      --datadir=/db
      --networkid=288
      --http
      --http.addr=0.0.0.0
      --http.port=9545
      --http.corsdomain=*
      --http.vhosts=*
      --authrpc.addr=0.0.0.0
      --authrpc.port=8551
      --authrpc.vhosts=*
      --authrpc.jwtsecret=/config/jwt-secret.txt
      --rollup.disabletxpoolgossip=true
      --http.api=eth,debug,net,web3
      --nodiscover
      --syncmode=full
      --maxpeers=0
      --rollup.sequencerhttp=https://mainnet.boba.network
    ports:
      - "7545:9545"
      - "7551:8551"
    volumes:
      - /mnt/boba-data/config-l2geth:/config
      - /mnt/boba-data/geth-1149019:/db
  op-node:
    depends_on:
      - op-erigon
    container_name: op-node
    image: us-docker.pkg.dev/boba-392114/bobanetwork-tools-artifacts/images/op-node:v1.6.3
    command: >
      op-node
      --l1=${ETH1_HTTP:-https://mainnet.gateway.tenderly.co}
      --l1.beacon=${ETH2_HTTP}
      --l2=http://l2:8551
      --l2.jwt-secret=/config/jwt-secret.txt
      --network=boba-mainnet
      --rpc.addr=0.0.0.0
      --rpc.port=8545
      --plasma.enabled=false
    ports:
      - "8545:8545"
    volumes:
      - /mnt/boba-data/config:/config
    restart: always
```

### **Start The Node**

`docker-compose -f [docker-compose-file] up -d`

### **Monitor Logs**

Use `docker logs` to monitor your boba node. The `-f` flag ensures you are following the log output

```bash
docker logs op-erigon -f
docker logs op-node -f
docker logs boba-legacy -f
```

### **Test RPC:**

```bash
curl --data '{"method":"eth_syncing","params":[],"id":1,"jsonrpc":"2.0"}' -H "Content-Type: application/json" -X POST https://{DOMAIN}
```

You should receive a result, after the node is synced:

```json
{
	"jsonrpc":"2.0",
	"id":1,
	"result":false
}
```
