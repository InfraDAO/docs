# 💻 Baremetal

Authors: \[man4ela | catapulta.eth]

## System Requirements <a href="#system-requirements" id="system-requirements"></a>

| **CPU**       | **OS**                 | **RAM**      | **DISK**           |
| ------------- | ---------------------- | ------------ | ------------------ |
| 16+ cores CPU | Debian 12/Ubuntu 22.04 | => 16 GB RAM | =1TB+(SSD or NVMe) |

{% hint style="info" %}
_The Ethereum Sepolia archive node has a size of 1TB on November 15th, 2024_
{% endhint %}

## Setup Production Erigon <a href="#setup-production-erigon" id="setup-production-erigon"></a>

{% hint style="success" %}
This guide covers the installation of`Erigon`, an implementation of Ethereum (execution layer), on the efficiency frontier, **Archive Node** by default, and `Lighthouse`, as a Consensus Layer.
{% endhint %}

## Pre-Requisites

{% code overflow="wrap" %}
```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y

sudo apt install -y libgtest-dev libomp-dev libgmp-dev git make wget aria2 gcc pkg-config libusb-1.0-0-dev libudev-dev jq g++ curl libssl-dev screen apache2-utils build-essential
```
{% endcode %}

### Setting up Firewall

Set explicit default UFW rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Allow SSH and peers

```bash
sudo ufw allow 22/tcp
sudo ufw allow 30303
sudo ufw allow 9001
```

Allow remote RPC connections with the Node

```bash
sudo ufw allow from ${REMOTE.HOST.IP} to any port 8545
sudo ufw allow from ${REMOTE.HOST.IP} to any port 5052
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

### Install GO

{% hint style="info" %}
Go version 1.21+ is required
{% endhint %}

```bash
sudo wget https://go.dev/dl/go1.21.6.linux-amd64.tar.gz && sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.21.6.linux-amd64.tar.gz && rm go1.21.6.linux-amd64.tar.gz

echo 'export PATH=$PATH:/usr/local/go/bin:/root/.local/bin' >> /root/.bashrc

source /root/.bashrc

#verify Go installation
go version
```

### Build Erigon RPC Node

```bash
git clone --recurse-submodules https://github.com/ledgerwatch/erigon.git

cd erigon 

git checkout v2.60.10

make erigon
```

#### Create Data directory and jwt secret file

```bash
cd ..

mkdir erigon-sepolia_data && cd erigon-sepolia_data

sudo openssl rand -hex -out /root/erigon-sepolia_data/jwtsecret 32
```

#### Create Systemd service for Erigon

```bash
sudo nano /etc/systemd/system/erigon-sepolia.service
```

Paste the configs and save by entering `ctrl+X` and `Y+ENTER`:

```bash
[Unit]
Description=Erigon Sepolia Service
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
WorkingDirectory=/root/erigon/build/bin/
ExecStart=/root/erigon/build/bin/erigon \
    --chain=sepolia \
    --port=30303 \
    --http.port=8545 \
    --torrent.port=42069 \
    --torrent.download.rate=80mb \
    --private.api.addr=127.0.0.1:9090 \
    --http \
    --ws \
    --http.api=eth,debug,net,trace,web3,erigon \
    --http.addr=0.0.0.0 \
    --http.corsdomain='*' \
    --metrics \
    --metrics.port=6060 \
    --metrics.addr=0.0.0.0 \
    --authrpc.jwtsecret=/root/erigon-sepolia_data/jwt.hex \
    --datadir=/root/erigon-sepolia_data \
    --rpc.gascap=5000000000 \
    --rpc.returndata.limit=1100000 \
    --pprof \
    --pprof.addr=0.0.0.0 \
    --pprof.port=6070

[Install]
WantedBy=multi-user.target
```

#### Launch Erigon

```bash
sudo systemctl daemon-reload #refresh systemd configuration when changes made

sudo systemctl enable erigon-sepolia.service #enable erigon-sepolia.service at system startup

sudo systemctl start erigon-sepolia.service #start erigon-sepolia.service

sudo nano /etc/systemd/system/erigon-sepolia.service #make changes in erigon-sepolia.service file
```

### Build Lighthouse

Go to the [Releases](https://github.com/sigp/lighthouse/releases) page and select the latest release:

```bash
cd /root/

mkdir lighthouse-sepolia_data

mkdir lighthouse-sepolia && cd lighthouse-sepolia

wget https://github.com/sigp/lighthouse/releases/download/v5.3.0/lighthouse-v5.3.0-x86_64-unknown-linux-gnu.tar.gz

tar -xzf lighthouse-v5.3.0-x86_64-unknown-linux-gnu.tar.gz #Extract the tar.gz archive

chmod +x /root/lighthouse-sepolia/ #Grant execute permissions to the files in the directory
```

#### Create systemd file for Lighthouse

```bash
sudo nano /etc/systemd/system/lighthouse-sepolia.service
```

Paste the configs and save by entering `ctrl+X` and `Y+ENTER`:

```bash
[Unit]
Description=Lighthouse-sepolia Beacon Node
After=network.target

[Service]
User=root
WorkingDirectory=/root/lighthouse-sepolia/
ExecStart=/root/lighthouse-sepolia/lighthouse beacon_node \
    --network sepolia \
    --datadir /root/lighthouse-sepolia_data \
    --http \
    --http-address 0.0.0.0 \
    --http-port 5052 \
    --execution-endpoint http://127.0.0.1:8551 \
    --checkpoint-sync-url https://sepolia.beaconstate.info \
    --execution-jwt /root/erigon-sepolia_data/jwt.hex \
    --disable-deposit-contract-sync \
    --prune-blobs false
Restart=on-failure
LimitNOFILE=1000000

[Install]
WantedBy=default.target
```

#### Launch Lighthouse

```bash
sudo systemctl daemon-reload #refresh systemd configuration when changes made

sudo systemctl enable lighthouse-sepolia.service #enable lighthouse-sepolia service at system startup

sudo systemctl start lighthouse-sepolia.service #start lighthouse-sepolia

sudo nano /etc/systemd/system/lighthouse-sepolia.service #make changes in lighthouse-sepolia.service file
```

### Monitor the logs for errors

```bash
journalctl -u erigon-sepolia.service -f -n 100 #follow logs of erigon-sepolia.service

journalctl -u lighthouse-sepolia.service -f -n 100 #follow logs of lighthouse-sepolia.service
```

During the synchonization, you are expected to get following log messages from`erigon`:

```bash
[INFO] [09-06|02:52:15.496] [4/12 Execution] Executed blocks         number=9421994 blk/s=112.3 tx/s=9589.1 Mgas/s=906.5 gasState=0.38 batch=246.4MB alloc=6.4GB sys=16.7GB
[INFO] [09-06|02:52:29.871] [] Flushed buffer file                   name=erigon-sortable-buf-4268134305
[INFO] [09-06|02:52:30.358] [] Flushed buffer file                   name=erigon-sortable-buf-140271917
[INFO] [09-06|02:52:30.405] [] Flushed buffer file                   name=erigon-sortable-buf-3356874711
[INFO] [09-06|02:52:43.500] Committed State                          gas reached=221060403580 gasTarget=549755813888 block=9423228 time=16.113674309s committedToDb=true
[INFO] [09-06|02:52:45.488] [4/12 Execution] Executed blocks         number=9423456 blk/s=48.7 tx/s=5474.7 Mgas/s=412.4 gasState=0.00 batch=3.1MB alloc=5.7GB sys=16.7GB
```

And `Lighthouse`:

{% code fullWidth="false" %}
```bash
Sep 06 01:05:36.659 INFO New block received                      root: 0x9bf6a56781caf6b6e57cb6a0cead5e9ada0c417a36d4dd3d6924d07e5993935b, slot: 9896726
Sep 06 01:05:41.000 WARN Head is optimistic                      execution_block_hash: 0x613050be274505439dda4867d07840bd2e2e6e9ba0cddd96aada49449861bbb2, info: chain not fully verified, block and attestation production disabled until execution engine syncs, service: slot_notifier
```
{% endcode %}

### Run _`curl`_ command in the terminal to check the status of your node

<pre class="language-bash"><code class="lang-bash"><strong>curl -H "Content-type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' http://localhost:8545
</strong></code></pre>

Expected output during synchronization:

{% code overflow="wrap" %}
```bash
{"jsonrpc":"2.0","id":1,"result":{"currentBlock":"0x0","highestBlock":"0x137477f","stages":[{"stage_name":"Snapshots","block_number":"0x137477f"},{"stage_name":"Headers","block_number":"0x137477f"},{"stage_name":"BorHeimdall","block_number":"0x0"},{"stage_name":"BlockHashes","block_number":"0x137477f"},{"stage_name":"Bodies","block_number":"0x137477f"},{"stage_name":"Senders","block_number":"0x137477f"},{"stage_name":"Execution","block_number":"0x90b383"},{"stage_name":"Translation","block_number":"0x0"},{"stage_name":"HashState","block_number":"0x0"},{"stage_name":"IntermediateHashes","block_number":"0x0"},{"stage_name":"AccountHistoryIndex","block_number":"0x0"},{"stage_name":"StorageHistoryIndex","block_number":"0x0"},{"stage_name":"LogIndex","block_number":"0x0"},{"stage_name":"CallTraces","block_number":"0x0"},{"stage_name":"TxLookup","block_number":"0x0"},{"stage_name":"Finish","block_number":"0x0"}]}}
```
{% endcode %}

When it returns `false` then your node is fully synchronized with the network

## References <a href="#references" id="references"></a>

{% embed url="https://github.com/erigontech/erigon" %}

{% embed url="https://github.com/sigp/lighthouse" %}

{% embed url="https://lighthouse-book.sigmaprime.io/intro.html" %}
