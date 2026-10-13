---
description: 'Authors: [man4ela | catapulta.eth]'
---

# 🐋 Op-Reth

## System Requirements

<table><thead><tr><th align="center">CPU</th><th align="center">OS</th><th width="254" align="center">RAM</th><th align="center">DISK</th></tr></thead><tbody><tr><td align="center">8-Core CPU</td><td align="center">Debian 12/Ubuntu 22.04</td><td align="center">=> 16 GB RAM</td><td align="center"><p>1 TB+</p><p> (NVMe)</p></td></tr></tbody></table>

{% hint style="info" %}
_Op-Reth Base Sepolia archive node has a size of 637GB on March 10th, 2025_
{% endhint %}

{% hint style="success" %}
Base is a secure, low-cost Ethereum L2 built on Optimism’s open-source [OP Stack](https://stack.optimism.io/). In this guide, we cover docker installation of `op-reth` and `op-node`to facilitate the node's synchronization on Sepolia Testnet Network. This method has proved to sync an archive node successfully in \~24 hours using the official snapshot provided by the Base team
{% endhint %}

{% hint style="warning" %}
## Before you start, make sure that you have your own synced Ethereum Sepolia L1 RPC URL and L1 Consensus Layer Beacon endpoint (e.g. Lighthouse Sepolia) ready
{% endhint %}

### Pre-Requisites <a href="#pre-requisties" id="pre-requisties"></a>

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y
​
sudo apt install -y wget curl screen git ufw
```

### Setting up Firewall <a href="#setting-up-firewall" id="setting-up-firewall"></a>

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw allow 22/tcp
sudo ufw allow 80
sudo ufw allow 443
```

### Enable Firewall

```bash
sudo ufw enable
```

## Install Docker

#### Run this command to remove any conflicting docker

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

#### Add Docker's official GPG key:

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

#### Add the repository to ppt sources:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
sudo apt-get update
```

#### Install docker

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Test docker is working
sudo docker run hello-world

#Install docker compose

sudo apt-get update
sudo apt-get install docker-compose-plugin

# Test the docker version
docker compose version
```

## Setting up a domain name to access RPC

Get the IP address of the host machine, you can use the following command in a terminal or command prompt

```bash
curl ifconfig.me
```

Set an A record for a domain, you need to access the domain's DNS settings and create an A record that points to the IP address of the host machine. This configuration allows users to reach your domain by resolving the domain name to the specific IP address associated with your host machine.

{% embed url="https://youtu.be/QcNBLSSn8Vg" %}

**Create Base Sepolia directory**

```bash
mkdir base-sepolia && cd base-sepolia
```

### Create .env file

```bash
sudo nano .env
```

Paste the following into the file.

<pre class="language-bash"><code class="lang-bash"><strong>EMAIL={YOUR_EMAIL} #Your email to receive SSL renewal emails
</strong>DOMAIN={YOUR_DOMAIN} #Domain should be something like rpc.mywebsite.com, e.g. base-sepolia.infradao.org
WHITELIST={YOUR_REMOTE_MACHINE_IP} #the server's own IP and comma separated list of IP's allowed to connect to RPC (e.g. Indexer)
LAYER_1_RPC={YOUR_L1_RPC} #Your ready synced L1 Ethereum Sepolia node RPC endpoint
L1_BEACON={YOUR_L1_BEACON} #Your synced L1 CL (Consensus Layer) Beacon endpoint, e.g. Lighthouse (Prysm, Lodestar) Sepolia
</code></pre>

{% hint style="info" %}
`Ctrl + x` and `y` to save file
{% endhint %}

### Create JWT secret file

```bash
mkdir -p /root/data/base-sepolia/op-reth/ && cd /root/data/base-sepolia/op-reth/

openssl rand -hex 32 | tr -d "\n" > "./jwt.hex"
```

### Optional/Recommended: Download Base Snapshot

{% hint style="success" %}
This is an optional step based on whether you want to sync the node from scratch or sync the node from a snapshot. Based on InfraDAO’s experience, we recommend downloading a snapshot and syncing the node from that snapshot. Syncing the node from the scratch will take weeks while the snapshot requires a couple of hours to download, some time to extract, and roughly 24 hours to sync from that point.
{% endhint %}

To sync from a snapshot, visit the Base Docs to validate the recommended approach for restoring from snapshot: [https://docs.base.org/chain/run-a-base-node](https://docs.base.org/chain/run-a-base-node).

As downloading a snapshot takes some time it is good idea to run it in a screen session

```
screen -S reth
```

Use `aria2c` to download the most recent **Base Sepolia Reth Archive** Snapshot

<pre class="language-bash" data-overflow="wrap"><code class="lang-bash"><strong>cd /root/base-sepolia
</strong>
aria2c --file-allocation=none -c -x 15 -s 15 https://sepolia-reth-archive-snapshots.base.org/$(curl https://sepolia-reth-archive-snapshots.base.org/latest)
</code></pre>

_press `ctrl+A and D` to return to previous screen and continue installation_

```bash
screen -r reth #will bring you back to monitor downloading progress
```

You'll need to extract the downloaded snapshot and move its contents to the `op-reth-data` directory, where Docker stores persistent data.

```bash
zstd -d base-sepolia-reth-1741404553.tar.zst -c | tar xvf -
# replace the archive with an actual name
```

{% hint style="warning" %}
If you initially tried to sync the node from scratch and are now trying with a snapshot make sure to empty the destination directory first:
{% endhint %}

<pre class="language-bash"><code class="lang-bash"><strong>cd /var/lib/docker/volumes/reth_op-reth_data/_data 
</strong>
ls 

#if directory isn't empty remove all contents

rm -rf blobstore  db  discovery-secret  invalid_block_hooks  known-peers.json  reth.toml  static_files

mv /root/base-sepolia/snapshots/sepolia/download/* /var/lib/docker/volumes/reth_op-reth_data/_data/

cd /var/lib/docker/volumes/reth_op-reth_data/_data/

ls
</code></pre>

{% hint style="warning" %}
If you haven't started the node yet, create `op-reth-data directory first:`

```bash
docker volume create reth_op-reth_data

cd /var/lib/docker/volumes/reth_op-reth_data/_data

mv /root/base-sepolia/snapshots/sepolia/download/* /var/lib/docker/volumes/reth_op-reth_data/_data/

ls
```
{% endhint %}

### Launch Base Sepolia

```bash
cd /root/base-sepolia

sudo nano docker-compose.yml
```

Paste the following into the `docker-compose.yml:`

```bash
networks:
  monitor-net:
    driver: bridge

volumes:
  op-reth_data: {}
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
      - "--certificatesresolvers.myresolver.acme.email=${EMAIL}"
      - "--certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json"
    volumes:
      - "traefik_letsencrypt:/letsencrypt"
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
    labels:
      - "traefik.enable=true"
      - "traefik.http.middlewares.ipwhitelist.ipwhitelist.sourcerange=${WHITELIST}"

  ######################################################################################
  #####################            OP-NODE CONTAINER             #######################
  ######################################################################################

  opnode:
    image: us-docker.pkg.dev/oplabs-tools-artifacts/images/op-node:v1.11.1
    container_name: opnode
    user: root  # Run as root
    networks:
      - monitor-net
    restart: unless-stopped
    expose:
      - "7545"  # RPC
      - "7373"  # METRICS
    ports:
      - "9222:9222"      # P2P TCP
      - "9222:9222/udp"  # P2P UDP
    volumes:
      - /root/data/base-sepolia/op-reth/jwt.hex:/root/data/base-sepolia/op-reth/jwt.hex:ro
    environment:
      - OP_GETH_SEQUENCER_HTTP=https://sepolia-sequencer.base.org
      - OP_SEQUENCER_HTTP=https://sepolia-sequencer.base.org
      - OP_NODE_NETWORK=base-sepolia
      - OP_NODE_L1_ETH_RPC=${LAYER_1_RPC}
      - OP_NODE_L1_BEACON=${L1_BEACON}
      - OP_NODE_L2_ENGINE_AUTH=/root/data/base-sepolia/op-reth/jwt.hex
      - OP_NODE_L2_ENGINE_RPC=http://op-reth:8551
      - OP_NODE_LOG_LEVEL=info
      - OP_NODE_METRICS_ADDR=0.0.0.0
      - OP_NODE_METRICS_ENABLED=true
      - OP_NODE_METRICS_PORT=7373
      - OP_NODE_P2P_AGENT=base
      - OP_NODE_P2P_LISTEN_IP=0.0.0.0
      - OP_NODE_P2P_LISTEN_TCP_PORT=9222
      - OP_NODE_P2P_LISTEN_UDP_PORT=9222
      - OP_NODE_ROLLUP_LOAD_PROTOCOL_VERSIONS=true
      - OP_NODE_RPC_ADDR=0.0.0.0
      - OP_NODE_RPC_PORT=7545
      - OP_NODE_SNAPSHOT_LOG=/tmp/op-node-snapshot-log
      - OP_NODE_VERIFIER_L1_CONFS=4
      - OP_NODE_L1_TRUST_RPC=true
      - OP_NODE_P2P_BOOTNODES=enr:-J64QBwRIWAco7lv6jImSOjPU_W266lHXzpAS5YOh7WmgTyBZkgLgOwo_mxKJq3wz2XRbsoBItbv1dCyjIoNq67mFguGAYrTxM42gmlkgnY0gmlwhBLSsHKHb3BzdGFja4S0lAUAiXNlY3AyNTZrMaEDmoWSi8hcsRpQf2eJsNUx-sqv6fH4btmo2HsAzZFAKnKDdGNwgiQGg3VkcIIkBg,enr:-J64QFa3qMsONLGphfjEkeYyF6Jkil_jCuJmm7_a42ckZeUQGLVzrzstZNb1dgBp1GGx9bzImq5VxJLP-BaptZThGiWGAYrTytOvgmlkgnY0gmlwhGsV-zeHb3BzdGFja4S0lAUAiXNlY3AyNTZrMaEDahfSECTIS_cXyZ8IyNf4leANlZnrsMEWTkEYxf4GMCmDdGNwgiQGg3VkcIIkBg

  ######################################################################################
  #####################            OP-RETH CONTAINER            ######################
  ######################################################################################

  op-reth:
    image: ghcr.io/paradigmxyz/op-reth:v1.1.5
    container_name: op-reth
    user: root  # Run as root
    restart: unless-stopped
    networks:
      - monitor-net
    expose:
      - "8545"  # RPC
      - "8546"  # WebSocket
      - "7300"  # Metrics
      - "8551"  # AuthRPC
    ports:
      - "30303:30303"      # Peers
      - "30303:30303/udp"  # Peers
      - "8551:8551"
    volumes:
      - /root/data/base-sepolia/op-reth/jwt.hex:/root/data/base-sepolia/op-reth/jwt.hex:ro
      - op-reth_data:/data
    command: node --authrpc.jwtsecret=/root/data/base-sepolia/op-reth/jwt.hex --datadir=/data --log.stdout.format log-fmt --ws --ws.origins="*" --ws.addr=0.0.0.0 --ws.port=8546 --ws.api="debug,eth,net,trace,txpool,web3,rpc,reth,admin" --http --http.corsdomain="*" --http.addr=0.0.0.0 --http.port=8545 --http.api="debug,eth,net,trace,txpool,web3,rpc,reth,admin" --authrpc.addr=0.0.0.0 --authrpc.port=8551 --metrics=0.0.0.0:7300 --chain=base-sepolia --rollup.sequencer-http=https://sepolia-sequencer.base.org --rollup.disable-tx-pool-gossip
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.base.service=base"
      - "traefik.http.services.base.loadbalancer.server.port=8545"
      - "traefik.http.routers.base.entrypoints=websecure"
      - "traefik.http.routers.base.tls.certresolver=myresolver"
      - "traefik.http.routers.base.rule=Host(`${DOMAIN}`)"
      - "traefik.http.routers.base.middlewares=ipwhitelist"
```

```bash

docker compose up -d
```

### Monitor Logs

Use `docker logs` to monitor your op-reth and op-node. The `-f` flag ensures you are following the log output

<pre><code>docker logs op-reth -f --tail 100

<strong>docker logs opnode -f --tail 100
</strong></code></pre>

Once your Base node starts syncing, the logs should look like this:

for op-reth:

```bash
ts=2025-03-10T02:53:38.709075686Z level=info target=reth_node_events::node message="Canonical chain committed" number=22903465 hash=0xa134676a4ed89f82eb1ce0ef5cf6fd58b0041e5e5c64e2f54cbc39a2bcfc34bf elapsed=132.57µs
ts=2025-03-10T02:53:40.4913852Z level=info target=reth_node_events::node message="Block added to canonical chain" number=22903466 hash=0x14dad5067b691a304f9b4167834cfd7664ffabf5dc87aef941e2854a60864158 peers=39 txs=77 gas="15.59 Mgas" gas_throughput="312.27 Mgas/second" full=26.0% base_fee=0.00gwei blobs=0 excess_blobs=0 elapsed=49.936088ms
```

for op-node:

```bash
t=2025-03-10T02:52:20+0000 lvl=info msg="Sync progress" reason="new chain head block" 
l2_finalized=0xaefc5cf3494778144b7be0b0f60ec021f62be261fe436408ec444d5beb96f59b:22902892 l2_safe=0x68567e5cbacbb9d765746e6057e15c52b20b403131f8910467cc1566deb10a8d:22903391 l2_pending_safe=0x68567e5cbacbb9d765746e6057e15c52b20b403131f8910467cc1566deb10a8d:22903391 
l2_unsafe=0x676daa0eff1ab38bc20afeae069231b7cb870d3280a89a4a0aa756f05be9a361:22903426 l2_backup_unsafe=0x0000000000000000000000000000000000000000000000000000000000000000:0 l2_time=1741575140
t=2025-03-10T02:52:20+0000 lvl=info msg="successfully processed payload" ref=0x676daa0eff1ab38bc20afeae069231b7cb870d3280a89a4a0aa756f05be9a361:22903426 txs=92
```

{% code overflow="wrap" %}
```bash
curl --data '{"method":"eth_syncing","params":[],"id":1,"jsonrpc":"2.0"}' -H "Content-Type: application/json" -X POST https://{YOUR_DOMAIN}
```
{% endcode %}

The result will return `false` if a node is fully synced

**Alternatively you can run**

```bash
curl -d '{"id":0,"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["latest",false]}' \
  -H "Content-Type: application/json" https://{YOUR_DOMAIN}
```

and it will return more details about syncing progress

{% hint style="warning" %}
Sync speed will be highly dependent on your Layer 1 RPC
{% endhint %}

### References

{% embed url="https://docs.base.org/chain/run-a-base-node" %}
