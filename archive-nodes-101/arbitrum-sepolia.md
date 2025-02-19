---
description: 'Authors: [man4ela | catapulta.eth]'
---

# 🔼 Arbitrum Sepolia

### System Requirements <a href="#system-requirements" id="system-requirements"></a>

| CPU          | OS                     | RAM          | DISK               |
| ------------ | ---------------------- | ------------ | ------------------ |
| 8+ cores CPU | Debian 12/Ubuntu 22.04 | => 16 GB RAM | =6TB+(SSD or NVMe) |

{% hint style="info" %}
_The Arbitrum Sepolia archive node has a size of 5.2TB on November 18th, 2024_
{% endhint %}

{% hint style="warning" %}
Before you start, make sure that you have your own synced Ethereum Sepolia RPC URL (e.g. Erigon) and Consensus Layer Beacon endpoint (e.g. Lighthouse) ready.
{% endhint %}

## Pre-Requisites

{% code overflow="wrap" %}
```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y

sudo apt install -y git make wget aria2 gcc pkg-config libusb-1.0-0-dev libudev-dev jq gcc g++ curl libssl-dev screen apache2-utils build-essential pkg-config
```
{% endcode %}

### Install Docker

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

# Install Docker Packages
 sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
 
 # Verify Docker Installation is Successful
sudo docker run hello-world
```

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
sudo ufw allow from ${REMOTE.HOST.IP} to any port 9545
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

<table><thead><tr><th width="115">Dependency</th><th width="110" align="center">Version</th><th width="233">Version Check Command</th></tr></thead><tbody><tr><td><mark style="color:green;">go</mark></td><td align="center"><code>^1.21</code></td><td><code>go version</code></td></tr><tr><td><mark style="color:orange;">node</mark></td><td align="center"><code>^20</code></td><td><code>node --version</code></td></tr><tr><td><mark style="color:blue;">pnpm</mark></td><td align="center"><code>^8</code></td><td><code>pnpm --version</code></td></tr><tr><td><mark style="color:green;">foundry</mark></td><td align="center"><code>^0.2.0</code></td><td><code>forge --version</code></td></tr><tr><td><mark style="color:orange;">make</mark></td><td align="center"><code>^4</code></td><td><code>make --version</code></td></tr><tr><td><mark style="color:green;">yarn</mark></td><td align="center"><code>1.22.21</code></td><td><code>yarn --version</code></td></tr><tr><td><mark style="color:blue;">nvm</mark></td><td align="center"><code>0.39.3</code></td><td><code>nvm --verison</code></td></tr></tbody></table>

### Install GO

{% code overflow="wrap" fullWidth="false" %}
```bash
sudo wget https://go.dev/dl/go1.21.6.linux-amd64.tar.gz && sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.21.6.linux-amd64.tar.gz && rm go1.21.6.linux-amd64.tar.gz

echo 'export PATH=$PATH:/usr/local/go/bin:/root/.local/bin' >> /root/.bashrc

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

<pre class="language-bash"><code class="lang-bash">source /root/.bashrc
<strong>
</strong><strong>foundryup
</strong></code></pre>

### Install node and yarn

```bash
nvm install 18.12.0 && npm install --global yarn && nvm use 18.12.0 && npm -g install pnpm

source /root/.bashrc
```

### Check if go and all dependancies are installed

```bash
go version
nvm -v
npm -v
yarn -v
pnpm -v
```

## Build the Nitro Node

```bash
git clone --branch v3.2.1 https://github.com/OffchainLabs/nitro.git

cd nitro

git submodule update --init --recursive --force

docker build . --tag nitro-node
```

To upgrade `nitro` check for latest version at [https://github.com/OffchainLabs/nitro/releases](https://github.com/OffchainLabs/nitro/releases):

```bash
#Copy Nitro binary from docker to /root/nitro/build/bin

docker pull offchainlabs/nitro-node:v3.2.1-d81324d

docker run -d --name nitro offchainlabs/nitro-node:v3.2.1-d81324d

docker cp nitro:/usr/local/bin/nitro /root/nitro/build/bin/
```

**Create Data directory and download latest snapshot**

```bash
cd

screen -S snapshot #start a screen session named snapshot to download a db archive for nitro:

mkdir snapshot && cd snapshot

#check for actual snapshot here 
https://snapshot-explorer.arbitrum.io/

#Download snapshot parts
aria2c -Z -x 16 "https://snapshot.arbitrum.io/sepolia-rollup/2024-11-03-4398c4dd/archive.tar.part0" "https://snapshot.arbitrum.io/sepolia-rollup/2024-11-03-4398c4dd/archive.tar.part1" "https://snapshot.arbitrum.io/sepolia-rollup/2024-11-03-4398c4dd/archive.tar.part2" "https://snapshot.arbitrum.io/sepolia-rollup/2024-11-03-4398c4dd/archive.tar.part3" "https://snapshot.arbitrum.io/sepolia-rollup/2024-11-03-4398c4dd/archive.tar.part4" "https://snapshot.arbitrum.io/sepolia-rollup/2024-11-03-4398c4dd/archive.tar.part5" "https://snapshot.arbitrum.io/sepolia-rollup/2024-11-03-4398c4dd/archive.tar.part6" "https://snapshot.arbitrum.io/sepolia-rollup/2024-11-03-4398c4dd/archive.tar.part7" "https://snapshot.arbitrum.io/sepolia-rollup/2024-11-03-4398c4dd/archive.tar.part8" "https://snapshot.arbitrum.io/sepolia-rollup/2024-11-03-4398c4dd/archive.tar.part9" "https://snapshot.arbitrum.io/sepolia-rollup/2024-11-03-4398c4dd/archive.tar.part10"

#To quit a session window during download progress use ctrl A+D and screen -r snapshot to attach again

#extract downloaded archive parts
cat archive.tar.part0 archive.tar.part1 archive.tar.part2 archive.tar.part3 archive.tar.part4 archive.tar.part5 archive.tar.part6 archive.tar.part7 archive.tar.part8 archive.tar.part9 archive.tar.part10 | tar -xvf -

mkdir -p /root/.local/share/nitro/datadir/nitro/nitro

#move contents into data directory:

mv arbitrumdata l2chaindata keystore nodes LOCK /root/.local/share/nitro/datadir/nitro/nitro
```

#### Create Systemd service for Nitro

```bash
sudo nano /etc/systemd/system/nitro-sepolia.service
```

Paste the configs and save by entering `ctrl+X` and `Y+ENTER`:

```bash
[Unit]
Description=Arbitrum Sepolia Nitro Service
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
WorkingDirectory=/root/nitro
ExecStart=/root/nitro/build/bin/nitro \
        --execution.caching.archive \
        --persistent.chain=/root/.local/share/nitro/datadir/nitro \
        --persistent.global-config=/root/.local/share/nitro/datadir \
        --parent-chain.connection.url={ETH SEPOLIA URL} \
        --chain.id=421614 \
        --http.api=net,web3,eth,debug \
        --http.corsdomain=* \
        --http.addr=0.0.0.0 \
        --http.port=9545 \
        --execution.rpc.gas-cap=0 \
        --http.vhosts=* \
        --log-level=3 \
        --parent-chain.blob-client.beacon-url={ETH SEPOLIA CL URL} \
        --validation.wasm.allowed-wasm-module-roots \
        --ws.addr=0.0.0.0 \
        --ws.port=9658 \
        --ws.api=net,web3,eth,debug \
        --ws.origins=*
KillSignal=SIGINT

[Install]
WantedBy=multi-user.target
```

{% hint style="info" %}
Replace `{ETH SEPOLIA URL}` and `{ETH SEPOLIA CL URL}` with your synced Ethereum Sepolia and Ethereum Sepolia Consensus Layer endpoints
{% endhint %}

#### Launch Nitro

```bash
sudo systemctl daemon-reload #refresh systemd configuration when changes made

sudo systemctl enable nitro-sepolia #enable nitro-sepolia.service at system startup

sudo systemctl start nitro-sepolia #start nitro-sepolia.service

sudo systemctl stop nitro-sepolia #stop nitro-sepolia.service

sudo nano /etc/systemd/system/nitro-sepolia.service #make changes in nitro-sepolia.service file
```

### Monitor the logs for errors

```bash
journalctl -u nitro-sepolia.service -f -n 100 #follow logs of nitro-sepolia.service
```

### Run _`curl`_ command in the terminal to check the status of your node

<pre class="language-bash"><code class="lang-bash"><strong>curl -H "Content-type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' http://localhost:9545
</strong></code></pre>

Expected output during synchronization:

{% code overflow="wrap" %}
```bash
{"jsonrpc":"2.0","id":1,"result":{"batchProcessed":346862,"batchSeen":346862,"blockNum":98302890,"consensusSyncTarget":98303169,"feedPendingMessageCount":0,"messageOfLastBlock":98302890,"messageOfProcessedBatch":98302538,"msgCount":98303174,"syncTargetMsgCount":98303169}}
```
{% endcode %}

When it returns `false` then your node is fully synchronized with the network

### References <a href="#references" id="references"></a>

{% embed url="https://github.com/OffchainLabs/nitro/releases" %}

{% embed url="https://docs.arbitrum.io/run-arbitrum-node/nitro/build-nitro-locally" %}
