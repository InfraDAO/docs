# 💻 Bare Metal

Authors: \[man4ela | catapulta.eth]

## System Requirements

<table><thead><tr><th width="157" align="center">CPU</th><th align="center">OS</th><th width="166" align="center">RAM</th><th align="center">DISK</th></tr></thead><tbody><tr><td align="center">8+ cores CPU</td><td align="center">Debian 12/Ubuntu 22.04</td><td align="center">=> 16 GB RAM</td><td align="center">1,5TB+ (SSD or NVMe)</td></tr></tbody></table>

{% hint style="info" %}
_Unichain Mainnet archive node has a size of 1,3TB on August 20th, 2025_
{% endhint %}

## Unichain

{% hint style="success" %}
Unichain operates within Optimism _Superchain_ ecosystem. It is powered by the [OP Stack](https://stack.optimism.io/), in collaboration with Optimism, leveraging the scalability and security of Optimism's Layer 2 infrastructure.

In this guide, we are walking through the process of setting up a Unichain Mainnet archive node using Optimism's `op-geth and op-node`.
{% endhint %}

{% hint style="warning" %}
Before you start, make sure that you have your own synced Ethereum L1 RPC URL (e.g. Erigon) and L1 Consensus Layer Beacon endpoint with **`all historical blobs data`** (e.g. Lighthouse) ready. <mark style="color:red;">A beacon endpoint meeting this criteria is essential for syncing to start.</mark>
{% endhint %}

## Pre-Requisites

{% code overflow="wrap" %}
```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y

sudo apt install -y git make wget aria2 gcc pkg-config libusb-1.0-0-dev libudev-dev jq gcc g++ curl libssl-dev screen apache2-utils build-essential pkg-configv
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

Allow remote RPC connections with Mode Node

```bash
sudo ufw allow from ${REMOTE.HOST.IP} to any port 3545
```

Allow remote P2P connections with Mode Node

```bash
sudo ufw allow 9222
sudo ufw allow 30303
```

{% hint style="warning" %}
Not advised to allow all or unknown IP address to RPC port
{% endhint %}

Enable Firewall

<pre class="language-bash"><code class="lang-bash"><strong>sudo ufw enable
</strong></code></pre>

To check the status of UFW and see the current rules

```bash
sudo ufw status verbose
```

## Install dependencies

#### Required Software Dependencies

<table><thead><tr><th width="115">Dependency</th><th width="110" align="center">Version</th><th width="233">Version Check Command</th></tr></thead><tbody><tr><td><mark style="color:green;">go</mark></td><td align="center"><code>^1.23</code></td><td><code>go version</code></td></tr><tr><td><mark style="color:orange;">docker</mark></td><td align="center"><code>^28</code></td><td><code>docker version</code></td></tr></tbody></table>

### Install GO

<pre class="language-bash" data-overflow="wrap" data-full-width="false"><code class="lang-bash">sudo rm -rf /usr/local/go

wget https://go.dev/dl/go1.23.9.linux-amd64.tar.gz

sudo tar -xzf go1.23.9.linux-amd64.tar.gz -C /usr/local/

rm -f go1.23.9.linux-amd64.tar.gz

<strong>#Setup Go language export paths:
</strong>
sudo tee /etc/profile.d/golang.sh > /dev/null &#x3C;&#x3C;EOT
export GOROOT=/usr/local/go
export GOPATH=\$HOME/go
export PATH=\$PATH:\$GOROOT/bin:\$GOPATH/bin

EOT

#Load Go environment variables into the current shell session:

source /etc/profile.d/golang.sh
</code></pre>

### Install Docker

```bash
sudo apt-get update

sudo apt-get install ca-certificates curl

sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Check if Go and all dependencies are installed

```bash
go version
docker version
```

## Build the Rollup Node (op-node)

#### Create working directory

```bash
mkdir unichain && cd unichain
```

#### Install Op-node

```bash
docker pull us-docker.pkg.dev/oplabs-tools-artifacts/images/op-node:v1.13.5

CONTAINER_ID=$(docker create us-docker.pkg.dev/oplabs-tools-artifacts/images/op-node:v1.13.5)

docker cp $CONTAINER_ID:/usr/local/bin/op-node ./op-node

docker rm $CONTAINER_ID

chmod +x ./op-node
```

## Build the Execution Engine (op-geth)

#### Build op-geth

```bash
git clone https://github.com/ethereum-optimism/op-geth.git

cd op-geth

git checkout v1.101511.1

make geth
```

#### Download genesis.json and create jwt.hex into database directory:

```bash
mkdir -p  /root/data/unichain/unichain-op-geth/ && cd /root/data/unichain/unichain-op-geth/

openssl rand -hex 32 | tr -d "\n" > /root/data/unichain/unichain-op-geth/jwt.hex

wget https://raw.githubusercontent.com/Uniswap/unichain-node/refs/heads/main/chaincon
```

### Create systemd service for op-node

```bash
sudo nano /etc/systemd/system/unichain-op-node.service
```

{% hint style="warning" %}
#### Paste the following configs replacing `{L1 RPC},{L1 BEACON RPC},{SERVER IP}` with own values
{% endhint %}

Save by entering `ctrl+X` and `Y+ENTER`

```bash
[Unit]
Description=Unichain OP Node Service
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
WorkingDirectory=/root/unichain/
Environment=OP_NODE_L2_ENGINE_AUTH=/root/data/unichain/unichain-op-geth/jwt.hex
Environment=OP_NODE_L2_ENGINE_RPC=http://0.0.0.0:8551
Environment=OP_NODE_NETWORK=unichain-mainnet
Environment=OP_NODE_LOG_LEVEL=info
Environment=OP_NODE_LOG_FORMAT=logfmt
Environment=OP_NODE_METRICS_ADDR=0.0.0.0
Environment=OP_NODE_METRICS_ENABLED=true
Environment=OP_NODE_METRICS_PORT=7007
Environment=OP_NODE_P2P_LISTEN_IP=0.0.0.0
Environment=OP_NODE_P2P_LISTEN_TCP_PORT=9222
Environment=OP_NODE_P2P_LISTEN_UDP_PORT=9222
Environment=OP_NODE_RPC_ADDR=0.0.0.0
Environment=OP_NODE_RPC_PORT=6545
Environment=OP_NODE_P2P_ADVERTISE_IP={SERVER IP}
Environment=OP_NODE_SNAPSHOT_LOG=/tmp/op-node-snapshot-log
Environment=OP_NODE_VERIFIER_L1_CONFS=4

ExecStart=/root/unichain/op-node \
                --network=unichain-mainnet \
                --l1={L1 RPC} \
                --l1.beacon={L1 BEACON RPC} \
                --l2=http://0.0.0.0:8551 \
                --l1.trustrpc=true \
                --syncmode=execution-layer
KillSignal=SIGTERM
[Install]
WantedBy=multi-user.target
```

### Create systemd service for op-geth

```bash
sudo nano /etc/systemd/system/unichain-op-geth.service
```

#### Paste the following configs:

```bash
[Unit]
Description=Unichain OP GETH Service
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

WorkingDirectory=/root/unichain/op-geth/
Environment=GETH_OP_NETWORK=unichain-mainnet
Environment=GETH_ROLLUP_SEQUENCERHTTP=https://mainnet-sequencer.unichain.org
Environment=GETH_LOG_FORMAT=logfmt
Environment=GETH_ROLLUP_DISABLETXPOOLGOSSIP=true
Environment=GETH_TXPOOL_NOLOCALS=true
Environment=GETH_GCMODE=archive
Environment=OP_NODE_L2_ENGINE_AUTH=/root/data/unichain/unichain-op-geth/jwt.hex

ExecStart=/root/unichain/op-geth/build/bin/geth \
        --datadir=/root/data/unichain/unichain-op-geth/ \
        --verbosity=3 \
        --op-network=unichain-mainnet \
        --state.scheme=hash \
        --http \
        --http.corsdomain="*" \
        --http.vhosts="*" \
        --http.addr=0.0.0.0 \
        --http.port=3545 \
        --http.api=eth,net,web3,debug,txpool,engine,admin,rpc \
        --authrpc.addr=0.0.0.0 \
        --authrpc.port=8551 \
        --authrpc.vhosts="*" \
        --authrpc.jwtsecret=/root/data/unichain/unichain-op-geth/jwt.hex \
        --ws \
        --ws.addr=0.0.0.0 \
        --ws.port=3546 \
        --ws.origins="*" \
        --ws.api=eth,net,web3,debug,txpool,engine,admin,rpc \
        --metrics \
        --metrics.addr=0.0.0.0 \
        --metrics.port=7900 \
        --syncmode=full \
        --gcmode=archive \
        --maxpeers=100 \
        --port=33303
KillSignal=SIGTERM

[Install]
WantedBy=multi-user.target
```

#### Initialize the node:

```bash
/root/unichain/op-geth/build/bin/geth init --datadir=/root/data/unichain/unichain-op-geth --state.scheme hash /root/data/unichain/unichain-op-geth/genesis-l2.json
```

## Launch Unichain

#### Start op-geth

{% hint style="info" %}
It's usually simpler to begin with starting`op-geth` before you start `op-node`. You can start `op-geth` even if `op-node` isn't running yet, but `op-geth` won't get any blocks until `op-node` starts.
{% endhint %}

```bash
sudo systemctl daemon-reload #refresh systemd configuration when changes made

sudo systemctl enable unichain-op-geth.service #enable unichain-op-geth service at system startup

sudo systemctl start unichain-op-geth.service #start unichain-op-geth

sudo nano /etc/systemd/system/unichain-op-geth.service #make changes in unichain-op-geth.service file
```

#### Start op-node

{% hint style="info" %}
Once you've started `op-geth`, you can start `op-node`. `op-node` will connect to `op-geth` and begin synchronizing the Mode network. `op-node` will begin sending block payloads to `op-geth` when it derives enough blocks from Ethereum
{% endhint %}

```bash
sudo systemctl daemon-reload #refresh systemd configuration when changes made

sudo systemctl enable unichain-op-node.service #enable unichain-op-node service at system startup

sudo systemctl start unichain-op-node.service #start unichain-op-node

sudo nano /etc/systemd/system/unichain-op-node.service #make changes in unichain-op-node.service file
```

### Monitor the logs for errors

```bash
sudo journalctl -fu unichain-op-node.service #follow logs of unichain-op-node.service

sudo journalctl -fu unichain-op-geth.service #follow logs of unichain-op-geth.service
```

You are expected to get following log messages from `op-node`

{% code overflow="wrap" %}
```log
Aug 15 13:05:58 Dawon op-node[3778198]: t=2025-08-15T13:05:58+0200 lvl=info msg="Optimistically queueing unsafe L2 execution payload" id=0x23309a24f044324a8af64eaaa823c04c9c8bc3e17826073adc62639aa7f03e8c:24507584
Aug 15 13:05:58 Dawon op-node[3778198]: t=2025-08-15T13:05:58+0200 lvl=info msg="Inserted new L2 unsafe block (synchronous)" hash=0x23309a24f044324a8af64eaaa823c04c9c8bc3e17826073adc62639aa7f03e8c number=24507584 newpayload_time=12.952ms fcu2_time=1.140ms total_time=14.093ms mgas=6.736009 mgasps=477.94030990442695
Aug 15 13:05:58 Dawon op-node[3778198]: t=2025-08-15T13:05:58+0200 lvl=info msg="Sync progress" reason="new chain head block" l2_finalized=0x7595a2fe9f3f2bb1ed9dc51ac704acff353f190b38ebf9ec3c5a1a3d037c6ba7:24465143 l2_safe=0x7595a2fe9f3f2bb1ed9dc51ac704acff353f190b38ebf9ec3c5a1a3d037c6ba7:24465143 l2_pending_safe=0x7595a2fe9f3f2bb1ed9dc51ac704acff353f190b38ebf9ec3c5a1a3d037c6ba7:24465143 l2_unsafe=0x23309a24f044324a8af64eaaa823c04c9c8bc3e17826073adc62639aa7f03e8c:24507584 l2_backup_unsafe=0x0000000000000000000000000000000000000000000000000000000000000000:0 l2_time=1755255943
Aug 15 13:05:58 Dawon op-node[3778198]: t=2025-08-15T13:05:58+0200 lvl=info msg="successfully processed payload" ref=0x23309a24f044324a8af64eaaa823c04c9c8bc3e17826073adc62639aa7f03e8c:24507584 txs=12
```
{% endcode %}

The expected log messages from `op-geth:`

```log
Aug 15 14:49:52 Dawon geth[3778023]: t=2025-08-15T14:49:52+0200 lvl=info msg="Imported new potential chain segment" number=24513817 hash=0x09a4bb509b2dec55d9d836f898c97e0231df024af5ebaf261ef6a309ced179d4 blocks=1 txs=9 mgas=1.306784 elapsed=9.150ms mgasps=142.81143064971036 snapdiffs="1.62 MiB" triedirty="0.00 B"
Aug 15 14:49:52 Dawon geth[3778023]: t=2025-08-15T14:49:52+0200 lvl=info msg="Chain head was updated" number=24513817 hash=0x09a4bb509b2dec55d9d836f898c97e0231df024af5ebaf261ef6a309ced179d4 root=0x1189a2de7dc35f1e11d7e9e95b5fd202d36406bd49fa2e685b965ec0e1aec12c elapsed=331.435µs
Aug 15 14:49:52 Dawon geth[3778023]: t=2025-08-15T14:49:52+0200 lvl=info msg="Imported new potential chain segment" number=24513818 hash=0x236968614141c40e8983b3513fdf00c3cc1c948fa98d778ea01e15836a47a7ea blocks=1 txs=9 mgas=1.1686 elapsed=6.441ms mgasps=181.41939954929248 snapdiffs="1.62 MiB" triedirty="0.00 B"
```

### Run _`curl`_ command in the terminal to check the status of your node

```bash
curl -H "Content-type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' http://localhost:3545
```

If it returns `false` then your node is fully synchronized with the network

#### Sync speed depends on your L1 node, as the majority of the chain is derived from data submitted to the L1.&#x20;

#### You can check your syncing status using the `optimism_syncStatus` RPC on the `op-node`

```bash
command -v jq  &> /dev/null || { echo "jq is not installed" 1>&2 ; }
echo Latest synced block behind by: \
$((($( date +%s )-\
$( curl -s -d '{"id":0,"jsonrpc":"2.0","method":"optimism_syncStatus"}' -H "Content-Type: application/json" http://localhost:6545 |
   jq -r .result.unsafe_l2.timestamp))/60)) minutes
```

## <sup>References</sup> <a href="#references" id="references"></a>

{% embed url="https://docs.unichain.org/docs/getting-started/set-up-a-node" %}

{% embed url="https://github.com/Uniswap/unichain-node" %}

{% embed url="https://docs.optimism.io/operators/node-operators/tutorials/run-node-from-source" %}
