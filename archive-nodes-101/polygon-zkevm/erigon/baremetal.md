# 🐳 Docker

Authors: \[ Ankur | Dapplooker]

## System Requirements

<table data-full-width="false"><thead><tr><th>CPU</th><th>OS</th><th>RAM</th><th>DISK</th></tr></thead><tbody><tr><td>4 vCPU</td><td>Ubuntu 22.04</td><td>64 GB</td><td>1TB (SSD)</td></tr></tbody></table>

{% hint style="info" %}
_The CDK-Erigon archival node has a size of 104GB on January 29, 2025._
{% endhint %}

## Pre-requisite

Before starting, clean the setup then update and upgrade. Install following:

* Docker
* Docker Compose
* Git
* Go v1.19 +
* L1 Ethereum node RPC&#x20;

### **Commands:**

{% code overflow="wrap" %}
```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt install docker.io docker-compose git ufw -y 
```
{% endcode %}

## Firewall Settings:

### Set explicit default UFW rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### Allow SSH, HTTP, and HTTPS

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80
sudo ufw allow 443
```

### Allow Remote connection

```bash
sudo ufw allow from ${REMOTE.HOST.IP} to any port 8545
```

## Setup Instructions:

**Clone the CDK Erigon Repository**

Clone the repository and navigate to its root directory:

```bash
git clone https://github.com/0xPolygonHermez/cdk-erigon.git
cd cdk-erigon/
```

**Build Libraries**

Install the relevant libraries for your architecture:

```bash
make build-libs
```

**Configure `.env` file**

Create a `.env` file to configure environment variables:

```bash
echo "NETWORK=mainnet" >> .env
echo "L1_RPC_URL=<ETH_RPC_URL>" >> .env
```

### Example docker compose file:

```yaml
version: '2.2'
services:
  erigon:
    image: hermeznetwork/cdk-erigon:${TAG:-latest}
    user: root
    build:
      args:
        UID: root
        GID: root
      context: .
    command: ${ERIGON_FLAGS-} --config mainnet.yaml --zkevm.l1-rpc-url=<ETH_RPC_URL>
    environment:
      - name=value
    ports:
      - "8545:8545"
    volumes:
      - /root/cdk-erigon/data:/home/erigon/.local/share/erigon
    restart: unless-stopped
    mem_swappiness: 0
```

**Start the Node**

```bash
 docker compose -f docker-compose-example.yml up -d
```

## Monitor Logs

Monitor Logs of Docker Container&#x20;

```bash
docker ps 
docker logs  cdk-erigon-erigon-1
```

## Sync Status

Run a query to check the latest synchronized L2 block:

```bash
curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber",
"params":[],"id":83}' http://localhost:8545
```

Response should look like:

```json
{"jsonrpc":"2.0","id":83,"result":"0x124ff31"}
```

### References <a href="#references" id="references"></a>

{% embed url="https://github.com/0xPolygonHermez/cdk-erigon" %}

{% embed url="https://zkevm.polygonscan.com/" %}
