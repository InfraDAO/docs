# 💻 Baremetal

Authors: \[man4ela | catapulta.eth]

## System Requirements

<table><thead><tr><th align="center">CPU</th><th align="center">OS</th><th width="254" align="center">RAM</th><th align="center">DISK</th></tr></thead><tbody><tr><td align="center">8+ cores CPU</td><td align="center">Debian 12/Ubuntu 22.04</td><td align="center">=> 16 GB RAM</td><td align="center"><p>=1TB+</p><p> (NVMe)</p></td></tr></tbody></table>

{% hint style="info" %}
_The BSC Testnet Chapel archive node has a size of 463GB on March 17th, 2025_
{% endhint %}

## Setup of BSC Erigon

{% hint style="warning" %}
This guide covers the installation of `BSC Erigon V3`, which is Erigon V3's fork managed by Node-Real. It has built-in snapshot download function and allows to sync BSC Testnet Chapel archive Node in effecteively 48 hours. It is important to keep in mind that the client is still under development and you need to be prepared to face difficulties during synchronization or other techincal issues
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
sudo ufw allow 30303 #p2p port
sudo ufw allow 42069 #torrent port
```

Allow remote RPC connections with the Node

```bash
sudo ufw allow from ${REMOTE.HOST.IP} to any port 8545 #http api port
sudo ufw allow from ${REMOTE.HOST.IP} to any port 8546 #ws port
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
Go version 1.22+ is required
{% endhint %}

```bash
sudo wget https://go.dev/dl/go1.22.4.linux-amd64.tar.gz && sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.22.4.linux-amd64.tar.gz && rm go1.22.4.linux-amd64.tar.gz

echo 'export PATH=$PATH:/usr/local/go/bin:/root/.local/bin' >> /root/.bashrc

source /root/.bashrc

#verify Go installation
go version
```

### Build Erigon RPC Node

```bash
git clone --recurse-submodules https://github.com/node-real/bsc-erigon.git

cd bsc-erigon
```

#### Check for the latest actual release at [https://github.com/node-real/bsc-erigon/releases](https://github.com/node-real/bsc-erigon/releases)

This guide has been tested and successfully synced the node with _**v1.3.2-beta2**_:

```bash
git checkout v1.3.2-beta2

make erigon
```

#### Create Data directory and jwt secret file

```bash
cd ..

mkdir -p /root/data/bsc-erigon/

openssl rand -hex 32 | tr -d "\n" > /root/data/bsc-erigon/jwt.hex
```

#### Create Systemd service for BSC Erigon

```bash
sudo nano /etc/systemd/system/bsc-erigon.service
```

Paste the configs and save by entering `ctrl+X` and `Y+ENTER`:

{% hint style="danger" %}
Replace {IP} with the actual address of the server in order to explicitly set the external IP address that the node should advertise to peers in a P2P network
{% endhint %}

```bash
[Unit]
Description=BSC Chapel Erigon Service
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
LimitNOFILE=500000

WorkingDirectory=/root/chapel/bsc-erigon/
ExecStart=/root/chapel/bsc-erigon/build/bin/erigon \
        --private.api.addr=127.0.0.1:9090 \
        --chain chapel \
        --prune.mode=archive \
        --torrent.download.rate=1g \
        --torrent.download.slots=400 \
        --batchSize=2g \
        --bsc.blobSidecars.no-pruning=true \
        --txpool.disable \
        --metrics \
        --metrics.addr=0.0.0.0 \
        --metrics.port=6060 \
        --pprof \
        --pprof.addr=0.0.0.0 \
        --pprof.port=6061 \
        --authrpc.jwtsecret=/root/data/bsc-erigon/jwt.hex \
        --authrpc.port=8551 \
        --datadir=/root/data/bsc-erigon/ \
        --http.addr=0.0.0.0 \
        --http.port=8545 \
        --http.api=eth,debug,net,trace,web3,erigon,bsc,admin \
        --http.vhosts=any \
        --http.corsdomain=* \
        --ws \
        --ws.port=8546 \
        --torrent.port=42069 \
        --nat=extip:{IP} \
        --db.pagesize=16k \
        --db.size.limit=4t \
        --port=30303
KillSignal=SIGTERM

[Install]
WantedBy=multi-user.target
```

#### Launch BSC Erigon

```bash
sudo systemctl daemon-reload #refresh systemd configuration when changes made

sudo systemctl enable bsc-erigon.service #enable bsc-erigon.service at system startup

sudo systemctl start bsc-erigon.service #start bsc-erigon.service

sudo nano /etc/systemd/system/bsc-erigon.service #make changes in bsc-erigon.service file
```

### Monitor the logs for errors

```bash
journalctl -u bsc-erigon.service -f -n 100 #follow logs of bsc-erigon.service
```

#### During the initialization, first you are expected to see logs of a snapshot download process:

```bash
[INFO] [03-14|03:34:50.959] [1/9 OtterSync] Downloading              progress="(3639/3639 files) 32.14% - 132.0GB/410.5GB" time-left=1hrs:25m total-time=33m0s download-rate=55.6MB/s completion-rate=55.6MB/s alloc=7.3GB sys=16.2GB
[INFO] [03-14|03:34:50.960] [p2p] GoodPeers                          eth68=5
[INFO] [03-14|03:34:51.027] [snapshots] no progress yet              files=282 list=idx/v1-logtopics.256-288.ef,accessor/v1-tracesfrom.128-192.efi,history/v1-code.0-64.v,accessor/v1-receipt.0-64.vi,domain/v1-code.300-302.bt,...
[INFO] [03-14|03:34:52.303] [mem] memory stats                       Rss=77.8GB Size=0B Pss=77.8GB SharedClean=3.6MB SharedDirty=0B PrivateClean=67.9GB PrivateDirty=9.9GB Referenced=73.4GB Anonymous=9.6GB Swap=117.0MB alloc=7.4GB sys=16.2GB
[INFO] [03-14|03:35:10.959] [1/9 OtterSync] Downloading              progress="(3639/3639 files) 32.39% - 132.9GB/410.5GB" time-left=1hrs:32m total-time=33m20s download-rate=51.0MB/s completion-rate=51.1MB/s alloc=7.7GB sys=16.2GB
[INFO] [03-14|03:35:11.055] [snapshots] no progress yet              files=281 list=domain/v1-storage.288-296.bt,accessor/v1-receipt.288-296.efi,accessor/v1-accounts.64-128.efi,domain/v1-commitment
```

{% hint style="danger" %}
If the download progress shows 0% for too long, try restarting the client. Once the snapshot has been fully downloaded and applied by the client, the node will start syncing process and will reach a chainhead in under 48 hrs
{% endhint %}

#### Run _`curl`_ command in the terminal to check the status of your node <a href="#run-curl-command-in-the-terminal-to-check-the-status-of-your-node" id="run-curl-command-in-the-terminal-to-check-the-status-of-your-node"></a>

```bash
curl -H "Content-type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' http://localhost:8545
```

When it returns `false` , your node is fully synchronized with the network

### References <a href="#references" id="references"></a>

{% embed url="https://github.com/node-real/bsc-erigon/issues/441" %}

{% embed url="https://docs.bnbchain.org/bnb-smart-chain/developers/node_operators/archive_node/" %}
