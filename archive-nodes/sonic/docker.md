---
description: 'Author: [ jLeopoldA ]'
---

# 🐳 Docker

## System Requirements

| CPU                  | OS                 | RAM                | DISK |
| -------------------- | ------------------ | ------------------ | ---- |
| Minimum: 4 Cores     | Ubuntu 24.04.2 LTS | Minimum: 32GB      | 1TB  |
| Recommended: 8 Cores | Ubuntu 24.04.2 LTS | Recommended: 128GB | 1TB  |

{% hint style="info" %}
The Sonic Archive Node has a size of 588GB as of 3/12/2025.
{% endhint %}

{% hint style="warning" %}
Sonic requires GO v1.22.0 or greater to run.
{% endhint %}

## Pre-Requisites

#### Update System

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y
sudo apt install -y git gcc make --fix-missing
```

#### Install GO

{% hint style="danger" %}
Sonic requires GO v1.22.0 or greater to run.
{% endhint %}

```bash
# Remove previous installation of GO
rm -rf /usr/local/go # For GO installations locacated within /usr/local/go
rm -rf /usr/local/bin/go # For GO installations located within /usr/local/bin/go

# Download GO
wget https://go.dev/dl/go1.22.0.linux-amd64.tar.gz

# Extract and place within /usr/local
tar -xzf go1.22.0.linux-amd64.tar.gz -C /usr/local && rm go1.22.0.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc
```

#### Install Docker & Docker-Compose

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

# Install Docker-Compose
sudo apt-get update
sudo apt-get install docker-compose-plugin

# Verify Docker-Compose installation by checking the version
docker compose version

# Expected output
Docker Compose version vN.N.N
```

### Firewall Configuration

#### Set Explicit UFW Rules

```bash
sudo ufw default deny incoming && sudo ufw default allow outgoing
```

#### Allow SSH

```bash
sudo ufw allow 22/tcp
```

#### Allow Connections for Sonic

```bash
sudo ufw allow 18545
sudo ufw allow 18546
sudo ufw allow 5050
```

#### Enable Firewall

```bash
sudo ufw enable
```

#### View Current UFW Rules / Status

```bash
sudo ufw status verbose
```

## Set up Sonic

### Create Directories

```bash
mkdir -p /var/lib/sonic/configuration
mkdir -p /var/lib/sonic/database
mkdir -p /root/docker
```

### Install Sonic

#### Download Sonic

```bash
cd /root
git clone https://github.com/0xsoniclabs/Sonic.git
cd Sonic
git fetch --tags && git checkout -b v2.0.1 tags/v2.0.1
```

#### Build Sonic

```bash
make all # Build Sonic
sudo cp build/sonic* /usr/local/bin/ 
```

### Prime Sonic State DB

```bash
cd /var/lib/sonic/configuration

# Download Genesis File
wget https://genesis.soniclabs.com/sonic-mainnet/genesis/sonic.g

# Prime DB
sonictool --datadir /var/lib/sonic/database --cache 12000 \
genesis /var/lib/configuration/sonic.g
```

## Set up Docker

#### Create Dockerfile

```bash
cd /root/docker

# Copy Sonic binary
cp /usr/local/bin/sonicd sonicd

echo "FROM ubuntu:24.04

RUN apt update && apt install -y \
    libc6 \
    libc-bin

WORKDIR /app

COPY sonicd /usr/local/bin/sonicd

RUN chmod +x /usr/local/bin/sonicd

# Expose necessary ports
EXPOSE 18545 18546

CMD ["/usr/local/bin/sonicd", \
    "--http", "--http.addr=0.0.0.0", "--http.port=18545", \
    "--http.corsdomain=*", "--http.vhosts=*", \
    "--http.api=eth,web3,net,ftm,txpool,abft,dag,trace,debug", \
    "--ws", "--ws.addr=0.0.0.0", "--ws.port=18546", "--ws.origins=*", \
    "--ws.api=eth,web3,net,ftm,txpool,abft,dag,trace,debug", \
    "--datadir", "/var/lib/sonic/database", \
    "--cache", "12000"]" > Dockerfile
    
# Build Dockerfile
docker build -t sonic-node .
```

#### Create docker-compose.yml

```bash
echo "services:
  sonic:
    image: sonic-node:latest
    container_name: sonic-node
    restart: unless-stopped
    volumes:
      - /var/lib/sonic/database:/var/lib/sonic/database  # Mount real database
    ports:
      - "18545:18545"
      - "18546:18546"
    command:
      - /usr/local/bin/sonicd
      - --http
      - --http.addr=0.0.0.0
      - --http.port=18545
      - --http.corsdomain=*
      - --http.vhosts=*
      - --http.api=eth,web3,net,ftm,txpool,abft,dag,trace,debug
      - --ws
      - --ws.addr=0.0.0.0
      - --ws.port=18546
      - --ws.origins=*
      - --ws.api=eth,web3,net,ftm,txpool,abft,dag,trace,debug
      - --datadir
      - /var/lib/sonic/database
      - --cache
      - "12000" " > docker-compose.yml
```

## Run Sonic Archive Node

#### Start Sonic Node

```bash
# Run the below from within /root/docker
docker compose up -d

# Stop Sonic Node
docker compose down
```

#### Get Logs from Sonic Node

````bash
docker logs sonic-node

# Should resemble the below
```
INFO [03-12|18:20:11.514] New block                                index=13298658 id=4fb356..584049   gas_used=1,565,765  gas_rate=4843392.945         base_fee=50000000000 txs=1/0   age=641.006ms   t=6.736ms     epoch=15220
INFO [03-12|18:20:11.829] New block                                index=13298659 id=854d2f..75f47e   gas_used=45800      gas_rate=158104.137          base_fee=50000000000 txs=2/0   age=666.251ms   t=1.398ms     epoch=15220
INFO [03-12|18:20:12.120] New block                                index=13298660 id=a74b42..917700   gas_used=806,101    gas_rate=2521272.818         base_fee=50000000000 txs=2/0   age=637.504ms   t=3.917ms     epoch=15220
INFO [03-12|18:20:12.435] New block                                index=13298661 id=77a9f1..40e2de   gas_used=794,133    gas_rate=2567401.378         base_fee=50000000000 txs=5/0   age=643.354ms   t=3.723ms     epoch=15220
INFO [03-12|18:20:12.749] New block                                index=13298662 id=0726ff..ff2d80   gas_used=179,388    gas_rate=582004.657          base_fee=50000000000 txs=2/0   age=649.186ms   t=1.897ms     epoch=15220
INFO [03-12|18:20:13.069] New block                                index=13298663 id=643a2f..c0c1d5   gas_used=508,248    gas_rate=1541704.805         base_fee=50000000000 txs=2/0   age=639.632ms   t=3.212ms     epoch=15220
INFO [03-12|18:20:13.377] New block                                index=13298664 id=bb537c..be7d8b   gas_used=1,609,831  gas_rate=5471962.423         base_fee=50000000000 txs=3/0   age=653.157ms   t=7.423ms     epoch=15220
INFO [03-12|18:20:13.411] New DAG summary                          new=2023  last_id=15220:5725:10c9ba  age=89.642ms    t=1.268s
```
````

### Query Sonic Node

#### Check Sync Status

```bash
curl -H "Content-Type: application/json" \
-X POST --data '{"jsonrpc":"2.0","method":"eth_syncing", "params":[], "id":1}' \
http://localhost:18545

# Response will resemble the below when node is done syncing. 
# Sync time was 6 days for the Author of this guide.
{"jsonrpc":"2.0","id":1,"result":false}
```

#### Get Current Block Number of Node

```bash
curl -H "Content-Type: application/json" \
-X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber", "params":[], "id":1}' \
http://localhost:18545

# Response should resemble the below:
{"jsonrpc":"2.0","id":1,"result":"0xca8227"}
```

## References

{% embed url="https://docs.soniclabs.com/sonic/node-deployment/archive-node" %}
