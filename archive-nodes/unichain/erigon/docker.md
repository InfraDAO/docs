# 🐳 Docker

Author \[godwin]

## System Requirements

| CPU      | OS           | RAM   | DISK      |
| -------- | ------------ | ----- | --------- |
| 8 cores+ | Ubuntu 24.04 | 32GB+ | >= 500 GB |

{% hint style="info" %}
_The Unichain Mainnet archive node has a size of 256 GB on August 12th, 2025_
{% endhint %}

Last updated at: 25th August 2025

## Pre-Requisites

First, update, upgrade, and clean the system:

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt install ufw -y
```

## Configure Firewall Settings&#x20;

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw allow 22/tcp
sudo ufw allow 80
sudo ufw allow 443
sudo ufw allow 8545
sudo ufw allow 8546
sudo ufw allow 30303
sudo ufw allow 6060
```

## Install Docker

Run this command to remove any conflicting docker

```bash
`for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done`
```

Add Docker's official GPG key:

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Add the repository to ppt sources:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

Install docker

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Test docker is working
sudo docker run hello-world

# Install docker compose

sudo apt-get update
sudo apt-get install docker-compose-plugin

# Test the docker version
docker compose version
```

## Setup Unichain Node

Make and switch to the working directory for the ronin node

```bash
mkdir /unichain
cd unichain
```

Go into the `docker` directory, create a `docker-compose.yml` file with the following configuration:

```bash
services:
  unichain-mainnet-archive:
    image: testinprod/op-erigon:v2.61.3-0.9.5
    container_name: unichain-erigon
    user: "0:0"
    ports:
      - "4145:4145"
      - "10415:10415"
      - "10415:10415/udp"
      - "4146:4146"
      - "25415:25415"
      - "25415:25415/udp"
      - "30415:30415"
      - "30415:30415/udp"
      - "8553:8553"
    entrypoint: ["erigon"]
    command:
      - --datadir=/data
      - --chain=unichain-mainnet
      - --port=10415
      - --p2p.allowed-ports=25415
      - --p2p.allowed-ports=30415 
      - --nat=extip:${IP}
      - --http.addr=0.0.0.0
      - --http.port=4145
      - --http.vhosts=*
      - --http.corsdomain=*
      - --authrpc.addr=0.0.0.0
      - --authrpc.port=8553
      - --authrpc.vhosts=*
      - --authrpc.jwtsecret=/shared/jwt.hex
      - --http.api=eth,erigon,web3,net,debug,trace,txpool,admin
      - --rpc.returndata.limit=1100000
      - --rpc.gascap=5000000000
      - --ws
      - --ws.port=4146
      - --batchSize=256MB
      - --rpc.evmtimeout=10m
    restart: unless-stopped
    stop_grace_period: 3m 
    volumes:
      - ${HOST_DATA_DIR}:/data:rw
      - shared:/shared


  unichain-mainnet-archive-node:
    image: us-docker.pkg.dev/oplabs-tools-artifacts/images/op-node:aa4db40408094803cf9bc05277734572df46088a
    container_name: unichain-erigon-opnode
    user: "0:0"
    ports:
      - "4345:4345"
      - "15415:15415"
      - "15415:15415/udp"
    entrypoint: [ "op-node" ]
    restart: unless-stopped
    volumes:
      - ${HOST_NODE_DATA_DIR}:/data
      - shared:/shared
    environment:
      - "OP_NODE_NETWORK=unichain-mainnet"
      - "OP_NODE_SYNCMODE=execution-layer"
      - "OP_NODE_L1_ETH_RPC=https://ethereum-rpc.publicnode.com"
      - "OP_NODE_L2_ENGINE_AUTH=/shared/jwt.hex"
      - "OP_NODE_L2_ENGINE_RPC=http://unichain-mainnet-archive:8553"
      - "OP_NODE_LOG_LEVEL=info"
      - "OP_NODE_METRICS_ADDR=0.0.0.0"
      - "OP_NODE_METRICS_ENABLED=true"
      - "OP_NODE_METRICS_PORT=7300"
      - "OP_NODE_P2P_LISTEN_IP=0.0.0.0"
      - "OP_NODE_P2P_LISTEN_TCP_PORT=15415"
      - "OP_NODE_P2P_LISTEN_UDP_PORT=15415"
      - "OP_NODE_RPC_ADDR=0.0.0.0"
      - "OP_NODE_P2P_ADVERTISE_IP=your-node-ip"
      - "OP_NODE_RPC_PORT=4345"
      - "OP_NODE_SNAPSHOT_LOG=/tmp/op-node-snapshot-log"
      - "OP_NODE_VERIFIER_L1_CONFS=4"
      - "OP_NODE_STATIC_PEERS="
      - "OP_NODE_L1_RPC_KIND=erigon"
      - "OP_NODE_L1_TRUST_RPC=false"
      - "OP_NODE_L1_BEACON=https://ethereum-beacon-api.publicnode.com"
      - "OP_NODE_L2_ENGINE_KIND=erigon"

volumes:
   shared:
```

Create a `.env` file and add the following content

```bash
HOST_DATA_DIR=./erigon-data
HOST_NODE_DATA_DIR=./opnode-data
IP=<your server ip>
ETHEREUM_MAINNET_EXECUTION_RPC=https://ethereum-rpc.publicnode.com
ETHEREUM_MAINNET_BEACON_REST=https://ethereum-beacon-api.publicnode.com
ETHEREUM_MAINNET_EXECUTION_KIND=erigon
ETHEREUM_MAINNET_EXECUTION_TRUST=false
```

## Run the node

```bash
docker-compose up -d
```

## Monitor the node

Use docker logs to monitor the rootstock node. The -f flag ensures you are following the log output.

```bash
docker logs unichain-erigon -f  --tail 100
```

You should see a response similar to this once your node starts syncing

```bash
[INFO] [08-25|18:39:53.455] [NewPayload] Handling new payload        height=25398834 hash=0xd220e5d3141ae8f00d7f5d17ed0cf3e73686973c8c4ad30f821f559ab83b8716
[INFO] [08-25|18:39:53.468] [updateForkchoice] Fork choice update: flushing in-memory state (built by previous newPayload) 
[INFO] [08-25|18:39:53.477] RPC Daemon notified of new headers       from=25398833 to=25398834 amount=1 hash=0xd220e5d3141ae8f00d7f5d17ed0cf3e73686973c8c4ad30f821f559ab83b8716 header sending=10.249µs log sending=261ns
[INFO] [08-25|18:39:53.477] head updated                             hash=0xd220e5d3141ae8f00d7f5d17ed0cf3e73686973c8c4ad30f821f559ab83b8716 number=25398834
[INFO] [08-25|18:39:54.464] [NewPayload] Handling new payload        height=25398835 hash=0x178ecdc07b2b98a1548669f927b51e7408d4ce972a0d348d8f56923b73ce8880
[INFO] [08-25|18:39:54.474] [updateForkchoice] Fork choice update: flushing in-memory state (built by previous newPayload) 
[INFO] [08-25|18:39:54.479] RPC Daemon notified of new headers       from=25398834 to=25398835 amount=1 hash=0x178ecdc07b2b98a1548669f927b51e7408d4ce972a0d348d8f56923b73ce8880 header sending=7.194µs log sending=170ns
[INFO] [08-25|18:39:54.479] head updated                             hash=0x178ecdc07b2b98a1548669f927b51e7408d4ce972a0d348d8f56923b73ce8880 number=25398835
```

```bash
docker logs unichain-erigon-opnode -f  --tail 100
```

```
t=2025-08-25T18:41:02+0000 lvl=info msg="successfully processed payload" ref=0x333a3441580a6f9dd9b7281fa6d6e1f451ba6881ab9c43c188e3e5c8678eca6e:25398903 txs=17
t=2025-08-25T18:41:03+0000 lvl=info msg="Optimistically queueing unsafe L2 execution payload" id=0x3946f700f9a51f93e869efc81d23e2c899d7b3c8a1e1163cfc20e22be52ad187:25398904
t=2025-08-25T18:41:03+0000 lvl=info msg="Inserted new L2 unsafe block (synchronous)" hash=0x3946f700f9a51f93e869efc81d23e2c899d7b3c8a1e1163cfc20e22be52ad187 number=25398904 newpayload_time=47.100ms fcu2_time=39.884ms total_time=86.986ms mgas=5.116018 mgasps=58.814060037884815
t=2025-08-25T18:41:03+0000 lvl=info msg="Sync progress" reason="new chain head block" l2_finalized=0xca27220fe3f2d1b53089834a7467abb70dc2e618e9ecd4b52fcef038942ada93:25397645 l2_safe=0x8285fd46fecc6610774e81acf406ca326e2a13a2ccdac2254fe15fc10bf1c335:25398753 l2_pending_safe=0x8285fd46fecc6610774e81acf406ca326e2a13a2ccdac2254fe15fc10bf1c335:25398753 l2_unsafe=0x3946f700f9a51f93e869efc81d23e2c899d7b3c8a1e1163cfc20e22be52ad187:25398904 l2_backup_unsafe=0x0000000000000000000000000000000000000000000000000000000000000000:0 l2_time=1756147263
t=2025-08-25T18:41:03+0000 lvl=info msg="successfully processed payload" ref=0x3946f700f9a51f93e869efc81d23e2c899d7b3c8a1e1163cfc20e22be52ad187:25398904 txs=30
```

## Query the node

Eth syncing status

```bash
curl -s -X POST -H "Content-Type: application/json"   --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}'   http://localhost:4145 | jq
```

Output

If the node is done syncing, you should see the below response

```bash
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": false
}
```

To check the block number

```bash
curl -s -X POST -H "Content-Type: application/json"   --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'   http://localhost:4145 | jq
```

Output

```bash
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "0x171c46f"
}
```

## References

{% embed url="https://github.com/testinprod-io/op-erigon" %}

{% embed url="https://docs.unichain.org/docs/getting-started/set-up-a-node" %}

{% embed url="https://github.com/Uniswap/unichain-node/tree/main" %}
