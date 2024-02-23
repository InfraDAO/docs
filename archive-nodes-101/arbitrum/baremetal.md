---
description: 'Authors: [Vince | Nodeify]'
---

# 💻 Baremetal

## System Requirements

<table><thead><tr><th align="center">CPU</th><th width="147" align="center">OS</th><th width="165" align="center">RAM</th><th align="center">Storage</th></tr></thead><tbody><tr><td align="center">4 + core CPU</td><td align="center">Ubuntu 22.04</td><td align="center">16GB+ for Nitro and 32GB+ for Classic</td><td align="center">>= 8 TiB NVMe SSD</td></tr></tbody></table>

## Offchain Labs ⛓️

Official Docs [https://docs.arbitrum.io/node-running/how-tos/running-an-archive-node](https://docs.arbitrum.io/node-running/how-tos/running-an-archive-node)

### Pre-requisites

Update, upgrade, and clean the system, and then install essential development tools (build-essential), firewall management (ufw), and the Git version control system.

**You must have a running Ethereum Mainnet RPC**

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt install -y git docker.io docker-compose aria2 clang cmake make \
	librocksdb-dev libboost-all-dev pkg-config libusb-1.0-0-dev \
	python3 python3-pip gperf libgoogle-perftools-dev autoconf \
	automake libgflags-dev libsnappy-dev zlib1g-dev libbz2-dev \
	liblz4-dev libzstd-dev libtool libudev-dev libssl-dev \
	libmpfr-dev libgmp-dev clang-format dtrx
```

Set explicit default UFW rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Allow SSH

```bash
sudo ufw allow 22/tcp
```

Allow remote RPC connections with Nitro Node

```bash
sudo ufw allow from ${REMOTE.HOST.IP} to any port 8546
```

{% hint style="danger" %}
Not advised to allow all or unknown IP address to RPC port
{% endhint %}

Enable Firewall

```bash
sudo ufw enable
```

### Install go

Download the Go programming language distribution archive, extracts it to the "/usr/local" directory, and then removes the downloaded archive, effectively installing Go version 1.20.6 on the system.

```bash
wget https://go.dev/dl/go1.20.6.linux-amd64.tar.gz && \
rm -rf /usr/local/go && \
tar -C /usr/local -xzf go1.20.6.linux-amd64.tar.gz && \
rm go1.20.6.linux-amd64.tar.gz
```

Please add the Go executable path to your system's `PATH` environment variable, and then test to ensure that Go is working correctly.

```bash
echo "export PATH=$PATH:/usr/local/go/bin:/root/.local/bin" >> /root/.bashrc
source /root/.bashrc
go version #test
```

### Install nvm

```bash
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash 
source /root/.bashrc
nvm -v #test 
```

### Install npm and yarn

```bash
nvm install 16.0.0 && npm install --global yarn
source /root/.bashrc
npm -v #test
yarn -v #test
```

### Build Nitro with Docker

Clone the Erigon repository from GitHub, including its submodules, changes the current directory to the Erigon directory, checks out the latest release tag, and then compile the project using the "make" build system.

```bash
git clone --recurse-submodules https://github.com/OffchainLabs/nitro/
cd nitro
docker build -t nitro .
```

### Copy Nitro binary from docker to `/root/nitro/build/bin`

```bash
docker cp $(docker run -it -d nitro):/usr/local/bin/nitro /root/nitro/build/bin/
/root/nitro/build/bin/nitro -h # test
```

### Download Nitro Snapshot

```bash
mkdir -p /root/.local/share/nitro/datadir
cd /root/.local/share/nitro/datadir
#Faster but requires double the storage
aria2c --file-allocation=none -c -x 10 -s 10 https://snapshot.arbitrum.foundation/arb1/nitro-archive.tar
dtrx -f nitro-archive.tar
#Saves space but much slower
wget -O - https://snapshot.arbitrum.foundation/arb1/nitro-archive.tar | tar -x -C /root/.local/share/nitro/datadir
```

### Create service to run Nitro Node

Change flag to your Ethereum RPC `--l1.url=1.2.3.4:8545`

<pre class="language-bash"><code class="lang-bash">sudo echo "[Unit]
Description=Arbitrum Nitro Service
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
        --node.caching.archive \
        --persistent.chain=/root/.local/share/nitro/datadir/ \
        --persistent.global-config=/root/.local/share/nitro/ \
        --l1.url=<a data-footnote-ref href="#user-content-fn-1">{YOUR_L1_RPC}</a> \
        --l2.chain-id=42161 \
        --http.api=net,web3,eth,debug \
        --http.corsdomain=* \
        --http.addr=0.0.0.0 \
        --http.port=8546 \
        --node.rpc.classic-redirect=http://0.0.0.0:8547 \
        --http.vhosts=* \
        --log-level=3

KillSignal=SIGHUP

[Install]
WantedBy=multi-user.target" >> /etc/systemd/system/nitro.service
</code></pre>

### Run Nitro

Reload the systemd manager configuration, start the Nitro service, and enable it to start automatically on system boot, ensuring that the Nitro Service is active and will be automatically started upon system startup.

```bash
sudo systemctl daemon-reload
sudo systemctl start nitro
sudo systemctl enable nitro
```

### Monitor Logs

Use journalctl to display real-time log messages and continuously follow the log output of the Nitro service, allowing you to monitor its activity and troubleshoot any issues as they occur.

```bash
sudo journalctl -fu nitro
```





[^1]: Required L1 RPC URL
