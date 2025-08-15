# 🐳 Docker L2

Authors: \[ Ankur | DappLooker]

## System Requirements

<table data-full-width="false"><thead><tr><th>CPU</th><th>OS</th><th>RAM</th><th>DISK</th></tr></thead><tbody><tr><td>4 - 8 vCPU</td><td>Ubuntu 22.04</td><td>16 GB</td><td>4 TiB SSD (NVME)</td></tr></tbody></table>

{% hint style="success" %}
_The Celo L2 node has a size of  2.2 TiB on 30, June, 2025._
{% endhint %}

## Pre-requisite

Before starting, clean the setup, then update and upgrade. Install the following:

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
sudo ufw allow from ${REMOTE.HOST.IP} to any port 9993
```

### Setup Instructions <a href="#setup-instructions" id="setup-instructions"></a>

### <sub>Download Celo-L1 Data</sub>&#x20;

{% hint style="info" %}
You can skip this step if you have already running L1 node&#x20;
{% endhint %}

#### Setup Directory

```bash
mkdir -p /root/celo-data/celo
mkdir -p /root/celo-data/celo-l2
```

#### Download Data

```bash
screen -S nodeConfiguration
wget https://storage.googleapis.com/cel2-rollup-files/celo/celo-mainnet-migrated-chaindata.tar.zst
```

#### Extract File

```bash
tar --zstd -xvf celo-mainnet-migrated-chaindata.tar.zst -C /root/celo-data/celo/ 
```

{% hint style="info" %}
Use ctrl + A + D to go back to your main terminal session.

`screen -r nodeConfiguration` to return to the screen window.
{% endhint %}

### <sub>Migrate L1 Data</sub>&#x20;

```bash
git clone https://github.com/celo-org/celo-l2-node-docker-compose.git
cd celo-l2-node-docker-compose
```

#### Migrate L1 Data to L2 Data&#x20;

{% hint style="info" %}
Run it in background or screen will take few hours to complete.
{% endhint %}

```bash
./migrate.sh pre mainnet /root/celo-data/ /root/celo-data/celo-l2/
```

### <sub>Configure Environment file</sub>

```bash
cd celo-l2-node-docker-compose
cp mainnet.env .env
```

#### Edit .env file&#x20;

Update `.env` file with the following configuration.

```bash
NODE_TYPE=archive
OP_GETH__SYNCMODE=full
HISTORICAL_RPC_DATADIR_PATH=/root/celo-data/
DATADIR_PATH=/root/celo-data/celo-l2/
```

Example `.env` file&#x20;

```
###############################################################################
#                                ↓ REQUIRED ↓                                 #
###############################################################################

# Network to run the node on ("mainnet","alfajores" or "baklava")
NETWORK_NAME=mainnet

# Type of node to run ("full" or "archive"), note that "archive" is 10x bigger
NODE_TYPE=archive

###############################################################################
#                            ↓ REQUIRED (BEDROCK) ↓                           #
###############################################################################

# L1 node that the op-node (Bedrock) will get chain data from.
# To ensure reliability node operators may wish to change this to point at a service they trust.
OP_NODE__RPC_ENDPOINT=https://ethereum-rpc.publicnode.com

# L1 beacon endpoint, you can setup your own or use Quicknode.
# To ensure reliability node operators may wish to change this to point at a service they trust.
OP_NODE__L1_BEACON=https://ethereum-beacon-api.publicnode.com

# Type of RPC that op-node is connected to, see README
OP_NODE__RPC_TYPE=basic

# Reference L2 node to run healthcheck against
HEALTHCHECK__REFERENCE_RPC_PROVIDER=https://forno.celo.org

###############################################################################
#                            ↓ OPTIONAL (BEDROCK) ↓                           #
###############################################################################

# Optional path to a datadir for an L1 node to serve RPC requests requiring historical states. If
# set a Celo L1 node will be run in archive mode to serve requests requiring state for blocks prior to the
# L2 hardfork and op-geth will be configured to proxy those requests to the Celo L1 node.
HISTORICAL_RPC_DATADIR_PATH=/root/celo-data/

# Optional provider to serve RPC requests requiring historical state, if set op-geth will proxy
# requests requiring state prior to the L2 start to here. If set this overrides the use of a local Celo L1
# node via HISTORICAL_RPC_DATADIR_PATH.
OP_GETH__HISTORICAL_RPC=

# Set to "full" to force op-geth to use --syncmode=full
OP_GETH__SYNCMODE=full

IS_CUSTOM_CHAIN=true

# Path to the datadir, If the datadir is empty then a new datadir will be
# initialised at the given path.
#
# The path can be absolute, or relative to the docker-compose.yml file.
DATADIR_PATH=/root/celo-data/celo-l2/

# If the datadir is on a disk that doesn't support unix domain sockets then you
# will need to specify a path to a disk that does support unix domain sockets.
# E.g. your normal hard drive. If left unset the default path inside the
# datadir will be used.
IPC_PATH=

# Controls how op-geth determines its public IP that is shared via the
# discovery mechanism. The value should be one of
# (any|none|upnp|pmp|pmp:<IP>|extip:<IP>|stun:<IP:PORT>) if any is selected
# op-geth will try to automatically determine its external IP. To explicitly
# set the IP that op-geth can be reached on use extip:<your-external-ip>. To
# check the value that op-geth is currently using look in the op-geth logs for
# an entry such as
# self=enode://b24c34e53adc6db27fe648615eca3b9062a58295242caeea1604690507d507d0138ddeecb3cc9805a1cbd441471790c66a6f886f0538c3cde3e0b5dbd145f1f1@127.0.0.1:30303
# alternatively you can use the admin_nodeInfo RPC to query this information.
# If unset other nodes will not be then other nodes on the network will not be
# able to discover and connect to your node.
OP_GETH__NAT=any

# This controls the IP that op-node shares with the network so that other nodes may discover and connect to it.
# To check the value that op-node is currently using you can look in the logs for an entry such as:
# msg="started p2p host" addrs="[/ip4/127.0.0.1/tcp/9222 /ip4/192.168.97.9/tcp/9222]" peerID=16Uiu2HAkv7PQ5hpa2HeWgjYQ7SChvipCWm3L95hUKejSKMM4rVPe
# alternatively you can use the op-node opp2p_self RPC to query this
# information. If unset other nodes will not be then other nodes on the network
# will not be able to discover and connect to your node.
OP_NODE__P2P_ADVERTISE_IP=

###############################################################################
#                            ↓ REQUIRED (EIGENDA) ↓                           #
###############################################################################

# Specifies the endpoint of the eigenda proxy to use. If this is unset then a local eigenda proxy will be used.
EIGENDA_PROXY_ENDPOINT=

EIGENDA_LOCAL_SVC_MANAGER_ADDR=0x870679e138bcdf293b7ff14dd44b70fc97e12fc0
EIGENDA_LOCAL_DISPERSER_RPC=disperser.eigenda.xyz:443
EIGENDA_LOCAL_SIGNER_PRIVATE_KEY_HEX=
EIGENDA_V2_LOCAL_SVC_MANAGER_ADDR=0x870679e138bcdf293b7ff14dd44b70fc97e12fc0
EIGENDA_V2_LOCAL_DISPERSER_RPC=disperser.eigenda.xyz:443
EIGENDA_V2_LOCAL_SIGNER_PAYMENT_KEY_HEX=
EIGENDA_V2_LOCAL_CERT_VERIFIER_ADDR=0xE1Ae45810A738F13e70Ac8966354d7D0feCF7BD6
EIGENDA_V2_LOCAL_BLS_OPERATOR_STATE_RETRIEVER_ADDR=0xEC35aa6521d23479318104E10B4aA216DBBE63Ce


###############################################################################
#                            ↓ OPTIONAL (EIGENDA) ↓                           #
###############################################################################

EIGENDA_LOCAL_S3_CREDENTIAL_TYPE="public"
EIGENDA_LOCAL_S3_ACCESS_KEY_ID=""
EIGENDA_LOCAL_S3_ACCESS_KEY_SECRET=""
EIGENDA_LOCAL_S3_BUCKET="eigenda-proxy-cache-mainnet"
EIGENDA_LOCAL_S3_PATH="blobs/"
EIGENDA_LOCAL_S3_ENDPOINT="storage.googleapis.com"
EIGENDA_LOCAL_ARCHIVE_BLOBS=${EIGENDA_LOCAL_S3_BUCKET:+0}

###############################################################################
#                                ↓ OPTIONAL ↓                                 #
###############################################################################

# MONITORING_ENABLED controls whether Grafana, Prometheus, Influxdb, and Healthcheck are started
# Set to "true" if you want to launch them, otherwise keep "false"
MONITORING_ENABLED=false

IMAGE_TAG__HEALTCHECK=
IMAGE_TAG__PROMETHEUS=
IMAGE_TAG__GRAFANA=
IMAGE_TAG__INFLUXDB=
IMAGE_TAG__OP_GETH=
IMAGE_TAG__OP_NODE=

# Exposed server ports (must be unique)
# See docker-compose.yml for default values
PORT__HISTORICAL_RPC_NODE_HTTP=
PORT__HISTORICAL_RPC_NODE_WS=
PORT__HEALTHCHECK_METRICS=
PORT__PROMETHEUS=
PORT__GRAFANA=
PORT__INFLUXDB=
PORT__TORRENT_UI=
PORT__TORRENT=
PORT__OP_GETH_HTTP=
PORT__OP_GETH_WS=
PORT__OP_GETH_P2P=30303
PORT__OP_NODE_P2P=
PORT__OP_NODE_HTTP=
PORT_EIGENDA_PROXY=
```

#### Example `docker-compose.yml` file

```yaml
services:
  historical-rpc-node:
    image: us-docker.pkg.dev/celo-org/us.gcr.io/geth-all:1.8.9
    restart: on-failure
    stop_grace_period: 5m
    entrypoint: /scripts/start-historical-rpc-node.sh
    env_file:
      - ./envs/common/historical-rpc-node.env
      - .env
    volumes:
      - ${HISTORICAL_RPC_DATADIR_PATH:-geth}:/geth
      - ./scripts/:/scripts/
    ports:
      - ${PORT__HISTORICAL_RPC_NODE_HTTP:-9991}:8545
      - ${PORT__HISTORICAL_RPC_NODE_WS:-9992}:8546

  healthcheck:
    platform: linux/amd64
    image: ethereumoptimism/replica-healthcheck:${IMAGE_TAG__HEALTHCHECK:-latest}
    restart: on-failure
    entrypoint: /opt/optimism/packages/replica-healthcheck/start-healthcheck.sh
    env_file:
      - ./envs/common/healthcheck.env
      - .env
    volumes:
      - ./scripts/start-healthcheck.sh:/opt/optimism/packages/replica-healthcheck/start-healthcheck.sh
    ports:
      - ${PORT__HEALTHCHECK_METRICS:-7300}:7300

  eigenda-proxy:
    platform: linux/amd64
    image: ghcr.io/layr-labs/eigenda-proxy:v1.8.2
    restart: on-failure
    stop_grace_period: 5m
    entrypoint: /scripts/start-eigenda-proxy.sh
    env_file:
      - .env
    volumes:
      - eigenda-data:/data
      - ./scripts/:/scripts
    ports:
      - ${PORT_EIGENDA_PROXY:-4242}:4242
    extra_hosts:
      - "host.docker.internal:host-gateway"

  op-geth:
    platform: linux/amd64
    image: us-west1-docker.pkg.dev/devopsre/celo-blockchain-public/op-geth:celo-v2.1.0-rc2
    restart: on-failure
    stop_grace_period: 5m
    entrypoint: /scripts/start-op-geth.sh
    env_file:
      - ./envs/${NETWORK_NAME}/op-geth.env
      - .env
    volumes:
      - ./envs/${NETWORK_NAME}/config:/chainconfig
      - ./scripts/:/scripts
      - shared:/shared
      - ${DATADIR_PATH}:/geth
    ports:
      - ${PORT__OP_GETH_HTTP:-9993}:8545
      - ${PORT__OP_GETH_WS:-9994}:8546
      - ${PORT__OP_GETH_P2P:-39393}:${PORT__OP_GETH_P2P:-39393}/udp
      - ${PORT__OP_GETH_P2P:-39393}:${PORT__OP_GETH_P2P:-39393}/tcp
    extra_hosts:
      - "host.docker.internal:host-gateway"

  op-node:
    platform: linux/amd64
    image: us-west1-docker.pkg.dev/devopsre/celo-blockchain-public/op-node:celo-v2.1.0-rc
    restart: on-failure
    stop_grace_period: 5m
    entrypoint: /scripts/start-op-node.sh
    env_file:
      - ./envs/${NETWORK_NAME}/op-node.env
      - .env
    volumes:
      - ./envs/${NETWORK_NAME}/config:/chainconfig
      - ./scripts/:/scripts
      - shared:/shared
    ports:
      - ${PORT__OP_NODE_P2P:-9222}:9222/udp
      - ${PORT__OP_NODE_P2P:-9222}:9222/tcp
      - ${PORT__OP_NODE_HTTP:-9545}:9545
    extra_hosts:
      - "host.docker.internal:host-gateway"
    depends_on:
      op-geth:
        condition: service_started

  prometheus:
    platform: linux/amd64
    image: prom/prometheus:${IMAGE_TAG__PROMETHEUS:-latest}
    restart: on-failure
    entrypoint: /scripts/start-prometheus.sh
    env_file:
      - .env
    volumes:
      - ./docker/prometheus:/etc/prometheus
      - prometheus_data:/prometheus
      - ./scripts/start-prometheus.sh:/scripts/start-prometheus.sh
    ports:
      - ${PORT__PROMETHEUS:-9090}:9090

  grafana:
    platform: linux/amd64
    image: grafana/grafana:${IMAGE_TAG__GRAFANA:-9.3.0}
    restart: on-failure
    entrypoint: /scripts/start-grafana.sh
    env_file:
      - ./envs/common/grafana.env
      - .env
    volumes:
      - ./docker/grafana/provisioning/:/etc/grafana/provisioning/:ro
      - ./docker/grafana/dashboards/simple_node_dashboard.json:/var/lib/grafana/dashboards/simple_node_dashboard.json
      - grafana_data:/var/lib/grafana
      - ./scripts/start-grafana.sh:/scripts/start-grafana.sh
    ports:
      - ${PORT__GRAFANA:-3000}:3000

  influxdb:
    platform: linux/amd64
    image: influxdb:${IMAGE_TAG__INFLUXDB:-1.8}
    restart: on-failure
    entrypoint: /scripts/start-influxdb.sh
    env_file:
      - ./envs/common/influxdb.env
      - .env
    volumes:
      - ./docker/influxdb/influx_init.iql:/docker-entrypoint-initdb.d/influx_init.iql
      - influxdb_data:/var/lib/influxdb
      - ./scripts/start-influxdb.sh:/scripts/start-influxdb.sh
    ports:
      - ${PORT__INFLUXDB:-8086}:8086

volumes:
  geth:
  eigenda-data:
  prometheus_data:
  grafana_data:
  influxdb_data:
  shared:
```

### <sub>Start the Node</sub>

```bash
docker compose up -d --build
```

## Monitoring

### Monitor Logs of Docker Container&#x20;

```bash
docker ps 
docker compose logs

# for Individual Containers Logs 
docker logs celo-l2-node-docker-compose-op-geth-1 
docker logs celo-l2-node-docker-compose-op-node-1
docker logs celo-l2-node-docker-compose-historical-rpc-node-1
docker logs celo-l2-node-docker-compose-eigenda-proxy-1
```

{% hint style="warning" %}
Total time required to sync is \~3 Days.
{% endhint %}

## Sync Status

### Latest Block

```bash
curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' http://localhost:9993
```

_Response should look like:_

```json
{"jsonrpc":"2.0","id":1,"result":"0x258fee5"}
```

## References

{% embed url="https://docs.celo.org/cel2/operators/run-node#running-an-archive-node" %}

{% embed url="https://celoscan.io/" %}
