---
description: 'Authors: [payne | stakesquid]'
---

# 💻 Baremetal

## System Requirements

|      CPU     |           OS           |      RAM     |           DISK          |
| :----------: | :--------------------: | :----------: | :---------------------: |
| 8+ cores CPU | Debian 12/Ubuntu 22.04 | => 16 GB RAM | 1TB+ (NVME preffered) |

{% hint style="info" %}
_The Base archive node reached a size of 804GB by May 2, 2024_
{% endhint %}

## Base <mark style="color:blue;">🔵</mark>

{% hint style="success" %}
Base is a secure, low-cost Ethereum L2 built on Optimism’s open-source [OP Stack](https://stack.optimism.io/). In this guide, Optimism's `op-erigon` and `op-node`binaries are built from source to facilitate the node's installation. This method has proved to sync an archive node successfully in \~48 hours using the official snapshot provided by the Base team.
{% endhint %}

{% hint style="warning" %}
Before you start, make sure that you have your own synced Ethereum L1 RPC URL (e.g. Erigon) and L1 Consensus Layer Beacon endpoint (e.g. Lighthouse) ready.
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

Allow remote RPC connections with Base Node

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
mkdir base && cd base

screen -S archive

aria2c --file-allocation=none -c -x 10 -s 10 "https://datadirs.testinprod.io/base-mainnet-db-14631082.zst"
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
mkdir -p /root/data/base/geth/op-node
mkdir -p /root/data/base/geth/op-erigon
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
You'll need your own synced Ethereum L1 RPC URL (e.g. Erigon) and L1 Consensus Layer Beacon endpoint (e.g. Lighthouse) in order to run Base
{% endhint %}

{% code overflow="wrap" %}
```bash
echo "[Unit]
Description=Base OP Node Service
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
WorkingDirectory=/root/data/base/op-node/
ExecStart=/root/data/optimism/op-node/bin/op-node \
        --l1=http://10.120.10.116:9656 \
        --l1.beacon=http://10.120.10.116:5052 \
        --l1.trustrpc=true \
        --l1.rpckind=erigon \
        --l2=http://0.0.0.0:8552 \
        --l2.jwt-secret=/root/data/base/erigon/jwt.hex \
        --rpc.addr=0.0.0.0 \
        --rpc.port=9546 \
        --rollup.config=/root/data/base/erigon/rollup.json \
        --metrics.enabled \
        --metrics.addr=0.0.0.0 \
        --metrics.port=7301 \
        --network=base-mainnet  \
        --p2p.listen.tcp=9923 \
        --p2p.listen.udp=9923 \
        --p2p.bootnodes=enr:-J24QNz9lbrKbN4iSmmjtnr7SjUMk4zB7f1krHZcTZx-JRKZd0kA2gjufUROD6T3sOWDVDnFJRvqBBo62zuF-hYCohOGAYiOoEyEgmlkgnY0gmlwhAPniryHb3BzdGFja4OFQgCJc2VjcDI1NmsxoQKNVFlCxh_B-716tTs-h1vMzZkSs1FTu_OYTNjgufplG4N0Y3CCJAaDdWRwgiQG,enr:-J24QH-f1wt99sfpHy4c0QJM-NfmsIfmlLAMMcgZCUEgKG_BBYFc6FwYgaMJMQN5dsRBJApIok0jFn-9CS842lGpLmqGAYiOoDRAgmlkgnY0gmlwhLhIgb2Hb3BzdGFja4OFQgCJc2VjcDI1NmsxoQJ9FTIv8B9myn1MWaC_2lJ-sMoeCDkusCsk4BYHjjCq04N0Y3CCJAaDdWRwgiQG,enr:-J24QDXyyxvQYsd0yfsN0cRr1lZ1N11zGTplMNlW4xNEc7LkPXh0NAJ9iSOVdRO95GPYAIc6xmyoCCG6_0JxdL3a0zaGAYiOoAjFgmlkgnY0gmlwhAPckbGHb3BzdGFja4OFQgCJc2VjcDI1NmsxoQJwoS7tzwxqXSyFL7g0JM-KWVbgvjfB8JA__T7yY_cYboN0Y3CCJAaDdWRwgiQG,enr:-J24QHmGyBwUZXIcsGYMaUqGGSl4CFdx9Tozu-vQCn5bHIQbR7On7dZbU61vYvfrJr30t0iahSqhc64J46MnUO2JvQaGAYiOoCKKgmlkgnY0gmlwhAPnCzSHb3BzdGFja4OFQgCJc2VjcDI1NmsxoQINc4fSijfbNIiGhcgvwjsjxVFJHUstK9L1T8OTKUjgloN0Y3CCJAaDdWRwgiQG,enr:-J24QG3ypT4xSu0gjb5PABCmVxZqBjVw9ca7pvsI8jl4KATYAnxBmfkaIuEqy9sKvDHKuNCsy57WwK9wTt2aQgcaDDyGAYiOoGAXgmlkgnY0gmlwhDbGmZaHb3BzdGFja4OFQgCJc2VjcDI1NmsxoQIeAK_--tcLEiu7HvoUlbV52MspE0uCocsx1f_rYvRenIN0Y3CCJAaDdWRwgiQG \
        --verifier.l1-confs=4 \
        --rollup.load-protocol-versions=true
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
cd /root/data/base/erigon

openssl rand -hex 32 > /root/data/base/erigon/jwt.txt
curl -LO https://raw.githubusercontent.com/base-org/node/main/mainnet/genesis-l2.json 
curl -LO https://raw.githubusercontent.com/base-org/node/main/mainnet/rollup.json

```

### Create systemd service

```bash
sudo echo "[Unit]
Description=Erigon Base Service
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
WorkingDirectory=/root/data/base/erigon/
ExecStart=/root/data/github/op-erigon/build/bin/erigon \
        --datadir=/root/data/base/erigon/datadir \
        --ethash.dagdir=/root/data/base/erigon/datadir/ethash \
        --authrpc.jwtsecret=/root/data/base/erigon/jwt.hex \
        --authrpc.port=8552 \
        --http \
        --http.addr=0.0.0.0 \
        --http.port=9660 \
        --http.compression \
        --http.vhosts=* \
        --http.corsdomain=* \
        --http.api=eth,debug,net,trace,web3,erigon \
        --private.api.addr=0.0.0.0:9095 \
        --ws --ws.compression \
        --metrics --metrics.addr=0.0.0.0 --metrics.port=9700 \
        --torrent.download.rate 80mb \
        --torrent.port=42070 \
        --rpc.returndata.limit=1000000 \
        --txpool.gossip.disable=true \
        --chain=base-mainnet \
        --db.size.limit=8TB \
        --nodiscover \
        --p2p.allowed-ports=30303,30304,30305,30306,30307,30308,30309,30310 \
        --rollup.sequencerhttp="https://mainnet-sequencer.base.org"
KillSignal=SIGTERM

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/op-erigon.service
```

## Sync using downloaded Snapshot

```bash
screen –r archive

ls #to see the name of downloaded archive

zstd --decompress base-mainnet-db-14631082.zst -o mdbx.dat
```

_#Unzipping takes \~3-4 hrs so you can go touch some grass_

Consider switching screen by pressing`ctrl A+D`to allow a process run in the background

#### After extracting is done move the contents of geth directory into op-erigon data directoy:

```bash
mv mdbx.dat /root/data/erigon/datadir/chaindata/
```

### Start op-erigon

```bash
sudo systemctl daemon-reload #refresh systemd configuration when changes made

sudo systemctl start op-erigon.service #start op-erigon

sudo systemctl enable op-erigon.service #enable op-erigon service at system startup

sudo journalctl -fu op-erigon.service #follow logs of op-erigon service
```

{% hint style="info" %}
To check or modify `op-erigon.service` parameters simply run&#x20;

`sudo nano /etc/systemd/system/op-erigon.service`

Ctrl+X and Y to save changes
{% endhint %}

