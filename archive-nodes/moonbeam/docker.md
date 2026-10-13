# 🐳 Docker

Author(s): BK | E\&N/Braindexer and man4ela | catapulta.eth

## System Requirements

|                CPU               |           OS           |  RAM  |             DISK             |
| :------------------------------: | :--------------------: | :---: | :--------------------------: |
| 8 Cores (Fastest per core speed) | Debian 12/Ubuntu 22.04 | 16 GB | 2TB+ (SSD or NVME preffered) |

{% hint style="info" %}
_Moonbeam tracing node has a size of 2TB on December 23, 2024_
{% endhint %}

## Run a tracing node

{% hint style="success" %}
Geth's `debug` and `txpool` APIs and OpenEthereum's `trace` module provide non-standard RPC methods for getting a deeper insight into transaction processing. Supporting these RPC methods is important because many projects, such as [The Graph](https://thegraph.com/), rely on them to index blockchain data.



To use the supported RPC methods, you need to run a **tracing node.** This guide covers the steps on how to setup and sync a tracing node on Moonbeam using Docker.
{% endhint %}

### Pre-Requisites <a href="#pre-requisties" id="pre-requisties"></a>

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y
​
sudo apt install -y wget curl screen git ufw// Some code
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

**Create Moonbeam directory**

The first command, `mkdir moonbeam`, will create a new directory named moonbeam in the current location. The second command, `cd moonbeam`, will change your current working directory to the newly created base directory. Now you are inside the base directory and can start storing docker-compose and related files in it.

```bash
mkdir moonbeam && cd moonbeam
```

### Create .env file

```bash
sudo nano .env
```

Paste the following into the file.

<pre class="language-bash"><code class="lang-bash"><strong>EMAIL={YOUR_EMAIL} #Your email to receive SSL renewal emails
</strong>DOMAIN={YOUR_DOMAIN} #Domain should be something like rpc.mywebsite.com, e.g. moonbeam.infradao.org
WHITELIST={YOUR_REMOTE_MACHINE_IP} #the server's IP itself and comma separated list of IP's allowed to connect to RPC (e.g. Indexer)
</code></pre>

{% hint style="info" %}
`Ctrl + x` and `y` to save file
{% endhint %}

### Make Database directory and set necessary permissions

```bash
mkdir /var/lib/moonbeam-data/

sudo chown -R $(id -u):$(id -g) /var/lib/moonbeam-data
```

### Create a directory for WASM overrides:

```bash
mkdir /var/lib/moonbeam-data/moonbeam

sudo chown -R $(id -u):$(id -g) /var/lib/moonbeam-data/moonbeam
```

### Create docker-compose.yml

{% hint style="info" %}
Instead of the standard `moonbeamfoundation/moonbeam` docker image, you will use the `moonbeamfoundation/moonbeam-tracing` image.&#x20;

The latest supported version can be found on the Docker Hub for the moonbeam-tracing image from these repos: [https://hub.docker.com/r/moonbeamfoundation/moonbeam-tracing/tags](https://hub.docker.com/r/moonbeamfoundation/moonbeam-tracing/tags)
{% endhint %}

_Create and paste the following into the docker-compose.yml_

```bash
sudo nano docker-compose.yml
```

{% hint style="warning" %}
Note that you have to:

* Replace `INSERT_YOUR_NODE_NAME` in two different places. This name can be whatever you want it to be named. For InfraDAO, you can use the name of the server
* Replace `INSERT_RAM_IN_MB` for 50% of the actual RAM your server has. For example, for 32 GB RAM, the value must be set to `16000`. The minimum value is `2000`, but it is below the recommended specs
* For an overview of the flags used in the following start-up commands, plus additional commonly used flags, please refer to the [Flags](https://docs.moonbeam.network/node-operators/networks/run-a-node/flags) page of our documentation
{% endhint %}

```bash
version: '3.9'

networks:
  monitor-net:
    driver: bridge

volumes:
  traefik_letsencrypt: {}
  moonbeam-data: {}

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
      - "traefik.http.middlewares.moonbeam-ipallowlist.ipallowlist.sourcerange=${WHITELIST}"

  moonbeam:
    image: moonbeamfoundation/moonbeam-tracing:v0.42.1-3400-latest
    user: root
    container_name: moonbeam
    volumes:
      - "/var/lib/moonbeam-data:/data"
    restart: unless-stopped
    command:
      - --base-path=/data
      - --chain=moonbeam
      - --name="INSERT_YOUR_NODE_NAME"
      - --rpc-port=9944
      - --rpc-cors=all
      - --unsafe-rpc-external
      - --state-pruning=archive
      - --trie-cache-size=1073741824
      - --db-cache=64000
      - --ethapi=debug,trace,txpool
      - --wasm-runtime-overrides=/moonbeam/moonbeam-substitutes-tracing
      - --runtime-cache-size=64
      - --
      - --name="INSERT_YOUR_NODE_NAME (Embedded Relay)"
    expose:
      - 9944 # rpc + ws parachain
      - 9945 # rpc + ws relay chain
      - 22057 # p2p parachain
      - 51555 # p2p relay chain
      - 9615 # prometheus parachain
      - 9616 # prometheus relay chain
    ports:
      - "9944:9944" # rpc + ws parachain
      - "9945:9945" # rpc + ws relay chain
      - "22057:22057"
      - "51555:51555"
    networks:
      - monitor-net
    labels:
    - "traefik.enable=true"
    - "traefik.http.routers.moonbeam.middlewares=moonbeam-ipallowlist"
    - "traefik.http.routers.moonbeam.rule=Host(`$DOMAIN`)"
    - "traefik.http.routers.moonbeam.entrypoints=websecure"
    - "traefik.http.routers.moonbeam.tls.certresolver=myresolver"
    - "traefik.http.services.moonbeam.loadbalancer.server.port=9944"// Some code
```

### Monitor logs for errors

```bash
docker logs moonbeam -f --tail 100
```

The expected output after successful launch should look like this:

```bash
2024-12-24 00:50:53 Moonbeam Parachain Collator
2024-12-24 00:50:53 ✌️  version 0.42.1-1432b45b797
2024-12-24 00:50:53 ❤️  by PureStake, 2019-2024
2024-12-24 00:50:53 📋 Chain specification: Moonbeam
2024-12-24 00:50:53 🏷  Node name: "InfraDAO B"
2024-12-24 00:50:53 👤 Role: FULL
2024-12-24 00:50:53 💾 Database: RocksDb at /data/chains/moonbeam/db/full
2024-12-24 00:50:54 Parachain Account: 5Ec4AhPVjsshXjh8ynp6MwaJTJBnen3pkHiiyDhHfie5VWkN
2024-12-24 00:50:54 Is collating: no
2024-12-24 00:53:51 [🌗] Found wasm override. version=moonbeam-3101 (moonbeam-0.tx2.au3) file=/moonbeam/moonbeam-substitutes-tracing/moonbeam-runtime-3101-substitute-tracing.wasm
2024-12-24 00:53:51 [🌗] Found wasm override. version=moonbeam-2600 (moonbeam-0.tx2.au3) file=/moonbeam/moonbeam-substitutes-tracing/moonbeam-runtime-2600-substitute-tracing.wasm
```

```bash
2024-12-24 00:54:38 [🌗] ⚙️  Syncing, target=#8873589 (6 peers), best: #7954971 (0x9ffd…e535), finalized #7954817 (0xbe30…f908), ⬇ 10.1MiB/s ⬆ 13.6kiB/s
2024-12-24 00:54:43 [Relaychain] ⚙️  Syncing 31.8 bps, target=#23974979 (8 peers), best: #23054735 (0xe5bd…0b0d), finalized #23054486 (0x0b99…0fd6), ⬇ 15.5MiB/s ⬆ 94.4kiB/s
2024-12-24 00:54:43 [🌗] ⚙️  Syncing 82.2 bps, target=#8873590 (6 peers), best: #7955382 (0x7551…156c), finalized #7954817 (0xbe30…f908), ⬇ 1.8MiB/s ⬆ 1.5kiB/s
2024-12-24 00:54:48 [Relaychain] ⚙️  Syncing 28.4 bps, target=#23974979 (8 peers), best: #23054877 (0x8c1a…c76c), finalized #23054848 (0x8eac…3a21), ⬇ 2.0MiB/s ⬆ 138.9kiB/s
```

### Test Moonbeam RPC

You can call the JSON-RPC API methods to confirm the node is running. For example, call [`eth_syncing`](https://besu.hyperledger.org/public-networks/reference/api#eth_syncing) to return the synchronization status. It will return the starting, current, and highest block, or `false` if not synchronizing (or if the head of the chain has been reached)

Replace `https://{YOUR_DOMAIN}` with actual domain name you set an A record for: &#x20;

```bash
curl https://{YOUR_DOMAIN} \
        -X POST \
        -H "Content-Type: application/json" \
        -d '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}'
```

#### Expected out:

```bash
{"jsonrpc":"2.0","result":{"startingBlock":"0x0","currentBlock":"0x85a337","highestBlock":"0x876c87","warpChunksAmount":null,"warpChunksProcessed":null},"id":1}
```

## References

* **Moonbeam Tracing Node Docs**: [https://docs.moonbeam.network/node-operators/networks/tracing-node/](https://docs.moonbeam.network/node-operators/networks/tracing-node/)
* **Moonbeam Full Node Docs**: [https://docs.moonbeam.network/node-operators/networks/run-a-node/docker/](https://docs.moonbeam.network/node-operators/networks/run-a-node/docker/)
* **Moonbeam Tracing Node Docker**: [https://hub.docker.com/r/moonbeamfoundation/moonbeam-tracing/tags](https://hub.docker.com/r/moonbeamfoundation/moonbeam-tracing/tags)
