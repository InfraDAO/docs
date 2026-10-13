# 🐳 Docker

Author \[godwin]

### System Requirements <a href="#system-requirements" id="system-requirements"></a>

| CPU      | OS           | RAM   | DISK     |
| -------- | ------------ | ----- | -------- |
| 8 cores+ | Ubuntu 24.04 | 32GB+ | >= 1.2TB |

{% hint style="info" %}
_The Ronin Mainnet archive node has a size of 1.2TB on November 14th, 2024_
{% endhint %}

Last updated at: 14th November 2024

Official docs - [https://docs.roninchain.com/rpc/mainnet-rpc](https://docs.roninchain.com/rpc/mainnet-rpc)​

## Pre-Requisites

First, update, upgrade, and clean the system:

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt install ufw -y
```

## Configure Firewall Settings&#x20;

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw allow 22/tcp
sudo ufw allow 80
sudo ufw allow 443
sudo ufw allow 8545
sudo ufw allow 8546
sudo ufw allow 30303
sudo ufw allow 6060
```

## Install Docker

Run this command to remove any conflicting docker

```bash
`for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done`
```

Add Docker's official GPG key:

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Add the repository to ppt sources:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

Install docker

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Test docker is working
sudo docker run hello-world

# Install docker compose

sudo apt-get update
sudo apt-get install docker-compose-plugin

# Test the docker version
docker compose version
```

## Setup Ronin Node

Make and switch to the working directory for the ronin node

```bash
mkdir -p ~/ronin/docker
cd ~/ronin
```

Make a directory for the chain data

```bash
mkdir -p chaindata/data/ronin
```

Go into the `docker` directory, create a `docker-compose.yml` file with the following configuration:

```bash
version: "3"
services:
  node:
    image: ${NODE_IMAGE}
    stop_grace_period: 5m
    stop_signal: SIGINT
    hostname: node
    container_name: node
    ports:
      - 127.0.0.1:8545:8545
      - 127.0.0.1:8546:8546
      - 30303:30303
      - 30303:30303/udp
      - 6060:6060
    volumes:
      - ~/ronin/chaindata:/ronin
    environment:
      - SYNC_MODE=full
      - PASSWORD=${PASSWORD}
      - NETWORK_ID=${NETWORK_ID}
      - RONIN_PARAMS=${RONIN_PARAMS}
      - VERBOSITY=${VERBOSITY}
      - MINE=${MINE}
      - GASPRICE=${GASPRICE}
      - ETHSTATS_ENDPOINT=${INSTANCE_NAME}:${CHAIN_STATS_WS_SECRET}@${CHAIN_STATS_WS_SERVER}:443
```

This compose file defines the `node` service that pulls a Ronin node image from the GitHub Container Registry.

Create an `.env` file and add the following content, replacing the `<...>` placeholder values with your information:

```bash
# The name of your node that you want displayed on https://ronin-stats.roninchain.com/
INSTANCE_NAME=<INSTANCE_NAME>

# The latest version of the node's image as listed in https://docs.roninchain.com/validators/setup/upgrade-validator
NODE_IMAGE=<NODE_IMAGE>

# The password used to encrypt the node's private key file
PASSWORD=<PASSWORD>

MINE=false

NETWORK_ID=2020
GASPRICE=20000000000
VERBOSITY=3

CHAIN_STATS_WS_SECRET=WSyDMrhRBe111
CHAIN_STATS_WS_SERVER=ronin-stats-ws.roninchain.com

RONIN_PARAMS=--http.api eth,net,web3,consortium --txpool.pricelimit 20000000000 --txpool.nolocals --cache 4096 --discovery.dns enrtree://AIGOFYDZH6BGVVALVJLRPHSOYJ434MPFVVQFXJDXHW5ZYORPTGKUI@nodes.roninchain.com
```

(Optional) Download the snapshot from the [ronin-snapshot](https://github.com/axieinfinity/ronin-snapshot) repo - If you want to sync the ronin chain data in time and not wait for weeks before it is fully synced.

```bash
cd ~/ronin/chaindata/data/ronin/
wget -q -O - <snapshot URL from the README file in the repo> | tar -I zstd -xvf -
```

## Run the node

```bash
cd ~/ronin/docker && docker-compose up -d
```

## Monitor the node

Use docker logs to monitor the rootstock node. The -f flag ensures you are following the log output.

```bash
docker logs node -f --tail 100
```

You should see a response similar to this once your node starts syncing

```
INFO [11-14|21:24:17.122] Imported new chain segment               blocks=1          txs=11          mgas=1.280   elapsed=16.911ms   mgasps=75.657  number=39,923,835 hash=1afff2..417928 dirty=0.00B
INFO [11-14|21:24:20.156] Imported new chain segment               blocks=1          txs=16          mgas=3.102   elapsed=25.749ms   mgasps=120.449 number=39,923,836 hash=52613d..18b334 dirty=0.00B
INFO [11-14|21:24:23.139] Imported new chain segment               blocks=1          txs=15          mgas=3.349   elapsed=30.903ms   mgasps=108.379 number=39,923,837 hash=e34fa3..50d71d dirty=0.00B
INFO [11-14|21:24:26.089] Imported new chain segment               blocks=1          txs=12          mgas=1.821   elapsed=20.431ms   mgasps=89.147  number=39,923,838 hash=7b568b..250cb0 dirty=0.00B
INFO [11-14|21:24:29.155] Imported new chain segment               blocks=1          txs=18          mgas=4.263   elapsed=26.391ms   mgasps=161.523 number=39,923,839 hash=f16921..a764e8 dirty=0.00B
INFO [11-14|21:24:30.882] Deep froze chain segment                 blocks=20         elapsed=5.064ms    number=39,833,839 hash=925558..afa3ca
INFO [11-14|21:24:32.101] Imported new chain segment               blocks=1          txs=7           mgas=1.347   elapsed=18.884ms   mgasps=71.350  number=39,923,840 hash=d1019d..13e80a dirty=0.00B
INFO [11-14|21:24:35.177] Imported new chain segment               blocks=1          txs=14          mgas=2.823   elapsed=23.487ms   mgasps=120.211 number=39,923,841 hash=5e0d2a..f2691b dirty=0.00B
```

## Query the node

To get the web3 client version

```bash
curl http://localhost:8545 -s -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"web3_clientVersion","params":[],"id":67}'
```

Output

```bash
{"jsonrpc":"2.0","id":67,"result":"ronin/v2.8.3-d27eb42e/linux-amd64/go1.20.10"}
```

To check the block number

```bash
curl -X POST http://localhost:8545/ -H "Content-Type: application/json" --data '{"jsonrpc":"2.0", "method":"eth_blockNumber","params":[],"id":1}'
```

Output

```bash
{"jsonrpc":"2.0","id":1,"result":"0x26130a0"}
```

## References

{% embed url="https://docs.roninchain.com/validators/setup/mainnet/run-archive" %}

{% embed url="https://docs.roninchain.com/rpc/mainnet-rpc" %}
