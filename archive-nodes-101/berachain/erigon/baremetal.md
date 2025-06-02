# 💻 Baremetal

Authors: \[ Godwin]

## System Requirements

<table data-full-width="false"><thead><tr><th>CPU</th><th>OS</th><th>RAM</th><th>DISK</th></tr></thead><tbody><tr><td>8 vCPU</td><td>Ubuntu 22.04</td><td>48 GB</td><td>400+ GB </td></tr></tbody></table>

{% hint style="success" %}
_The Berachain node has a size of  237 GB on 19, May, 2025. Erigon: 165 and Beacond: 71GB_
{% endhint %}

## Pre-requisite

{% hint style="info" %}
At the time when this node was set up, Erigon v2.61.x and GO v1.23.x were the required version for the Berachain setup.

Check out  [https://docs.berachain.com/nodes/evm-execution](https://docs.berachain.com/nodes/evm-execution) to find the correct required version when setting up the node.
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

Depending on your execution client you intend to use, you delete the other client files. For a full list of the clients and versions go to this page - [EVM Execution Layer ⟠ | Berachain Core Docs](https://docs.berachain.com/nodes/evm-execution)

For this guide, I chose erigon client, so this guide will follow the installation of erigon as the execution layer as well as beacond for the consensus layer.

### Install Erigon

### First Install go

```bash
sudo wget https://go.dev/dl/go1.23.9.linux-amd64.tar.gz && sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.23.9.linux-amd64.tar.gz && rm go1.23.9.linux-amd64.tar.gz

export PATH=$PATH:/usr/local/go/bin
```

### Build Erigon

Berachain node requires erigon v1.26.x for mainnet. With Rust and the dependencies installed, you're ready to build Erigon. First, clone the repository:

```bash
git clone https://github.com/erigontech/erigon.git
cd erigon/
git checkout v2.61.1
make erigon 
```

Install Beacond Consensus Layer Note: Beacond requires go version 1.17.x - 1.23.x

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
export CHAIN_SPEC=mainnet   # "mainnet" or "testnet"
export MONIKER_NAME=anyname
export WALLET_ADDRESS_FEE_RECIPIENT=0x9BcaA41DC32627776b1A4D714Eef627E640b3EF5
export EL_ARCHIVE_NODE=true # set to true if you want to run an archive node on CL and EL
export MY_IP=`curl -s ipv4.canhazip.com`

########
# VALUES YOU MIGHT WANT TO CHANGE
export LOG_DIR=/root/erigon-beranode/scripts/logs
export BEACOND_BIN=/root/erigon-beranode/beacon-kit/build/bin/beacond
export BEACOND_DATA=/root/erigon-beranode/scripts/var/beacond
export BEACOND_CONFIG=$BEACOND_DATA/config  # can't change this. sorry.
export JWT_PATH=$BEACOND_CONFIG/jwt.hex

# need at least one of these
#export RETH_BIN=$(command -v reth || echo $(pwd)/reth)
#export GETH_BIN=$(command -v geth || echo $(pwd)/geth)
#export NETHERMIND_BIN=$(command -v Nethermind.Runner || echo $(pwd)/Nethermind.Runner)
export ERIGON_BIN=/root/erigon-beranode/erigon/build/bin/erigon 
```

You need to set these constants:

1. **CHAIN\_SPEC**: Set to `testnet` or `mainnet`.
2. **MONIKER\_NAME**: Should be a name of your choice for your node.
3. **WALLET\_ADDRESS\_FEE\_RECIPIENT**: This is the address that will receive the priority fees for blocks sealed by your node. If your node will not be a validator, this won't matter.
4. **EL\_ARCHIVE\_NODE**: Set to `true` if you want the execution client to be a full archive node.
5. **MY\_IP**: This is used to set the IP address your chain clients advertise to other peers on the network. If you leave it blank, `geth` and `reth` will discover the address with UPnP (if you are behind a NAT gateway) or assign the node's ethernet IP (which is OK if your computer is directly on the internet and has a public IP). In a cloud environment such as AWS or GCP where you are behind a NAT gateway, you **must** specify this address or allow the default `curl canhazip.com` to auto-detect it, if connections to that address lead back to your instance.

You should verify these constants:

* **LOG\_DIR**: This directory stores log files.
* **BEACOND\_BIN**: Set this to the full path where you installed `beacond`. The expression provided finds it in your $PATH.
* **BEACOND\_DATA**: Set this to where the consensus data and config should be kept. BEACOND\_CONFIG must be under BEACOND\_PATH as shown. Don't change it.
* **ERIGON\_BIN** or other chain client: Set this to the full path where you installed `erigon`. The expression provided finds it in your $PATH.

### Fetch Mainnet Parameters

Run

```bash
 # ./fetch-berachain-params.sh 
0038db129d91238c9bff8e495c5fa93f  /root/erigon-beranode/scripts/seed-data-80094/app.toml
8e5601f00d14d3694b4eccc8c101b15b  /root/erigon-beranode/scripts/seed-data-80094/config.toml
5aeab3cb885d8f32892685fc8e44151b  /root/erigon-beranode/scripts/seed-data-80094/el-bootnodes.txt
b00257ebcaa13f02559861696b55c5da  /root/erigon-beranode/scripts/seed-data-80094/el-peers.txt
cd3a642dc78823aea8d80d5239231557  /root/erigon-beranode/scripts/seed-data-80094/eth-genesis.json
c0b7dc21e089f9074d97957526fcd08f  /root/erigon-beranode/scripts/seed-data-80094/eth-nether-genesis.json
c66dbea5ee3889e1d0a11f856f1ab9f0  /root/erigon-beranode/scripts/seed-data-80094/genesis.json
5d0d482758117af8dfc20e1d52c31eef  /root/erigon-beranode/scripts/seed-data-80094/kzg-trusted-setup.json
```

### Set up the Consensus Client

The script `setup-beacond.sh` invokes `beacond init` and `beacond jwt generate`. This script:

1. Runs `beacond init` to create the file `var/beacond/config/priv_validator_key.json`. This contains your node's private key, and especially if you intend to become a validator, this file should be kept safe. It cannot be regenerated, and losing it means you will not be able to participate in the consensus process.
2. Runs `beacond jwt generate` to create the file `jwt.hex`. This contains a secret shared between the consensus client and execution client so they can securely communicate. Protect this file. If you suspect it has been leaked, generate a new one with `beacond jwt generate -o $JWT_PATH`.
3. Rewrites the beacond configuration files to reflect settings chosen in `env.sh`.
4. Places the mainnet parameters fetched above where Beacon-Kit expects them, and shows you an important hash from the genesis file.

```bash
# ./setup-beacond.sh 
BEACOND_DATA: /root/erigon-beranode/scripts/var/beacond
BEACOND_BIN: /root/erigon-beranode/beacon-kit/build/bin/beacond
  Version: v1.2.0.rc2-28-gcbb6cfac3
✓ Private validator key generated in /root/erigon-beranode/scripts/var/beacond/config/priv_validator_key.json
✓ JWT secret generated at /root/erigon-beranode/scripts/var/beacond/config/jwt.hex
✓ Config files in /root/erigon-beranode/scripts/var/beacond/config updated
Genesis validator root: 0xdf609e3b062842c6425ff716aec2d2092c46455d9b2e1a2c9e32c6ba63ff0bda
✓ Beacon-Kit set up. Confirm genesis root is correct.
```

### Set up the Execution Client [​](https://docs.berachain.com/nodes/quickstart#set-up-the-execution-client-%F0%9F%9B%A0%EF%B8%8F)

The provided scripts `setup-reth`, `setup-geth` and `setup-nether` create a runtime directory and configuration for those respective chain clients. The node is configured with pruning settings according to the `EL_ARCHIVE_NODE` setting in `config.sh`.

Here's an example of `setup-erigon`:

```bash
# ./setup-erigon.sh 
ERIGON_DATA: /root/erigon-beranode/scripts/var/erigon
ERIGON_BIN: /root/erigon-beranode/erigon/build/bin/erigon
  Version: erigon version 2.61.1-b3129c00
INFO[05-14|21:51:19.609] logging to file system                   log dir=/root/erigon-beranode/scripts/var/erigon/logs file prefix=erigon log level=info json=false
INFO[05-14|21:51:21.246] Starting Erigon on Ethereum mainnet... 
INFO[05-14|21:51:21.247] Maximum peer count                       ETH=100 total=100
INFO[05-14|21:51:21.247] starting HTTP APIs                       port=8545 APIs=eth,erigon,engine
INFO[05-14|21:51:21.247] Opening Database                         label=chaindata path=/root/erigon-beranode/scripts/var/erigon/chaindata
INFO[05-14|21:51:21.248] [db] open                                label=chaindata sizeLimit=12TB pageSize=8192
INFO[05-14|21:51:21.249] Re-Opening DB in exclusive mode to apply migrations 
INFO[05-14|21:51:21.250] [db] open                                label=chaindata sizeLimit=12TB pageSize=8192
INFO[05-14|21:51:21.251] Apply migration                          name=db_schema_version5
INFO[05-14|21:51:21.251] Applied migration                        name=db_schema_version5
INFO[05-14|21:51:21.251] Apply migration                          name=txs_begin_end
INFO[05-14|21:51:21.251] Applied migration                        name=txs_begin_end
INFO[05-14|21:51:21.251] Apply migration                          name=txs_v3
INFO[05-14|21:51:21.251] Applied migration                        name=txs_v3
INFO[05-14|21:51:21.251] Apply migration                          name=prohibit_new_downloads_lock
INFO[05-14|21:51:21.251] Applied migration                        name=prohibit_new_downloads_lock
INFO[05-14|21:51:21.251] Apply migration                          name=prohibit_new_downloads_lock2
INFO[05-14|21:51:21.251] Applied migration                        name=prohibit_new_downloads_lock2
INFO[05-14|21:51:21.252] Updated DB schema to                     version=6.1.0
INFO[05-14|21:51:21.253] [db] open                                label=chaindata sizeLimit=12TB pageSize=8192
INFO[05-14|21:51:27.874] Writing custom genesis block             hash=0xd57819422128da1c44339fc7956662378c17e2213e669b427ac91cd11dfcfb38
INFO[05-14|21:51:27.981] Successfully wrote genesis state         hash=0xd57819422128da1c44339fc7956662378c17e2213e669b427ac91cd11dfcfb38

✓ Erigon set up.
```

Your genesis block hash **must** agree with the above

### Run the Client

Setup Services for both beacond and erigon

Beacond service

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
WorkingDirectory=/root/erigon-beranode/scripts/
ExecStart=/root/erigon-beranode/scripts/run-beacond.sh

KillSignal=SIGTERM
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
" > /etc/systemd/system/erigon-beacond.service
```

Erigon Service

```bash
sudo echo "[Unit]
Description=erigon
After=network.target

[Service]
Restart=on-failure
RestartSec=5
TimeoutSec=900
User=root
Nice=0
LimitNOFILE=200000
WorkingDirectory=/root/erigon-beranode/scripts/
ExecStart=/root/erigon-beranode/scripts/run-erigon.sh

KillSignal=SIGTERM
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/erigon.service
```

Contents of run-erigon.sh

This file can be configured based on your needs

```bash
#!/bin/bash

set -e
. ./env.sh

BOOTNODES_OPTION=${EL_BOOTNODES:+--bootnodes $EL_BOOTNODES}
PEERS_OPTION=${EL_PEERS:+--staticpeers $EL_PEERS}
IP_OPTION=${MY_IP:+--nat extip:$MY_IP}

$ERIGON_BIN                                     \
        --datadir $ERIGON_DATA                  \
        $BOOTNODES_OPTION                       \
        $PEERS_OPTION                           \
        $IP_OPTION                              \
        --metrics                               \
        --metrics.addr 0.0.0.0                  \
        --metrics.port $EL_PROMETHEUS_PORT      \
        --http                                  \
        --http.addr 0.0.0.0                     \
        --http.port $EL_ETHRPC_PORT             \
        --http.api eth,erigon,web3,net,debug,trace \
        --http.vhosts "*"                       \
        --http.corsdomain "*"                   \
        --ws                                    \
        --ws.port 8549                          \
        --port $EL_ETH_PORT                     \
        --p2p.allowed-ports 30305,30306 \
        --authrpc.addr 127.0.0.1                \
        --authrpc.port $EL_AUTHRPC_PORT         \
        --authrpc.jwtsecret $JWT_PATH           \
        --authrpc.vhosts localhost
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
sudo systemctl enable erigon-beacond.service erigon.service
sudo systemctl start erigon-beacond.service erigon.service
```

### Confirm Node Syncing

```bash
journalctl -xeu erigon-beacond.service
journalctl -xeu erigon.service
```

Beacon

```bash
May 09 11:30:45 run-beacond.sh[1047966]: 2025-05-09T11:30:45+02:00 INFO Processed withdrawals service=state-processor num_withdrawals=1 evm_inflation=0
May 09 11:30:45 run-beacond.sh[1047966]: 2025-05-09T11:30:45+02:00 INFO Forkchoice updated service=execution-engine head_block_hash=0xb06fb69887d4966c9122a1da3b3211a6ec516f00e11127b3ddf22c2b9c311a8e safe_block_hash=0x8a12334f9fabd>May 09 11:30:45 run-beacond.sh[1047966]: 2025-05-09T11:30:45+02:00 INFO Finalized block module=state height=6440 num_txs_res=2 num_val_updates=0 block_app_hash=D353B3D4ED32B3BC614B4BC641690AF3AD78E6C9B9826D0F5F73EF97D8F4EDB0 synci>May 09 11:30:45 run-beacond.sh[1047966]: 2025-05-09T11:30:45+02:00 INFO Committed state module=state height=6440 block_app_hash=B368447416331674634AC8BE6D870565C6D3BC4152361EA94B322471E37F9224
```

Erigon

```bash
May 19 15:34:57 Sufax run-erigon.sh[1238087]: [INFO] [05-19|15:34:57.146] [NewPayload] Handling new payload        height=3792435 hash=0x>May 19 15:34:57 Sufax run-erigon.sh[1238087]: [INFO] [05-19|15:34:57.152] [updateForkchoice] Fork choice update: flushing in-memory state>May 19 15:34:57 Sufax run-erigon.sh[1238087]: [INFO] [05-19|15:34:57.153] RPC Daemon notified of new headers       from=3792434 to=379243>May 19 15:34:57 Sufax run-erigon.sh[1238087]: [INFO] [05-19|15:34:57.153] head updated           
```

#### Testing Local RPC Node

**Get the current execution block number**

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
