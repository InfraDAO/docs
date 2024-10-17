# 🐳 Docker

## System Requirements

| **CPU** | **OS**    | **RAM** | **DISK**   |
| ------- | --------- | ------- | ---------- |
| 8 vCPU  | Ubuntu 22 | 16GB    | 1TB+ (SSD) |

{% hint style="info" %}
_The Zora Mainnet archive node has a size of 693GB on October 10th, 2024_
{% endhint %}

## Zora

Zora operates within the Optimism _Superchain_ ecosystem and is built using the OP stack, which leverages the scalability and security of Optimism's Layer 2 infrastructure.

In this guide, we will walk you through the process of setting up a Zora Mainnet archive node using Optimism's `op-geth` and `op-node` tools.

Before you begin, ensure you have a synced Ethereum L1 RPC URL (such as Erigon) and an L1 Consensus Layer Beacon endpoint that includes **all historical blobs data** (for example, Lighthouse). Having a suitable beacon endpoint is crucial for the syncing process to initiate.

**Tip:** You can create a free account at [Chainstack](https://console.chainstack.com/user/account/create) to obtain enough resources for syncing your node.

## Pre-Requisites

To the archive node using Docker we need the following installed:

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

### Allow SSH, HTTP, and HTTPS <a href="#allow-ssh-http-and-https" id="allow-ssh-http-and-https"></a>

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

```yaml
services:
  op-geth: # this is Optimism's geth client
    ...
    volumes:
      - /mnt/zora-data:/data # enable to have persistency between restarts
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
    "gasUsed": "0xab4b",
    "hash": "0x3ef9dfddbcf06e31ea6fa488834682287e8405b685299b9c1acb939ec81308bd",
    "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
    "miner": "0x4200000000000000000000000000000000000011",
    "mixHash": "0xf4fd464784037be144c017dd88a03f1fc234c65728caa8dd2195e1303f84c107",
    "nonce": "0x0000000000000000",
    "number": "0x13edd8a",
    "parentBeaconBlockRoot": "0x94b9941e2ac80450845656ad054e6cbeb656e6f408194e55025be9d4da6bae70",
    "parentHash": "0x6b77bb5a6ef5a31cbc014867cb9a666c589a90a937b4b0e060aa4734b5ddc59e",
    "receiptsRoot": "0x49af9e31125af317a81e30bcf9db9419afdbb4afd808c2a16bef3ff9167db3a1",
    "sha3Uncles": "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
    "size": "0x348",
    "stateRoot": "0xba115af0f6448b6bc1333880b5c7a1b6a210b8774d698446b1f3810ed17f7fbc",
    "timestamp": "0x6706a2e3",
    "totalDifficulty": "0x0",
    "transactions": [
      "0xaf32e05388d009011ad28fd010af52ee9da6decc0a04313843419fd6414ae935"
    ],
    "transactionsRoot": "0x854a1e051237917389c365790d1b1bb18059bf8f91602ef2e6e51555f71eae49",
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

{% embed url="https://docs.zora.co/zora-network/intro" %}
