# 🐳 Docker

Authors: \[ Ankur | Dapplooker]

## System Requirements

<table data-full-width="false"><thead><tr><th>CPU</th><th>OS</th><th>RAM</th><th>DISK</th></tr></thead><tbody><tr><td>8 vCPU</td><td>Ubuntu 22.04</td><td>16 GB</td><td>500 GB </td></tr></tbody></table>

{% hint style="success" %}
_The Chapel node has a size of 463 GB on March 25, 2025._
{% endhint %}

## Pre-requisite

Before starting, clean the setup then update and upgrade. Install following:

* Docker

### **Commands**

{% code overflow="wrap" %}
```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt install docker.io ufw -y jq -y
```
{% endcode %}

## Firewall Settings

### Check status & enable UFW&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>sudo ufw enable
</strong>sudo ufw status verbose
</code></pre>

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

## Setup Instructions&#x20;

### Pull Docker Image

Pull the image from the GitHub container registry (ghr):

```bash
docker pull ghcr.io/node-real/bsc-erigon:v1.3.2-beta2
```

{% hint style="info" %}
Get latest version: [https://github.com/node-real/bsc-erigon/pkgs/container/bsc-erigon](https://github.com/node-real/bsc-erigon/pkgs/container/bsc-erigon)
{% endhint %}

### Create docker compose file

Create project directory

```bash
mkdir /mnt/erigon-bsc/
```

Create _docker-compose.yml_ file

```yaml
version: '3.8'

services:
  bsc-erigon:
    container_name: bsc-erigon
    image: ghcr.io/node-real/bsc-erigon:v1.3.2-beta2
    restart: on-failure
    user: root
    volumes:
      - /mnt/erigon-bsc:/root/data/bsc-erigon
    ports:
      - "8545:8545"   # HTTP RPC
      - "8546:8546"   # WebSocket
      - "30303:30303" # P2P TCP
      - "30303:30303/udp" # P2P UDP
      - "42069:42069" # Torrent Port
      - "6060:6060"   # Metrics
      - "6061:6061"   # PProf Debugging
      - "9090:9090"   # Private API
      - "8551:8551"   # Auth RPC
    command:
      - --private.api.addr=127.0.0.1:9090
      - --chain=chapel
      - --prune.mode=archive
      - --torrent.download.rate=1g
      - --torrent.download.slots=400
      - --batchSize=2g
      - --bsc.blobSidecars.no-pruning=true
      - --txpool.disable
      - --metrics
      - --metrics.addr=0.0.0.0
      - --metrics.port=6060
      - --pprof
      - --pprof.addr=0.0.0.0
      - --pprof.port=6061
      - --authrpc.jwtsecret=/root/data/bsc-erigon/jwt.hex
      - --authrpc.port=8551
      - --datadir=/root/data/bsc-erigon/
      - --http.addr=0.0.0.0
      - --http.port=8545
      - --http.api=eth,debug,net,trace,web3,erigon,bsc,admin
      - --http.vhosts=any
      - --http.corsdomain=*
      - --ws
      - --ws.port=8546
      - --torrent.port=42069
      - --nat=extip:157.90.180.249
      - --db.pagesize=16k
      - --db.size.limit=3t
      - --port=30303
    ulimits:
      nofile:
        soft: 500000
        hard: 500000
```

### Start the Node

```bash
docker compose up -d 
```

## Monitoring

### Monitor Logs of Docker Container&#x20;

```bash
docker ps 
docker logs bsc-erigon
```

{% hint style="warning" %}
Might need to restart the container if snapshot download gets stuck for a longer duration. Total time required to sync is \~28 hr
{% endhint %}

## Sync Status

### Latest Block

```bash
curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' http://localhost:8545
```

_Response should look like:_

```json
{"jsonrpc":"2.0","id":1,"result":"0x2f19521"}
```

### References

{% embed url="https://docs.bnbchain.org/bnb-smart-chain/developers/node_operators/archive_node/" %}

{% embed url="https://testnet.bscscan.com/" %}
