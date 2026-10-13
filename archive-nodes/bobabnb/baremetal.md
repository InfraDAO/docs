# 💻 Baremetal

Authors: \[man4ela | catapulta.eth]

## System Requirements

|      CPU     |           OS           |      RAM     |         DISK         |
| :----------: | :--------------------: | :----------: | :------------------: |
| 8+ cores CPU | Debian 12/Ubuntu 22.04 | => 16 GB RAM | 550GB+ (SSD or NVMe) |

{% hint style="info" %}
_The BobaBNB archive node has a size of 525GB on August 8th, 2024_
{% endhint %}

## BobaBNB

{% hint style="success" %}
Boba Network is built on the Optimistic Rollup developed by [Optimism](https://optimism.io/), which ensures EVM and Solidity compatibility, minimizing the efforts required to migrate smart contracts from L1 to L2.&#x20;

In this guide, we are walking through the process of setting up a **BobaBNB** archive node using `l2geth and DTL (Data Transport Layer)`.&#x20;

**BobaBNB** is a Layer 2 scaling solution for the Binance Smart Chain (BSC), designed to enhance transaction throughput and reduce fees while maintaining the security of BSC. By deploying a **BobaBNB** archive node, you gain access to the complete transaction history, enabling advanced queries and analytics
{% endhint %}

## Pre-Requisites

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y

sudo apt install -y git make wget aria2 gcc pkg-config libusb-1.0-0-dev libudev-dev jq gcc g++ curl libssl-dev screen apache2-utils build-essential pkg-config
```

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

Allow remote RPC connections with Blast Node

```bash
sudo ufw allow from ${REMOTE.HOST.IP} to any port 8545
```

{% hint style="warning" %}
Not advised to allow all or unknown IP address to RPC port
{% endhint %}

Enable Firewall

<pre class="language-bash"><code class="lang-bash"><strong>sudo ufw enable
</strong></code></pre>

To check the status of UFW and see the current rules

<pre class="language-bash"><code class="lang-bash"><strong>sudo ufw status verbose
</strong></code></pre>

## Install dependencies

#### Required Software Dependencies

<table><thead><tr><th width="154">Dependency</th><th width="110" align="center">Version</th><th width="233">Version Check Command</th></tr></thead><tbody><tr><td><mark style="color:green;">go</mark></td><td align="center"><code>^1.21</code></td><td><code>go version</code></td></tr><tr><td><mark style="color:orange;">node</mark></td><td align="center"><code>^20</code></td><td><code>node --version</code></td></tr><tr><td><mark style="color:blue;">pnpm</mark></td><td align="center"><code>^8</code></td><td><code>pnpm --version</code></td></tr><tr><td><mark style="color:green;">foundry</mark></td><td align="center"><code>^0.2.0</code></td><td><code>forge --version</code></td></tr><tr><td><mark style="color:orange;">make</mark></td><td align="center"><code>^4</code></td><td><code>make --version</code></td></tr><tr><td><mark style="color:green;">yarn</mark></td><td align="center"><code>1.22.21</code></td><td><code>yarn --version</code></td></tr><tr><td><mark style="color:blue;">nvm</mark></td><td align="center"><code>0.39.3</code></td><td><code>nvm --verison</code></td></tr></tbody></table>

### Install GO

{% code overflow="wrap" fullWidth="false" %}
```bash
sudo wget https://go.dev/dl/go1.21.6.linux-amd64.tar.gz && sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.21.6.linux-amd64.tar.gz && rm go1.21.6.linux-amd64.tar.gz

#to verify Go installation
go version

#If it returns Command 'go' not found simply run 
echo 'export PATH=$PATH:/usr/local/go/bin:/root/.local/bin' >> /root/.bashrc

#and then apply changes with

source /root/.bashrc
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
source /root/.bashrc

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

## Build the Execution Engine (l2geth)

#### Clone the Boba Legacy Monorepo and build l2geth:

```bash
mkdir bobabnb && cd bobabnb

git clone https://github.com/bobanetwork/boba_legacy.git

cd boba_legacy

cd l2geth

make geth
```

#### Create database directories for l2geth and DTL:

```bash
mkdir -p /root/data/bobabnb/geth/dtl

mkdir -p /root/data/bobabnb/geth/l2geth
```

#### Creating password and block-signer key

<pre class="language-bash"><code class="lang-bash">cd /root/data/bobabnb/geth/l2geth/

touch password

<strong>echo "6587ae678cf4fc9a33000cdbf9f35226b71dcc6a4684a31203241f9bcfd55d27" > /root/data/bobabnb/geth/l2geth/block-signer-key
</strong>
/root/bobabnb/boba_legacy/l2geth/build/bin/geth account import --datadir=/root/data/bobabnb/geth/l2geth/ --password /root/data/bobabnb/geth/l2geth/password /root/data/bobabnb/geth/l2geth/block-signer-key
</code></pre>

#### Create systemd service for l2geth

```bash
sudo nano /etc/systemd/system/l2geth.service
```

Paste the configs and save by entering `ctrl+X` and `Y+ENTER`:

```bash
[Unit]
Description=BobaBNB L2 GETH Service
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
WorkingDirectory=/root/data/bobabnb/geth/l2geth/
#EnvironmentFile=/root/bobabnb/boba_legacy/packages/data-transport-layer/.env
Environment=DATADIR='/root/data/bobabnb/geth/l2geth/'
Environment=CHAIN_ID=56288
Environment=NETWORK_ID=56288
Environment=NO_DISCOVER=true
Environment=NO_USB=true
Environment=GASPRICE=0
Environment=TARGET_GAS_LIMIT=15000000
Environment=RPC_ADDR=0.0.0.0
Environment=RPC_API="eth,rollup,net,web3,debug"
Environment=RPC_CORS_DOMAIN=*
Environment=RPC_ENABLE=true
Environment=RPC_PORT=8545
Environment=RPC_VHOSTS=*
Environment=NODE_TYPE='archive'
Environment=ROLLUP_TIMESTAMP_REFRESH=5s
Environment=ROLLUP_STATE_DUMP_PATH=http://127.0.0.1:8081/state-dump.latest.json
Environment=ROLLUP_CLIENT_HTTP=http://127.0.0.1:7878
Environment=ROLLUP_BACKEND='l2'
Environment=ROLLUP_VERIFIER_ENABLE='true'
Environment=RETRIES=60
Environment=BLOCK_SIGNER_KEY="6587ae678cf4fc9a33000cdbf9f35226b71dcc6a4684a31203241f9bcfd55d27"
Environment=BLOCK_SIGNER_ADDRESS="0x00000398232E2064F896018496b4b44b3D62751F"
Environment=ROLLUP_POLL_INTERVAL_FLAG="10s"
Environment=ROLLUP_ENFORCE_FEES='true'
Environment=TURING_CREDIT_ADDRESS="0x4200000000000000000000000000000000000020"
Environment=L2_BOBA_TOKEN_ADDRESS="0x4200000000000000000000000000000000000023"
Environment=BOBA_GAS_PRICE_ORACLE_ADDRESS="0x4200000000000000000000000000000000000024"
Environment=SEQUENCER_CLIENT_HTTP='https://bnb.boba.network/'
Environment=ETH1_HTTP='https://bsc-dataseed.binance.org/'
Environment=ETH1_SYNC_SERVICE_ENABLE=true
Environment=ETHERBASE=0x7E5F4552091A69125d5DfCb7b8C2659029395Bdf
Environment=IPC_DISABLE=true
Environment=NO_DISCOVER=true
Environment=TARGET_GAS_LIMIT=11000000
Environment=ROLLUP_ENABLE_L2_GAS_POLLING=true
Environment=ETH1_CONFIRMATION_DEPTH=0
Environment=ETH1_CTC_DEPLOYMENT_HEIGHT=1305672
Environment=USING_OVM=true
ExecStart=/root/bobabnb/boba_legacy/l2geth/build/bin/geth \
        --datadir=/root/data/bobabnb/geth/l2geth/ \
        --password=/root/data/bobabnb/geth/l2geth/password \
        --allow-insecure-unlock \
        --unlock 0x00000398232E2064F896018496b4b44b3D62751F \
        --mine \
        --miner.etherbase 0x00000398232E2064F896018496b4b44b3D62751F \
        --gcmode=archive \
        --port=30301 \
        --ws=false \
        --nousb \
        --rollup.clienthttp http://127.0.0.1:7878 \
        --rpc \
        --rpcaddr 0.0.0.0 \
        --rpcport 8545 \
        --rpcapi eth,net,rollup,web3,debug,personal \
        --rangelimit \
        --rpc.gascap 501000000
KillSignal=SIGTERM

[Install]
WantedBy=multi-user.target
```

## Build Data Transport Layer (DTL)

```bash
cd /root/bobabnb/boba_legacy/

yarn

yarn build
```

#### Create systemd service for DTL

```bash
sudo nano /etc/systemd/system/dtl.service
```

Paste DTL configs and save by entering `ctrl+X` and `Y+ENTER`:

```bash
[Unit]
Description=BobaBNB DTL Service
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
WorkingDirectory=/root/bobabnb/boba_legacy/packages/data-transport-layer
ExecStart=/bin/bash -c '. /root/.nvm/nvm.sh && /root/.nvm/versions/node/v18.12.0/bin/node /root/.nvm/versions/node/v18.12.0/bin/yarn start'
#EnvironmentFile=/root/bobabnb/boba_legacy/packages/data-transport-layer/.env
Environment=DATA_TRANSPORT_LAYER__L1_RPC_ENDPOINT='https://bsc-dataseed.binance.org/'
Environment=DATA_TRANSPORT_LAYER__L2_RPC_ENDPOINT='https://replica.bnb.boba.network'
Environment=DATA_TRANSPORT_LAYER__L2_CHAIN_ID=56288
Environment=DATA_TRANSPORT_LAYER__TRANSACTIONS_PER_POLLING_INTERVAL=1000
Environment=DATA_TRANSPORT_LAYER__POLLING_INTERVAL=4000
Environment=DATA_TRANSPORT_LAYER__LOGS_PER_POLLING_INTERVAL=2000
Environment=DATA_TRANSPORT_LAYER__ETH1_CTC_DEPLOYMENT_HEIGHT=1305672
Environment=DATA_TRANSPORT_LAYER__ADDRESS_MANAGER='0xeb989B25597259cfa51Bd396cE1d4B085EC4c753'
Environment=DATA_TRANSPORT_LAYER__BSS_HARDFORK_1_INDEX=0
Environment=DATA_TRANSPORT_LAYER__TURING_V0_HEIGHT=0
Environment=DATA_TRANSPORT_LAYER__TURING_V1_HEIGHT=0
Environment=DATA_TRANSPORT_LAYER__DB_PATH="/root/data/bobabnb/geth/dtl"
Environment=DATA_TRANSPORT_LAYER__DANGEROUSLY_CATCH_ALL_ERRORS=true
Environment=DATA_TRANSPORT_LAYER__SERVER_HOSTNAME=0.0.0.0
Environment=DATA_TRANSPORT_LAYER__SERVER_PORT=7878
Environment=DATA_TRANSPORT_LAYER__SYNC_FROM_L1=false
Environment=DATA_TRANSPORT_LAYER__SYNC_FROM_L2=true
KillSignal=SIGTERM
[Install]
WantedBy=multi-user.target
```

#### Create Environment file for DTL:

```bash
sudo nano /root/bobabnb/boba_legacy/packages/data-transport-layer/.env
```

#### Paste the configs and save changes by entering `ctrl+X` and `Y+ENTER`:

```
DATA_TRANSPORT_LAYER__L1_RPC_ENDPOINT='https://bsc-dataseed.binance.org/'
DATA_TRANSPORT_LAYER__L2_RPC_ENDPOINT='https://replica.bnb.boba.network'
DATA_TRANSPORT_LAYER__L2_CHAIN_ID=56288
DATA_TRANSPORT_LAYER__TRANSACTIONS_PER_POLLING_INTERVAL=1000
DATA_TRANSPORT_LAYER__POLLING_INTERVAL=4000
DATA_TRANSPORT_LAYER__LOGS_PER_POLLING_INTERVAL=2000
DATA_TRANSPORT_LAYER__ETH1_CTC_DEPLOYMENT_HEIGHT=1305672
DATA_TRANSPORT_LAYER__ADDRESS_MANAGER='0xeb989B25597259cfa51Bd396cE1d4B085EC4c753'
DATA_TRANSPORT_LAYER__BSS_HARDFORK_1_INDEX=0
DATA_TRANSPORT_LAYER__TURING_V0_HEIGHT=0
DATA_TRANSPORT_LAYER__TURING_V1_HEIGHT=0
DATA_TRANSPORT_LAYER__DB_PATH="/root/data/bobabnb/geth/dtl"
DATA_TRANSPORT_LAYER__DANGEROUSLY_CATCH_ALL_ERRORS=true
DATA_TRANSPORT_LAYER__SERVER_HOSTNAME=0.0.0.0
DATA_TRANSPORT_LAYER__SERVER_PORT=7878
DATA_TRANSPORT_LAYER__SYNC_FROM_L1=false
DATA_TRANSPORT_LAYER__SYNC_FROM_L2=true
DATADIR='/root/data/bobabnb/geth/l2geth/'
CHAIN_ID=56288
NETWORK_ID=56288
NO_DISCOVER=true
NO_USB=true
GASPRICE=0
TARGET_GAS_LIMIT=15000000
RPC_ADDR=0.0.0.0
RPC_API="eth,rollup,net,web3,debug"
RPC_CORS_DOMAIN=*
RPC_ENABLE=true
RPC_PORT=8545
RPC_VHOSTS=*
NODE_TYPE='archive'
ROLLUP_TIMESTAMP_REFRESH=5s
ROLLUP_STATE_DUMP_PATH=http://127.0.0.1:8081/state-dump.latest.json
ROLLUP_CLIENT_HTTP=http://127.0.0.1:7878
ROLLUP_BACKEND='l2'
ROLLUP_VERIFIER_ENABLE='true'
RETRIES=60
BLOCK_SIGNER_KEY="6587ae678cf4fc9a33000cdbf9f35226b71dcc6a4684a31203241f9bcfd55d27"
BLOCK_SIGNER_ADDRESS="0x00000398232E2064F896018496b4b44b3D62751F"
ROLLUP_POLL_INTERVAL_FLAG="10s"
ROLLUP_ENFORCE_FEES='true'
TURING_CREDIT_ADDRESS="0x4200000000000000000000000000000000000020"
L2_BOBA_TOKEN_ADDRESS="0x4200000000000000000000000000000000000023"
BOBA_GAS_PRICE_ORACLE_ADDRESS="0x4200000000000000000000000000000000000024"
SEQUENCER_CLIENT_HTTP='https://bnb.boba.network/'
ETH1_HTTP='https://bsc-dataseed.binance.org/'
ETH1_SYNC_SERVICE_ENABLE=true
ETHERBASE=0x7E5F4552091A69125d5DfCb7b8C2659029395Bdf
IPC_DISABLE=true
NO_DISCOVER=true
TARGET_GAS_LIMIT=11000000
ROLLUP_ENABLE_L2_GAS_POLLING=true
ETH1_CONFIRMATION_DEPTH=0
ETH1_CTC_DEPLOYMENT_HEIGHT=1305672
USING_OVM=true
```

#### Import genesis information and initialize l2geth

```bash
cd /root/bobabnb/boba_legacy/packages/data-transport-layer

wget https://raw.githubusercontent.com/bobanetwork/boba_legacy/develop/boba_community/boba-node/state-dumps/bobabnb/state-dump.latest.json -O /root/bobabnb/boba_legacy/packages/data-transport-layer/state-dump.latest.json

set -o allexport; source /root/bobabnb/boba_legacy/packages/data-transport-layer/.env; set +o allexport; /root/bobabnb/boba_legacy/l2geth/build/bin/geth init --datadir=/root/data/bobabnb/geth/l2geth /root/bobabnb/boba_legacy/packages/data-transport-layer/state-dump.latest.json --nousb
```

## Launch BobaBNB

#### Start DTL

```bash
sudo systemctl daemon-reload #refresh systemd configuration when changes made

sudo systemctl enable dtl.service #enable dtl service at system startup

sudo systemctl start dtl.service #start dtl

sudo nano /etc/systemd/system/dtl.service #make changes in dtl.service file
```

#### Start l2geth

```bash
sudo systemctl daemon-reload #refresh systemd configuration when changes made

sudo systemctl enable l2geth.service #enable l2geth service at system startup

sudo systemctl start l2geth.service #start l2geth

sudo nano /etc/systemd/system/l2geth.service #make changes in l2geth.service file
```

### Run _`curl`_ command in the terminal to check the status of your node

<pre class="language-bash"><code class="lang-bash"><strong>curl -H "Content-type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' http://localhost:8545
</strong></code></pre>

When it returns `false` then your node is fully synchronized with the network

### Monitor the logs for errors

```bash
sudo journalctl -fu dtl.service #follow logs of dtl.service

sudo journalctl -fu l2geth.service #follow logs of l2geth.service
```

During the synchonization, you are expected to get following log messages from `DTL`:

```bash
{"level":30,"time":1722479386601,"method":"GET","url":"/transaction/latest?backend=l2","elapsed":0,"msg":"Served HTTP Request"}
{"level":30,"time":1722479393551,"fromBlock":39649105,"toBlock":39649106,"msg":"Synchronizing unconfirmed transactions from Layer 2 (Optimism)"}
```

and `l2geth`:

```bash
INFO [08-01|04:29:56.601] Syncing transaction range                start=39649105 end=39649105 backend=l2
INFO [08-01|04:29:56.609] New block                                index=39649105 l1-timestamp=1722479389 l1-blocknumber=40970591 tx-hash=0x0256f7a95b88f10495ae3a67642009b7ee681c730aac249df16472a90e7be>
INFO [08-01|04:30:07.488] Deep froze chain segment                 blocks=3   elapsed=119.358ms number=39559105 hash=256255…ce8334
```

### References

{% embed url="https://docs.boba.network/" %}

{% embed url="https://github.com/bobanetwork/boba_legacy/blob/develop/boba_community/boba-node/docker-compose-bobabnb.yml" %}

