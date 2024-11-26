# 🐳 Docker

Authors: \[jLeopoldA]

### System Requirements <a href="#system-requirements" id="system-requirements"></a>

| CPU        | OS                          | Ram          | Disk          |
| ---------- | --------------------------- | ------------ | ------------- |
| vCpu 4 min | Used: Debian / Ubuntu 22.04 | 16GB minimum | 13.5 TB (SSD) |

{% hint style="info" %}
Archival Node size is 973GB as of 11/20/2024
{% endhint %}

## Required Installations

{% hint style="info" %}
Requirements to create an X Layer node are Docker, Docker-Compose and an Ethereum L1  node.
{% endhint %}

## Pre-Requisites

### Update

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y
```

### Setting up Firewall

#### Set explicit default UFW rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

#### Allow SSH

```bash
sudo ufw allow 22/tcp
```

#### Allow remote RPC connections with X Layer node

```bash
sudo ufw allow 8545
```

### Install Docker and Docker-Compose

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Install Docker Packages
 sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
 
 # Verify Docker Installation is Successful
sudo docker run hello-world

# Install Docker-Compose
sudo apt-get update
sudo apt-get install docker-compose-plugin

# Verify Docker-Compose installation by checking the version
docker compose version

# Expected output
Docker Compose version vN.N.N
```

## Set up X Layer Node

<pre class="language-bash"><code class="lang-bash"># Make a directory
mkdir -p /root/xlayer-node &#x26;&#x26; cd /root/xlayer-node

# Download mainnet X-Layer script
curl -fsSL https://raw.githubusercontent.com/okx/Deploy/main/setup/zknode/run_xlayer_mainnet.sh | bash -s init &#x26;&#x26; cp ./mainnet/example.env ./mainnet/.env
<strong>
</strong><strong># Alternatively: Download testnet X-Layer script
</strong>curl -fsSL https://raw.githubusercontent.com/okx/Deploy/main/setup/zknode/run_xlayer_testnet.sh | bash -s init &#x26;&#x26; cp ./testnet/example.env ./testnet/.env

# Edit mainnet .env
nano ./mainnet/.env

# Alternatively: Edit testnet .env
nano ./testnet/.env
</code></pre>

### Values for .env file

```bash
# URL of a JSON RPC for Ethereum mainnet or Sepolia testnet
XLAYER_NODE_ETHERMAN_URL = "http://your.L1node.url"

# PATH WHERE THE STATEDB POSTGRES CONTAINER WILL STORE PERSISTENT DATA
XLAYER_NODE_STATEDB_DATA_DIR = "./xlayer_mainnet_data/statedb" # OR ./xlayer_testnet_datastatedb/ for testnet

# PATH WHERE THE POOLDB POSTGRES CONTAINER WILL STORE PERSISTENT DATA #
XLAYER_NODE_POOLDB_DATA_DIR = "./xlayer_mainnet_data/pooldb" # OR ./xlayer_testnet_data/pooldb/ for testnet
```

### Restore latest Layer 2 snapshot

{% hint style="info" %}
Restoring the Layer 2 snapshot database locally will allow synchronizing of Layer 2 data quickly.&#x20;
{% endhint %}

```bash
# mainnet
./run_xlayer_mainnet.sh restore

# If using testnet
./run_xlayer_testnet.sh restore
```

## Start X Layer Node

```bash
# mainnet
./run_xlayer_mainnet.sh start

# If using testnet
./run_xlayer_testnet.sh start

docker ps -a

# The following containers should be available: 
xlayer-rpc
xlayer-sync
xlayer-state-db
xlayer-pool-db
xlayer-prover
```

### Query X Layer Node

```bash
curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":83}' http://localhost:8545

# The response should resemble the follow
{"jsonrpc":"2.0","id":1,"result":"0xcab5ab"}
```

### Additional Commands

```bash
# Stop X Layer Node
./run_xlayer_mainnet.sh stop

# Restart
./run_xlayer_mainnet.sh restart

# Updating
./run_xlayer_mainnet.sh update
```

## Access Logs

```bash
# TO VIEW LOGS
docker logs [container id OR container name]
```
