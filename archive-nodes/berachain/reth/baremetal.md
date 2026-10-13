# 💻 Baremetal

Authors: \[ Godwin]

## System Requirements

<table data-full-width="false"><thead><tr><th>CPU</th><th>OS</th><th>RAM</th><th>DISK</th></tr></thead><tbody><tr><td>8 vCPU</td><td>Ubuntu 22.04</td><td>48 GB</td><td>400+ GB </td></tr></tbody></table>

{% hint style="success" %}
_The Berachain node has a size of  237 GB on 19, May, 2025. Reth: 165 and Beacond: 71GB_
{% endhint %}

## Pre-requisite

{% hint style="info" %}
At the time when this node was set up, Reth v1.3.12 and GO v1.23 were the required version for the Berachain setup.

Check out [https://docs.berachain.com/nodes/evm-execution](https://docs.berachain.com/nodes/evm-execution) to find the version required when setting up the node.
{% endhint %}

### **Commands**

{% code overflow="wrap" %}
```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
```
{% endcode %}

## Firewall Settings

### Check status & enable UFW&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>sudo ufw enable
</strong>sudo ufw status verbose
</code></pre>

### Set explicit default UFW rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### Allow SSH, HTTP, and HTTPS

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80
sudo ufw allow 443
sudo ufw allow 30303
```

### Allow Remote connection

```bash
sudo ufw allow from ${REMOTE.HOST.IP} to any port 9944 
```

## Setup Instructions&#x20;

```bash
mkdir beranode-setup
cd beranode-setup
git clone https://github.com/berachain/guides
mv guides/apps/node-scripts/* ./
rm -r guide
```

Depending on the execution client you intend to use, delete the other client files.&#x20;

For a full list of the clients and versions, go to this page - [EVM Execution Layer ⟠ | Berachain Core Docs](https://docs.berachain.com/nodes/evm-execution)

For this guide, I chose Reth client, so this guide will follow the installation of Reth as the execution layer, as well as Beacond for the consensus layer.

### Install Reth

First, **install Rust** using [rustup](https://rustup.rs/)：&#x20;

`curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`

The rustup installer provides an easy way to update the Rust compiler and works on all platforms.

Install the following dependencies if running on Ubuntu:&#x20;

`apt-get install libclang-dev pkg-config build-essential`

### Build Reth

Berachain node requires reth v1.1.x for mainnet. With Rust and the dependencies installed, you're ready to build Reth. First, clone the repository:

```bash
git clone https://github.com/paradigmxyz/reth
cd reth
git checkout v1.1.5
```

Then, install Reth into your PATH directly via:

`cargo install --locked --path bin/reth --bin reth` The binary will now be accessible as reth via the command line, and exist under your default `.cargo/bin` folder.

Beacond Consensus Layer Note: Beacond requires go version 1.17.x - 1.23.x

### Install go

```bash
sudo wget https://go.dev/dl/go1.23.9.linux-amd64.tar.gz && sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.23.9.linux-amd64.tar.gz && rm go1.23.9.linux-amd64.tar.gz

export PATH=$PATH:/usr/local/go/bin
```

### Install Beacond

```bash
git clone https://github.com/berachain/beacon-kit.git
make install

make build
```

Edit Env.sh The file `env.sh` contains environment variables used in the other scripts. `fetch-berachain-params.sh` obtains copies of the genesis file and other configuration files. Then we have `setup-` and `run-` scripts for various execution clients and `beacond.`

```bash
#!/bin/bash

# CHANGE THESE VALUES
export CHAIN_SPEC=mainnet   # or "testnet"
export MONIKER_NAME=anyname
export WALLET_ADDRESS_FEE_RECIPIENT=0x9BcaA41DC32627776b1A4D714Eef627E640b3EF5
export EL_ARCHIVE_NODE=false # set to true if you want to run an archive node on CL and EL
export MY_IP=`curl -s canhazip.com`

# VALUES YOU MIGHT WANT TO CHANGE
export LOG_DIR=$(pwd)/logs
export BEACOND_BIN=/root/beacon-kit/build/bin/beacond
export BEACOND_DATA=$(pwd)/data/beacond
export BEACOND_CONFIG=$BEACOND_DATA/config  # can't change this. sorry.
export JWT_PATH=$BEACOND_CONFIG/jwt.hex

export RETH_BIN=/root/.cargo/bin/reth
#export GETH_BIN=$(command -v geth || echo $(pwd)/geth)
#export NETHERMIND_BIN=$(command -v Nethermind.Runner || echo #$(pwd)/Nethermind.Runner)
#export ERIGON_BIN=$(command -v erigon || echo $(pwd)/erigon)
```

You need to set these constants:

1. **CHAIN\_SPEC**: Set to `testnet` or `mainnet`.
2. **MONIKER\_NAME**: Should be a name of your choice for your node.
3. **WALLET\_ADDRESS\_FEE\_RECIPIENT**: This is the address that will receive the priority fees for blocks sealed by your node. If your node will not be a validator, this won't matter.
4. **EL\_ARCHIVE\_NODE**: Set to `true` if you want the execution client to be a full archive node.
5. **MY\_IP**: This is used to set the IP address that your chain clients advertise to other peers on the network. If you leave it blank, `geth` and `reth` will discover the address with UPnP (if you are behind a NAT gateway) or assign the node's ethernet IP (which is OK if your computer is directly on the internet and has a public IP). In a cloud environment such as AWS or GCP where you are behind a NAT gateway, you **must** specify this address or allow the default `curl canhazip.com` to auto-detect it, if connections to that address lead back to your instance.

You should verify these constants:

* **LOG\_DIR**: This directory stores log files.
* **BEACOND\_BIN**: Set this to the full path where you installed `beacond`. The expression provided finds it in your $PATH.
* **BEACOND\_DATA**: Set this to where the consensus data and config should be kept. BEACOND\_CONFIG must be under BEACOND\_PATH as shown. Don't change it.
* **RETH\_BIN** or other chain client: Set this to the full path where you installed `reth`. The expression provided finds it in your $PATH.

### Fetch Mainnet Parameters

Run

```bash
# FROM: ~/beranode-setup

./fetch-berachain-params.sh;

# [Expected Output for mainnet]: # cd3a642dc78823aea8d80d5239231557 seed-data-80094/eth-genesis.json # c0b7dc21e089f9074d97957526fcd08f seed-data-80094/eth-nether-genesis.json # c66dbea5ee3889e1d0a11f856f1ab9f0 seed-data-80094/genesis.json # 5d0d482758117af8dfc20e1d52c31eef seed-data-80094/kzg-trusted-setup.json
```

### Set up the Consensus Client

The script `setup-beacond.sh` invokes `beacond init` and `beacond jwt generate`.&#x20;

This script:

1. Runs `beacond init` to create the file `var/beacond/config/priv_validator_key.json`. This contains your node's private key, and especially if you intend to become a validator, this file should be kept safe. It cannot be regenerated, and losing it means you will not be able to participate in the consensus process.
2. Runs `beacond jwt generate` to create the file `jwt.hex`. This contains a secret shared between the consensus client and execution client so they can securely communicate. Protect this file. If you suspect it has been leaked, generate a new one with `beacond jwt generate -o $JWT_PATH`.
3. Rewrites the beacond configuration files to reflect settings chosen in `env.sh`.
4. Places the mainnet parameters fetched above where Beacon-Kit expects them, and shows you an important hash from the genesis file.

```bash
# FROM: ~/beranode-setup

./setup-beacond.sh;

# expected output:
BEACOND_DATA: /root/beranode-setup/data/beacond
BEACOND_BIN: /root/beacon-kit/build/bin/beacond
  Version: v1.2.0.rc2-19-g0da575879
✓ Private validator key generated in /root/beranode-setup/data/beacond/config/priv_validator_key.json
✓ JWT secret generated at /root/beranode-setup/data/beacond/config/jwt.hex
✓ Config files in /root/beranode-setup/data/beacond/config updated
0xdf609e3b062842c6425ff716aec2d2092c46455d9b2e1a2c9e32c6ba63ff0bda
✓ Beacon-Kit set up. Confirm genesis root is correct.
```

### Set up the Execution Client [​](https://docs.berachain.com/nodes/quickstart#set-up-the-execution-client-%F0%9F%9B%A0%EF%B8%8F)

The provided scripts `setup-reth`, `setup-geth` and `setup-nether` create a runtime directory and configuration for those respective chain clients. The node is configured with pruning settings according to the `EL_ARCHIVE_NODE` setting in `config.sh`.

Here's an example of `setup-reth`:

```bash
# FROM: ~/beranode-setup

./setup-reth.sh;

# [Expected Output]:
RETH_DATA: /root/beranode-setup/var/reth/data
RETH_BIN: /root/.cargo/bin/reth
  Version: reth Version: 1.1.5
2025-05-09T08:35:56.850901Z  INFO Initialized tracing, debug log directory: /root/.cache/reth/logs/80094
2025-05-09T08:35:56.853494Z  INFO reth init starting
2025-05-09T08:35:56.853891Z  INFO Opening storage db_path="/root/beranode-setup/var/reth/data/db" sf_path="/root/beranode-setup/var/reth/data/static_files"
2025-05-09T08:35:56.861490Z  INFO Verifying storage consistency.
2025-05-09T08:35:59.340630Z  INFO Genesis block written hash=0xd57819422128da1c44339fc7956662378c17e2213e669b427ac91cd11dfcfb38

✓ Reth set up.
```

Your genesis block hash **must** agree with the above

### Run the Client

Setup Services for both beacond and reth

**Beacond service**

```bash
sudo echo "[Unit]
Description=beacond-node
After=network.target

[Service]
Restart=on-failure
RestartSec=5
TimeoutSec=900
User=root
Nice=0
LimitNOFILE=200000
WorkingDirectory=/root/beranode-setup
ExecStart=/root/beranode-setup/run-beacond.sh

KillSignal=SIGTERM
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
" > /etc/systemd/system/beacond.service
```

**Reth Service**

```bash
sudo echo "[Unit]
Description=reth
After=network.target

[Service]
Restart=on-failure
RestartSec=5
TimeoutSec=900
User=root
Nice=0
LimitNOFILE=200000
WorkingDirectory=/root/beranode-setup
ExecStart=/root/beranode-setup/run-reth.sh

KillSignal=SIGTERM
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/reth.service
```

Contents of run-reth.sh

This file can be configured based on your needs

```bash
#!/bin/bash

set -e
. ./env.sh

PEERS_OPTION=${EL_PEERS:+--trusted-peers $EL_PEERS}
BOOTNODES_OPTION=${EL_BOOTNODES:+--bootnodes $EL_BOOTNODES}
ARCHIVE_OPTION=$([ "$EL_ARCHIVE_NODE" = true ] && echo "" || echo "--full")
IP_OPTION=${MY_IP:+--nat extip:$MY_IP}

$RETH_BIN node                                  \
        --datadir $RETH_DATA                    \
        --chain $RETH_GENESIS_PATH              \
        $ARCHIVE_OPTION                         \
        $BOOTNODES_OPTION                       \
        $PEERS_OPTION                           \
        $IP_OPTION                              \
        --authrpc.addr 127.0.0.1                \
        --authrpc.port $EL_AUTHRPC_PORT         \
        --authrpc.jwtsecret $JWT_PATH           \
        --port $EL_ETH_PORT                     \
        --metrics $EL_PROMETHEUS_PORT           \
        --http                                  \
        --http.addr 0.0.0.0                     \
        --http.port $EL_ETHRPC_PORT             \
        --http.api web3,debug,eth,txpool,trace,net,reth,rpc  \
        --ws                                    \
        --ws.addr 0.0.0.0                       \
        --ws.port 8546                          \
        --ws.api web3,debug,eth,txpool,trace,net,reth,rpc  \
        --ipcpath /tmp/reth.ipc.$EL_ETHRPC_PORT \
        --discovery.port $EL_ETH_PORT           \
        --http.corsdomain '*'                   \
        --log.file.directory $LOG_DIR           \
        --engine.persistence-threshold 0        \
        --engine.memory-block-buffer-target 0 
```

Contents of run-beacon.sh

```bash
!/bin/bash

set -e
. ./env.sh

$BEACOND_BIN start --home $BEACOND_DATA
```

#### Start the service

```bash
sudo systemctl daemon-reload
sudo systemctl enable beacond.service reth.service
sudo systemctl start beacond.service reth.service
```

### Confirm Node Syncing

```bash
journalctl -xeu beacond.service
journalctl -xeu reth.service
```

beacon

```bash
May 09 11:30:45 run-beacond.sh[1047966]: 2025-05-09T11:30:45+02:00 INFO Processed withdrawals service=state-processor num_withdrawals=1 evm_inflation=0
May 09 11:30:45 run-beacond.sh[1047966]: 2025-05-09T11:30:45+02:00 INFO Forkchoice updated service=execution-engine head_block_hash=0xb06fb69887d4966c9122a1da3b3211a6ec516f00e11127b3ddf22c2b9c311a8e safe_block_hash=0x8a12334f9fabd>May 09 11:30:45 run-beacond.sh[1047966]: 2025-05-09T11:30:45+02:00 INFO Finalized block module=state height=6440 num_txs_res=2 num_val_updates=0 block_app_hash=D353B3D4ED32B3BC614B4BC641690AF3AD78E6C9B9826D0F5F73EF97D8F4EDB0 synci>May 09 11:30:45 run-beacond.sh[1047966]: 2025-05-09T11:30:45+02:00 INFO Committed state module=state height=6440 block_app_hash=B368447416331674634AC8BE6D870565C6D3BC4152361EA94B322471E37F9224
```

reth

```bash
May 09 11:31:08 run-reth.sh[1049997]: 2025-05-09T09:31:08.198893Z  INFO Forkchoice updated head_block_hash=0x17cafd43d064ed2e68653dd5099712e836dc815f09e68c768c1a4fe91782a978 safe_block_hash=0x7d5c2d440f097fd2b4b34eece9431d486b69d4>May 09 11:31:08 run-reth.sh[1049997]: 2025-05-09T09:31:08.209230Z  INFO Block added to canonical chain number=7349 hash=0x0133d491f063346adb431d321a52371d3ae9335c6992bd7dcfc6c07da638f7d6 peers=24 txs=0 gas=0.00 Kgas gas_throughput>May 09 11:31:08 run-reth.sh[1049997]: 2025-05-09T09:31:08.210002Z  INFO Canonical chain committed number=7349 hash=0x0133d491f063346adb431d321a52371d3ae9335c6992bd7dcfc6c07da638f7d6 elapsed=4.428µs
May 09 11:31:08 Sufax run-reth.sh[1049997]: 2025-05-09T09:31:08.210014Z  INFO Forkchoice updated head_block_hash=0x0133d491f063346adb431d321a52371d3ae9335c6992bd7dcfc6c07da638f7d6 safe_block_hash=0x17cafd43d064ed2e68653dd5099712e836dc81>May 09 11:31:08 run-reth.sh[1049997]: 2025-05-09T09:31:08.222898Z  INFO Block added to canonical chain number=7350 hash=0xf11199dc86a79d5e09cc2388ebe4ce1a96038a983cf23ff5ccc49e170a11dc4b peers=24 txs=0 gas=0.00 Kgas gas_throughput>May 09 11:31:08 run-reth.sh[1049997]: 2025-05-09T09:31:08.223694Z  INFO Canonical chain committed number=7350 hash=0xf11199dc86a79d5e09cc2388ebe4ce1a96038a983cf23ff5ccc49e170a11dc4b elapsed=6.382µs
```

#### Testing Local RPC Node

**Get current execution block number**

```bash
curl --location 'http://localhost:8545' \ --header 'Content-Type: application/json' \ --data '{ "jsonrpc":"2.0", "method":"eth_blockNumber", "params":[], "id":420 }';
```

Result

```bash
{"jsonrpc":"2.0","id":420,"result":"0x1860b"}
```

**Get Current Consensus Block Number**

```bash
curl -s http://localhost:26657/status | jq '.result.sync_info.latest_block_height'
```

Result

```bash
"99730"
```

{% embed url="https://docs.berachain.com/nodes/quickstart" %}
