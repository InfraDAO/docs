# 🐳 Docker

Authors: \[Vikash Choubey | Dapplooker]

### System Requirements

| **CPU** | **OS**    | **RAM** | **DISK**   |
| ------- | --------- | ------- | ---------- |
| 8 vCPU  | Ubuntu 22 | 16GB    | 1TB+ (SSD) |

> _The Mode Mainnet archive node has a size of_ 562G _on October 21st, 2024_

## Mode

{% hint style="success" %}
Mode operates within Optimism _Superchain_ ecosystem. It is powered by the [OP Stack](https://stack.optimism.io/), leveraging the scalability and security of Optimism's Layer 2 infrastructure.

In this guide, we are walking through the process of setting up a Mode Mainnet archive node using Optimism's `op-geth and op-node`.
{% endhint %}

{% hint style="warning" %}
Before you start, make sure that you have your own synced Ethereum L1 RPC URL (e.g. Erigon) and L1 Consensus Layer Beacon endpoint with **`all historical blobs data`** (e.g. Lighthouse) ready. <mark style="color:red;">A beacon endpoint meeting this criteria is essential for syncing to start.</mark>

<mark style="color:orange;">**Hint:**</mark> [https://console.chainstack.com/user/account/create](https://console.chainstack.com/user/account/create) has a free plan enough to sync a node
{% endhint %}

## Pre-Requisites

To run the archive node using Docker, we need the following installed:

* Docker
* Python3
* git

#### Commands:

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt install docker.io docker-compose git ufw -y
```

## Firewall Setting: <a href="#set-explicit-default-ufw-rules" id="set-explicit-default-ufw-rules"></a>

### Set explicit default UFW rules <a href="#set-explicit-default-ufw-rules" id="set-explicit-default-ufw-rules"></a>

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### Allow SSH, HTTP and HTTPS <a href="#allow-ssh-http-and-https" id="allow-ssh-http-and-https"></a>

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80
sudo ufw allow 443
```

### Allow Remote connection: <a href="#setup-process" id="setup-process"></a>

```bash
sudo ufw allow from ${REMOTE.HOST.IP} to any port 8545 
sudo ufw allow from ${REMOTE.HOST.IP} to any port 7545
```

### Firewall Basic commands:

```
sudo ufw enable
```

## Running the node

### Clone repo:

```bash
cd /mnt/
git clone git@github.com:mode-network/rollup-node.git
```

### Set Environment variable

```bash
export CONDUIT_NETWORK=mode-mainnet-0
```

### Download network configuration with

```bash
cd /mnt/rollup-node
./download-config.py $CONDUIT_NETWORK
cp .env.example .env
```

### Update Environment Variable (.env):

```
OP_NODE_L1_ETH_RPC=
OP_NODE_L1_BEACON=
CONDUIT_NETWORK=mode-mainnet-0
```

### Create Data Directory:

You can create a data directory wherever you want. For this tutorial, we have created it at `/mnt/mode-data`

### Update Docker Compose file:

```yaml
services:
  op-geth: # this is Optimism's geth client
    ...
    volumes:
      - /mnt/mode-data:/data # enable to have persistency between restarts
      ...

```

### Example Docker Compose:

```yaml
version: '3.8'

services:
  op-geth: # this is Optimism's geth client
    pull_policy: always
    build:
      context: .
      dockerfile: op-geth.Dockerfile
    ports:
      - 8545:8545       # RPC
      - 8546:8546       # websocket
      - 30303:30303     # P2P TCP (currently unused)
      - 30303:30303/udp # P2P UDP (currently unused)
      - 7301:6060       # metrics
    env_file:
      - .env.default
      - networks/${CONDUIT_NETWORK:?set network}/.env
      - .env
    volumes:
      #- ./geth-data/:/data # enable to have persistency between restarts
      - ./networks/${CONDUIT_NETWORK:?set network}/genesis.json:/genesis.json
  op-node:
    pull_policy: always
    build:
      context: .
      dockerfile: op-node.Dockerfile
    depends_on:
      - op-geth
    ports:
      - 7545:8545     # RPC
      - 9222:9222     # P2P TCP
      - 9222:9222/udp # P2P UDP
      - 7300:7300     # metrics
      - 6060:6060     # pprof
    env_file:
      - .env.default
      - networks/${CONDUIT_NETWORK:?set network}/.env
      - .env
    volumes:
      - ./networks/${CONDUIT_NETWORK:?set network}/rollup.json:/rollup.json
      - ./networks/${CONDUIT_NETWORK:?set network}/genesis.json:/genesis.json
```

### Start Services Containers:

```bash
docker compose up -d --build
```

### Check Status:

Below is the command for the request:

```bash
curl -d '{"id":0,"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["latest",false]}' -H "Content-Type: application/json" http://localhost:8545 | jq .
```

You will see a response like:

```json
{
  "jsonrpc": "2.0",
  "id": 0,
  "result": {
    "baseFeePerGas": "0xfc",
    "blobGasUsed": "0x0",
    "difficulty": "0x0",
    "excessBlobGas": "0x0",
    "extraData": "0x",
    "gasLimit": "0x1c9c380",
    "gasUsed": "0xab57",
    "hash": "0xe479844f85d8dd6008b4aa9352ff3e3d0fa550235bf10bf4ae0c6e893dc13704",
    "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
    "miner": "0x4200000000000000000000000000000000000011",
    "mixHash": "0x93aa8ce0f2f44eabacbb94058b5780cb6d645d457adba101540c6382ccf15f57",
    "nonce": "0x0000000000000000",
    "number": "0xdfaab9",
    "parentBeaconBlockRoot": "0x4f90158e2222808d53cad04b9b49ce98b3b52d311ec31b8234879a0fa81ab98c",
    "parentHash": "0x339d3321322b0f5f70bd69a0bf124025f9b980cd3b02b71b61e221c731521f54",
    "receiptsRoot": "0xf02f480a926e6a825c18788f6442060b973a4bb60ad88dbff3018b4982a1a071",
    "sha3Uncles": "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
    "size": "0x347",
    "stateRoot": "0x017662cd279f827fcf1eef1c8c362bd4e7c069b36eb9d13a0e92bedbef94e16c",
    "timestamp": "0x6715d511",
    "totalDifficulty": "0x0",
    "transactions": [
      "0xaf0ce001508b96ce0d38bb8b14480487a788181e74690b513920513e50f033d6"
    ],
    "transactionsRoot": "0x35ccc1581f44d178fb5d0059d8a8ee6198117deab115e020591a7979c1dcbc1a",
    "uncles": [],
    "withdrawals": [],
    "withdrawalsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421"
  }
}
```

## Sync Status:

```bash
command -v jq  &> /dev/null || { echo "jq is not installed" 1>&2 ; }
echo Latest synced block behind by: \
$((($( date +%s )-\
$( curl -s -d '{"id":0,"jsonrpc":"2.0","method":"optimism_syncStatus"}' -H "Content-Type: application/json" http://localhost:7545 |
   jq -r .result.unsafe_l2.timestamp))/60)) minutes
```

## References:

{% embed url="https://docs.mode.network/" %}

{% embed url="https://github.com/mode-network/rollup-node" %}
