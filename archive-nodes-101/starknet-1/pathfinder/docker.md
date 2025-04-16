---
description: 'Authors: [man4ela | catapulta.eth]'
---

# 🐳 Docker

## System Requirements

<table><thead><tr><th align="center">CPU</th><th align="center">OS</th><th width="254" align="center">RAM</th><th align="center">DISK</th></tr></thead><tbody><tr><td align="center">4-Core CPU</td><td align="center">Debian 12/Ubuntu 22.04</td><td align="center">=> 8 GB RAM</td><td align="center"><p>250 GB+</p><p> (SSD or NVMe)</p></td></tr></tbody></table>

{% hint style="info" %}
_Starknet Pathfinder full node has a size of 168GB on April 10th, 2025_
{% endhint %}

{% hint style="success" %}
Pathfinder is a Rust implementation of a [Starknet](https://www.starknet.io/) full node that provides a secure view into the Starknet blockchain. As a full node, Pathfinder provides access to Starknet's entire state history, allowing users to query contract code, storage, and transactions.
{% endhint %}

{% hint style="warning" %}
## Before you start, make sure that you have your own synced Ethereum mainnet L1 RPC URL ready with WS port enabled
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

{% embed url="https://www.youtube.com/watch?ab_channel=TonyTeachesTech&v=QcNBLSSn8Vg" %}

**Create Starknet directory**

```bash
mkdir starkent && cd starknet
```

### Create .env file

```bash
sudo nano .env
```

Paste the following into the file.

<pre class="language-bash"><code class="lang-bash"><strong>EMAIL={YOUR_EMAIL} #Your email to receive SSL renewal emails
</strong>DOMAIN={YOUR_DOMAIN} #Domain should be something like rpc.mywebsite.com, e.g. starknet.infradao.org
WHITELIST={YOUR_REMOTE_MACHINE_IP} #the server's own IP and comma separated list of IP's allowed to connect to RPC (e.g. Indexer)
PATHFINDER_ETHEREUM_API_URL={YOUR_L1_RPC} #Your ready synced L1 Ethereum Mainnet node RPC endpoint set as wss://
</code></pre>

`Ctrl + x` and `y` to save file

{% hint style="danger" %}
**Pathfinder** only recognizes an endpoint through WebSocket protocol, so make sure to specify it using the `wss://` scheme (e.g., `wss://your-endpoint`)
{% endhint %}

### Launch Starknet full node

```bash
sudo nano docker-compose.yml
```

Paste the following into the `docker-compose.yml:`&#x20;

```bash
networks:
monitor-net:
driver: bridge
volumes:
traefik_letsencrypt: {}
starknet-data: {}
services:
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
- "traefik.http.middlewares.starknet-ipallowlist.ipallowlist.sourcerange=${WHITELIST}"
starknet-node:
container_name: starknet-node
image: eqlabs/pathfinder:v0.16.3
environment:
- RUST_LOG=info
env_file:
- .env
volumes:
- starknet-data:/usr/share/pathfinder/data
ports:
- "9545:9545"
restart: unless-stopped
stop_grace_period: 1m
networks:
- monitor-net
labels:
- "traefik.enable=true"
- "traefik.http.routers.starknet.rule=Host(${DOMAIN}) && PathPrefix(/rpc/v0_8)"
- "traefik.http.routers.starknet.entrypoints=websecure"
- "traefik.http.routers.starknet.tls.certresolver=myresolver"
- "traefik.http.services.starknet.loadbalancer.server.port=9545"
- "traefik.http.middlewares.starknet-stripprefix.stripprefix.prefixes=/rpc/v0_8"
- "traefik.http.routers.starknet.middlewares=starknet-ipallowlist,starknet-stripprefix"
```

```bash
sudo docker compose up -d
```

### Monitor Logs

Use `docker logs` to monitor your starknet node. The `-f` flag ensures you are following the log output

```bash
docker logs starknet-node -f --tail 100
```

Once your Pathfinder Starknet node starts syncing, the logs are expected to look like this:

```log
2025-02-14T01:00:42  INFO 🏁 Starting node. version="v0.16.3"
2025-02-14T01:00:42  INFO Pubsub service request channel closed. Shutting down.
2025-02-14T01:00:42  INFO Performing database migrations current_revision=40 latest_revision=65 migrations=25
2025-02-14T01:00:42  INFO db_migration: Creating Bloom filters for events revision=46

....

2025-02-14T01:00:42  INFO Created new database with Merkle trie pruning enabled.
2025-02-14T01:00:42  INFO Merkle trie pruning enabled history_kept=20
2025-02-14T01:00:42  INFO Database migrated. location="./mainnet.sqlite"
2025-02-14T01:00:42  INFO 📡 HTTP-RPC server started on: 0.0.0.0:9545
2025-02-14T01:00:42  INFO Pubsub service request channel closed. Shutting down.
2025-02-14T01:00:42  INFO Pubsub service request channel closed. Shutting down.
2025-02-14T01:00:42  INFO L1 sync updated to block 1148917
2025-02-14T01:00:43  INFO Updated Starknet state with block 0
2025-02-14T01:00:43  INFO Updated Starknet state with block 1
2025-02-14T01:00:43  INFO Updated Starknet state with block 2
2025-02-14T01:00:43  INFO Updated Starknet state with block 3
2025-02-14T01:00:43  INFO Updated Starknet state with block 4
```

#### 1. Check the Chain ID

This confirms which Starknet network the node is connected to:

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"starknet_chainId","params":[],"id":1}' \
  https://{DOMAIN}/rpc/v0_8
```

**Expected Response:**

```json
{"id":1,"jsonrpc":"2.0","result":"0x534e5f4d41494e"}
```

| Network | Chain ID (Hex)     | Chain ID (Decoded) |
| ------- | ------------------ | ------------------ |
| Mainnet | `0x534e5f4d41494e` | `SN_MAIN`          |

#### 2. Check the Latest Block Number

This ensures that your node is syncing correctly and producing up-to-date data.

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"starknet_blockNumber","params":[],"id":1}' \
  https://{DOMAIN}/rpc/v0_8
```

**Expected Response:**

```json
{"id":1,"jsonrpc":"2.0","result":1303384}
```

The value returned in `result`field indicates a current block number and should increase over time, confirming that your node is syncing properly.

{% hint style="success" %}
**Pathfinder Starkent node** takes **approximately 7 days** to fully catch up to the latest chain head when syncing from Genesis
{% endhint %}

#### References <a href="#references" id="references"></a>

{% embed url="https://eqlabs.github.io/pathfinder/" %}
