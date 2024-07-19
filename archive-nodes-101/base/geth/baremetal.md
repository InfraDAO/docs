---
description: 'Authors: [man4ela | catapulta.eth]'
---

# 💻 Baremetal

## System Requirements

|      CPU     |           OS           |      RAM     |          DISK         |
| :----------: | :--------------------: | :----------: | :-------------------: |
| 8+ cores CPU | Debian 12/Ubuntu 22.04 | => 16 GB RAM | 6TB+ (NVME preffered) |

{% hint style="info" %}
_The Base archive node reached a size of 6.1TB by July 15, 2024_
{% endhint %}

## Base <mark style="color:blue;">🔵</mark>

{% hint style="success" %}
Base is a secure, low-cost Ethereum L2 built on Optimism’s open-source [OP Stack](https://stack.optimism.io/). In this guide, Optimism's `op-geth` and `op-node`binaries are built from source to facilitate the node's installation. This method has proved to sync an archive node successfully in \~48 hours using the official snapshot provided by the Base team.
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

_Create a directory and start downloading an archive in screen session as it takes \~9 hours_

{% code overflow="wrap" %}
```bash
mkdir base && cd base

screen -S archive

aria2c --file-allocation=none -c -x 10 -s 10 "https://base-snapshots-mainnet-archive.s3.amazonaws.com/$(curl https://base-snapshots-mainnet-archive.s3.amazonaws.com/latest)"
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

<pre class="language-bash" data-overflow="wrap" data-full-width="false"><code class="lang-bash">sudo wget https://go.dev/dl/go1.21.6.linux-amd64.tar.gz &#x26;&#x26; sudo rm -rf /usr/local/go &#x26;&#x26; sudo tar -C /usr/local -xzf go1.21.6.linux-amd64.tar.gz &#x26;&#x26; rm go1.21.6.linux-amd64.tar.gz

#to verify Go installation
go version

#If it returns Command 'go' not found simply run 
echo 'export PATH=$PATH:/usr/local/go/bin:/root/.local/bin' >> /root/.bashrc
<strong>
</strong><strong>#and then apply changes with
</strong>
source /root/.bashrc
</code></pre>

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
source /root/.bashrc #or start a new terminal session to use foundryup

foundryup
```

### Install node and yarn

```bash
nvm install 18.12.0 && npm install --global yarn && nvm use 18.12.0 && npm -g install pnpm

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
mkdir -p /root/data/base/geth/op-geth
```

### Build op-node

```bash
cd /root/github/

git clone https://github.com/ethereum-optimism/optimism.git

cd optimism

git checkout v1.7.7

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

WorkingDirectory=/root/data/base/geth/op-node
Environment=OP_GETH_GENESIS_FILE_PATH=/root/data/base/geth/op-geth/genesis-l2.json \
OP_GETH_SEQUENCER_HTTP=https://mainnet-sequencer.base.org \
OP_GETH_BOOTNODES=enode://87a32fd13bd596b2ffca97020e31aef4ddcc1bbd4b95bb633d16c1329f654f34049ed240a36b449fda5e5225d70fe40bc667f53c304b71f8e68fc9d448690b51@3.231.138.188:30301,enode://ca21ea8f176adb2e229ce2d700830c844af0ea941a1d8152a9513b966fe525e809c3a6c73a2c18a12b74ed6ec4380edf91662778fe0b79f6a591236e49e176f9@184.72.129.189:30301,enode://acf4507a211ba7c1e52cdf4eef62cdc3c32e7c9c47998954f7ba024026f9a6b2150cd3f0b734d9c78e507ab70d59ba61dfe5c45e1078c7ad0775fb251d7735a2@3.220.145.177:30301,enode://8a5a5006159bf079d06a04e5eceab2a1ce6e0f721875b2a9c96905336219dbe14203d38f70f3754686a6324f786c2f9852d8c0dd3adac2d080f4db35efc678c5@3.231.11.52:30301,enode://cdadbe835308ad3557f9a1de8db411da1a260a98f8421d62da90e71da66e55e98aaa8e90aa7ce01b408a54e4bd2253d701218081ded3dbe5efbbc7b41d7cef79@54.198.153.150:30301 \
OP_NODE_L1_ETH_RPC={L1 RPC URL} \
OP_NODE_L1_BEACON={L1 BEACON RPC URL} \
OP_NODE_L2_ENGINE_AUTH= /root/data/base/geth/op-geth/jwt.hex \
OP_NODE_L2_ENGINE_RPC=http://0.0.0.0:8551 \
OP_NODE_LOG_LEVEL=info \
OP_NODE_METRICS_ADDR=0.0.0.0 \
OP_NODE_METRICS_ENABLED=true \
OP_NODE_METRICS_PORT=7200 \
OP_NODE_NETWORK=base-mainnet \
OP_NODE_P2P_AGENT=base \
OP_NODE_P2P_LISTEN_IP=0.0.0.0 \
OP_NODE_P2P_LISTEN_TCP_PORT=9222 \
OP_NODE_P2P_LISTEN_UDP_PORT=9222 \
OP_NODE_ROLLUP_CONFIG=/root/data/base/geth/op-geth/rollup.json \
OP_NODE_P2P_BOOTNODES=enr:-J24QNz9lbrKbN4iSmmjtnr7SjUMk4zB7f1krHZcTZx-JRKZd0kA2gjufUROD6T3sOWDVDnFJRvqBBo62zuF-hYCohOGAYiOoEyEgmlkgnY0gmlwhAPniryHb3BzdGFja4OFQgCJc2VjcDI1NmsxoQKNVFlCxh_B-716tTs-h1vMzZkSs1FTu_OYTNjgufplG4N0Y3CCJAaDdWRwgiQG,enr:-J24QH-f1wt99sfpHy4c0QJM-NfmsIfmlLAMMcgZCUEgKG_BBYFc6FwYgaMJMQN5dsRBJApIok0jFn-9CS842lGpLmqGAYiOoDRAgmlkgnY0gmlwhLhIgb2Hb3BzdGFja4OFQgCJc2VjcDI1NmsxoQJ9FTIv8B9myn1MWaC_2lJ-sMoeCDkusCsk4BYHjjCq04N0Y3CCJAaDdWRwgiQG,enr:-J24QDXyyxvQYsd0yfsN0cRr1lZ1N11zGTplMNlW4xNEc7LkPXh0NAJ9iSOVdRO95GPYAIc6xmyoCCG6_0JxdL3a0zaGAYiOoAjFgmlkgnY0gmlwhAPckbGHb3BzdGFja4OFQgCJc2VjcDI1NmsxoQJwoS7tzwxqXSyFL7g0JM-KWVbgvjfB8JA__T7yY_cYboN0Y3CCJAaDdWRwgiQG,enr:-J24QHmGyBwUZXIcsGYMaUqGGSl4CFdx9Tozu-vQCn5bHIQbR7On7dZbU61vYvfrJr30t0iahSqhc64J46MnUO2JvQaGAYiOoCKKgmlkgnY0gmlwhAPnCzSHb3BzdGFja4OFQgCJc2VjcDI1NmsxoQINc4fSijfbNIiGhcgvwjsjxVFJHUstK9L1T8OTKUjgloN0Y3CCJAaDdWRwgiQG,enr:-J24QG3ypT4xSu0gjb5PABCmVxZqBjVw9ca7pvsI8jl4KATYAnxBmfkaIuEqy9sKvDHKuNCsy57WwK9wTt2aQgcaDDyGAYiOoGAXgmlkgnY0gmlwhDbGmZaHb3BzdGFja4OFQgCJc2VjcDI1NmsxoQIeAK_--tcLEiu7HvoUlbV52MspE0uCocsx1f_rYvRenIN0Y3CCJAaDdWRwgiQG \
OP_NODE_RPC_ADDR=0.0.0.0 \
OP_NODE_RPC_PORT=7545 \
OP_NODE_SNAPSHOT_LOG=/tmp/op-node-snapshot-log \
OP_NODE_VERIFIER_L1_CONFS=4 \
OP_NODE_ROLLUP_LOAD_PROTOCOL_VERSIONS=true \
OP_NODE_L1_TRUST_RPC=true

ExecStart=/root/github/optimism/op-node/bin/op-node \
--l1={L1 RPC URL} \
--l1.beacon={L1 BEACON RPC URL} \
--l2=http://0.0.0.0:8551 \
--l2.jwt-secret=/root/data/base/geth/op-geth/jwt.hex \
--rollup.config=/root/data/base/geth/op-geth/rollup.json
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

## Compile op-geth

```bash
cd /root/github/

git clone https://github.com/ethereum-optimism/op-geth.git

cd op-geth

git checkout v1.101315.2

make geth
```

_#The binary is built at /root/github/op-geth/build/bin/geth_

#### Create JWT secret file and download genesis and rollup .json files

```bash
cd /root/data/base/geth/op-geth/

openssl rand -hex 32 > /root/data/base/geth/op-geth/jwt.txt

curl -LO https://raw.githubusercontent.com/base-org/node/main/mainnet/genesis-l2.json 
curl -LO https://raw.githubusercontent.com/base-org/node/main/mainnet/rollup.json

```

### Create systemd service

```bash
sudo echo "[Unit]
Description=BASE OP GETH Service
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
ExecStart=/root/github/op-geth/build/bin/geth \
            --datadir=/root/data/base/geth/op-geth \
            --verbosity=3 \
            --http \
            --http.corsdomain=* \
            --http.vhosts=* \
            --http.addr=0.0.0.0 \
            --http.port=8545 \
            --http.api=web3,debug,eth,txpool,net,engine \
            --authrpc.addr=0.0.0.0 \
            --authrpc.port=8551 \
            --authrpc.vhosts=* \
            --authrpc.jwtsecret=/root/data/base/geth/op-geth/jwt.hex \
            --ws \
            --ws.addr=0.0.0.0 \
            --ws.port=8546 \
            --ws.origins=* \
            --ws.api=debug,eth,txpool,net,engine \
            --metrics \
            --metrics.addr=0.0.0.0 \
            --metrics.port=7300 \
            --syncmode=full \
            --gcmode=archive \
            --nodiscover \
            --maxpeers=100 \
            --networkid=8453 \
            --nat=extip:0.0.0.0 \
            --rollup.sequencerhttp=https://mainnet-sequencer.base.org
KillSignal=SIGTERM
[Install]
WantedBy=multi-user.target" > /etc/systemd/system/op-geth.service

```

{% hint style="info" %}
If you wish to sync from scratch, consider bootstraping the node first by running

`/root/github/op-geth/build/bin/geth --datadir /root/data/base/geth/op-geth init /root/data/base/geth/op-geth/genesis-l2.json`
{% endhint %}

## Sync using downloaded Snapshot

```bash
screen –r archive

ls #to see the name of downloaded archive

dtrx -f base-mainnet-archive-xxxxxx.tar.gz
```

_#Unzipping takes \~3-4 hrs so you can go touch some grass_

Consider switching screen by pressing`ctrl A+D`to allow a process run in the background

#### After extracting is done move the contents of geth directory into op-geth data directoy:

```bash
mv /root/base/snapshots/mainnet/download/geth/* /root/data/base/geth/op-geth/geth/
```

### Start op-geth

```bash
sudo systemctl daemon-reload #refresh systemd configuration when changes made

sudo systemctl start op-geth.service #start op-geth

sudo systemctl enable op-geth.service #enable op-geth service at system startup

sudo journalctl -fu op-geth.service #follow logs of op-geth service
```

{% hint style="info" %}
To check or modify `op-geth.service` parameters simply run&#x20;

`sudo nano /etc/systemd/system/op-geth.service`

Ctrl+X and Y to save changes
{% endhint %}

#### You can also run `curl` command in the terminal to check the status of your node

```bash
curl -d '{"id":0,"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["latest",false]}' \
  -H "Content-Type: application/json" http://localhost:8545
```

You can see If blocks increase at [https://base.blockscout.com/](https://base.blockscout.com/) by entering returned hash. It means the node is catching up and the setup is successful.

## References

{% embed url="https://docs.optimism.io/builders/node-operators/tutorials/node-from-source" %}

{% embed url="https://github.com/base-org/node" %}

{% embed url="https://docs.base.org/tutorials/run-a-base-node" %}

