# 💻 Baremetal

Authors: \[ Ankur | Dapplooker]

## System Requirements

| CPU    | OS           | RAM   | DISK        |
| ------ | ------------ | ----- | ----------- |
| 8 vCPU | Ubuntu 22.04 | 16 GB | 1 TB (SSD)  |

{% hint style="success" %}
_The node has a size of 585 GB as of March 14th, 2025._
{% endhint %}

## Pre-requisite <a href="#pre-requisite" id="pre-requisite"></a>

Before starting, clean the setup then update and upgrade. Install following:

* Git
* Go v1.23+
* rustc
* make
* just
* op-node&#x20;
* op-reth
* jq&#x20;
* Ethereum Sepolia L1 RPC URL
* L1 Consensus Layer Beacon URL

{% hint style="warning" %}
### Before you start, make sure that you have your own synced `Ethereum Sepolia L1 RPC URL` (Ethereum not base ) & `L1 Consensus Layer Beacon endpoint` (e.g. Lighthouse Sepolia) ready. <a href="#before-you-start-make-sure-that-you-have-your-own-synced-ethereum-sepolia-l1-rpc-url-and-l1-consensu" id="before-you-start-make-sure-that-you-have-your-own-synced-ethereum-sepolia-l1-rpc-url-and-l1-consensu"></a>
{% endhint %}

### Installation Command

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt install -y git make just wget gcc pkg-config libusb-1.0-0-dev libudev-dev jq gcc g++ curl libssl-dev screen apache2-utils build-essential pkg-config
```

### Install Rustc

```bash
curl --proto '=https' --tlsv1.3 https://sh.rustup.rs -sSf | sh # Follow the prompt instruction 
rustc --version #Verifying the installation
```

### Install Op-node

```bash
git clone https://github.com/ethereum-optimism/optimism.git
cd optimism/op-node
just VERSION=v1.12.1 op-node # build op-node
cp ./bin/op-node /usr/bin #Placing op-node binary in /usr/bin
op-node -v #Verify version 
```

{% hint style="warning" %}
Version (v1.12.1) may change in future look for op-node latest version [https://github.com/ethereum-optimism/optimism/releases?q=op-node\&expanded=true](https://github.com/ethereum-optimism/optimism/releases?q=op-node\&expanded=true)&#x20;
{% endhint %}

### Install Op-reth

```bash
git clone https://github.com/paradigmxyz/reth.git
cd reth
make build-op # build op-reth must install/upgrade dependencies
cp ./target/release/op-reth /usr/bin #Placing op-reth binary in /usr/bin
op-reth -V # Verify Version 
```

## Setting up Firewall <a href="#setting-up-firewall" id="setting-up-firewall"></a>

### Set explicit default UFW rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### Allow SSH

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80
sudo ufw allow 443
sudo ufw allow 8545
sudo ufw allow 5052
```

### Allow Remote connection

```bash
sudo ufw allow from ${REMOTE.HOST.IP} to any port 8545
```

### Enable Firewall

```bash
sudo ufw enable
```

## Setup Instructions <a href="#setup-instructions" id="setup-instructions"></a>

{% stepper %}
{% step %}
## &#x20;Cloning Base node repository <a href="#id-71fb" id="id-71fb"></a>

```bash
git clone https://github.com/base/node.git
mv node /mnt/base-sepolia
cd base-sepolia
```
{% endstep %}

{% step %}
### Creating op-node service file

```bash
echo "[Unit]
Description=Optimistic Node Client
After=network.target

[Service]
User=root
Environment=HOME="/mnt/base-sepolia"
Environment=OP_GETH_GENESIS_FILE_PATH="/mnt/base-sepolia/node/sepolia/genesis-l2.json"
Environment=OP_GETH_SEQUENCER_HTTP="https://sepolia-sequencer.base.org"
Environment=OP_NODE_L1_ETH_RPC="http://L1_ETH_ENDPOINT_URL"
Environment=OP_NODE_L1_BEACON="http://L1_BEACON_URL"
Environment=OP_NODE_P2P_ADVERTISE_IP=YOUR_PUBLIC_IP_ADDRESS
Environment=OP_NODE_BETA_EXTRA_NETWORKS="true"
Environment=OP_NODE_L2_ENGINE_AUTH="/mnt/base-sepolia/data/engine-auth-jwt"
Environment=OP_NODE_L2_ENGINE_AUTH_RAW="688f5d737bad920bdfb2fc2f488d6b6209eebda1dae949a8de91398d932c517a"
Environment=OP_NODE_L2_ENGINE_RPC="http://localhost:8551"
Environment=OP_NODE_LOG_LEVEL="info"
Environment=OP_NODE_METRICS_ADDR=0.0.0.0
Environment=OP_NODE_METRICS_ENABLED="true"
Environment=OP_NODE_METRICS_PORT=7300
Environment=OP_NODE_NETWORK="base-sepolia"
Environment=OP_NODE_P2P_AGENT="base"
Environment=OP_NODE_P2P_BOOTNODES="enr:-J64QBwRIWAco7lv6jImSOjPU_W266lHXzpAS5YOh7WmgTyBZkgLgOwo_mxKJq3wz2XRbsoBItbv1dCyjIoNq67mFguGAYrTxM42gmlkgnY0gmlwhBLSsHKHb3BzdGFja4S0lAUAiXNlY3AyNTZrMaEDmoWSi8hcsRpQf2eJsNUx-sqv6fH4btmo2HsAzZFAKnKDdGNwgiQGg3VkcIIkBg,enr:-J64QFa3qMsONLGphfjEkeYyF6Jkil_jCuJmm7_a42ckZeUQGLVzrzstZNb1dgBp1GGx9bzImq5VxJLP-BaptZThGiWGAYrTytOvgmlkgnY0gmlwhGsV-zeHb3BzdGFja4S0lAUAiXNlY3AyNTZrMaEDahfSECTIS_cXyZ8IyNf4leANlZnrsMEWTkEYxf4GMCmDdGNwgiQGg3VkcIIkBg"
Environment=OP_NODE_P2P_LISTEN_IP=0.0.0.0
Environment=OP_NODE_P2P_LISTEN_TCP_PORT=9222
Environment=OP_NODE_P2P_LISTEN_UDP_PORT=9222
Environment=OP_NODE_RPC_ADDR=0.0.0.0
Environment=OP_NODE_RPC_PORT=8547
Environment=OP_NODE_SNAPSHOT_LOG="/mnt/base-sepolia/data/Environment=op-node-snapshot-log"
Environment=OP_NODE_VERIFIER_L1_CONFS=4
Environment=OP_NODE_ROLLUP_LOAD_PROTOCOL_VERSIONS="true"
Environment=OP_NODE_L1_TRUST_RPC="true"
Environment=OP_NODE_SYNCMODE="execution-layer"
Environment=OP_GETH_BOOTNODES="enode://548f715f3fc388a7c917ba644a2f16270f1ede48a5d88a4d14ea287cc916068363f3092e39936f1a3e7885198bef0e5af951f1d7b1041ce8ba4010917777e71f@18.210.176.114:30301,enode://6f10052847a966a725c9f4adf6716f9141155b99a0fb487fea3f51498f4c2a2cb8d534e680ee678f9447db85b93ff7c74562762c3714783a7233ac448603b25f@107.21.251.55:30301"
Environment=GETH_DATA_DIR="/mnt/base-sepolia/data"
Environment=VERBOSITY=3
Environment=RPC_PORT=8545
Environment=WS_PORT=8546
Environment=AUTHRPC_PORT=8551
Environment=METRICS_PORT=6060
Environment=HOST_IP=0.0.0.0
Environment=P2P_PORT=30304
Environment=OP_GETH_GCMODE="archive"
Environment=OP_GETH_SYNCMODE="full"
Type=simple
ExecStart=/usr/bin/op-node
TimeoutStopSec=90
Restart=on-failure
RestartSec=10s
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=op-node

[Install]
WantedBy=multi-user.target"  > /etc/systemd/system/op-node.service
```

{% hint style="warning" %}
If you don’t know your public IP address `curl http://api.ipify.org`&#x20;

* Replace `http://L1_ETH_ENDPOINT_URL` , `http://L1_BEACON_URL` & `YOUR_PUBLIC_IP_ADDRESS` .
* Create a data directory under project :

{% code fullWidth="false" %}
```bash
mkdir /mnt/base-sepolia/data
```
{% endcode %}

* Create secret file :

```
cd /mnt/base-sepolia/data
openssl rand -hex 32 | tr -d "\n" > "./engine-auth-jwt"
```
{% endhint %}
{% endstep %}

{% step %}
### Creating op-reth service file

```bash
echo "[Unit]
Description=Optimism Go-arbiturum client
After=network.target

[Service]
User=root
Type=simple
ExecStart=/usr/bin/op-reth node \
--datadir=/mnt/base-sepolia/data-reth \
--ws \
--ws.origins="*" \
--ws.addr=0.0.0.0 \
--ws.port=8546 \
--ws.api=debug,eth,net,trace,txpool,web3,rpc,reth,admin \
--http \
--http.corsdomain="*" \
--http.addr=0.0.0.0 \
--http.port=8545 \
--http.api=debug,eth,net,trace,txpool,web3,rpc,reth,admin \
--authrpc.addr=0.0.0.0 \
--authrpc.port=8551 \
--authrpc.jwtsecret=/mnt/base-sepolia/data-reth/engine-auth-jwt \
--metrics=0.0.0.0:6060 \
--chain=base-sepolia \
--rollup.sequencer-http=https://sepolia-sequencer.base.org \
--rollup.disable-tx-pool-gossip

KillMode=process
KillSignal=SIGINT
TimeoutStopSec=90
Restart=on-failure
RestartSec=10s
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/op-reth.service
```

{% hint style="warning" %}
Create a data directory:

```bash
mkdir -p /mnt/base-sepolia/data-reth
cp /mnt/base-sepolia/data/engine-auth-jwt /mnt/base-sepolia/data-reth/engine-auth-jwt
```
{% endhint %}
{% endstep %}

{% step %}
### Enable Services&#x20;

```bash
systemctl daemon-reload
systemctl enable op-node.service
systemctl enable op-reth.service
```
{% endstep %}

{% step %}
### Start Services

```bash
systemctl start op-node.service
systemctl start op-reth.service
```
{% endstep %}
{% endstepper %}

## Monitoring <a href="#monitor-logs" id="monitor-logs"></a>

{% hint style="warning" %}
The Sync time is \~24 hours ; depending upon L1 RPC, Peers , Disk & Internet speed .
{% endhint %}

* Check is Service file is running and status is <mark style="color:green;">active(running)</mark>

```bash
systemctl status op-node.service
systemctl status op-reth.service
```

* Check op-node service file logs&#x20;

```bash
journalctl -fu op-node.service # Live logs
```

&#x20;  _result_&#x20;

```json
Mar 15 08:08:58 op-node[2644281]: t=2025-03-15T08:08:58+0100 lvl=info msg="Optimistically queueing unsafe L2 execution payload" id=0x90e199e558a43d057755c3b10375098888c71c24e59e0b68752d0e2779cd8637:23127125
Mar 15 08:08:58 op-node[2644281]: t=2025-03-15T08:08:58+0100 lvl=info msg="Inserted new L2 unsafe block (synchronous)" hash=0x90e199e558a43d057755c3b10375098888c71c24e59e0b68752d0e2779cd8637 number=23127125 newpayload_time=46.682ms fcu2_time=422.404µs total_time=47.106ms mgas=20.235851 mgasps=429.57957359986085
Mar 15 08:08:58 op-node[2644281]: t=2025-03-15T08:08:58+0100 lvl=info msg="Sync progress" reason="new chain head block" l2_finalized=0xad05950d632d858da449f885a4d6216af4793521b636adc42e20d8d01344049a:23096311 l2_safe=0xad05950d632d858da449f885a4d6216af4793521b636adc42e20d8d01344049a:23096311 l2_pending_safe=0xad05950d632d858da449f885a4d6216af4793521b636adc42e20d8d01344049a:23096311 l2_unsafe=0x90e199e558a43d057755c3b10375098888c71c24e59e0b68752d0e2779cd8637:23127125 l2_backup_unsafe=0x0000000000000000000000000000000000000000000000000000000000000000:0 l2_time=1742022538
Mar 15 08:08:58 op-node[2644281]: t=2025-03-15T08:08:58+0100 lvl=info msg="successfully processed payload" ref=0x90e199e558a43d057755c3b10375098888c71c24e59e0b68752d0e2779cd8637:23127125 txs=72
```

* Check op-reth service file logs

```bash
journalctl -fu op-reth.service # Live logs
```

_result_

```json
Mar 15 08:08:48 op-reth[2644142]: 2025-03-15T07:08:48.796637Z  INFO Canonical chain committed number=23127120 hash=0xe65860150e39a3f1784e38c9f2eced0913b8697106a910ebfa6bfb5d0c68c0a8 elapsed=131.183µs
Mar 15 08:08:50 op-reth[2644142]: 2025-03-15T07:08:50.465654Z  INFO State root task finished state_root=0x1611ce8389868100d8fec8c4edf019cb7937bd0f2f3b97be628dc76481242f4c elapsed=4.161285ms
Mar 15 08:08:50 op-reth[2644142]: 2025-03-15T07:08:50.465808Z  INFO Block added to canonical chain number=23127121 hash=0x7527aee3323166d9e173d3014b11547e998fd4f7ee96d71cc56ef7ba96f3dfaf peers=60 txs=69 gas=14.87 Mgas gas_throughput=423.14 Mgas/second full=24.8% base_fee=0.01gwei blobs=0 excess_blobs=0 elapsed=35.151491ms
Mar 15 08:08:50 op-reth[2644142]: 2025-03-15T07:08:50.466326Z  INFO Canonical chain committed number=23127121 hash=0x7527aee3323166d9e173d3014b11547e998fd4f7ee96d71cc56ef7ba96f3dfaf elapsed=100.196µs
```

## Sync Status

_Run a query to check the latest synchronized L2 block:_

```bash
curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber",
"params":[],"id":83}' http://localhost:8545
```

_Response should look like:_

```json
{"jsonrpc":"2.0","id":83,"result":"0x160e561"}
```

## References



{% embed url="https://sepolia.basescan.org/" %}
