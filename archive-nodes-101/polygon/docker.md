---
description: 'Authors: [Vince | Nodeify]'
---

# 🐳 Docker

## System Requirements

<table data-full-width="false"><thead><tr><th align="center">CPU</th><th width="140" align="center">OS</th><th width="180" align="center">RAM</th><th align="center">DISK</th></tr></thead><tbody><tr><td align="center">16c CPU</td><td align="center">Ubuntu 22.04</td><td align="center">>= 32GB</td><td align="center">>= 10TB SSD/NVME</td></tr></tbody></table>

## Polygon ♾️

Official Docs [https://wiki.polygon.technology/docs/pos/operate/node/archive-node/](https://wiki.polygon.technology/docs/pos/operate/node/archive-node/)

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

{% embed url="https://www.youtube.com/watch?v=QcNBLSSn8Vg" %}

### Create Polygon directory

The first command, `mkdir polygon`, will create a new directory named polygon in the current location. The second command, `cd polygon`, will change your current working directory to the newly created polygon directory. Now you are inside the polygon directory and can start storing docker-compose and related files in it.

```bash
mkdir polygon
cd polygon
```

### Create .env file

```bash
sudo nano .env
```

Paste the following into the file.

```bash
EMAIL={YOUR_EMAIL} #Your email to receive SSL renewal emails
DOMAIN={YOUR_DOMAIN} #Domain of your reth node you set earlier, polygon.indexerdao.com
WHITELIST={YOUR_REMOTE_MACHINE_IP} # Remote IP's allowed to connect to RPC
```

{% hint style="info" %}
ctrl + x and y to save file
{% endhint %}

### Create heimdall  and erigon directories

```
mkdir /var/lib/heimdall
mkdir /var/lib/erigon
```

### Initialize Heimdall

Run a Docker container from the `0xpolygon/heimdall` image. By using the `-v` flag, you're mapping the host directory `/var/lib/heimdall` to the container's `/root/.heimdalld` directory. The `init --home /root/.heimdalld` command initializes configurations and data on the host machine.

```
docker run -it -v /var/lib/heimdall:/root/.heimdalld 0xpolygon/heimdall:latest init --home /root/.heimdalld
```

{% hint style="info" %}
You should now have `/var/lib/heimdall/config` and `/var/lib/heimdall/data` directories on the host.
{% endhint %}

### Download genesis.json to `/var/lib/heimdall/config`

```
wget https://raw.githubusercontent.com/maticnetwork/launch/master/mainnet-v1/without-sentry/heimdall/config/genesis.json -O /var/lib/heimdall/config/genesis.json
```

### Modify seeds and cors\_allowed\_origins

```
sed -i '/^seeds/c\seeds = "f4f605d60b8ffaaf15240564e58a81103510631c@159.203.9.164:26656,4fb1bc820088764a564d4f66bba1963d47d82329@44.232.55.71:26656,2eadba4be3ce47ac8db0a3538cb923b57b41c927@35.199.4.13:26656,3b23b20017a6f348d329c102ddc0088f0a10a444@35.221.13.28:26656,25f5f65a09c56e9f1d2d90618aa70cd358aa68da@35.230.116.151:26656"' /var/lib/heimdall/config/config.toml
sed -i 's/cors_allowed_origins = \[\]/cors_allowed_origins = ["*"]/g' /var/lib/heimdall/config/config.toml
```

### Create docker-compose.yml

```bash
sudo nano docker-compose.yml
```

Paste the following into the docker-compose.yml

```docker
version: '3.8'

networks:
  monitor-net:
    driver: bridge

volumes:
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
#####################              ERIGON CONTAINER            #######################
###################################################################################### 

  erigon:
    image: thorax/erigon:v2.52.0
    container_name: erigon
    restart: unless-stopped
    user: root
    expose:
      - "8545" #rpc
      - "8551" #auth
      - "6060" #metrics
      - "9090" #api
    ports:
      - 30303:30303 #p2p
      - 30303:30303/udp #p2p
      - 30304:30304 #p2p
      - 30304:30304/udp #p2p
      - 42069:42069 #torrent
      - 42069:42069/udp #torrent
      - 4000:4000/udp #lightclient discovery
      - 4001:4001 #lightclient tcp
    networks:
      - monitor-net
    command: >
      --chain bor-mainnet
      --datadir /root/.local/share/erigon
      --bor.heimdall=http://heimdallr:1317
      --torrent.download.rate 1024mb
      --metrics
      --metrics.addr 0.0.0.0
      --metrics.port 6060
      --http.addr 0.0.0.0
      --http.port 8545
      --http.vhosts *
      --http.api eth,debug,net,trace
      --db.pagesize 16kb
      --batchSize 2048MB
      --etl.bufferSize 512MB
      --rpc.returndata.limit 1000000
      --snapshots=false
      --maxpeers 200
      --sentry.drop-useless-peers=true
      --db.size.limit=12TB
    volumes:
      - /var/lib/erigon:/root/.local/share/erigon:rw
    labels:
      - "traefik.enable=true"
      - "traefik.http.services.erigon.loadbalancer.server.port=8545"
      - "traefik.http.routers.erigon.entrypoints=websecure"
      - "traefik.http.routers.erigon.tls.certresolver=myresolver"
      - "traefik.http.routers.erigon.rule=Host(`$DOMAIN`)"
      - "traefik.http.routers.erigon.middlewares=ipwhitelist"

######################################################################################
#####################              HEIMDALLD CONTAINER          #######################
######################################################################################

  heimdalld:
    image: 0xpolygon/heimdall:1.0.2
    container_name: heimdalld
    restart: unless-stopped
    expose:
      - "26657" # RPC
      - "1317" # REST
    ports:
      - "26656:26656" # P2P
    networks:
      - monitor-net
    command: |
      - start
      - --home /root/.heimdalld
      - --p2p.laddr=tcp://0.0.0.0:26656
      - --rpc.laddr=tcp://0.0.0.0:26657
      - --chain=mainnet
      - --rest-server
    volumes:
      - /var/lib/heimdall:/root/.heimdalld:rw
```

{% hint style="info" %}
ctrl + x and y to save file
{% endhint %}

### Run Polygon Node

```bash
docker-compose up -d
```

### Monitor Logs

Use `docker logs` to monitor your erigon and heimdall nodes. The `-f` flag ensures you are following the log output

```
docker logs erigon -f
docker logs heimdalld -f
docker logs heimdallr -f
```

## Test Polygon RPC 🧪

{% code overflow="wrap" %}
```bash
curl --data '{"method":"eth_syncing","params":[],"id":1,"jsonrpc":"2.0"}' -H "Content-Type: application/json" -X POST https://{DOMAIN}
```
{% endcode %}
