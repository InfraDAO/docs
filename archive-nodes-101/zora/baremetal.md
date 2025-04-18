# 💻 Baremetal

Authors: \[man4ela | catapulta.eth]

### System Requirements <a href="#system-requirements" id="system-requirements"></a>

Comment

| CPU          | OS                     | RAM          | DISK               |
| ------------ | ---------------------- | ------------ | ------------------ |
| 8+ cores CPU | Debian 12/Ubuntu 22.04 | => 16 GB RAM | 1TB+ (SSD or NVMe) |

{% hint style="info" %}
_The Zora Mainnet archive node has a size of 693GB on October 10th, 2024_
{% endhint %}

## Zora

{% hint style="success" %}
Zora operates within the Optimism _Superchain_ ecosystem and is built using the OP stack, leveraging the scalability and security of Optimism's Layer 2 infrastructure.

In this guide, we are walking through the process of setting up a Zora Mainnet archive node using Optimism's `op-gethand op-node`.
{% endhint %}

{% hint style="warning" %}
Before you start, make sure that you have your own synced Ethereum L1 RPC URL (e.g. Erigon) and L1 Consensus Layer Beacon endpoint with **`all historical blobs data`** (e.g. Lighthouse) ready. <mark style="color:red;">A beacon endpoint meeting this criteria is essential for syncing to start.</mark>

<mark style="color:orange;">**Hint:**</mark> [https://console.chainstack.com/user/account/create](https://console.chainstack.com/user/account/create) has a free plan enough to sync a node
{% endhint %}

## Pre-Requisites

{% code overflow="wrap" %}
```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y

sudo apt install -y git make wget aria2 gcc pkg-config libusb-1.0-0-dev libudev-dev jq gcc g++ curl libssl-dev screen apache2-utils build-essential pkg-config
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

Allow remote RPC connections with Zora Node

```bash
sudo ufw allow from ${REMOTE.HOST.IP} to any port 8545 
sudo ufw allow from ${REMOTE.HOST.IP} to any port 7545
```

{% hint style="warning" %}
Not advised to allow all or unknown IP address to RPC port
{% endhint %}

```bash
sudo ufw allow 9222
sudo ufw allow 30303
```

Enable Firewall

```bash
sudo ufw enable
```

To check the status of UFW and see the current rules

```bash
sudo ufw status verbose
```

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

### Check if go and all dependencies are installed

```bash
go version
nvm -v
npm -v
yarn -v
pnpm -v
```

## Build the Rollup Node (op-node)

#### Create database directory and jwt secret file

```bash
mkdir zora && cd zora

mkdir -p  /root/data/zora/zora-op-node/

mkdir -p  /root/data/zora/zora-op-geth/ && cd /root/data/zora/zora-op-geth/

openssl rand -hex 32 | tr -d "\n" > /root/data/zora/zora-op-geth/jwt.hex
```

#### Download genesis.json and rollup.json files

```bash
cd #to return to /root/ directory

git clone https://github.com/mehdius1364/node

cd node

export CONDUIT_NETWORK=zora-mainnet-0

./download-config.py $CONDUIT_NETWORK

#move genesis.json and rollup.json into op-geth directory 

mv /root/node/networks/zora-mainnet-0/genesis.json /root/node/networks/zora-mainnet-0/rollup.json /root/data/zora/zora-op-geth/
```

#### Build op-node

```bash
cd /root/zora/

git clone https://github.com/ethereum-optimism/optimism.git

cd optimism

git checkout v1.9.1

make op-node

# The binary is built at /root/zora/optimism/op-node/bin/op-node
```

### Create systemd service for op-node

```bash
sudo nano /etc/systemd/system/zora-op-node.service
```

#### Paste the following configs replacing `{L1 RPC},{L1 BEACON RPC},{SERVER IP},{JWT}` with own values

Save by entering `ctrl+X` and `Y+ENTER`

```bash
[Unit]
Description=Zora OP Node Service
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
WorkingDirectory=/root/data/zora/zora-op-node/
Environment=OP_NODE_L1_ETH_RPC={L1 RPC}
Environment=OP_NODE_L2_ENGINE_AUTH_RAW={JWT}
Environment=OP_NODE_L2_ENGINE_AUTH=/root/data/zora/zora-op-geth/jwt.hex
Environment=OP_NODE_L2_ENGINE_RPC=http://0.0.0.0:8551
Environment=OP_NODE_LOG_LEVEL=info
Environment=OP_NODE_METRICS_ADDR=0.0.0.0
Environment=OP_NODE_METRICS_ENABLED=true
Environment=OP_NODE_METRICS_PORT=7300
Environment=OP_NODE_P2P_BOOTNODES=enode://cd3730ae0a02324d4f529b1a0b492a4047552025c48dc8c9d6685af386dbe8de7780cb35567f76b9542537e96b9a5b160bee79edbd15fefa5a90371c12e57bed@34.127.98.251:9222?discport=30301,enode://d25ce99435982b04d60c4b41ba256b84b888626db7bee45a9419382300fbe907359ae5ef250346785bff8d3b9d07cd3e017a27e2ee3cfda3bcbb0ba762ac9674@bootnode.conduit.xyz:0?discport=30301,enode://2d4e7e9d48f4dd4efe9342706dd1b0024681bd4c3300d021f86fc75eab7865d4e0cbec6fbc883f011cfd6a57423e7e2f6e104baad2b744c3cafaec6bc7dc92c1@34.65.43.171:0?discport=30305,enode://9d7a3efefe442351217e73b3a593bcb8efffb55b4807699972145324eab5e6b382152f8d24f6301baebbfb5ecd4127bd3faab2842c04cd432bdf50ba092f6645@34.65.109.126:0?discport=30305"
Environment=OP_NODE_P2P_AGENT=conduit
Environment=OP_NODE_P2P_NAT=true
Environment=OP_NODE_P2P_ADVERTISE_IP={SERVER IP}
Environment=OP_NODE_P2P_LISTEN_IP=0.0.0.0
Environment=OP_NODE_P2P_LISTEN_TCP_PORT=9222
Environment=OP_NODE_P2P_LISTEN_UDP_PORT=9222
Environment=OP_NODE_ROLLUP_CONFIG=/root/data/zora/zora-op-geth/rollup.json
Environment=OP_NODE_RPC_ADDR=0.0.0.0
Environment=OP_NODE_RPC_PORT=7545
Environment=OP_NODE_SNAPSHOT_LOG=/tmp/op-node-snapshot-log
Environment=OP_NODE_VERIFIER_L1_CONFS=4
Environment=OP_NODE_L1_TRUST_RPC=true
Environment=OP_NODE_L1_BEACON={L1 BEACON RPC}
Environment=OP_NODE_OVERRIDE_CANYON=1704992401
Environment=OP_NODE_OVERRIDE_DELTA=1708560000
Environment=OP_NODE_OVERRIDE_ECOTONE=1710374401
Environment=OP_NODE_OVERRIDE_FJORD=1720627201
Environment=OP_NODE_OVERRIDE_GRANITE=1726070401

ExecStart=/root/zora/optimism/op-node/bin/op-node \
                --l1={L1 RPC} \
                --l2=http://0.0.0.0:8551 \
                --l1.trustrpc=true
KillSignal=SIGTERM
[Install] 
WantedBy=multi-user.target	

```

```bash
cat /root/data/zora/zora-op-geth/jwt.hex #to retrieve {JWT} value
```

## Build the Execution Engine (op-geth)

#### Build op-geth

```bash
cd /root/zora/

git clone https://github.com/ethereum-optimism/op-geth.git

cd  op-geth

git checkout v1.101408.0

make geth

# The binary is built at /root/github/op-geth/build/bin/geth
```

### Create systemd service for op-geth

```bash
sudo nano /etc/systemd/system/zora-op-geth.service
```

#### Paste the following configs:

<pre class="language-bash"><code class="lang-bash"><strong>[Unit]
</strong>Description=Zora OP GETH Service
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

WorkingDirectory=/root/zora/op-geth/build/bin/
Environment=GETH_GENESIS_FILE_PATH=/root/data/zora/zora-op-geth/genesis.json
Environment=GETH_ROLLUP_SEQUENCERHTTP=https://rpc-zora-mainnet-0.t.conduit.xyz
Environment=GETH_GCMODE=archive
Environment=HOST_IP=0.0.0.0
Environment=P2P_PORT=30303
Environment=WS_PORT=8546
Environment=OP_NODE_L2_ENGINE_AUTH=/root/data/zora/zora-op-geth/jwt.hex
Environment=GETH_OVERRIDE_CANYON=1704992401
Environment=GETH_OVERRIDE_DELTA=1708560000
Environment=GETH_OVERRIDE_ECOTONE=1710374401
Environment=GETH_OVERRIDE_FJORD=1720627201
Environment=GETH_OVERRIDE_GRANITE=1726070401

ExecStart=/root/zora/op-geth/build/bin/geth \
        --datadir=/root/data/zora/zora-op-geth/ \
        --verbosity=3 \
        --state.scheme=hash \
        --http \
        --http.corsdomain="*" \
        --http.vhosts="*" \
        --http.addr=0.0.0.0 \
        --http.port=8545 \
        --http.api=web3,debug,eth,net,engine \
        --authrpc.addr=0.0.0.0 \
        --authrpc.port=8551 \
        --authrpc.vhosts="*" \
        --authrpc.jwtsecret=/root/data/zora/zora-op-geth/jwt.hex \
        --ws \
        --ws.addr=0.0.0.0 \
        --ws.port=8546 \
        --ws.origins="*" \
        --ws.api=debug,eth,net,engine \
        --metrics \
        --metrics.addr=0.0.0.0 \
        --metrics.port=7200 \
        --syncmode=full \
        --gcmode=archive \
        --nodiscover \
        --maxpeers=100 \
        --nat=extip:0.0.0.0 \
        --rollup.sequencerhttp=https://rpc-zora-mainnet-0.t.conduit.xyz \
        --port=30303 \
        --op-network=zora-mainnet \
        --override.granite=1726070401
KillSignal=SIGTERM

[Install]
WantedBy=multi-user.target

</code></pre>

Save by entering `ctrl+X` and `Y+ENTER`

#### Initialize op-geth

```bash
/root/zora/op-geth/build/bin/geth init --datadir=/root/data/zora/zora-op-geth --state.scheme hash /root/data/zora/zora-op-geth/genesis.json
```

## Launch Zora

#### Start op-geth

{% hint style="info" %}
It's usually simpler to begin with starting`op-geth` before you start `op-node`. You can start `op-geth` even if `op-node` isn't running yet, but `op-geth` won't get any blocks until `op-node` starts.
{% endhint %}

```bash
sudo systemctl daemon-reload #refresh systemd configuration when changes made

sudo systemctl enable zora-op-geth.service #enable zora-op-geth service at system startup

sudo systemctl start zora-op-geth.service #start op-geth

sudo nano /etc/systemd/system/zora-op-geth.service #make changes in zora-op-geth.service fi
```

#### Start op-node

{% hint style="info" %}
Once you've started `op-geth`, you can start `op-node`. `op-node` will connect to `op-geth` and begin synchronizing the Zora network. `op-node` will begin sending block payloads to `op-geth` when it derives enough blocks from Ethereum
{% endhint %}

```bash
sudo systemctl daemon-reload #refresh systemd configuration when changes made

sudo systemctl enable zora-op-node.service #enable zora-op-node service at system startup

sudo systemctl start zora-op-node.service #start zora-op-node

sudo nano /etc/systemd/system/zora-op-node.service #make changes in zora-op-node.service file
```

### Monitor the logs for errors

```bash
sudo journalctl -fu zora-op-node.service #follow logs of zora-op-node.service

sudo journalctl -fu zora-op-geth.service #follow logs of zora-op-geth.service
```

You are expected to get the following log messages from `op-node`

{% code overflow="wrap" %}
```bash
t=2024-09-18T03:23:58+0200 lvl=info msg="generated attributes in payload queue" txs=1 timestamp=1686723279
t=2024-09-18T03:23:58+0200 lvl=info msg="Inserted block" hash=0xa320a436f3f0aba175d8a5a29df758e791cac7ef9b159c07d623fbc9cfdcee7e number=14720 state_root=0x868445e40d3f282c735d768e07966ce153d4539057dd0cd602f531b2b1f96af6 timestamp=1686723279 parent=0xbcf66c21b09dbf880aa803016432407b38f86bf7bbc4d10bebee4f988f6525d1 prev_randao=0x057fd8e4e0b088b42d7a8c6ef2aeaeeb59da6f56cbff2ee34b13fa1fd5aa45ed fee_recipient=0x4200000000000000000000000000000000000011 txs=1 last_in_span=true derived_from=0xda5525e81c1267b4b0adb46ad23483d6eb67cc39aeadd289a618afed237f2687:17476348
Sep 18 03:23:58 Podaga op-node[33764]: t=2024-09-18T03:23:58+0200 lvl=info msg="Found next batch" batch_type=SingularBatch batch_timestamp=1686723281 parent_hash=0xa320a436f3f0aba175d8a5a29df758e791cac7ef9b159c07d623fbc9cfdcee7e batch_epoch=0x4b12a6767d5f8362ed09c2b1e6e371b4737eeec2f66632427143cc4687b62bff:17476337 txs=0 compression_algo=zlib
```
{% endcode %}

### Run _`curl`_ command in the terminal to check the status of your node

```bash
curl -H "Content-type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' http://localhost:8545
```

If it returns `false` then your node is fully synchronized with the network

#### Sync speed depends on your L1 node, as most of the chain is derived from data submitted to the L1.&#x20;

#### You can check your syncing status using the `optimism_syncStatus` RPC on the `op-node`

```bash
command -v jq  &> /dev/null || { echo "jq is not installed" 1>&2 ; }
echo Latest synced block behind by: \
$((($( date +%s )-\
$( curl -s -d '{"id":0,"jsonrpc":"2.0","method":"optimism_syncStatus"}' -H "Content-Type: application/json" http://localhost:7545 |
   jq -r .result.unsafe_l2.timestamp))/60)) minutes
```

### References

{% embed url="https://docs.zora.co/zora-network/intro" %}

{% embed url="https://docs.conduit.xyz/guides/run-a-node/op-stack-node" %}
