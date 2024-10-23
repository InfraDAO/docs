# 🐳 Docker

Author \[abstractCube]

### System Requirements <a href="#system-requirements" id="system-requirements"></a>

Comment

| CPU      | OS           | RAM  | DISK     |
| -------- | ------------ | ---- | -------- |
| 2 cores+ | Ubuntu 24.04 | 8GB+ | >= 128GB |

{% hint style="info" %}
_The Rootstock Mainnet archive node has a size of 132GB on September 22nd, 2024_
{% endhint %}

Last updated: 22nd Sept 2024

Official docs - [https://dev.rootstock.io/node-operators/](https://dev.rootstock.io/node-operators/)

## Pre-Requisites

First, update, upgrade, and clean the system:

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt install ufw -y
```

## Configure Firewall Settings&#x20;

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw allow 22/tcp
sudo ufw allow 80
sudo ufw allow 443
sudo ufw allow 4444
```

## Install Docker

Run this command to remove any conflicting docker

```bash
`for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done`
```

Add Docker's official GPG key:

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Add the repository to ppt sources:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

Install docker

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Test docker is working
sudo docker run hello-world

# Install docker compose

sudo apt-get update
sudo apt-get install docker-compose-plugin

# Test the docker version
docker compose version
```

## Setup Rootstock Node

Make and switch to the working directory for the Rootstock node

```bash
mkdir rootstock && cd rootstock
```

Create and edit the configuration file:

```bash
nano node.conf
```

Paste the following content into the file You can find all the configuration options [here](https://dev.rootstock.io/node-operators/setup/configuration/reference/)

If you are interested in running the config for other networks, you can find the configs [here](https://github.com/rsksmart/rskj/tree/master/rskj-core/src/main/resources/config)

```bash
blockchain.config.name = "main"

database.dir = /var/lib/rsk/database/mainnet

rpc {
providers : {
    web: {
        cors: "localhost",
        http: {
            enabled: true,
            bind_address = "0.0.0.0",
            hosts = ["localhost"]
            port: 4444,
            }
        ws: {
            enabled: false,
            bind_address: "0.0.0.0",
            port: 4445,
            }
        }
    }

    modules = [
        {
            name: "eth",
            version: "1.0",
            enabled: "true",
        },
        {
            name: "net",
            version: "1.0",
            enabled: "true",
        },
        {
            name: "rpc",
            version: "1.0",
            enabled: "true",
        },
        {
            name: "web3",
            version: "1.0",
            enabled: "true",
        },
        {
            name: "evm",
            version: "1.0",
            enabled: "true"
        },
        {
            name: "sco",
            version: "1.0",
            enabled: "false",
        },
        {
            name: "txpool",
            version: "1.0",
            enabled: "true",
        },
        {
            name: "debug",
            version: "1.0",
            enabled: "false",
        },        
        {
            name: "personal",
            version: "1.0",
            enabled: "true"
        }
    ]
}
```

Create and edit the docker compose file

```bash
nano docker-compose.yml
```

Paste the content into the compose file.

In this Docker Compose file, we are utilizing the prebuilt Rootstock node image available on [dockerhub](https://hub.docker.com/r/rsksmart/rskj), where you can also find other prebuilt images.

````bash
services:
  rsk-node:
    image: rsksmart/rskj:ARROWHEAD-6.3.1
    container_name: rsk-node
    ports:
      - 5050:5050
      - 4444:4444
    volumes:
      - rsk-data:/var/lib/rsk/.rsk
      - ./node.conf:/etc/rsk/node.conf
    restart: unless-stopped
volumes:
  rsk-data:
```
This section mounts the host's data directory for blockchain storage and the node.conf file for configuration into the container,
```bash
volumes:
      - ./data:/var/lib/rsk/.rsk
      - ./node.conf:/etc/rsk/node.conf
````

## Run the node

```bash
docker compose up -d
```

## Monitor the node

Use docker logs to monitor the rootstock node. The -f flag ensures you are following the log output.

```bash
docker logs -f rsk-node
```

You should see a response similar to this once your node starts syncing

```
2024-09-16-21:23:20.0646 INFO [blockchain] [message handler] [blockHash=c6225969d4795419027f98269b1c5c0f2a6afc2a0a5f399fcaa607c27a2c484a, peerMsgId=iCvIT12JZvyWvki, peerSID=8dfb1d6268b4b13, blockHeight=1309986]  block: num: [1309986] hash: [c6225969d4795419027f98269b1c5c0f2a6afc2a0a5f399fcaa607c27a2c484a], processed after: [0.005189]seconds, result IMPORTED_BEST
2024-09-16-21:23:20.0676 INFO [blockchain] [message handler] [blockHash=39a7299155cebca0b4f22ddbf49ce1c39f7d230e21aa7061874f2894d43b4847, peerMsgId=g7k643wFWTOWBfZ, peerSID=8dfb1d6268b4b13, blockHeight=1309987]  block: num: [1309987] hash: [39a7299155cebca0b4f22ddbf49ce1c39f7d230e21aa7061874f2894d43b4847], processed after: [0.005022]seconds, result IMPORTED_BEST
2024-09-16-21:23:20.0706 INFO [blockchain] [message handler] [blockHash=9c5b971fb0094e922c5e517f11324e17babed9c14a33e426a5f7984b1701fe37, peerMsgId=akXn9XtqflGiK4y, peerSID=8dfb1d6268b4b13, blockHeight=1309988]  block: num: [1309988] hash: [9c5b971fb0094e922c5e517f11324e17babed9c14a33e426a5f7984b1701fe37], processed after: [0.005072]seconds, result IMPORTED_BEST
2024-09-16-21:23:20.0735 INFO [blockchain] [message handler] [blockHash=53677df4e16affaf09da709a42a288e4df376ee16c97d76f25e862113f59680d, peerMsgId=n5RGPtCFbshskYG, peerSID=8dfb1d6268b4b13, blockHeight=1309989]  block: num: [1309989] hash: [53677df4e16affaf09da709a42a288e4df376ee16c97d76f25e862113f59680d], processed after: [0.004886]seconds, result IMPORTED_BEST
```

## Query the node

To get the web3 client version

```bash
curl http://localhost:4444 -s -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"web3_clientVersion","params":[],"id":67}'
```

Output

```bash
{"jsonrpc":"2.0","id":67,"result":"RskJ/6.3.1/Mac OS X/Java1.8/ARROWHEAD-202f1c5"}
```

To check the block number

```bash
curl -X POST http://localhost:4444/ -H "Content-Type: application/json" --data '{"jsonrpc":"2.0", "method":"eth_blockNumber","params":[],"id":1}'
```

Output

```bash
{"jsonrpc":"2.0","id":1,"result":"0x14144a"}
```

## References

{% embed url="https://hub.docker.com/r/rsksmart/rskj" %}

{% embed url="https://dev.rootstock.io/node-operators/" %}
