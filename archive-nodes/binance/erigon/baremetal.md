---
description: 'Authors: [Payne | Stake🦑Squid]'
---

# 💻 Baremetal

## System Requirements

<table><thead><tr><th align="center">CPU</th><th width="147" align="center">OS</th><th width="119" align="center">RAM</th><th align="center">Storage</th></tr></thead><tbody><tr><td align="center">8 Cores / 16 Threads</td><td align="center">Ubuntu 22.04</td><td align="center">>= 32GB</td><td align="center">>= 14 TiB NVMe SSD</td></tr></tbody></table>

BNB archive size on June 20th 2024 was 11TB

## Erigon 🦦

Official Docs [https://github.com/node-real/bsc-erigon](https://github.com/node-real/bsc-erigon)

### Snapshots

To download archive snapshots, please use the following: https://github.com/binance-chain/bsc-snapshots

### Install go

Download the Go programming language distribution archive, extracts it to the "/usr/local" directory, and then removes the downloaded archive, effectively installing Go version 1.22.4 on the system.

```bash
wget https://go.dev/dl/go1.22.4.linux-amd64.tar.gz && \
rm -rf /usr/local/go && \
tar -C /usr/local -xzf go1.22.4.linux-amd64.tar.gz && \
rm go1.22.4.linux-amd64.tar.gz
```

Please add the Go executable path to your system's `PATH` environment variable, and then test to ensure that Go is working correctly.

```bash
echo "export PATH="$PATH:/root/.foundry/bin:/usr/local/go/bin"" >> /root/.bashrc
source /root/.bashrc
go version #test
```

### Build Erigon

Clone the Erigon repository from GitHub, including its submodules, changes the current directory to the Erigon directory, checks out the latest release tag, and then compile the project using the "make" build system.

```bash
mkdir -p /root/data/github
cd /root/data/github/
git clone --recurse-submodules https://github.com/node-real/bsc-erigon
cd erigon
git checkout <latest release tag>
make
```

### Configure Erigon

Append a systemd service configuration for the Erigon BNB Mainnet Service to the "/etc/systemd/system/erigon.service" file, specifying its description, dependencies, and executable parameters for proper execution and monitoring.

```bash
[Unit]
Description=Erigon BSC Mainnet Service
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
WorkingDirectory=/root/data/bsc/erigon
ExecStart=/root/data/github/bsc-erigon/build/bin/erigon \
        --datadir=/root/data/bsc/erigon/datadir \
        --chain=bsc \
        --http \
        --http.addr=0.0.0.0 \
        --http.port=9656 \
        --http.compression \
        --http.vhosts=* \
        --http.corsdomain=* \
        --http.api=eth,debug,net,trace,web3,erigon \
        --private.api.addr=0.0.0.0:9092 \
        --ws --ws.compression \
        --metrics --metrics.addr=0.0.0.0 --metrics.port=9696 \
        --rpc.returndata.limit=1000000 \
        --db.pagesize=16kb \
        --db.size.limit=14TB \
        --snapshots=false
KillSignal=SIGTERM

[Install]
WantedBy=multi-user.target
```

### Run Erigon

Reload the systemd manager configuration, start the Erigon service, and enable it to start automatically on system boot, ensuring that the Erigon BNB Mainnet Service is active and will be automatically started upon system startup.

```bash
sudo systemctl daemon-reload
sudo systemctl start erigon
sudo systemctl enable erigon
```

### Monitor Logs

Use journalctl to display real-time log messages and continuously follow the log output of the Erigon service, allowing you to monitor its activity and troubleshoot any issues as they occur.

```bash
sudo journalctl -fu erigon
```
