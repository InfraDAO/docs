# 🐳 Docker

Author \[godwin]

## System Requirements

| CPU      | OS           | RAM   | DISK     |
| -------- | ------------ | ----- | -------- |
| 8 cores+ | Ubuntu 24.04 | 32GB+ | >= 1.3TB |

{% hint style="info" %}
_The Unichain Mainnet archive node has a size of 1.3TB on August 12th, 2025_
{% endhint %}

Last updated: 12th August 2025

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

Run this command to remove any conflicting Docker

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

## Set up Unichain Node

Make and switch to the working directory for the ronin node

```bash
mkdir /unichain
cd unichain
```

Go into the `docker` directory, create a `docker-compose.yml` file with the following configuration:

```bash
volumes:
  shared:

services:
  execution-client:
    image: us-docker.pkg.dev/oplabs-tools-artifacts/images/op-geth:v1.101511.1
    env_file:
      - .env
      - .env.mainnet
    ports:
      - 30303:30303/udp
      - 30303:30303/tcp
      - 8545:8545/tcp
      - 8546:8546/tcp
    volumes:
      - ${HOST_DATA_DIR}:/data
      - shared:/shared
      - ./op-geth-entrypoint.sh:/entrypoint.sh
    healthcheck:
      start_interval: 5s
      start_period: 240s
      test: wget --no-verbose --tries=1 --spider http://localhost:8545 || exit 1
    restart: always
    entrypoint: /entrypoint.sh

  op-node:
    image: us-docker.pkg.dev/oplabs-tools-artifacts/images/op-node:v1.13.5
    env_file:
      - .env
      - .env.mainnet
    ports:
      - 9222:9222/udp
      - 9222:9222/tcp
      - 9545:9545/tcp
    volumes:
      - ${HOST_NODE_DATA_DIR}:/data
      - shared:/shared
    healthcheck:
      start_interval: 5s
      start_period: 240s
      test: wget --no-verbose --tries=1 --spider http://localhost:9545 || exit 1
    depends_on:
      execution-client:
        condition: service_healthy
    restart: always
```

Create a `.env` file and add the following content

```bash
HOST_DATA_DIR=./geth-data
HOST_NODE_DATA_DIR=./opnode-data
```

Create a `.env.mainnet` file in the project directory

```bash
# op-node configuration

# [required] replace with your preferred L1 (Ethereum, not Unichain) node RPC URL:
OP_NODE_L1_ETH_RPC=https://ethereum-rpc.publicnode.com

# [required] replace with your preferred L1 CL beacon endpoint:
OP_NODE_L1_BEACON=https://ethereum-beacon-api.publicnode.com


OP_NODE_NETWORK=unichain-mainnet
OP_NODE_L2_ENGINE_AUTH=/shared/jwt.hex
OP_NODE_L2_ENGINE_RPC=ws://execution-client:8551
OP_NODE_LOG_LEVEL=info
OP_NODE_LOG_FORMAT=logfmt
OP_NODE_METRICS_ADDR=0.0.0.0
OP_NODE_METRICS_ENABLED=true
OP_NODE_METRICS_PORT=3300
OP_NODE_P2P_LISTEN_IP=0.0.0.0
OP_NODE_P2P_LISTEN_TCP_PORT=9222
OP_NODE_P2P_LISTEN_UDP_PORT=9222
OP_NODE_RPC_ADDR=0.0.0.0
OP_NODE_RPC_PORT=9545
OP_NODE_VERIFIER_L1_CONFS=4
OP_NODE_P2P_NAT=true
OP_NODE_P2P_DISCOVERY_PATH=/data/opnode_discovery_db
OP_NODE_P2P_PEERSTORE_PATH=/data/opnode_peerstore_db
OP_NODE_P2P_PRIV_PATH=/data/opnode_p2p_priv.txt
OP_NODE_L2_ENGINE_KIND=geth
OP_NODE_L1_TRUST_RPC=false
# Execution Layer Sync
# op-node can drive the Execution Client to sync from the EL layer. This enables Snap Sync in op-geth or staged sync in>
# This requires the EL Client to be peered.
# By default, OP geth uses snap sync, but can use full sync (executes every block) while OP_NODE_SYNCMODE=execution-lay>
OP_NODE_SYNCMODE=execution-layer
OP_NODE_L1_RPC_KIND=standard

OP_NODE_P2P_NAT=true
OP_NODE_P2P_ADVERTISE_IP=       # Replace with your real IP
OP_NODE_P2P_ADVERTISE_TCP=9222  # External port from docker-compose
OP_NODE_P2P_ADVERTISE_UDP=9222


GETH_NAT_EXTIP=<node-ext-ip>:30303

GETH_SYNCMODE=full


# op-geth configuration

GETH_OP_NETWORK=unichain-mainnet
GETH_ROLLUP_SEQUENCERHTTP=https://mainnet-sequencer.unichain.org
GETH_LOG_FORMAT=logfmt
GETH_VERBOSITY=3
GETH_DATADIR=/data
GETH_HTTP=true
GETH_HTTP_ADDR=0.0.0.0
GETH_HTTP_VHOSTS="*"
GETH_HTTP_CORSDOMAIN="*"
GETH_HTTP_API=web3,debug,eth,txpool,net,admin,rpc
GETH_WS=true
GETH_WS_ADDR=0.0.0.0
GETH_WS_API=web3,debug,eth,txpool,net,admin,rpc
GETH_AUTHRPC_JWTSECRET=/shared/jwt.hex
GETH_AUTHRPC_PORT=8551
GETH_AUTHRPC_VHOSTS="*"
GETH_AUTHRPC_ADDR=0.0.0.0
GETH_METRICS=true
GETH_METRICS_ADDR=0.0.0.0
GETH_ROLLUP_DISABLETXPOOLGOSSIP=true
GETH_TXPOOL_NOLOCALS=true
GETH_GCMODE=archive
```

Create `op-geth-entrypoint.sh` in the project directory

```bash
#!/bin/sh
set -euxo pipefail

if [ -n "${GENESIS_FILE-}" ]; then
	geth init "$GENESIS_FILE"
fi

exec geth $@
```

## Run the node

```bash
docker-compose up -d
```

## Monitor the node

Use docker logs to monitor the rootstock node. The -f flag ensures you are following the log output.

```bash
docker logs unichain-op-geth -f --tail 50
```

You should see a response similar to this once your node starts syncing

```
t=2025-08-12T06:46:57+0000 lvl=info msg="Imported new potential chain segment" number=24232858 hash=0xc300e3b6308be8fe5371978f01c6fab909284067fd4c43314e0a88fb65623cc1 blocks=1 txs=13 mgas=2.240231 elapsed=14.457ms mgasps=154.9476210004033 snapdiffs="1.49 MiB" triedirty="0.00 B"
t=2025-08-12T06:46:57+0000 lvl=info msg="Chain head was updated" number=24232858 hash=0xc300e3b6308be8fe5371978f01c6fab909284067fd4c43314e0a88fb65623cc1 root=0x21ef672c25a814c992dc6afda8e7bba7099b1f4af04ca534615488700f1fad6b elapsed=291.627µs
t=2025-08-12T06:46:58+0000 lvl=info msg="Imported new potential chain segment" number=24232859 hash=0xb10c669d0efc6a2faeadc3787fdf3d1fe3d9d11975a5007ea260243700d21907 blocks=1 txs=12 mgas=2.687842 elapsed=20.289ms mgasps=132.47773055462045 snapdiffs="1.50 MiB" triedirty="0.00 B"
t=2025-08-12T06:46:58+0000 lvl=info msg="Chain head was updated" number=24232859 hash=0xb10c669d0efc6a2faeadc3787fdf3d1fe3d9d11975a5007ea260243700d21907 root=0x4e69964b1c86a52c91deea8345152a6078280ba17d58d6b8872256ca3f464c70 elapsed=528.692µs
```

```bash
docker logs unichain-op-node -f --tail 50
```

```
t=2025-08-12T06:48:25+0000 lvl=info msg="successfully processed payload" ref=0x50665f0807456c318313e9006e79c1b6a0f711c033968e63ac7567c600cc0415:24232946 txs=13
t=2025-08-12T06:48:26+0000 lvl=info msg="Optimistically queueing unsafe L2 execution payload" id=0x2f01786fa44447e0df230dbd4219f49305a1d10abbfbb9c049f481af3486eb7d:24232947
t=2025-08-12T06:48:26+0000 lvl=info msg="Inserted new L2 unsafe block (synchronous)" hash=0x2f01786fa44447e0df230dbd4219f49305a1d10abbfbb9c049f481af3486eb7d number=24232947 newpayload_time=13.308ms fcu2_time=983.566µs total_time=14.292ms mgas=1.47295 mgasps=103.05431020568254
t=2025-08-12T06:48:26+0000 lvl=info msg="Sync progress" reason="new chain head block" l2_finalized=0x7322764ce54f000bd8ec66a188365365f36e862a6aa24a0492ec82d0dad6b81b:24231349 l2_safe=0x642c37ca5213562fe2a16cd2f808885b78ca2a67ebe6d555473eb38729c48460:24232683 l2_pending_safe=0x642c37ca5213562fe2a16cd2f808885b78ca2a67ebe6d555473eb38729c48460:24232683 l2_unsafe=0x2f01786fa44447e0df230dbd4219f49305a1d10abbfbb9c049f481af3486eb7d:24232947 l2_backup_unsafe=0x0000000000000000000000000000000000000000000000000000000000000000:0 l2_time=1754981306
t=2025-08-12T06:48:26+0000 lvl=info msg="successfully processed payload" ref=0x2f01786fa44447e0df230dbd4219f49305a1d10abbfbb9c049f481af3486eb7d:24232947 txs=10
t=2025-08-12T06:48:26+0000 lvl=info msg="Advancing bq origin" origin=0xdfa78d3e53df14ca1359db22a0f5e66320ecc465d97a7c46a018710e84bfada0:23123270 originBehind=false
```

## Query the node

Eth syncing status

```bash
curl -s -X POST -H "Content-Type: application/json"   --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}'   http://localhost:8545 | jq
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
curl -s -X POST -H "Content-Type: application/json"   --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'   http://localhost:8545 | jq
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

{% embed url="https://docs.unichain.org/docs/getting-started/set-up-a-node" %}

{% embed url="https://github.com/Uniswap/unichain-node/tree/main" %}
