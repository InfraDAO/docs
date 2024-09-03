---
description: 'Authors: [Payne | StakeSquid]'
---

# 🖥️ Baremetal

## System Requirements

<table data-full-width="false"><thead><tr><th align="center">CPU</th><th width="140" align="center">OS</th><th width="180" align="center">RAM</th><th align="center">DISK</th></tr></thead><tbody><tr><td align="center">4c/8T</td><td align="center">Ubuntu 22.04</td><td align="center">>= 16GB</td><td align="center">>= 8TB</td></tr></tbody></table>

## Pre-requisites

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y

sudo apt install -y git make wget gcc pkg-config libusb-1.0-0-dev libudev-dev jq gcc g++ curl libssl-dev apache2-utils build-essential pkg-config 

# Add to PATH
echo "export PATH="$PATH:/root/.foundry/bin:/usr/local/go/bin:/root/.local/bin"" >> /root/.bashrc

source /root/.bashrc

# Install go
sudo wget https://go.dev/dl/go1.21.6.linux-amd64.tar.gz && sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.21.6.linux-amd64.tar.gz && rm go1.21.6.linux-amd64.tar.gz

# Install nvm
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash 

# Download foundry
curl -L https://foundry.paradigm.xyz | bash

# Install foundry
foundryup

source /root/.bashrc

# Install node and yarn
nvm install 16 && npm install --global yarn && nvm use 16 && npm -g install pnpm

source /root/.bashrc

# Check if go and npm are installed:
go version
nvm -v
npm -v
yarn -v
pnpm -v

# Create directories
mkdir -p /root/github
mkdir -p /root/data/optimism/geth/dtl
mkdir -p /root/data/optimism/geth/l2geth
mkdir -p /root/data/optimism/geth/op-node
mkdir -p /root/data/optimism/geth/op-geth

# Download snapshots
https://docs.optimism.io/builders/node-operators/management/snapshots#op-mainnet

# Verify tree structure
/root/data/optimism/geth ❯ tree -d
.
|-- dtl
|-- l2geth
|   |-- geth
|   |   |-- chaindata
|   |   |   `-- ancient
|   |   |-- chaindata-part1
|   |   |   `-- ancient
|   |   |-- chaindata-part2
|   |   |-- lightchaindata
|   |   `-- nodes
|   `-- keystore
|-- op-geth
|   |-- geth
|   |   |-- blobpool
|   |   |-- chaindata
|   |   |   `-- ancient
|   |   |-- nodes
|   |   `-- triecache
|   `-- keystore
`-- op-node
    |-- opnode_discovery_db
    `-- opnode_peerstore_db

22 directories
```

## DTL

### Building Optimism DTL

```bash
mkdir -p /root/github
mkdir -p /root/data/optimism/geth/dtl

cd /root/github

git clone https://github.com/ethereum-optimism/optimism-legacy.git

cd optimism-legacy

yarn

yarn build
```

### Setting up the Systemd file for DTL

```bash
#######################  !!! EDIT YOUR OWN L1 RPC ENDPOINT !!! ###################

sudo echo "[Unit]
Description=Optimism DTL Service
After=network.target
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
Type=simple
Restart=on-failure
RestartSec=5
TimeoutSec=900
User=root
Nice=0
LimitNOFILE=200000
WorkingDirectory=/root/github/optimism-legacy/packages/data-transport-layer
ExecStart=/bin/bash -c '. /root/.nvm/nvm.sh && /root/.nvm/versions/node/v16.20.2/bin/node /root/.nvm/versions/node/v16.20.2/bin/yarn start'
#EnvironmentFile=/root/github/optimism-legacy/packages/data-transport-layer/.env
Environment=DATA_TRANSPORT_LAYER__ADDRESS_MANAGER=0xdE1FCfB0851916CA5101820A69b13a4E276bd81F \
        DATA_TRANSPORT_LAYER__L1_START_HEIGHT=13596466 \
        DATA_TRANSPORT_LAYER__CONFIRMATIONS=12  \
        DATA_TRANSPORT_LAYER__DANGEROUSLY_CATCH_ALL_ERRORS=true \
        DATA_TRANSPORT_LAYER__DB_PATH=/root/data/optimism/geth/dtl/ \
        DATA_TRANSPORT_LAYER__ENABLE_METRICS=true \
        DATA_TRANSPORT_LAYER__ETH_NETWORK_NAME=mainnet \
        DATA_TRANSPORT_LAYER__L2_CHAIN_ID=10 \
        DATA_TRANSPORT_LAYER__LOGS_PER_POLLING_INTERVAL=2000 \
        DATA_TRANSPORT_LAYER__NODE_ENV=production \
        DATA_TRANSPORT_LAYER__POLLING_INTERVAL=500 \
        DATA_TRANSPORT_LAYER__SENTRY_TRACE_RATE=0.05 \
        DATA_TRANSPORT_LAYER__SERVER_HOSTNAME=0.0.0.0 \
        DATA_TRANSPORT_LAYER__SERVER_PORT=7878 \
        DATA_TRANSPORT_LAYER__TRANSACTIONS_PER_POLLING_INTERVAL=1000 \
        DATA_TRANSPORT_LAYER__SYNC_FROM_L1=true \
        DATA_TRANSPORT_LAYER__L1_RPC_ENDPOINT=http://IP:PORT \
        DATA_TRANSPORT_LAYER__DEFAULT_BACKEND=l1 \
        DATA_TRANSPORT_LAYER__L1_GAS_PRICE_BACKEND=l1 \
        DATA_TRANSPORT_LAYER__SYNC_FROM_L2=false
KillSignal=SIGTERM
[Install]
WantedBy=multi-user.target" > /etc/systemd/system/dtl.service
```

### To verify that DTL is running correctly you can run a command

#### If synchronizing from L1:

```bash
curl -s http://localhost:7878/eth/syncing?backend=l1  | jq .currentTransactionIndex
```

#### If synchronizing from L2:

```bash
curl -s http://localhost:7878/eth/syncing?backend=l2  | jq .currentTransactionIndex
```

## L2-GETH <a href="#optimism-l2-geth" id="optimism-l2-geth"></a>

### Building and preparing L2 GETH

```bash
cd /root/github/optimism-legacy/l2geth

make geth

mkdir -p /root/data/optimism/geth/l2geth/genesis/

# Download the genesis file

wget -O /root/data/optimism/geth/l2geth/genesis/genesis.json https://storage.googleapis.com/optimism/mainnet/genesis-berlin.json

# Set basic envs to load

curl -s "https://gist.githubusercontent.com/trader-payne/0fce05f7a8e7d9c4724fb2a771fff7f1/raw/b91b291445ff41cd4498d322da967167d4430f22/gistfile1.txt" > /root/github/optimism-legacy/packages/data-transport-layer/.env

# Init L2 GETH - this takes a while depending on your machine's specs

set -o allexport; source /root/github/optimism-legacy/packages/data-transport-layer/.env; set +o allexport; /root/github/optimism-legacy/l2geth/build/bin/geth init --datadir=/root/data/optimism/geth/l2geth /root/data/optimism/geth/l2geth/genesis/genesis.json --no-usb

# Import the block signer account

touch $DATADIR/password
echo $BLOCK_SIGNER_PRIVATE_KEY > $DATADIR/block-signer-key

/root/github/optimism-legacy/l2geth/build/bin/geth account import --datadir=$DATADIR --password $DATADIR/password $DATADIR/block-signer-key
```

### Setting up the Systemd file for L2GETH

```bash
sudo echo "[Unit]
Description=Optimism L2 GETH Service
After=network.target
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
Type=simple
Restart=on-failure
RestartSec=5
TimeoutSec=900
User=root
Nice=0
LimitNOFILE=200000
WorkingDirectory=/root/data/optimism/geth/l2geth/
#EnvironmentFile=/root/github/optimism-legacy/packages/data-transport-layer/.env
Environment=NETWORK_NAME=mainnet \
        NODE_TYPE=archive \
        USING_OVM=true \
        SEQUENCER_CLIENT_HTTP=https://mainnet.optimism.io \
        BLOCK_SIGNER_ADDRESS=0x00000398232E2064F896018496b4b44b3D62751F \
        BLOCK_SIGNER_PRIVATE_KEY=6587ae678cf4fc9a33000cdbf9f35226b71dcc6a4684a31203241f9bcfd55d27 \
        ETH1_CTC_DEPLOYMENT_HEIGHT=13596466 \
        ETH1_SYNC_SERVICE_ENABLE=false \
        ROLLUP_CLIENT_HTTP=http://0.0.0.0:7878 \
        ROLLUP_MAX_CALLDATA_SIZE=40000 \
        ROLLUP_POLL_INTERVAL_FLAG=1s \
        ROLLUP_VERIFIER_ENABLE=true \
        ROLLUP_DISABLE_TRANSFERS=false \
        ROLLUP_ENABLE_L2_GAS_POLLING=false \
        ROLLUP_GAS_PRICE_ORACLE_OWNER_ADDRESS=0x648E3e8101BFaB7bf5997Bd007Fb473786019159 \
        ROLLUP_BACKEND=l1 \
        GCMODE=archive \
        CACHE=32768 \
        DATADIR=/root/data/optimism/geth/l2geth/ \
        CHAIN_ID=10 \
        NETWORK_ID=10 \
        NO_DISCOVER=true \
        NO_USB=true \
        GASPRICE=0 \
        TARGET_GAS_LIMIT=15000000 \
        RPC_ADDR=0.0.0.0 \
        RPC_API=eth,rollup,net,web3,debug \
        RPC_CORS_DOMAIN=* \
        RPC_ENABLE=true \
        RPC_PORT=9656 \
        RPC_VHOSTS=* \
        WS_ADDR=0.0.0.0 \
        WS_API=eth,rollup,net,web3,debug \
        WS_ORIGINS=* \
        WS=true
ExecStart=/root/github/optimism-legacy/l2geth/build/bin/geth \
        --datadir=/root/data/optimism/geth/l2geth/ \
        --password=/root/data/optimism/geth/l2geth/password \
        --allow-insecure-unlock \
        --unlock=0x00000398232E2064F896018496b4b44b3D62751F \
        --mine \
        --miner.etherbase=0x00000398232E2064F896018496b4b44b3D62751F \
        --vmodule=eth/*=5,miner=4,rpc=5,rollup=4,consensus/clique=1 \
        --cache=32768
KillSignal=SIGTERM

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/l2geth.service
```

#### To check if l2geth is running correctly, open another command line window and run these commands:

```bash
/root/optimism/l2geth/build/bin/geth attach --datadir=/root/.local/share/optimism/l2geth/datadir

eth.blockNumber
```

Wait a few seconds and then look at the blocknumber again and exit:

```bash
eth.blockNumber

exit
```

## OP NODE

### Building and preparing OP Node

```bash
mkdir -p /root/data/optimism/geth/op-node/

cd /root/github/

git clone https://github.com/ethereum-optimism/optimism.git

cd optimism

git checkout v1.6.0

nvm install && npm install --global yarn && nvm use node && npm -g install pnpm

pnpm install

pnpm build

make op-node

# The binary is built at /root/github/optimism/op-node/bin/op-node
```

### Setting up the Systemd file for OP Node

```bash
#######################  !!! EDIT YOUR OWN L1 RPC ENDPOINT !!! ###################

echo "[Unit]
Description=Optimism OP Node Service
After=network.target
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
Type=simple
Restart=on-failure
RestartSec=5
TimeoutSec=900
User=root
Nice=0
LimitNOFILE=200000

WorkingDirectory=/root/data/optimism/geth/op-node/
#EnvironmentFile=/root/optimism/packages/data-transport-layer/.env
#Environment=

ExecStart=/root/github/optimism/op-node/bin/op-node \
                --l1=http://ethereum:port \
                --l2=http://0.0.0.0:8551 \
                --network=mainnet \
                --rpc.addr=0.0.0.0 \
                --rpc.port=9545 \
                --l2.jwt-secret=/root/data/optimism/geth/op-geth/jwt.txt \
                --l1.trustrpc \
                --l1.rpckind=erigon \
                --metrics.enabled \
                --metrics.addr=0.0.0.0 \
                --metrics.port=7300
KillSignal=SIGTERM
[Install]
WantedBy=multi-user.target" > /etc/systemd/system/op-node.service
```

## OP-GETH

### Building and preparing OP-GETH

```bash
cd /root/github/

git clone https://github.com/ethereum-optimism/op-geth.git

cd op-geth

git checkout v1.101308.1

make geth

# The binary is built at /root/github/op-geth/build/bin/geth
```

```bash
openssl rand -hex 32 > /root/data/optimism/geth/op-geth/jwt.txt
```

### Setting up the Systemd file for OP GETH

```bash
sudo echo "[Unit]
Description=Optimism OP GETH Service
After=network.target
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
Type=simple
Restart=on-failure
RestartSec=5
TimeoutSec=900
User=root
Nice=0
LimitNOFILE=200000

WorkingDirectory=/root/github/op-geth/build/bin/
#EnvironmentFile=/root/optimism/packages/data-transport-layer/.env
#Environment=

ExecStart=/root/github/op-geth/build/bin/geth \
		--datadir=/root/data/optimism/geth/op-geth \
		--http \
		--http.corsdomain="*" \
		--http.vhosts="*" \
		--http.addr=0.0.0.0 \
		--http.port=9657 \
		--http.api=web3,debug,eth,txpool,net,engine \
		--ws \
		--ws.addr=0.0.0.0 \
		--ws.port=8546 \
		--ws.origins="*" \
		--ws.api=debug,eth,txpool,net,engine,web3 \
		--metrics \
		--syncmode=full \
		--gcmode=archive \
		--nodiscover \
		--maxpeers=0 \
		--networkid=420 \
		--authrpc.vhosts="*" \
		--authrpc.addr=0.0.0.0 \
		--authrpc.port=8551 \
		--authrpc.jwtsecret=/root/data/optimism/geth/op-geth/jwt.txt \
		--rollup.sequencerhttp="<https://mainnet-sequencer.optimism.io>" \
		--rollup.disabletxpoolgossip=true \
		--rollup.historicalrpc="<http://0.0.0.0:9656>"
KillSignal=SIGTERM
[Install]
WantedBy=multi-user.target" > /etc/systemd/system/op-geth.service
```
