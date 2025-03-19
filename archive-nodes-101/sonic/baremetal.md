---
description: 'Author: [ Godwin]'
---

# 💻 Baremetal

System Requirements

| CPU        | OS                 | RAM        | DISK |
| ---------- | ------------------ | ---------- | ---- |
| 4 -8 Cores | Ubuntu 22.04.4 LTS | 34 - 128GB | 1TB  |

{% hint style="info" %}
The Sonic Archive Node has a size of 590GB as of 3/10/2025.
{% endhint %}

## Pre-Requisites

#### Update System

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y
sudo apt install -y git gcc make --fix-missing
```

#### Install GO

{% hint style="warning" %}
Sonic requires go version 1.22 or greater to run
{% endhint %}

```bash
# Remove previous installation of GO
rm -rf /usr/local/go # For GO installations locacated within /usr/local/go
rm -rf /usr/local/bin/go # For GO installations located within /usr/local/bin/go

# Download GO
wget https://go.dev/dl/go1.24.1.linux-amd64.tar.gz

# Extract and place within /usr/local
tar -xzf go1.24.0.linux-amd64.tar.gz -C /usr/local && rm go1.24.0.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc
```

## Firewall Configuration

{% hint style="info" %}
The system firewall must allow TCP and UDP connections from/to port 5050.
{% endhint %}

#### Set Explicit Firewall Configuration

```bash
sudo ufw default deny incoming && sudo ufw default allow outgoing
```

#### Allow SSH

```bash
sudo ufw allow 22/tcp
```

#### Allow Connections for sonic ports

```bash
sudo ufw allow 80
sudo ufw allow 443
sudo ufw allow 18545
sudo ufw allow 18546
sudo ufw allow 5050/tcp
sudo ufw allow 5050/udp
```

#### Enable Firewall Rules

```bash
sudo ufw enable
```

#### Check Status of Firewall Rules (UFW)

```bash
sudo ufw status verbose
```

## Build Sonic Client

Create directory to store data and code repository

```bash
mkdir sonic-index && cd sonic-index

mkdir sonic-data
```

Download the Sonic source code from the following GitHub repository.

```bash
git clone https://github.com/0xsoniclabs/Sonic.git
cd Sonic
```

Switch to the most recent Sonic release.

```bash
git fetch --tags && git checkout -b v2.0.1 tags/v2.0.1
```

Build the Sonic binary using the provided configuration.

```bash
make all
```

Transfer the new binaries to the bin folder for system-wide access.

```bash
sudo cp build/sonic* /usr/local/bin/
```

### Prime Sonic State DB <a href="#id-2.-prime-sonic-state-db" id="id-2.-prime-sonic-state-db"></a>

Download the most recent network archive genesis file for the [**Sonic mainnet or Blaze testnet**](https://genesis.soniclabs.com/)**.**

```bash
wget https://genesis.soniclabs.com/sonic-mainnet/genesis/sonic.g
```

The genesis file will be used to prime your local state database and will allow you to join the network and synchronize with it. Please check the downloaded genesis file using the provided checksum.

```bash
wget https://genesis.soniclabs.com/sonic-mainnet/genesis/sonic.g.md5
md5sum --check sonic.g.md5
```

The expected output is `sonic.g: OK`.

#### Prime Sonic Database <a href="#prime-sonic-database" id="prime-sonic-database"></a>

Use the sonictool app (created during the building process as build/sonictool) to prime a validated archive state database for the Sonic client. Start the genesis expansion.

```
GOMEMLIMIT=50GiB sonictool --datadir sonic-data --cache 12000 genesis <genesis path>
```

The last step of the genesis processing is the state validation. Please double-check that the output log contains the following messages with details about the verified state:

```
StateDB imported successfully, stateRoot matches module=gossip-store index=1 root="1ccba39...9ea2be"
```

## Create System Services

```bash
sudo nano /etc/systemd/system/sonic.service
```

#### Copy and past the config settings

```bash
[Unit]
Description=Sonic Archival Node
After=network.target

[Service]
Type=simple
Restart=always
RestartSec=5
TimeoutSec=900
User=root
ExecStart=/usr/local/bin/sonicd \
        --datadir /root/sonic-index/sonic-data \
        --cache 12000 --nat extip:<your-node-external-ip> \
        --http \
        --http.addr=0.0.0.0 \
        --http.port=18545 \
        --http.corsdomain=* \
        --http.vhosts=* \
        --http.api=eth,web3,net,ftm,txpool,abft,dag,trace,debug \
        --ws \
        --ws.addr=0.0.0.0 \
        --ws.port=18546 \
        --ws.origins=* \
        --ws.api=eth,web3,net,ftm,txpool,abft,dag,trace,debug
Environment="GOMEMLIMIT=90GiB"
LimitNOFILE=200000

KillSignal=SIGTERM

[Install]
WantedBy=multi-user.target
```

## Run System Services

#### Reload System Services

```bash
systemctl daemon-reload
```

#### Run Sonic Service

```bash
systemctl enable sonic.service 
systemctl start sonic.service
```

## Query Node

#### Check Logs

```bash
journalctl -xeu sonic.service -o cat
```

Response should follow this

```
Mar 12 22:12:14 sonicd[45638]: INFO [03-12|22:12:14.055] New block                                index=13323260 id=563709..664f13   gas_used=910,015     gas_rate=2936366.148           base_fee=50000000000  txs=2/0      age=666.223ms         t=4.721ms     epoch=15237
Mar 12 22:12:14 sonicd[45638]: INFO [03-12|22:12:14.371] New block                                index=13323261 id=52e8c7..ddd174   gas_used=396,112     gas_rate=1249237.219           base_fee=50000000000  txs=1/0      age=665.575ms         t=8.661ms     epoch=15237
Mar 12 22:12:14 sonicd[45638]: INFO [03-12|22:12:14.673] New block                                index=13323262 id=20a5d7..5cc02f   gas_used=251,646     gas_rate=811205.918            base_fee=50000000000  txs=1/0      age=657.086ms         t=2.020ms     epoch=15237
Mar 12 22:12:14 sonicd[45638]: INFO [03-12|22:12:14.998] New block                                index=13323263 id=48ac9c..ac0c96   gas_used=1,911,456   gas_rate=6212358.041           base_fee=50000000000  txs=3/0      age=674.246ms         t=7.683ms     epoch=15237
```

#### Check Sync Status

```bash
curl -H "Content-Type: application/json" \
-X POST --data '{"jsonrpc":"2.0", "method":"eth_syncing", "params":[], "id":1}' \
http://localhost:18545

# If node is done syncing - the response should resemble the below.
{"jsonrpc":"2.0","id":1,"result":false}
```

#### Check Block Number

```bash
curl -H "Content-Type: application/json" \
-X POST --data '{"jsonrpc":"2.0", "method":"eth_blockNumber", "params":[], "id":1}' \
http://localhost:18545

# Response should resemble the below.
{"jsonrpc":"2.0","id":1,"result":"0xcb4b6a"}
```

## References

{% embed url="https://docs.soniclabs.com/sonic/node-deployment/archive-node" %}

{% embed url="https://genesis.soniclabs.com/" %}

{% embed url="https://github.com/0xsoniclabs/Sonic.git" %}
