# 💻 Baremetal

Authors: \[man4ela | catapulta.eth]

### System Requirements <a href="#system-requirements" id="system-requirements"></a>

| CPU          | OS                     | RAM          | DISK                  |
| ------------ | ---------------------- | ------------ | --------------------- |
| 8+ cores CPU | Debian 12/Ubuntu 22.04 | => 16 GB RAM | 4TB+ (NVME preffered) |

_The Blast archive node has a size of 3.9TB on January 10th, 2025_

## Blast

{% hint style="success" %}
Blast is a fork of Optimism’s open-source [OP Stack](https://stack.optimism.io/). In this guide, Blast forked Optimism's `op-geth` and `op-node`binaries are built from source to facilitate the node's installation.
{% endhint %}

{% hint style="warning" %}
Before you start, make sure that you have your own synced Ethereum L1 RPC URL (e.g. Erigon) and L1 Consensus Layer Beacon endpoint (e.g. Lighthouse) ready.
{% endhint %}

## Pre-Requisites



```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y

sudo apt install -y git make wget gcc pkg-config libusb-1.0-0-dev libudev-dev jq gcc g++ curl libssl-dev screen apache2-utils build-essential pkg-config
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

Allow remote RPC connections with Blast Node

```bash
sudo ufw allow from ${REMOTE.HOST.IP} to any port 8545
```

{% hint style="warning" %}
Not advised to allow all or unknown IP address to RPC port
{% endhint %}

Allow P2P Connections

```bash
sudo ufw allow 30304/tcp
sudo ufw allow 30304/udp
```

Enable Firewall

```bash
sudo ufw enable
```

To check the status of UFW and see the current rules

```bash
sudo ufw status verbose
```

## Install dependencies

#### Required Software Dependencies

<table><thead><tr><th width="154">Dependency</th><th width="110" align="center">Version</th><th width="233">Version Check Command</th></tr></thead><tbody><tr><td><mark style="color:green;">go</mark></td><td align="center"><code>^1.21</code></td><td><code>go version</code></td></tr><tr><td><mark style="color:orange;">node</mark></td><td align="center"><code>^20</code></td><td><code>node --version</code></td></tr><tr><td><mark style="color:blue;">pnpm</mark></td><td align="center"><code>^8</code></td><td><code>pnpm --version</code></td></tr><tr><td><mark style="color:green;">foundry</mark></td><td align="center"><code>^0.2.0</code></td><td><code>forge --version</code></td></tr><tr><td><mark style="color:orange;">make</mark></td><td align="center"><code>^4</code></td><td><code>make --version</code></td></tr><tr><td><mark style="color:green;">yarn</mark></td><td align="center"><code>1.22.21</code></td><td><code>yarn --version</code></td></tr><tr><td><mark style="color:blue;">nvm</mark></td><td align="center"><code>0.39.3</code></td><td><code>nvm --verison</code></td></tr></tbody></table>

### Install GO

```bash
sudo wget https://go.dev/dl/go1.21.6.linux-amd64.tar.gz && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf go1.21.6.linux-amd64.tar.gz && \
rm go1.21.6.linux-amd64.tar.gz

# Verify the installation
/usr/local/go/bin/go version
```

### Install nvm

```bash
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
```

### Download foundry

```bash
curl -L https://foundry.paradigm.xyz | bash
```

### Install foundry

```bash
foundryup

source /root/.bashrc
```

### Install node and yarn

```bash
nvm install 18.12.0 && npm install --global yarn && nvm use 18.12.0 && npm -g install pnpm

source /root/.bashrc
```

### Check if go and all dependencies are installed

```bash
go version
nvm -v
npm -v
yarn -v
pnpm -v
```

### Create directories

```bash
mkdir -p /root/data/blast/geth/blast-geth
mkdir -p /root/data/blast/geth/blast-optimism
```

## Compile Blast-geth

```bash
git clone https://github.com/blast-io/blast.git

cd blast/blast-geth

git checkout tags/v1.1.0 -b v1.1.0

make
```

_#The binary is built at /root/blast/blast-geth/build/bin/geth_

#### Create JWT secret file and download genesis and rollup .json files

```bash
cd /root/data/blast/geth/blast-geth

openssl rand -hex 32 | tr -d "\n" > /root/data/blast/geth/blast-geth/jwt.hex

cd /root/blast

git clone https://github.com/blast-io/deployment.git #contains genesis and rollup configs
```

### Create systemd service

```bash
sudo echo "[Unit]
Description=Blast-geth Service
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
WorkingDirectory=/root/blast/blast-geth/build/bin/
ExecStart=/root/blast/blast-geth/build/bin/geth \
  --datadir=/root/data/blast/geth/blast-geth/ \
  --http \
  --http.corsdomain=* \
  --http.vhosts=* \
  --http.addr=0.0.0.0 \
  --http.port=8545 \
  --http.api=web3,debug,eth,txpool,net,engine \
  --ws \
  --ws.addr=0.0.0.0 \
  --ws.port=8546 \
  --ws.origins=* \
  --ws.api=debug,eth,txpool,net,engine \
  --authrpc.addr=0.0.0.0 \
  --authrpc.port=8551 \
  --authrpc.vhosts=* \
  --authrpc.jwtsecret=/root/data/blast/geth/blast-geth/jwt.hex \
  --syncmode=full \
  --gcmode=archive \
  --networkid=81457 \
  --nodiscover \
  --maxpeers=0 \
  --rollup.disabletxpoolgossip=true \
  --override.canyon=0 \
  --override.ecotone=1716843599 \
  --rollup.sequencerhttp=https://sequencer.blast.io

KillSignal=SIGTERM

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/blast-geth.service

```

Bootstrap the node by running

`/root/blast/blast-geth/build/bin/geth --datadir /root/data/blast/geth/blast-geth/ init /root/blast/deployment/mainnet/genesis.json`

### Start blast-geth

<pre class="language-bash"><code class="lang-bash">sudo systemctl daemon-reload #refresh systemd configuration when changes made
<strong>
</strong>sudo systemctl start blast-geth.service #start blast-geth

sudo systemctl enable blast-geth.service #enable blast-geth service at system startup

sudo journalctl -fu blast-geth.service #follow logs of blast-geth service
</code></pre>



{% hint style="info" %}
To check or modify `blast-geth.service` parameters simply run&#x20;

`sudo nano /etc/systemd/system/blast-geth.service`

Ctrl+X and Y to save changes
{% endhint %}

#### You can run _`curl`_ command in the terminal to check the status of your node

```bash
curl -d '{"id":0,"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["latest",false]}' \
  -H "Content-Type: application/json" http://localhost:8545
```

## Compile Op-node (Blast-optimism)

```bash
cd blast/blast-optimism

make op-node
```

#### Create systemd service

{% hint style="warning" %}
Make sure to replace `--l1` and `--l1.beacon` flags with your own synced Ethereum L1 RPC URL (e.g. Erigon) and L1 Consensus Layer Beacon endpoint (e.g. Lighthouse)
{% endhint %}

```bash
echo "[Unit]
Description=Blast-optimism node Service
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
WorkingDirectory=/root/blast/blast-optimism/op-node/bin/
ExecStart=/root/blast/blast-optimism/op-node/bin/op-node \
    --l1={L1 RPC URL} \
    --l1.rpckind=any \
    --l1.beacon={L1 BEACON RPC URL} \
    --l2=http://0.0.0.0:8551 \
    --l2.jwt-secret=/root/data/blast/blast-geth/jwt.hex \
    --rollup.config=/root/blast/deployment/mainnet/rollup.json \
    --p2p.listen.tcp=30304 \
    --p2p.listen.udp=30304 \
    --p2p.bootnodes=enr:-J64QGwHl9uYLfC_cnmxSA6wQH811nkOWJDWjzxqkEUlJoZHWvI66u-BXgVcPCeMUmg0dBpFQAPotFchG67FHJMZ9OSGAY3d6wevgmlkgnY0gmlwhANizeSHb3BzdGFja4Sx_AQAiXNlY3AyNTZrMaECg4pk0cskPAyJ7pOmo9E6RqGBwV-Lex4VS9a3MQvu7PWDdGNwgnZhg3VkcIJ2YQ,enr:-J64QDge2jYBQtcNEpRqmKfci5E5BHAhNBjgv4WSdwH1_wPqbueq2bDj38-TSW8asjy5lJj1Xftui6Or8lnaYFCqCI-GAY3d6wf3gmlkgnY0gmlwhCO2D9yHb3BzdGFja4Sx_AQAiXNlY3AyNTZrMaEDo4aCTq7pCEN8om9U5n_VyWdambGnQhwHNwKc8o-OicaDdGNwgnZhg3VkcIJ2YQ \

KillSignal=SIGTERM

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/blast-optimism.service

```

### Start blast-optimism

```bash
sudo nano /etc/systemd/system/blast-optimism.service #make changes in blast-optimism service file

sudo systemctl daemon-reload #refresh systemd configuration when changes made

sudo systemctl enable blast-optimism.service #enable blast-optimism service at system startup

sudo systemctl start blast-optimism.service #start blast-optimism
```

### Monitor the logs for errors

```bash
sudo journalctl -fu blast-optimism.service #follow logs of blast-optimism service

sudo journalctl -fu blast-geth.service #follow logs of blast-geth service
```

#### Run _`curl`_ command in the terminal to check the status of your node

```bash
curl -d '{"id":0,"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["latest",false]}' \
  -H "Content-Type: application/json" http://localhost:8545
```

{% embed url="https://github.com/blast-io/deployment" %}

{% embed url="https://docs.optimism.io/builders/node-operators/tutorials/node-from-source" %}

### References
