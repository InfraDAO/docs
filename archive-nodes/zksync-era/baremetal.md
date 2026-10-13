# 💻 Baremetal

Author: \[ jLeopoldA ]

## System Requirements

| CPU     | OS                 | RAM  | DISK |
| ------- | ------------------ | ---- | ---- |
| 4+Cores | Ubuntu 24.04.1 LTS | 32GB | 16TB |

{% hint style="info" %}
The Zksync Archive Node has a size of  10TB as of 2/28/2025.
{% endhint %}

## Pre-Requisites

{% hint style="info" %}
Zksync requires PostgreSQL, Docker, Docker-Compose, an Ethereum L1, and a database dump.
{% endhint %}

### Update and Clean System

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
```

### Install Docker & Docker-Compose

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Install Docker Packages including Docker Compose
 sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
 
# Verify Docker Engine Installation
sudo docker run hello-world
```

### Firewall Configuration

#### Set Explicit Default Firewall Rules

```bash
sudo ufw default deny incoming && sudo ufw default allow outgoing
```

#### Allow SSH

```bash
sudo ufw allow 22/tcp
```

#### Allow RPC Connections with Zksync

```bash
sudo ufw allow 3060
```

#### Allow P2P & Metrics

```bash
sudo ufw allow 3322 && sudo ufw allow 3061 
```

#### Enable Firewall

```bash
sudo ufw enable
```

#### Check Status & Current Rules of UFW

```bash
sudo ufw status verbose
```

### PostgreSQL

#### Install PostgreSQL

```bash
sudo apt install postgresql postgresql-contrib
```

#### Set Password

```bash
# Enter the psql terminal
psql -U postgres

# Create password
ALTER ROLE postgres WITH PASSWORD 'password';

# Create Database for Zksync
CREATE DATABASE zksync_local_ext_node;

# Exit psql terminal
\q
```

## Download and Import Database Dump

#### Download Database Dump

```bash
# Make Directory for Database
mkdir -p /root/zksync/dump
cd /root/zksync/dump

# This website provides database dumps
# https://en-backups.matterlabs.dev/
# Right click and copy the link you would like 
nohup wget <selected_link> & disown

# Example Download Dump
nohup wget http://37.27.135.10:8080/ipfs/QmVWVRjw5DEuG5noaD7Ltcc4BGGCKEcjvL8CYFw5GcAzWD/external_node_latest.pgdump.sql.gz & disown

# Check Status of Download
tail -f nohup.out
```

#### Import Database Dump

```bash
# Remove previous nohup.out
rm nohup.out

# Import Database into zksync_local_ext_node
zcat external_node_backup_2024_12_13T05_05.sql.gz | nohup psql -U postgres -d zksync_local_ext_node

# Vague layout
zcat <your_downloaded_file> | nohup psql -U your_username_here -d your_database_name_here

# Check import status
tail -f nohup.out

# Check Database Size
psql -U postgres
SELECT pg_size_pretty(pg_database_size('zksync_local_ext_node'));

# Abstract Database Size Check
psql -U your_username_here
SELECT pg_size_pretty(pg_database_size('your_database_name_here'));

# Ensure database is done
ps aux | grep "nohup"

# If there are multiple responses like the below then it is still processing.
root     2602454  0.0  0.0   6544  2048 pts/0    S+   22:33   0:00 grep --color=auto nohup
```

{% hint style="warning" %}
Ensure that your database has finished importing before proceeding to the next step.
{% endhint %}

## Set up Zksync

#### Create Directories

```bash
# Extracted Content Directory
mkdir -p /root/zksync/repo/contracts
mkdir -p /root/zksync/repo/etc
mkdir -p /root/zksync/repo/usr/bin
mkdir -p /root/zksync/repo/migrations

# External Node Data Directory
mkdir -p /var/lib/zksync/db/ext-node/state_keeper
mkdir -p /var/lib/zksync/db/ext-node/lightweight

# Directory for Consensus Config
mkdir -p /etc/zksync/
```

#### Extract Zksync Binaries from Docker

```bash
# Pull latest Zksync Image
docker pull matterlabs/external-node:latest

# Copy necessary components to our created directories
docker cp $(docker run -it -d matterlabs/external-node:latest):/usr/bin/zksync_external_node /root/zksync/repo/usr/bin
docker cp $(docker run -it -d matterlabs/external-node:latest):/usr/bin/block_reverter /root/zksync/repo/usr/bin
docker cp $(docker run -it -d matterlabs/external-node:latest):/usr/bin/sqlx /root/zksync/repo/usr/bin
docker cp $(docker run -it -d matterlabs/external-node:latest):/contracts/. /root/zksync/repo/contracts
docker cp $(docker run -it -d matterlabs/external-node:latest):/etc/tokens/. /root/zksync/repo/etc/tokens/
docker cp $(docker run -it -d matterlabs/external-node:latest):/etc/ERC20/. /root/zksync/repo/etc/ERC20/
docker cp $(docker run -it -d matterlabs/external-node:latest):/etc/multivm_bootloaders/. /root/zksync/repo/etc/multivm_bootloaders/
docker cp $(docker run -it -d matterlabs/external-node:latest):/migrations/. /root/zksync/repo/migrations/
```

#### Perform Database Migration

```bash
cd /root/zksync/repo/usr/bin

# Set migration folder as source and specify database
nohup ./sqlx migrate info --source /root/FINAL/repo/migrations --database-url postgres://postgres:password@localhost/zksync_local_ext_node & disown

# When the above is done - remove uncessary file
rm nohup.out

# Run Database Migration
nohup ./sqlx migrate run --source /root/FINAL/repo/migrations --database-url postgres://post
```

### Create Mainnet Consensus Configuration&#x20;

```bash
# Create Mainnet Configuration
echo "server_addr: '0.0.0.0:3054'
public_addr: '127.0.0.1:3054'
debug_page_addr: '127.0.0.1:5000'
max_payload_size: 5000000
gossip_dynamic_inbound_limit: 100
gossip_static_outbound:
  # preconfigured ENs owned by Matterlabs that you can connect to
  - key: 'node:public:ed25519:68d29127ab03408bf5c838553b19c32bdb3aaaae9bf293e5e078c3a0d265822a'
    addr: 'external-node-consensus-mainnet.zksync.dev:3054'
  - key: 'node:public:ed25519:b521e1bb173d04bc83d46b859d1296378e94a40427a6beb9e7fdd17cbd934c11'
    addr: 'external-node-moby-consensus-mainnet.zksync.dev:3054'" > /etc/zksync/mainnet_consensus_config.yaml
```

### Create System Service

```bash
echo "[Unit]
Description=zkSync Era Node
After=network.target

[Service]
User=root
Environment=DATABASE_URL="postgres://postgres:password@localhost/zksync_local_ext_node"
Environment=DATABASE_POOL_SIZE=10
Environment=EN_HTTP_PORT=3060
Environment=EN_WS_PORT=3061
Environment=EN_HEALTHCHECK_PORT=3081
Environment=EN_PROMETHEUS_PORT=3322
Environment=EN_ETH_CLIENT_URL="<YOUR_L1_ETH_URL_HERE"
Environment=EN_MAIN_NODE_URL="https://zksync2-mainnet.zksync.io"
Environment=EN_L1_CHAIN_ID=1
Environment=EN_L2_CHAIN_ID=324
Environment=EN_PRUNING_ENABLED=false
Environment=EN_STATE_CACHE_PATH="/var/lib/zksync/db/ext-node/state_keeper"
Environment=EN_MERKLE_TREE_PATH="/var/lib/zksync/db/ext-node/lightweight"
Environment=EN_SNAPSHOTS_RECOVERY_ENABLED="false"
Environment=EN_SNAPSHOTS_OBJECT_STORE_BUCKET_BASE_URL="zksync-era-mainnet-external-node-snapshots"
Environment=EN_SNAPSHOTS_OBJECT_STORE_MODE="GCSAnonymousReadOnly"
Environment=RUST_LOG="warn,zksync=info,zksync_core:metadata_calculator=debug,zksync_state=debug,zksync_utils=debug,zksync_web3_decl::client=error"
Environment=EN_CONSENSUS_CONFIG_PATH="/etc/zksync/mainnet_consensus_config.yaml"
Environment=EN_CONSENSUS_SECRETS="/var/lib/zksync/mainnet_consensus_secrets.yaml"
WorkingDirectory=/root/zksync/repo
ExecStart=/root/zksync/repo/usr/bin/zksync_external_node 
Restart=on-failure
LimitNOFILE=1000000
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/zksync.service
```

## Run Zksync Archive Node

#### Systemctl Commands for Zksync&#x20;

```bash
systemctl daemon-reload # Reload systemctl 
systemctl enable zksync.service # Enable Zksync 
systemctl start zksync.service # Start Zksync
systemctl stop zksync.service # Stop Zksync
systemctl restart zksync.service # Restart Zksync
```

### Query Zksync Node

```bash
# CHECK STATUS OF SYNC
curl -H "Content-Type: application/json" \
-X POST --data '{"jsonrpc":"2.0", "method":"eth_syncing", "params":[], "id":1}' http://localhost:3060

##### The response should resemble the below
{"jsonrpc":"2.0","id":1,"result":{"startingBlock":"0x0","currentBlock":"0x3219750","highestBlock":"0x35fc3d7"}}

# CHECK LATEST BLOCK
curl -H "Content-Type: application/json" \ 
-X POST --data '{"jsonrpc":"2.0", "method":"eth_blockNumber", "params":[], "id":1}' http://localhost:3060

##### The response should resemble the below
{"jsonrpc":"2.0","id":1,"result":"0x321afb7"}
```
