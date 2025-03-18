---
description: 'Author: [ jLeopoldA ]'
---

# 💻 Baremetal

## System Requirements

| CPU      | OS                 | RAM  | DISK |
| -------- | ------------------ | ---- | ---- |
| 4+ Cores | Ubuntu 22.04.4 LTS | 16GB | 5TB  |

{% hint style="info" %}
The Optimism Sepolia Archive Node has a size of 2.3TB as of 3/10/2025.
{% endhint %}

{% hint style="info" %}
Subgraphs using callHandlers requires the RPC client to support the Parity Tracing API. With Optimism sepolia we observe that `erigon` supports the parity tracing API, while `geth` does not.\
\
This guide shows how to set up a `geth` client.&#x20;
{% endhint %}

## Pre-Requisites

#### Update System

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y
sudo apt install -y git gcc make --fix-missing
```

#### Install GO

{% hint style="warning" %}
OP-NODE and OP-GETH specifically require GO  v1.22.0.\
OP-NODE requires an L1 and an L1 Beacon.
{% endhint %}

```bash
# Remove previous installation of GO
rm -rf /usr/local/go # For GO installations locacated within /usr/local/go
rm -rf /usr/local/bin/go # For GO installations located within /usr/local/bin/go

# Download GO
wget https://go.dev/dl/go1.22.0.linux-amd64.tar.gz

# Extract and place within /usr/local
tar -xzf go1.22.0.linux-amd64.tar.gz -C /usr/local && rm go1.22.0.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc
```

## Firewall Configuration

#### Set Explicit Firewall Configuration

```bash
sudo ufw default deny incoming && sudo ufw default allow outgoing
```

#### Allow SSH

```bash
sudo ufw allow 22/tcp
```

#### Allow Connections for OP-NODE & OP-GETH

```bash
sudo ufw allow 9222
sudo ufw allow 9545
sudo ufw allow 8545
sudo ufw allow 30303
```

#### Enable Firewall Rules

```bash
sudo ufw enable
```

#### Check Status of Firewall Rules (UFW)

```bash
sudo ufw status verbose
```

## Download and Set up OP-Node & OP-Geth

### Create Directories

```bash
mkdir -p /var/lib/optimism/database
mkdir -p /var/lib/optimism/configuration
mkdir -p /root/chain
```

### Create JWT Secret

```bash
openssl rand -hex 32 > /var/lib/optimism/configuration/jwt.txt
```

## Set up OP-Node&#x20;

#### Download & Build OP-Node

```bash
cd /root/chain

# Clone Optimism Repo
git clone https://github.com/ethereum-optimism/optimism.git
cd optimism

# Check out Latest Git version
git checkout v1.10.0

# Build OP-Node
make op-node
```

### Set up OP-Geth

```bash
cd /root/chain

# Clone OP-Geth repo
git clone https://github.com/ethereum-optimism/op-geth.git
cd op-geth

# Check out latest Git version
git checkout v1.101500.1

# Build OP-Geth
make geth
```

## Create System Services

#### Create Service for OP-Node

```bash
echo "[Unit]
Description=op-node
After=network.target
StartLimitIntervalSec=200
StartLimitBurst=5

[Service]
Type=simple
Restart=on-failure
RestartSec=5
TimeoutSec=900
User=root
Nice=0
LimitNOFILE=200000
WorkingDirectory=/root/chain/optimism/
ExecStart=/root/chain/optimism/op-node/bin/op-node \
	--l1={L1_URL_HERE} \
	--l1.rpckind=any \
	--l1.beacon={L1_BEACON_URL_HERE} \
	--l2=ws://localhost:8551 \
	--l2.jwt-secret=/var/lib/optimism/configuration/jwt.txt \
	--network=op-sepolia \
	--syncmode=execution-layer
KillSignnal=SIGTERM
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target" > 
```

#### Create Service for OP-Geth

```bash
echo "[Unit]
Description=op-geth
After=network.target

[Service]
Restart=on-failure
RestartSec=5
TimeoutSec=900
User=root
Nice=0
LimitNOFILE=200000
WorkingDirectory=/root/chain/op-geth
ExecStart=/root/chain/op-geth/build/bin/geth \
    --http --http.port=8545 --http.addr=localhost \
    --authrpc.addr=localhost \
    --authrpc.jwtsecret=/var/lib/optimism/configuration/jwt.txt \
    --verbosity=3 \
    --rollup.sequencerhttp=https://sepolia-sequencer.optimism.io/ \
    --op-network=op-sepolia \
    --datadir=/var/lib/optimism/database \
    --syncmode=full --gcmode=archive
KillSignal=SIGTERM
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/op-geth.service
```

## Run System Services

#### Reload System Services

```bash
systemctl daemon-reload
```

#### Run OP-Node Service

```bash
systemctl enable op-node.service 
systemctl start op-node.service
```

#### Run OP-Geth Service

```bash
systemctl enable op-geth.service
systemctl start op-geth.service
```

## Query Node

#### Check Logs

```bash
# Check OP-Node
journalctl -xeu op-node.service -o cat

# Check OP-Geth
journalctl -xeu op-geth.service -o cat
```

#### Check Sync Status

```bash
curl -H "Content-Type: application/json" \
-X POST --data '{"jsonrpc":"2.0", "method":"eth_syncing", "params":[], "id":1}' \
http://localhost:8545

# If node is done syncing - the response should resemble the below.
{"jsonrpc":"2.0","id":1,"result":false}
```

#### Check Block Number

```bash
curl -H "Content-Type: application/json" \
-X POST --data '{"jsonrpc":"2.0", "method":"eth_blockNumber", "params":[], "id":1}' \
http://localhost:8545

# Response should resemble the below.
{"jsonrpc":"2.0","id":1,"result":"0x17c07de"}
```

{% embed url="https://docs.optimism.io/operators/node-operators/tutorials/node-from-source" %}

{% embed url="https://docs.optimism.io/operators/node-operators/tutorials/run-node-from-source" %}

{% embed url="https://github.com/ethereum-optimism/optimism" %}

{% embed url="https://github.com/ethereum-optimism/op-geth" %}
