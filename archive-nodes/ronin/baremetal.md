---
description: 'Author: [ jleopoldA ]'
---

# 💻 Baremetal

## System Requirements

| CPU    | OS                 | RAM    | DISK    |
| ------ | ------------------ | ------ | ------- |
| 8 Core | Ubuntu 24.04.1 LTS | 32 GB  | 7TB SSD |

{% hint style="info" %}
Ronin has a size of 5.8TB as of 1/15/2025.
{% endhint %}

## Pre-Requisites

{% hint style="info" %}
Ronin requires the installation of Go.
{% endhint %}

### Update System

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y
```

### Set up Firewall

#### Set Explicit Default Firewall Rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

#### Allow SSH

```bash
sudo ufw allow 22/tcp
```

#### Allow Remote RPC Connections with Ronin Node

```bash
sudo ufw allow 8545
sudo ufw allow 8546
```

#### Allow P2P Connections

```bash
sudo ufw allow 30303/tcp && sudo ufw allow 30303/udp
```

#### Enable Firewall

```bash
sudo ufw enable
```

#### To Check Status / Current Rules of UFW&#x20;

```bash
sudo ufw status verbose
```

### Install GO

{% hint style="info" %}
This step is necessary if GO is not installed.
{% endhint %}

#### Check for latest version of GO

```bash
# This will return the latest version of GO
curl -s https://go.dev/VERSION?m=text

# Example response
go1.23.3
time 2024-11-06T18:46:45Z
```

#### Download the latest GO Tarball

```bash
# Downloading using the above example response
wget https://go.dev/dl/go1.23.3.linux-amd64.tar.gz

# Example command if the above version is different
wget https://go.dev/dl/VERSION.linux-amd64.tar.gz
```

#### Extract and Install GO

```bash
sudo tar -C /usr/local -xzf go1.23.3.linux-amd64.tar.gz 
```

#### Set Up Environment Variables

```bash
echo "export PATH=\$PATH:/usr/local/go/bin" >> ~/.bashrc
source ~/.bashrc
```

#### Check Installation

```bash
go version

# Example response
go version go1.23.3 linux/amd64
```

## Install Ronin

### Clone the Ronin Repo

```bash
cd /root

# The below command will create a directory called 'ronin'
# The path to it will be /root/ronin
git clone https://github.com/axieinfinity/ronin
```

### Build Ronin

```bash
cd /ronin/cmd/ronin
go build -o ronin
```

### Initialize Ronin Genesis Block

{% hint style="info" %}
Ronin requires the initialization of its Genesis Block before being run.
{% endhint %}

```bash
./ronin init --datadir /opt/ronin /root/ronin/genesis/mainnet.json
```

## Create System Service

```bash
# Copy and paste the code below and run it within your terminal.
sudo echo "[Unit]
Description=Ronin Node
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
WorkingDirectory=/root/ronin/
ExecStart=/root/ronin/cmd/ronin/ronin \
	--gcmode archive --syncmode full \
	--http --http.addr 0.0.0.0 --http.api eth,net,web3 --http.port 8545 \
	--ws --ws.addr 0.0.0.0 --ws.port 8546 --ws.api eth,net,web3 \
	--datadir /opt/ronin \
        --port 30303 --networkid 2020 \
	--discovery.dns enrtree://AIGOFYDZH6BGVVALVJLRPHSOYJ434MPFVVQFXJDXHW5ZYORPTGKUI@nodes.roninchain.com
Restart=on-failure
LimitNOFILE=1000000
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/ronin.service
```

## Run Ronin Node

```bash
sudo systemctl daemon-reload # Refresh after systemd configuration changes
sudo systemctl enable ronin.service # Enable ronin.service at start up
sudo systemctl start ronin.service # Starts ronin.service
sudo systemctl stop ronin.service # Stops ronin.service
sudo systemctl restart ronin.service # Restarts ronin.service
```

### View Logs for Debugging

```bash
journactl -fu ronin.service -xe
```

### Query Ronin Node

```bash
curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' http://localhost:8545

# Example response
{"jsonrpc":"2.0","id":1,"result":"0x40e2d5"}
```

## References

{% embed url="https://github.com/axieinfinity/ronin" %}

{% embed url="https://docs.roninchain.com/protocol/validators/setup/overview" %}
