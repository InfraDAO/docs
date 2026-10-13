# 🐳 Docker

Authors: \[Vikash Choubey | Dapplooker]

### **System Requirements**

| CPU    | OS        | RAM  | DISK   |
| ------ | --------- | ---- | ------ |
| 8 vCPU | Ubuntu 22 | 16GB | 500GB+ |

_The Boba BNB archival node has a size of 508GB on September 19th, 2024_

### Pre-requisite

Before starting, clean the setup then update and upgrade. Install following:

* Docker & Docker Compose
* Git

**Commands:**

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt install docker.io docker-compose git ufw -y
```

### Set explicit default UFW rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### Allow SSH, HTTP and HTTPS

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80
sudo ufw allow 443
```

### **Setup**

Clone the `boba_legacy` repository to get started

```bash
git clone git@github.com:bobanetwork/boba_legacy.git
cd boba_community/boba_node
```

#### Setting up Environment Variable

You can locate environment files at `boba_legacy/ops/envs/` folder.

#### **dtl.env**

```bash
DATA_TRANSPORT_LAYER__SYNC_FROM_L1=true
DATA_TRANSPORT_LAYER__SYNC_FROM_L2=false
DATA_TRANSPORT_LAYER__DB_PATH=/db
DATA_TRANSPORT_LAYER__SERVER_PORT=7878
DATA_TRANSPORT_LAYER__TRANSACTIONS_PER_POLLING_INTERVAL=1000
DATA_TRANSPORT_LAYER__CONFIRMATIONS=0
DATA_TRANSPORT_LAYER__POLLING_INTERVAL=4000
DATA_TRANSPORT_LAYER__LOGS_PER_POLLING_INTERVAL=2000
DATA_TRANSPORT_LAYER__DANGEROUSLY_CATCH_ALL_ERRORS=true
DATA_TRANSPORT_LAYER__SERVER_HOSTNAME=0.0.0.0

DATA_TRANSPORT_LAYER__ADDRESS_MANAGER=
DATA_TRANSPORT_LAYER__L1_RPC_ENDPOINT=
DATA_TRANSPORT_LAYER__L2_RPC_ENDPOINT=
DATA_TRANSPORT_LAYER__L2_CHAIN_ID=
```

#### **geth.env**

```bash
ETH1_HTTP=
ETH1_CTC_DEPLOYMENT_HEIGHT=
ETH1_SYNC_SERVICE_ENABLE=true
ETH1_CONFIRMATION_DEPTH=0

ROLLUP_CLIENT_HTTP=
ROLLUP_POLL_INTERVAL_FLAG=4500ms
ROLLUP_ENABLE_L2_GAS_POLLING=true
# ROLLUP_ENFORCE_FEES=

ETHERBASE=0x7E5F4552091A69125d5DfCb7b8C2659029395Bdf

RPC_ENABLE=true
RPC_ADDR=0.0.0.0
RPC_PORT=8545
RPC_API=eth,net,rollup,web3,debug
RPC_CORS_DOMAIN=*
RPC_VHOSTS=*

WS=true
WS_ADDR=0.0.0.0
WS_PORT=8546
WS_API=eth,net,rollup,web3
WS_ORIGINS=*

CHAIN_ID=31338
DATADIR=/root/.ethereum
GASPRICE=0
GCMODE=archive
IPC_DISABLE=true
NETWORK_ID=31338
NO_USB=true
NO_DISCOVER=true
TARGET_GAS_LIMIT=11000000
USING_OVM=true
```

### Example docker-compose file:

```bash
x-l1_rpc_dtl: &l1_rpc_dtl
  DATA_TRANSPORT_LAYER__L1_RPC_ENDPOINT: '<https://bsc-dataseed.binance.org/>'

x-l1_rpc_geth: &l1_rpc_geth
  ETH1_HTTP: '<https://bsc-dataseed.binance.org/>'

version: "3.9"

services:
  dtl:
    container_name: dtl
    image: bobanetwork/data-transport-layer:latest
    env_file:
      -  ../../ops/envs/dtl.env
    environment:
      << : *l1_rpc_dtl
      DATA_TRANSPORT_LAYER__L2_RPC_ENDPOINT: '<https://replica.bnb.boba.network>'
      DATA_TRANSPORT_LAYER__SYNC_FROM_L1: 'false'
      DATA_TRANSPORT_LAYER__SYNC_FROM_L2: 'true'
      DATA_TRANSPORT_LAYER__L2_CHAIN_ID: 56288
      DATA_TRANSPORT_LAYER__POLLING_INTERVAL: 10000
      DATA_TRANSPORT_LAYER__ETH1_CTC_DEPLOYMENT_HEIGHT: 1305672
      DATA_TRANSPORT_LAYER__ADDRESS_MANAGER: '0xeb989B25597259cfa51Bd396cE1d4B085EC4c753'
      DATA_TRANSPORT_LAYER__BSS_HARDFORK_1_INDEX: 0
      DATA_TRANSPORT_LAYER__TURING_V0_HEIGHT: 0
      DATA_TRANSPORT_LAYER__TURING_V1_HEIGHT: 0
    volumes:
      - ./state-dumps/bobabnb:/opt/optimism/packages/data-transport-layer/state-dumps/
    logging:
      driver: "json-file"
      options:
        max-file: "5"
        max-size: "10m"
    ports:
      - ${DTL_PORT:-7878}:7878
      - ${REGISTRY_PORT:-8080}:8081

  replica:
    container_name: replica
    depends_on:
      - dtl
    image: bobanetwork/l2geth:latest
    deploy:
      replicas: 1
    entrypoint: sh ./geth.sh
	    env_file:
      - ../../ops/envs/geth.env
    volumes:
      - <DATA_DIR>:/root/.ethereum/
    environment:
      << : *l1_rpc_geth
      ROLLUP_TIMESTAMP_REFRESH: 5s
      ROLLUP_STATE_DUMP_PATH: <http://dtl:8081/state-dump.latest.json>
      ROLLUP_CLIENT_HTTP: <http://dtl:7878>
      ROLLUP_BACKEND: 'l2'
      ROLLUP_VERIFIER_ENABLE: 'true'
      RETRIES: 60
      # no need to keep this secret, only used internally to sign blocks
      BLOCK_SIGNER_KEY: "6587ae678cf4fc9a33000cdbf9f35226b71dcc6a4684a31203241f9bcfd55d27"
      BLOCK_SIGNER_ADDRESS: "0x00000398232E2064F896018496b4b44b3D62751F"
      ROLLUP_POLL_INTERVAL_FLAG: "10s"
      ROLLUP_ENFORCE_FEES: 'true'
      # turing
      TURING_CREDIT_ADDRESS: "0x4200000000000000000000000000000000000020"
      # fee token
      L2_BOBA_TOKEN_ADDRESS: "0x4200000000000000000000000000000000000023"
      BOBA_GAS_PRICE_ORACLE_ADDRESS: "0x4200000000000000000000000000000000000024"
      # sequencer http endpoint
      SEQUENCER_CLIENT_HTTP: <https://bnb.boba.network/>
      ROLLUP_HISTORICALRPC: <https://bnb.boba.network/>
    logging:
      driver: "json-file"
      options:
        max-file: "5"
        max-size: "10m"
    ports:
      - ${L2GETH_HTTP_PORT:-8549}:8545
      - ${L2GETH_WS_PORT:-8550}:8546

```

### **Start The Node**

```bash
docker-compose -f [docker-compose-file] up -d
```

### **Monitor Logs**

Use `docker logs` to monitor your boba node. The `-f` flag ensures you are following the log output

```bash
docker logs dtl -f
docker logs replica -f
```

### **Test RPC:**

```bash
curl --data '{"method":"eth_blockNumber","params":[],"id":1,"jsonrpc":"2.0"}' 
-H "Content-Type: application/json" -X POST https://{DOMAIN}
```

#### **You should receive a result after the node is synced:**

```bash
{
	"jsonrpc":"2.0",
	"id":1,
	"result":{HEX_VALUE}
}
```
