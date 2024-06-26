# 🐳 Docker

Author: \[man4ela | catapulta.eth]&#x20;

_Last updated at date: 15 May 2024_

#### System Requirements <a href="#system-requirements" id="system-requirements"></a>

| CPU                      | OS           | RAM   | DISK     |
| ------------------------ | ------------ | ----- | -------- |
| A fast CPU with 4+ cores | Ubuntu 22.04 | 16GB+ | >= 1.5TB |

{% hint style="info" %}
_Note: The Linea archive node consumes 1.5 TB of space on May 14.2024_
{% endhint %}

## 🔲 Linea

* [Official Docs](https://docs.linea.build/build-on-linea/run-a-node#step-3-1)

### Pre-requisites

Update, upgrade, and clean the system, and then firewall management (ufw), Docker, and the Git version control system.

```
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt install docker.io docker-compose git ufw -y
```

Set explicit default UFW rules

```
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Allow SSH, HTTP and HTTPS

```
sudo ufw allow 22/tcp
sudo ufw allow 80
sudo ufw allow 443
```

Enable Firewall

```
sudo ufw enable
```

## Setting up a domain name to access RPC

Get the IP address of the host machine, you can use the following command in a terminal or command prompt

```
curl ifconfig.me
```

Set an A record for a domain, you need to access the domain's DNS settings and create an A record that points to the IP address of the host machine. This configuration allows users to reach your domain by resolving the domain name to the specific IP address associated with your host machine.

{% embed url="https://youtu.be/QcNBLSSn8Vg" %}

### Create Linea directory

The first command, `mkdir Linea`, will create a new directory named Linea in the current location. The second command, `cd Linea`, will change your current working directory to the newly created base directory. Now you are inside the base directory and can start storing docker-compose and related files in it.

```
mkdir Linea
cd Linea
```

### Create .env file

```
sudo nano .env
```

Paste the following into the file.

```
EMAIL={YOUR_EMAIL} #Your email to receive SSL renewal emails
DOMAIN={YOUR_DOMAIN} #Domain should be something like rpc.mywebsite.com, e.g. linea.infradao.org
WHITELIST={YOUR_REMOTE_MACHINE_IP} #the server's IP itself and comma separated list of IP's allowed to connect to RPC (e.g. Indexer)
```

{% hint style="info" %}
ctrl + x and y to save file
{% endhint %}

#### Make configuration directory

```
mkdir config

cd config
```

#### Download genesis.json

```
curl -LO https://docs.linea.build/files/geth/mainnet/genesis.json
```

### Create docker-compose.yml

_Return to Linea directory_

```
cd ~/linea
```

_Create and paste the following into the docker-compose.yml_

```
sudo nano docker-compose.yml
```

```
version: '3.9'

networks:
  monitor-net:
    driver: bridge

volumes:
  traefik_letsencrypt: {}
  linea-mainnet: {}

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
      - "traefik.http.middlewares.ipwhitelist.ipwhitelist.sourcerange=${WHITELIST}"

  init:
    image: ethereum/client-go:v1.13.4
    container_name: linea-init
    command:
      - init
      - /genesis.json
    volumes:
      - ./config/genesis.json:/genesis.json:ro
      - linea-mainnet:/root/.ethereum

  node:
    image: ethereum/client-go:v1.13.4
    container_name: linea-mainnet
    restart: unless-stopped
    depends_on:
      init:
        condition: service_completed_successfully
    command:
      - --networkid=59144
      - --gcmode=archive
      - --syncmode=full
      - --rpc.allow-unprotected-txs
      - --txpool.accountqueue=50000
      - --txpool.globalqueue=50000
      - --txpool.globalslots=50000
      - --txpool.pricelimit=1000000
      - --txpool.pricebump=1
      - --txpool.nolocals
      - --http
      - --http.addr=0.0.0.0
      - --http.port=8545
      - --http.corsdomain=*
      - --http.api=admin,web3,eth,txpool,net
      - --http.vhosts=*
      - --ws
      - --ws.addr=0.0.0.0
      - --ws.port=8546
      - --ws.origins=*
      - --ws.api=web3,eth,txpool,net
      - --metrics
      - --metrics.addr=0.0.0.0
      - --metrics.port=6060
      - --bootnodes=enode://ca2f06aa93728e2883ff02b0c2076329e475fe667a48035b4f77711ea41a73cf6cb2ff232804c49538ad77794185d83295b57ddd2be79eefc50a9dd5c48bbb2e@3.23.106.165:30303,enode://eef91d714494a1ceb6e06e5ce96fe5d7d25d3701b2d2e68c042b33d5fa0e4bf134116e06947b3f40b0f22db08f104504dd2e5c790d8bcbb6bfb1b7f4f85313ec@3.133.179.213:30303,enode://cfd472842582c422c7c98b0f2d04c6bf21d1afb2c767f72b032f7ea89c03a7abdaf4855b7cb2dc9ae7509836064ba8d817572cf7421ba106ac87857836fa1d1b@3.145.12.13:30303
      - --verbosity=3
    ports:
      - "30303:30303"
      - "30303:30303/udp"
      - "8545:8545"
      - "8546:8546"
      - "6060:6060"
    volumes:
      - ./config/genesis.json:/genesis.json:ro
      - linea-mainnet:/root/.ethereum
    networks:
      - monitor-net
    labels:
      - "traefik.enable=true"
      - "traefik.http.middlewares.linea-stripprefix.stripprefix.prefixes=/linea-mainnet"
      - "traefik.http.routers.linea.service=linea"
      - "traefik.http.services.linea.loadbalancer.server.port=8545"
      - "traefik.http.routers.linea.entrypoints=websecure"
      - "traefik.http.routers.linea.tls.certresolver=myresolver"
      - "traefik.http.routers.linea.rule=Host(`${DOMAIN}`)"
      - "traefik.http.routers.linea.middlewares=ipwhitelist"

```

{% hint style="info" %}
ctrl + x and y to save file
{% endhint %}

### Run Linea Node

```
docker-compose up -d
```

### Monitor Logs

Use `docker logs` to monitor your Linea node. The `-f` flag ensures you are following the log output

```
docker logs linea-mainnet -f --tail 100
```

Allow a few minutes for the Linea node to connect with peers. Once it starts syncing, the logs should look like this:

```
Feb 26 01:14:36 tiberian-infradao geth[118326]: INFO [02-26|01:14:36.231] Looking for peers                        peercount=0 tried=131 static=0
Feb 26 01:14:40 tiberian-infradao geth[118326]: INFO [02-26|01:14:40.661] Imported new chain segment               number=1 hash=78df32..d51eac blocks=1 txs=1 mgas=0.021 elapsed=1.487ms     mgasps=14.115 age=7mo3w3d snapdiffs=133.00B triedirty=0.00B
Feb 26 01:14:40 tiberian-infradao geth[118326]: INFO [02-26|01:14:40.776] Imported new chain segment               number=118 hash=9d4545..f40fa6 blocks=117 txs=128 mgas=42.794 elapsed=82.126ms    mgasps=521.076 age=7mo2w3d snapdiffs=26.73KiB triedirty=0.00B
```

## Test Linea RPC

You can call the JSON-RPC API methods to confirm the node is running. For example, call [`eth_syncing`](https://besu.hyperledger.org/public-networks/reference/api#eth\_syncing) to return the synchronization status. For example the starting, current, and highest block, or `false` if not synchronizing (or if the head of the chain has been reached)

```
curl https://{YOUR_DOMAIN} \
        -X POST \
        -H "Content-Type: application/json" \
        -d '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}'
```

