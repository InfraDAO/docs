---
description: 'Authors: [Vince | Nodeify]'
---

# 🐳 Docker

_Last updated at date: 12 May 2024_

## System Requirements

<table data-full-width="false"><thead><tr><th align="center">CPU</th><th width="140" align="center">OS</th><th width="180" align="center">RAM</th><th align="center">DISK</th></tr></thead><tbody><tr><td align="center">8c CPU</td><td align="center">Ubuntu 22.04</td><td align="center">>= 16GB</td><td align="center">>= 3.5TB</td></tr></tbody></table>

{% hint style="info" %}
_Note: The Base archive node consumes 3.5 TB of space on May 12.2024_
{% endhint %}

## Base 🔵

Official Docs [https://docs.base.org/guides/run-a-base-node/](https://docs.base.org/guides/run-a-base-node/)

### Pre-requisites

Update, upgrade, and clean the system, and then firewall management (ufw), Docker, and the Git version control system.

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt install docker.io docker-compose git ufw -y
```

Set explicit default UFW rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Allow SSH, HTTP and HTTPS

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80
sudo ufw allow 443
```

Get the IP address of the host machine, you can use the following command in a terminal or command prompt

```bash
curl ifconfig.me
```

Set an A record for a domain, you need to access the domain's DNS settings and create an A record that points to the IP address of the host machine. This configuration allows users to reach your domain by resolving the domain name to the specific IP address associated with your host machine.

{% hint style="info" %}
_This step can be skipped if you aren't going to use a domain name to access your RPC. The guide specifically covers the setup using a domain generated on the server_
{% endhint %}

{% embed url="https://www.youtube.com/watch?v=QcNBLSSn8Vg" %}

### Create base directory

The first command, `mkdir base`, will create a new directory named base in the current location. The second command, `cd base`, will change your current working directory to the newly created base directory. Now you are inside the base directory and can start storing docker-compose and related files in it.

```bash
mkdir base
cd base
```

### Create .env file

```bash
sudo nano .env
```

Paste the following into the file.

```bash
EMAIL={YOUR_EMAIL} #Your email to receive SSL renewal emails
DOMAIN={YOUR_DOMAIN} #Quickly register and query your free domain by entering curl -X PUT bash-st.art
WHITELIST={YOUR_REMOTE_MACHINE_IP} #the server's IP itself and IP's allowed to connect to RPC (eg. Indexer)
LAYER_1_RPC={YOUR_L1_RPC} #Your preferred L1 (Ethereum, not Base) node RPC URL
L1_BEACON={YOUR_L1_BEACON} #Your preferred L1 CL (Consensus Layer) Beacon endpoint, e.g. Lighthouse
```

```wasm
{YOUR_DOMAIN} should look something like 117-230-108-65.bash-st.art
```

{% hint style="info" %}
ctrl + x and y to save file
{% endhint %}

Ensure you have an Ethereum L1 full node RPC available. It needs to be synced before Base will be able to fully sync

### Make configuration directory

```
mkdir config
cd config
```

### Download genesis.json and rollup.json

```
curl -LO https://raw.githubusercontent.com/base-org/node/main/mainnet/genesis-l2.json
curl -LO https://raw.githubusercontent.com/base-org/node/main/mainnet/rollup.json
```

Create `base_geth_data` docker volume

```
docker volume create base_geth_data
```

### Initialize Geth

{% hint style="warning" %}
It's required to initialize Geth if you plan to sync from scratch.

If you are going to sync using the snapshot, you shouldn't need to initialize Geth. The docker-compose would do the trick
{% endhint %}

This command runs a Docker container using the `op-geth` image. It mounts two volumes: `base_geth_data` to `/data` inside the container and `/root/base/config` to `/config`. The container then initializes the Ethereum client with a genesis file located at `/config/genesis-l2.json` using the `--datadir` option to specify the data directory as `/data`

```
docker run -v base_geth_data:/data -v /root/base/config:/config us-docker.pkg.dev/oplabs-tools-artifacts/images/op-geth:b5fecf58ec77909c70bd15b96b32b593af0b38ff --datadir=/data init /config/genesis-l2.json
```

{% hint style="info" %}
You should receive a quick output that your genesis file has been initialized.&#x20;

`INFO [08-14|19:31:15.573] Successfully wrote genesis state`
{% endhint %}

### Create jwt.hex

```
openssl rand -hex 32 | tr -d "\n" > "./jwt.hex"
```

### Create docker-compose.yml

```bash
cd ~/base
sudo nano docker-compose.yml
```

Assuming that this guide is current, you’ll be able to paste the following into the docker-compose.yml and then ctrl + x and y to save file. The more likely scenario is that this .yml template is a bit outdated and you will need to update the version under the opnode > image as well as the geth > image sections. You can find the latest releases of the op-node and geth nodes here: [https://docs.optimism.io/builders/node-operators/releases](https://docs.optimism.io/builders/node-operators/releases).

Paste the following into the docker-compose.yml

```docker
version: '3.8'

networks:
  monitor-net:
    driver: bridge

volumes:
    geth_data: {}
    traefik_letsencrypt: {}

services:

######################################################################################
#####################         TRAEFIK PROXY CONTAINER          #######################
######################################################################################     

  traefik:
    image: traefik:latest
    container_name: traefik
    restart: always
    ports:
      - "443:443"
    networks:
      - monitor-net
    command:
      - "--api=true"
      - "--api.insecure=true"
      - "--api.dashboard=true"
      - "--log.level=DEBUG"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.myresolver.acme.tlschallenge=true"
      - "--certificatesresolvers.myresolver.acme.email=$EMAIL"
      - "--certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json"
    volumes:
      - "traefik_letsencrypt:/letsencrypt"
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
    labels:
      - "traefik.enable=true"
      - "traefik.http.middlewares.ipwhitelist.ipwhitelist.sourcerange=$WHITELIST"

######################################################################################
#####################            OP-NODE CONTAINER             #######################
###################################################################################### 

  opnode:
    image: us-docker.pkg.dev/oplabs-tools-artifacts/images/op-node:v1.7.0
    container_name: opnode
    networks:
     - monitor-net
    restart: unless-stopped
    expose:
      - "8545" # RPC
      - "7300" # METRICS
    ports:
      - "9222:9222"     # P2P TCP
      - "9222:9222/udp" # P2P UDP
    volumes:
      - ./config/rollup.json:/mainnet/rollup.json
      - ./config/genesis-l2.json:/mainnet/genesis-l2.json
      - ./config/jwt.hex:/root/jwt/jwt.hex:ro
    environment:
      - OP_GETH_GENESIS_FILE_PATH=/mainnet/genesis-l2.json
      - OP_GETH_SEQUENCER_HTTP=https://mainnet-sequencer.base.org
      - OP_NODE_L1_ETH_RPC=${LAYER_1_RPC}
      - OP_NODE_L1_BEACON=${L1_BEACON}
      - OP_NODE_L2_ENGINE_AUTH=/root/jwt/jwt.hex
      - OP_NODE_L2_ENGINE_RPC=http://geth:8551
      - OP_NODE_LOG_LEVEL=info
      - OP_NODE_METRICS_ADDR=0.0.0.0
      - OP_NODE_METRICS_ENABLED=true
      - OP_NODE_METRICS_PORT=7300
      - OP_NODE_NETWORK=base-mainnet
      - OP_NODE_P2P_AGENT=base
      - OP_NODE_P2P_BOOTNODES=enr:-J24QNz9lbrKbN4iSmmjtnr7SjUMk4zB7f1krHZcTZx-JRKZd0kA2gjufUROD6T3sOWDVDnFJRvqBBo62zuF-hYCohOGAYiOoEyEgmlkgnY0gmlwhAPniryHb3BzdGFja4OFQgCJc2VjcDI1NmsxoQKNVFlCxh_B-716tTs-h1vMzZkSs1FTu_OYTNjgufplG4N0Y3CCJAaDdWRwgiQG,enr:-J24QH-f1wt99sfpHy4c0QJM-NfmsIfmlLAMMcgZCUEgKG_BBYFc6FwYgaMJMQN5dsRBJApIok0jFn-9CS842lGpLmqGAYiOoDRAgmlkgnY0gmlwhLhIgb2Hb3BzdGFja4OFQgCJc2VjcDI1NmsxoQJ9FTIv8B9myn1MWaC_2lJ-sMoeCDkusCsk4BYHjjCq04N0Y3CCJAaDdWRwgiQG,enr:-J24QDXyyxvQYsd0yfsN0cRr1lZ1N11zGTplMNlW4xNEc7LkPXh0NAJ9iSOVdRO95GPYAIc6xmyoCCG6_0JxdL3a0zaGAYiOoAjFgmlkgnY0gmlwhAPckbGHb3BzdGFja4OFQgCJc2VjcDI1NmsxoQJwoS7tzwxqXSyFL7g0JM-KWVbgvjfB8JA__T7yY_cYboN0Y3CCJAaDdWRwgiQG,enr:-J24QHmGyBwUZXIcsGYMaUqGGSl4CFdx9Tozu-vQCn5bHIQbR7On7dZbU61vYvfrJr30t0iahSqhc64J46MnUO2JvQaGAYiOoCKKgmlkgnY0gmlwhAPnCzSHb3BzdGFja4OFQgCJc2VjcDI1NmsxoQINc4fSijfbNIiGhcgvwjsjxVFJHUstK9L1T8OTKUjgloN0Y3CCJAaDdWRwgiQG,enr:-J24QG3ypT4xSu0gjb5PABCmVxZqBjVw9ca7pvsI8jl4KATYAnxBmfkaIuEqy9sKvDHKuNCsy57WwK9wTt2aQgcaDDyGAYiOoGAXgmlkgnY0gmlwhDbGmZaHb3BzdGFja4OFQgCJc2VjcDI1NmsxoQIeAK_--tcLEiu7HvoUlbV52MspE0uCocsx1f_rYvRenIN0Y3CCJAaDdWRwgiQG
      - OP_NODE_P2P_LISTEN_IP=0.0.0.0
      - OP_NODE_P2P_LISTEN_TCP_PORT=9222
      - OP_NODE_P2P_LISTEN_UDP_PORT=9222
      - OP_NODE_ROLLUP_CONFIG=/mainnet/rollup.json
      - OP_NODE_RPC_ADDR=0.0.0.0
      - OP_NODE_RPC_PORT=8545
      - OP_NODE_SNAPSHOT_LOG=/tmp/op-node-snapshot-log
      - OP_NODE_VERIFIER_L1_CONFS=4
      - OP_NODE_L1_TRUST_RPC=true

######################################################################################
#####################            GETH CONTAINER             #######################
######################################################################################

  geth:
    image: us-docker.pkg.dev/oplabs-tools-artifacts/images/op-geth:v1.101308.2
    container_name: geth
    restart: unless-stopped
    networks:
      - monitor-net
    expose:
      - "8545"       # RPC
      - "8546"       # websocket
      - "7300"       # metrics
    ports:
      - "30303:30303" # Peers
      - "30303:30303/udp" # Peers
    volumes:
      - ./config/jwt.hex:/root/jwt/jwt.hex:ro
      - geth_data:/data
    command:
      - --datadir=/data
      - --verbosity=3
      - --http
      - --http.corsdomain=*
      - --http.vhosts=*
      - --http.addr=0.0.0.0
      - --http.port=8545
      - --http.api=web3,debug,eth,txpool,net,engine
      - --authrpc.addr=0.0.0.0
      - --authrpc.port=8551
      - --authrpc.vhosts=*
      - --authrpc.jwtsecret=/root/jwt/jwt.hex
      - --ws
      - --ws.addr=0.0.0.0
      - --ws.port=8546
      - --ws.origins=*
      - --ws.api=debug,eth,txpool,net,engine
      - --metrics
      - --metrics.addr=0.0.0.0
      - --metrics.port=7300
      - --syncmode=full
      - --gcmode=archive
      - --nodiscover
      - --maxpeers=100
      - --networkid=8453
      - --nat=extip:0.0.0.0
      - --rollup.sequencerhttp=https://mainnet-sequencer.base.org
    labels:
      - "traefik.enable=true"
      - "traefik.http.middlewares.base-stripprefix.stripprefix.prefixes=/geth"
      - "traefik.http.routers.base.service=base" #https
      - "traefik.http.services.base.loadbalancer.server.port=8545"
      - "traefik.http.routers.base.entrypoints=websecure"
      - "traefik.http.routers.base.tls.certresolver=myresolver"
      - "traefik.http.routers.base.rule=Host(`$DOMAIN`) && PathPrefix(`/geth`)"
      - "traefik.http.routers.base.middlewares=base-stripprefix, ipwhitelist"
```

{% hint style="info" %}
ctrl + x and y to save file
{% endhint %}

### Optional/Recommended: Download Base Snapshot

{% hint style="success" %}
This is an optional step based on whether you want to sync the node from scratch or sync the node from a snapshot. Based on InfraDAO’s experience, we recommend downloading a snapshot and syncing the node from that snapshot. Syncing the node from the scratch took around two weeks while the snapshot took requires 9-10 hours to download, 4-6 hours to unzip, and roughly 24 hours to sync from that point.
{% endhint %}

To sync from a snapshot, visit the Base Docs to validate the recommended approach for restoring from snapshot: [https://docs.base.org/tutorials/run-a-base-node/#snapshots](https://docs.base.org/tutorials/run-a-base-node/#snapshots). Next, in the home directory of your (i.e. the `base` folder), create a folder named `geth-data`. If you already have this folder, remove it to clear the existing state and then recreate it. Next, run the following code and wait for the operation to complete.

As downloading a snapshot takes about 9 hrs it is better to run it in a screen session

```
screen -S archive
```

Use `aria2c` to download the most recent Mainnet Archive Snapshot

<pre class="language-bash" data-overflow="wrap"><code class="lang-bash"><strong>cd ~/base
</strong>
aria2c --file-allocation=none -c -x 10 -s 10 "https://base-snapshots-mainnet-archive.s3.amazonaws.com/$(curl https://base-snapshots-mainnet-archive.s3.amazonaws.com/latest)"
</code></pre>

_press `ctrl+A and D` to return to previous screen and continue installation_

```bash
screen -r archive #will bring you back to monitor downloading progress
```

You'll then need to untar the downloaded snapshot and place the geth subfolder inside of it in the geth-data folder you created (unless you changed the location of your data directory)

```bash
tar -xvzf filename.tar.gz
# tar -xvzf base-mainnet-archive-1712388985.tar.gz
```

Next, you’ll need to move the snapshot to the where the geth data was stored in the docker container. If you initially tried to sync the node from scratch and are now trying with a snapshot:

```bash
cd /var/lib/docker/volumes/base_geth_data/_data 

rm -rf geth

cd ~/base/snapshots/mainnet/download/

mv ~/base/snapshots/mainnet/download/geth /var/lib/docker/volumes/base_geth_data/_data/

cd /var/lib/docker/volumes/base_geth_data/_data/

ls
```

### Run Base Node

```bash
cd ~/base

docker-compose up -d
```

### Monitor Logs

Use `docker logs` to monitor your geth and op nodes. The `-f` flag ensures you are following the log output

```
docker logs geth -f --tail 100

docker logs opnode -f --tail 100
```

## Test Base RPC 🧪

{% code overflow="wrap" %}
```bash
curl --data '{"method":"eth_syncing","params":[],"id":1,"jsonrpc":"2.0"}' -H "Content-Type: application/json" -X POST https://{DOMAIN}/geth
```
{% endcode %}

The result will return `false` if a node is fully synced

**Alternatively you can run**

```
curl -d '{"id":0,"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["latest",false]}' \
  -H "Content-Type: application/json" https://{DOMAIN}/geth
```

{% hint style="warning" %}
Sync speed will be highly dependent on your Layer 1 RPC
{% endhint %}
