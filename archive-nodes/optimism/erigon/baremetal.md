---
description: 'Authors: [payne | stakesquid]'
---

# 💻 Baremetal

## System Requirements

|      CPU     |           OS           |      RAM     |                         DISK                         |
| :----------: | :--------------------: | :----------: | :--------------------------------------------------: |
| 8+ cores CPU | Debian 12/Ubuntu 22.04 | => 16 GB RAM | <p>2TB+ op-erigon<br><br>3.5TB+ l2geth (legacy) </p> |

{% hint style="info" %}
_Op-erigon reached a size of 2TB by Jan 10, 2025_\
_L2-geth is 3.4TB_
{% endhint %}

## Optimism <mark style="color:blue;">🔵</mark>

{% hint style="warning" %}
Before you start, make sure that you have your own synced Ethereum L1 RPC URL (e.g. Erigon) and L1 Consensus Layer Beacon endpoint (e.g. Lighthouse) ready.
{% endhint %}

{% hint style="warning" %}
To serve pre-bedrock eth\_calls, you will also need an l2geth (legacy) node. Instructions for how to set up an l2geth node can be found in [#optimism-l2-geth](../geth/baremetal.md#optimism-l2-geth "mention")
{% endhint %}

## Pre-Requisites

{% code overflow="wrap" %}
```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y

sudo apt install -y git make wget gcc pkg-config libusb-1.0-0-dev libudev-dev jq gcc g++ curl libssl-dev screen apache2-utils build-essential pkg-config
```
{% endcode %}

### Setting up Firewall

Set explicit default UFW rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Allow SSH

```bash
sudo ufw allow 22/tcp
```

Allow remote RPC connections with Optimism Node

```bash
sudo ufw allow from ${REMOTE.HOST.IP} to any port 8545
```

{% hint style="warning" %}
Not advised to allow all or unknown IP address to RPC port
{% endhint %}

Enable Firewall

```bash
sudo ufw enable
```

## Download a snapshot

Snapshots URL: https://snapshot.testinprod.io/

_Create a directory and start downloading an archive in screen session as it takes \~9 hours_

{% code overflow="wrap" %}
```bash
mkdir Optimism && cd Optimism

screen -S archive

aria2c --file-allocation=none -c -x 10 -s 10 "https://datadirs.testinprod.io/op-mainnet-db-120229131.zst"
```
{% endcode %}

```bash
#to return to previous screen and continue installation press 

Ctrl+a+d
```

## Compile Op-node

### Required Software Dependencies

<table><thead><tr><th width="154">Dependency</th><th width="110" align="center">Version</th><th width="233">Version Check Command</th></tr></thead><tbody><tr><td><mark style="color:green;">go</mark></td><td align="center"><code>^1.21</code></td><td><code>go version</code></td></tr><tr><td><mark style="color:orange;">node</mark></td><td align="center"><code>^20</code></td><td><code>node --version</code></td></tr><tr><td><mark style="color:blue;">pnpm</mark></td><td align="center"><code>^8</code></td><td><code>pnpm --version</code></td></tr><tr><td><mark style="color:green;">foundry</mark></td><td align="center"><code>^0.2.0</code></td><td><code>forge --version</code></td></tr><tr><td><mark style="color:orange;">make</mark></td><td align="center"><code>^4</code></td><td><code>make --version</code></td></tr><tr><td><mark style="color:green;">yarn</mark></td><td align="center"><code>1.22.21</code></td><td><code>yarn --version</code></td></tr><tr><td><mark style="color:blue;">nvm</mark></td><td align="center"><code>0.39.3</code></td><td><code>nvm --verison</code></td></tr></tbody></table>

### Install go

{% code overflow="wrap" fullWidth="false" %}
```bash
sudo wget https://go.dev/dl/go1.21.6.linux-amd64.tar.gz && sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.21.6.linux-amd64.tar.gz && rm go1.21.6.linux-amd64.tar.gz
```
{% endcode %}

### Install nvm

```bash
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
```

### Download foundry

```bash
curl -L https://foundry.paradigm.xyz | bash
```

### Install foundry

```bash
foundryup

source /root/.bashrc
```

### Install node and yarn

```bash
nvm install 16 && npm install --global yarn && nvm use 16 && npm -g install pnpm

source /root/.bashrc
```

### Check if go and all dependancies are installed

```bash
go version
nvm -v
npm -v
yarn -v
pnpm -v
```

### Create directories

```bash
mkdir -p /root/github
mkdir -p /root/data/optimism/op-node
mkdir -p /root/data/optimism/op-erigon
```

### Build op-node

```bash
cd /root/github/

git clone https://github.com/ethereum-optimism/optimism.git

cd optimism

git checkout v1.7.0

nvm install && npm install --global yarn && nvm use node && npm -g install pnpm

pnpm install

pnpm build

make op-node
```

_#The binary is built at /root/github/optimism/op-node/bin/op-node_

### Create systemd service

{% hint style="warning" %}
You'll need your own synced Ethereum L1 RPC URL (e.g. Erigon) and L1 Consensus Layer Beacon endpoint (e.g. Lighthouse) in order to run optimism
{% endhint %}

{% code overflow="wrap" %}
```bash
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
WorkingDirectory=/root/data/optimism/op-node/
ExecStart=/root/data/github/optimism/op-node/bin/op-node \
        --l1=http://<your_l1_eth_rpc> \
        --l2=http://0.0.0.0:8551 \
        --network=mainnet \
        --rpc.addr=0.0.0.0 \
        --rpc.port=9545 \
        --l2.jwt-secret=/root/data/optimism/erigon/jwt.hex \
        --l1.trustrpc \
        --l1.rpckind=erigon \
        --metrics.enabled \
        --l1.beacon=http://<your_l1_beacon_rpc> \
        --metrics.addr=0.0.0.0 \
        --metrics.port=7300
KillSignal=SIGTERM
[Install]
WantedBy=multi-user.target" > /etc/systemd/system/op-node.service
```
{% endcode %}

```bash
sudo nano /etc/systemd/system/op-node.service #make changes in op-node service file

sudo systemctl daemon-reload #refresh systemd configuration when changes made

sudo systemctl start op-node.service #start op-node

sudo systemctl enable op-node.service #enable op-node service at system startup

sudo journalctl -fu op-node.service #follow logs of op-node service
```

## Compile Erigon

```bash
cd /root/github/

git clone https://github.com/testinprod-io/op-erigon

cd op-erigon

git checkout v2.60.0-0.6.1

make
```

#### Create JWT secret file and download genesis and rollup .json files

```bash
cd /root/data/optimism/erigon

openssl rand -hex 32 > /root/data/optimism/erigon/jwt.txt

```

### Create systemd service

```bash
sudo echo "[Unit]
Description=Erigon Optimism Service
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
WorkingDirectory=/root/data/optimism/erigon/
ExecStart=/root/data/github/op-erigon/build/bin/erigon \
        --datadir=/root/data/optimism/erigon/datadir \
        --ethash.dagdir=/root/data/optimism/erigon/datadir/ethash \
        --authrpc.jwtsecret=/root/data/optimism/erigon/jwt.hex \
        --authrpc.port=8551 \
        --http \
        --http.addr=0.0.0.0 \
        --http.port=9659 \
        --http.compression \
        --http.vhosts=* \
        --http.corsdomain=* \
        --http.api=eth,debug,net,trace,web3,erigon \
        --private.api.addr=0.0.0.0:9094 \
        --ws --ws.compression \
        --metrics --metrics.addr=0.0.0.0 --metrics.port=9698 \
        --torrent.download.rate 80mb \
        --rpc.returndata.limit=1000000 \
        --txpool.gossip.disable=true \
        --chain=op-mainnet \
        --db.size.limit=8TB \
        --nodiscover \
        --rollup.sequencerhttp="https://mainnet-sequencer.optimism.io" \
        --rollup.historicalrpc="http://<your_l2-geth_endpoint>:9656"
KillSignal=SIGTERM

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/op-erigon.service
```

## Sync using downloaded Snapshot

```bash
screen –r archive

ls #to see the name of downloaded archive

zstd --decompress op-mainnet-db-120229131.zst -o mdbx.dat
```

_#Unzipping takes \~3-4 hrs so you can go touch some grass_

Consider switching screen by pressing`ctrl A+D`to allow a process run in the background

#### After extracting is done move the contents of geth directory into op-erigon data directoy:

```bash
mv mdbx.dat /root/data/op-erigon/datadir/chaindata/
```

### Start op-erigon

```bash
sudo systemctl daemon-reload #refresh systemd configuration when changes made

sudo systemctl start op-erigon.service #start op-erigon

sudo systemctl enable op-erigon.service #enable op-erigon service at system startup

sudo journalctl -fu op-erigon.service #follow logs of op-erigon service
```

{% hint style="info" %}
To check or modify `op-erigon.service` parameters simply run

`sudo nano /etc/systemd/system/op-erigon.service`

Ctrl+X and Y to save changes
{% endhint %}
