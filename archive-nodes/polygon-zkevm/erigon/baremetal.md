# 💻 Baremetal

Author: \[ jLeopoldA ]

### System Requirements <a href="#system-requirements" id="system-requirements"></a>

| CPU    | OS                  | RAM  | DISK  |
| ------ | ------------------- | ---- | ----- |
| 4 Core | Ubunutu 24.04.1 LTS | 16GB | 128GB |

{% hint style="info" %}
The Polygon zkEVM archive node has a size of 103GB as of 1/15/2025
{% endhint %}

### Pre-Requisites <a href="#pre-requisites" id="pre-requisites"></a>

{% hint style="info" %}
CDK-Erigon requires the installation of Go.
{% endhint %}

### Update System

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -ye
```

### Set up Firewall <a href="#set-up-firewall" id="set-up-firewall"></a>

**Set Explicit Default Firewall Rules**

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

#### Allow SSH

```bash
sudo ufw allow 22/tcp
```

#### Allow Remote RPC Connections with Polygon Zkevm

```bash
sudo ufw allow 8545
```

#### Allow P2P Connections

```bash
sudo ufw allow 30303/tcp && sudo ufw allow 30303/udp
```

#### Enable Firewall

```bash
sudo ufw enable
```

#### Check Status / Current Rules of UFW

```bash
sudo ufw status verbose
```

### Install GO

#### Check for Latest Version of GO

```bash
# This will return the latest version of GO
curl -s https://go.dev/VERSION?m=text

# Example response
go1.23.3
time 2024-11-06T18:46:45Z
```

#### Download the Latest GO Tarball

```bash
# Downloading using the above example response
wget https://go.dev/d1/go1.23.3.linux-amd64.tar.gz

# Example command if the above version is different
wget https://go.dev/d1/VERSION.linux-amd64.tar.gz
```

#### Extract and Install GO

```bash
sudo tar -C /usr/local -xzf go.1.23.3.linux-amd64.tar.gz
```

#### Set Environment Variables

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

## Set up Polygon zkEVM with Erigon

### Clone the Polygon Hermez Repo for Erigon and Build Erigon

```bash
cd /root
git clone https://github.com/0xPolygonHermez/cdk-erigon
cd cdk-erigon/cmd/cdk-erigon
go build -o erigon
```

### Create Directory for DB

```bash
mkdir -p /var/lib/zkevm/db/
```

### Create YAML Configuration

#### Create Configuration Directory

```bash
mkdir /root/config/
nano /root/config/config.yaml
```

#### Paste and modify parameters. Save by entering ctrl+X and Y+ENTER&#x20;

```yaml
datadir: /var/lib/zkevm/db/
chain: hermez-mainnet
http: true
private.api.addr: localhost:9091
zkevm.l2-chain-id: 1101
zkevm.l2-sequencer-rpc-url: https://zkevm-rpc.com
zkevm.l2-datastreamer-url: stream.zkevm-rpc.com:6900
zkevm.l1-chain-id: 1
zkevm.l1-rpc-url: {YOUR_L1_RPC_URL_HERE}

zkevm.address-sequencer: "0x148Ee7dAF16574cD020aFa34CC658f8F3fbd2800"
zkevm.address-zkevm: "0x519E42c24163192Dca44CD3fBDCEBF6be9130987"
zkevm.address-rollup: "0x5132A183E9F3CB7C848b0AAC5Ae0c4f0491B7aB2"
zkevm.address-ger-manager: "0x580bda1e7A0CFAe92Fa7F6c20A3794F169CE3CFb"

zkevm.default-gas-price: 1000000000
zkevm.max-gas-price: 0
zkevm.gas-price-factor: 0.0375

zkevm.l1-rollup-id: 1
zkevm.l1-block-range: 20000
zkevm.l1-query-delay: 6000
zkevm.l1-first-block: 16896700
zkevm.datastream-version: 2

# debug.timers: true # Uncomment to enable timers

externalcl: true
http.port: 8545
http.api: [eth, debug, net, trace, web3, erigon, zkevm]
http.addr: 0.0.0.0
http.vhosts: any
http.corsdomain: any
ws: true
```

## Launch Erigon Node

### Create Systemd Service

```bash
sudo nano /etc/systemd/system/erigon.service
```

Paste the below configuration and save by entering ctrl+X and Y+ENTER

```bash
[Unit]
Description=Zkevm Node
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
WorkingDirectory=/root/cdk-erigon/
ExecStart=/root/cdk-erigon/cmd/cdk-erigon/erigon \
	--config="/root/config/config.yaml"
KillSignal=SIGTERM
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

### Start CDK-Erigon

```bash
sudo systemctl daemon-reload # Refresh after systemd configuration changes
sudo systemctl enable erigon.service # Enable erigon.service at start up
sudo systemctl start erigon.service # Starts erigon.service
sudo systemctl stop erigon.service # Stops erigon.service
sudo systemctl restart erigon.service # Restarts erigon.service
```

### View Logs for Debugging

```bash
journalctl -fu erigon.service -xe
```

#### Alternatively, you can view logs minus server name and time and receive the below

```bash
journalctl -fu erigon.service -o cat
```

<figure><img src="../../../.gitbook/assets/Screenshot from 2025-01-13 20-13-35.png" alt=""><figcaption></figcaption></figure>

### Query Polygon zkEVM Node

{% hint style="info" %}
CDK-Erigon does take time to process blocks before you can fully query it.
{% endhint %}

```bash
curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' http://localhost:8545

# Example response
{"jsonrpc":"2.0","id":1,"result":"0x40e2d5"}
```

### References <a href="#references" id="references"></a>

{% embed url="https://github.com/0xPolygonHermez/cdk-erigon" %}
