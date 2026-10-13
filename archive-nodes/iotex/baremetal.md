---
description: 'Author: [ jLeopoldA ]'
---

# 💻 Baremetal

## System Requirements

| CPU                    | OS       | RAM   | DISK                |
| ---------------------- | -------- | ----- | ------------------- |
| Debian 12/Ubuntu 22.04 | 8+ Cores | 32GB+ | 10TB+ (SSD or NVMe) |

{% hint style="info" %}
The Iotex Archive Node has a size of 5.5TB as of 3/5/2025
{% endhint %}

{% hint style="warning" %}
The Iotex Archive Node requires an L1 Ethereum RPC
{% endhint %}

## Pre-Requisites

#### Update System

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y
sudo apt install -y git gcc make --fix-missing
```

#### Install GO

```bash
curl -LO https://go.dev/dl/go1.21.8.linux-amd64.tar.gz
sudo tar xzf go1.21.8.linux-amd64.tar.gz -C /usr/local && rm go1.21.8.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc

# Check Installation
go version
```

## Firewall Configuration

#### Set Explicit Firewall Rules

```bash
sudo ufw default deny incoming 
sudo ufw default allow outgoing
```

#### Allow SSH

```bash
sudo ufw allow 22/tcp
```

#### Allow Connections for Iotex

```bash
sudo ufw allow 4689
sudo ufw allow 8080
sudo ufw allow 14014
sudo ufw allow 16014
sudo ufw allow 15014
```

#### Enable Firewall Rules

```bash
sudo ufw enable
```

#### Check Status of Firewall Rules (UFW)

```bash
sudo ufw status verbose
```

## Set up Iotex Configuration and Data

#### Create Directories

```bash
# Make Directory for Iotex Server and Data
mkdir -p /var/lib/iotex
mkdir -p /var/lib/iotex/etc
mkdir -p /var/lib/iotex/data
mkdir -p /var/lib/iotex/log

# Make Directory for Configuration
mkdir -p /root/configuration
```

#### Download trie.db.patch & poll.db

```bash
curl https://raw.githubusercontent.com/iotexproject/iotex-bootstrap/master/trie.db.patch > /var/lib/iotex/data/trie.db.patch
curl https://storage.googleapis.com/blockchain-golden/poll.mainnet.db > /var/lib/iotex/data/poll.db
```

### Create Configuration

#### Download genesis.yaml

```bash
curl https://raw.githubusercontent.com/iotexproject/iotex-bootstrap/master/genesis_mainnet.yaml > /root/configuration/genesis.yaml
```

#### Create general\_config.yaml

This portion requires an L1\_RPC within the configuration.  Specifically, the object route of chain > committee > gravityChainAPIs

```bash
echo "
---
network:
  # externalHost: SET YOUR EXTERNAL IP HERE (e.g., 12.34.56.78)
  externalPort: 4689
  bootstrapNodes:
    - /dns4/bootnode-0.mainnet.iotex.one/tcp/4689/p2p/12D3KooWPfQDF8ASjd4r7jS9e7F1wn6zod7Mf4ERr8etoY6ctQp5
    - /dns4/bootnode-1.mainnet.iotex.one/tcp/4689/p2p/12D3KooWN4TQ1CWRA7yvJdQCdti1qARLXXu2UEHJfycn3XbnAnRh
    - /dns4/bootnode-2.mainnet.iotex.one/tcp/4689/p2p/12D3KooWSiktocuUke16bPoW9zrLawEBaEc1UriaPRwm82xbr2BQ
    - /dns4/bootnode-3.mainnet.iotex.one/tcp/4689/p2p/12D3KooWEsmwaorbZX3HRCnhkMPjMAHzwu3om1pdGrtVm2QaM35n
    - /dns4/bootnode-4.mainnet.iotex.one/tcp/4689/p2p/12D3KooWHRcgNim4Nau73EEu7aKJZRZPZ21vQ7BE3fG6vENXkduB
    - /dns4/bootnode-5.mainnet.iotex.one/tcp/4689/p2p/12D3KooWGeHkVDQQFxXpTX1WpPhuuuWYTxPYDUTmaLWWSYx5rmUY

chain:
  # If you are a delegate, make sure producerPrivKey is the key for the
  # operator address you have registered.
  # producerPrivKey: SET YOUR PRIVATE KEY HERE (e.g.,
  # 96f0aa5e8523d6a28dc35c927274be4e931e74eaa720b418735debfcbfe712b8)
  enableStakingIndexer: true
  chainDBPath: "/var/lib/iotex/data/chain.db"
  trieDBPatchFile: "/var/lib/iotex/data/trie.db.patch"
  trieDBPath: "/var/lib/iotex/data/archive.db"
  stakingPatchDir: "/var/lib/iotex/data"
  indexDBPath: "/var/lib/iotex/data/index.db"
  blobStoreDBPath: "/var/lib/iotex/data/blob.db"
  bloomfilterIndexDBPath: "/var/lib/iotex/data/bloomfilter.index.db"
  candidateIndexDBPath: "/var/lib/iotex/data/candidate.index.db"
  stakingIndexDBPath: "/var/lib/iotex/data/staking.index.db"
  contractStakingIndexDBPath: "/var/lib/iotex/data/contractstaking.index.db"
  enableArchiveMode: true
  maxCacheSize: 1000
  committee:
    gravityChainAPIs:
      # Please change the Infura key to your key (e.g.,
      # https://mainnet.infura.io/v3/YOUR_KEY)
       - {YOUR_L1_RPC_HERE}
    numOfRetries: 20
    paginationSize: 255
    cacheSize: 1000
  gravityChainDB:
    dbPath: "/var/lib/iotex/data/poll.db"
    numRetries: 8

actPool:
  minGasPrice: "1000000000000"
  store:
    datadir: "/var/lib/iotex/data/actpool.cache"

api:
  gasStation:
    defaultGas: 1000000000000

consensus:
  scheme: ROLLDPOS
  rollDPoS:
    fsm:
      unmatchedEventTTL: 3s
      unmatchedEventInterval: 100ms
      acceptBlockTTL: 4s
      acceptProposalEndorsementTTL: 2s
      acceptLockEndorsementTTL: 2s
    delay: 10s
    consensusDBPath: "/var/lib/iotex/data/consensus.db"

blockSync:
  interval: 2s
  bufferSize: 400
  maxRepeat: 3
  repeatDecayStep: 3

log:
  zap:
    level: info
    encoding: json
    disableStacktrace: true
    outputPaths: ["stderr", "stdout"]
    errorOutputPaths: ["stderr"]
  stderrRedirectFile: /var/lib/iotex/log/s.log
  stdLogRedirect: true
" > /root/configuration/general_config.yaml
```

### Download Necessary Data Files

#### Download iotex-data

```bash
cd /var/lib/iotex

# Multiple Data Files (Block, Blob Storage, Indexing Files)
nohup curl -LO https://storage.googleapis.com/blockchain-golden/archive/iotex-data.tar.gz & disown

# Check the status of download
tail -f /var/lib/iotex/nohup.out

# When done downloading - unpack.
tar -xzf iotex-data.tar.gz
```

#### Download State Database

```bash
cd /var/lib/iotex/data

# Download State / Address Database
nohup curl -LO https://storage.googleapis.com/blockchain-golden/archive/archive.db & disown

# The State Database download will take a long time as it is uncompressed
# Check the status of State Database download
tail -f /var/lib/iotex/data/nohup.out
```

{% hint style="warning" %}
Wait for your data to finish downloading before proceeding to the next step.
{% endhint %}

## Set up Iotex Chain

#### Clone Repository and Build Binary

```bash
# Clone the Repository
cd /root
git clone https://github.com/iotexproject/iotex-core.git
cd iotex-core

# Checkout the Archive node branch
git checkout origin/archive

# Build binary
make build

# Copy Server
cp ./bin/server /var/lib/iotex/server
```

### Set up Iotex as a System Service

#### Create System Service File

```bash
echo "[Unit]
Description=Iotex Node
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
WorkingDirectory=/var/lib/iotex
ExecStart=/var/lib/iotex/server \
	-config-path=/root/configuration/general_config.yaml \
	-genesis-path=/root/configuration/genesis.yaml 
	-plugin=gateway &
KillSignal=SIGTERM
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/iotex.service
```

#### Start System Service

```bash
sudo systemctl daemon-reload # Reload SystemCtl
sudo systemctl start iotex.service # Start Service
sudo systemctl stop iotex.service # Stop Service
sudo systemctl status iotex.service # Check status
```

## Query Node

#### Check Logs of Node

````bash
journalctl -fu iotex.service -o cat

# The response should resemble the below
```
{"level":"info","ts":"2025-02-18T18:34:18.829+0100","caller":"chainservice/builder.go:628","msg":"Successfully committed block.","ioAddr":"io1p2xu7xve7nqp85rvyrmqgy9yd3pjuy2xe7ejtr","height":33734123}
{"level":"info","ts":"2025-02-18T18:34:18.949+0100","caller":"chainservice/builder.go:628","msg":"Successfully committed block.","ioAddr":"io1p2xu7xve7nqp85rvyrmqgy9yd3pjuy2xe7ejtr","height":33734124}
{"level":"info","ts":"2025-02-18T18:34:19.078+0100","caller":"chainservice/builder.go:628","msg":"Successfully committed block.","ioAddr":"io1p2xu7xve7nqp85rvyrmqgy9yd3pjuy2xe7ejtr","height":33734125}
{"level":"info","ts":"2025-02-18T18:34:19.156+0100","caller":"chainservice/builder.go:628","msg":"Successfully committed block.","ioAddr":"io1p2xu7xve7nqp85rvyrmqgy9yd3pjuy2xe7ejtr","height":33734126}
{"level":"info","ts":"2025-02-18T18:34:19.305+0100","caller":"chainservice/builder.go:628","msg":"Successfully committed block.","ioAddr":"io1p2xu7xve7nqp85rvyrmqgy9yd3pjuy2xe7ejtr","height":33734127}
{"level":"info","ts":"2025-02-18T18:34:19.430+0100","caller":"chainservice/builder.go:628","msg":"Successfully committed block.","ioAddr":"io1p2xu7xve7nqp85rvyrmqgy9yd3pjuy2xe7ejtr","height":33734128}
{"level":"info","ts":"2025-02-18T18:34:19.641+0100","caller":"chainservice/builder.go:628","msg":"Successfully committed block.","ioAddr":"io1p2xu7xve7nqp85rvyrmqgy9yd3pjuy2xe7ejtr","height":33734129}
{"level":"info","ts":"2025-02-18T18:34:20.047+0100","caller":"chainservice/builder.go:628","msg":"Successfully committed block.","ioAddr":"io1p2xu7xve7nqp85rvyrmqgy9yd3pjuy2xe7ejtr","height":33734130}
{"level":"info","ts":"2025-02-18T18:34:20.157+0100","caller":"chainservice/builder.go:628","msg":"Successfully committed block.","ioAddr":"io1p2xu7xve7nqp85rvyrmqgy9yd3pjuy2xe7ejtr","height":33734131}
{"level":"info","ts":"2025-02-18T18:34:20.222+0100","caller":"chainservice/builder.go:628","msg":"Successfully committed block.","ioAddr":"io1p2xu7xve7nqp85rvyrmqgy9yd3pjuy2xe7ejtr","height":33734132}
{"level":"info","ts":"2025-02-18T18:34:20.312+0100","caller":"chainservice/builder.go:628","msg":"Successfully committed block.","ioAddr":"io1p2xu7xve7nqp85rvyrmqgy9yd3pjuy2xe7ejtr","height":33734133}
{"level":"info","ts":"2025-02-18T18:34:20.904+0100","caller":"chainservice/builder.go:628","msg":"Successfully committed block.","ioAddr":"io1p2xu7xve7nqp85rvyrmqgy9yd3pjuy2xe7ejtr","height":33734134}
{"level":"info","ts":"2025-02-18T18:34:20.992+0100","caller":"chainservice/builder.go:628","msg":"Successfully committed block.","ioAddr":"io1p2xu7xve7nqp85rvyrmqgy9yd3pjuy2xe7ejtr","height":33734135}
{"level":"info","ts":"2025-02-18T18:34:21.120+0100","caller":"chainservice/builder.go:628","msg":"Successfully committed block.","ioAddr":"io1p2xu7xve7nqp85rvyrmqgy9yd3pjuy2xe7ejtr","height":33734136}
{"level":"info","ts":"2025-02-18T18:34:21.252+0100","caller":"chainservice/builder.go:628","msg":"Successfully committed block.","ioAddr":"io1p2xu7xve7nqp85rvyrmqgy9yd3pjuy2xe7ejtr","height":33734137}
```
````

#### Check Sync Status

```bash
curl -H "Content-Type: application/json" -X POST --data \
'{"jsonrpc":"2.0", "method":"eth_syncing", "params":[], "id":1}' \
http://localhost:15014
```

#### Check Block Number

```bash
curl -H "Content-Type: application/json" -X POST --data \
'{"jsonrpc":"2.0", "method":"eth_blockNumber", "params":[], "id":1}' \
http://localhost:15014

# Response should look similar to the below
{"jsonrpc":"2.0","id":1,"result":"0x202c35d"}
```

### References <a href="#references" id="references"></a>

{% embed url="https://github.com/iotexproject/iotex-bootstrap/blob/master/archive-node.md" %}
