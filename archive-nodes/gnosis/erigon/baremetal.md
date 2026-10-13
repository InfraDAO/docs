---
description: 'Authors: [Vince | Nodeify, Payne | Stake🦑Squid]'
---

# 💻 Baremetal

## System Requirements

<table><thead><tr><th align="center">CPU</th><th width="147" align="center">OS</th><th width="119" align="center">RAM</th><th align="center">Storage</th></tr></thead><tbody><tr><td align="center">6 Cores / 12 Threads</td><td align="center">Ubuntu 22.04</td><td align="center">>= 16GB</td><td align="center">>= 3 TiB NVMe SSD</td></tr></tbody></table>

## Erigon 🦦

Official Docs [https://erigon.gitbook.io/](https://erigon.gitbook.io/)

### Pre-requisites

Update, upgrade, and clean the system, and then install essential development tools (build-essential), firewall management (ufw), and the Git version control system.

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt-get install -y build-essential ufw git cmake
```

Set explicit default UFW rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Allow P2P connections with Erigon, Consensus peers and SSH

```bash
sudo ufw allow 30303
sudo ufw allow 9001
sudo ufw allow 22/tcp
```

Allow remote RPC connections with Execution Client

```bash
sudo ufw allow from ${REMOTE.HOST.IP} to any port 9656
```

{% hint style="danger" %}
Not advised to allow all or unknown IP address to RPC port
{% endhint %}

Enable Firewall

```
sudo ufw enable
```

### Install go

Download the Go programming language distribution archive, extracts it to the "/usr/local" directory, and then removes the downloaded archive, effectively installing Go version 1.20.6 on the system.

```bash
wget https://go.dev/dl/go1.20.6.linux-amd64.tar.gz && \
rm -rf /usr/local/go && \
tar -C /usr/local -xzf go1.20.6.linux-amd64.tar.gz && \
rm go1.20.6.linux-amd64.tar.gz
```

Please add the Go executable path to your system's `PATH` environment variable, and then test to ensure that Go is working correctly.

```bash
echo "export PATH="$PATH:/root/.foundry/bin:/usr/local/go/bin"" >> /root/.bashrc
source /root/.bashrc
go version #test
```

### Create Erigon Directory

Create a new directory named erigon within the "/root/.local/share" directory, providing a location for storing data related to Erigon.

```bash
mkdir /root/.local/share/erigon
```

### Generate JWT token

Generate a 32-byte random hexadecimal value using OpenSSL, removes any newline characters, and then save the result into the file "jwt.hex" located in the "/root/.local/share/erigon/" directory.

```bash
openssl rand -hex 32 | sudo tee /root/.local/share/erigon/jwt.hex > /dev/null
```

### Build Erigon

Clone the Erigon repository from GitHub, including its submodules, changes the current directory to the Erigon directory, checks out the latest release tag, and then compile the project using the "make" build system.

```bash
git clone --recurse-submodules https://github.com/ledgerwatch/erigon.git
cd erigon
git checkout <latest release tag>
make
```

### Configure Erigon

Append a systemd service configuration for the Erigon Gnosis Mainnet Service to the "/etc/systemd/system/erigon.service" file, specifying its description, dependencies, and executable parameters for proper execution and monitoring.

```bash
sudo echo "[Unit]
Description=Erigon Gnosis Mainnet Service
After=network.target
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
Type=simple
Restart=on-failure
RestartSec=5
TimeoutSec=900
User=root
Nice=0
LimitNOFILE=200000
WorkingDirectory=/root/.local/share/erigon/
ExecStart=/root/erigon/build/bin/erigon \
        --datadir=/root/.local/share/erigon/datadir \
        --ethash.dagdir=/root/.local/share/erigon/datadir/ethash \
        --chain gnosis \
        --authrpc.jwtsecret=/root/.local/share/erigon/jwt.hex \
        --authrpc.port=9663 \
        --http \
        --http.addr=0.0.0.0 \
        --http.port=9656 \
        --http.compression \
        --http.vhosts=* \
        --http.corsdomain=* \
        --http.api=eth,debug,net,trace,web3,erigon \
        --private.api.addr=0.0.0.0:9092 \
        --ws --ws.compression \
        --metrics --metrics.addr=0.0.0.0 --metrics.port=6060 \
        --torrent.download.rate 1024mb \
        --rpc.returndata.limit=1000000
KillSignal=SIGHUP

[Install]
WantedBy=multi-user.target" >> /etc/systemd/system/erigon.service
```

### Run Erigon

Reload the systemd manager configuration, start the Erigon service, and enable it to start automatically on system boot, ensuring that the Erigon Gnosis Mainnet Service is active and will be automatically started upon system startup.

```bash
sudo systemctl daemon-reload
sudo systemctl start erigon
sudo systemctl enable erigon
```

### Monitor Logs

Use journalctl to display real-time log messages and continuously follow the log output of the Erigon service, allowing you to monitor its activity and troubleshoot any issues as they occur.

```bash
sudo journalctl -fu erigon
```

## Consensus Clients

{% hint style="danger" %}
For Erigon to operate, it requires the use of a consensus client. It is crucial to consider client diversity while doing so. -> [https://clientdiversity.org/](https://clientdiversity.org/)
{% endhint %}

## Lodestar 🤠

Official Docs [https://chainsafe.github.io/lodestar/](https://chainsafe.github.io/lodestar/)

### Install yarn and nodeJS

```bash
#yarn
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt update -y && sudo apt install yarn -y
#nodeJS
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### Create Lodestar Directory

Create a new directory named lodestar within the "/root/.local/share" directory, providing a location for storing data related to Lodestar.

```bash
mkdir /root/.local/share/lodestar
```

### Build Lodestar

Clone the lodestar repository from GitHub, navigate to the lodestar directory, switch to the stable branch, and then build the project with yarn.

```bash
git clone https://github.com/chainsafe/lodestar.git
cd lodestar
git checkout stable
yarn install
yarn run build
```

### Configure Lodestar

Append a systemd service configuration for the Lodestar Gnosis Service to the "/etc/systemd/system/lodestar.service" file, specifying its description, dependencies, and executable parameters for proper execution and monitoring.

```bash
sudo echo "[Unit]
Description=Lodestar Gnosis Service
After=network.target
StartLimitIntervalSec=60
StartLimitBurst=3

Type=simple
Restart=on-failure
RestartSec=5
TimeoutSec=900
User=root
Nice=0
LimitNOFILE=200000
WorkingDirectory=/root/lodestar/
ExecStart=/root/lodestar/lodestar beacon \
  --datadir=/root/.local/share/lodestar \
  --network=gnosis \
  --rest \
  --rest.port=6061 \
  --metrics \
  --metrics.port=6062 \
  --port 9001 \
  --checkpointSyncUrl=https://checkpoint.gnosischain.com/ \
  --execution.urls=http://127.0.0.1:9663 \
  --jwt-secret=/root/.local/share/erigon/jwt.hex

KillSignal=SIGHUP  

[Install]
WantedBy=multi-user.target" >> /etc/systemd/system/lodestar.service
```

### Run Lodestar

Reload the systemd manager configuration, restart the Lodestar and Erigon services, and enable the Lodestar service to start automatically on system boot. This ensures that both services are running with the latest configuration and that the Lodestar service will be automatically started upon system startup.

```bash
systemctl daemon-reload
systemctl restart lodestar erigon
sudo systemctl enable lodestar
```

### Monitor Logs

Use journalctl to display real-time log messages and continuously follow the log output of the Lodestar service, allowing you to monitor its activity and troubleshoot any issues as they occur.

```bash
sudo journalctl -fu lodestar
```

## Lighthouse 🕯️

Official Docs [https://lighthouse-book.sigmaprime.io/](https://lighthouse-book.sigmaprime.io/)

### Install rust

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Please add the Rust executable path to your system's `PATH` environment variable, and then test to ensure that Rust is working correctly.

```bash
echo "export PATH="$PATH:/root/.local/bin:/root/.cargo/env"" >> /root/.bashrc
source /root/.bashrc
cargo --version #test
```

### Create Lighthouse Directory

Create a new directory named "lighthouse" within the "/root/.local/share" directory, providing a location for storing data related to Lighthouse.

<pre class="language-bash"><code class="lang-bash"><strong>mkdir /root/.local/share/lighthouse
</strong></code></pre>

### Build Lighthouse

Clone the lighthouse repository from GitHub, navigate to the lighthouse directory, switch to the stable branch, and then build the project with the gnosis feature enabled using the "make" build system.

```bash
git clone https://github.com/sigp/lighthouse.git
cd lighthouse
git checkout stable
FEATURES=gnosis make
```

### Configure Lighthouse

Append a systemd service configuration for the Lighthouse Gnosis Service to the "/etc/systemd/system/lighthouse.service" file, specifying its description, dependencies, and executable parameters for proper execution and monitoring.

```bash
sudo echo "[Unit]
Description=LightHouse Gnosis Service
After=network.target
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
Type=simple
Restart=on-failure
RestartSec=5
TimeoutSec=900
User=root
Nice=0
LimitNOFILE=200000
WorkingDirectory=/root/lighthouse/
ExecStart=/root/lighthouse/target/release/lighthouse bn \
        --datadir=/root/.local/share/lighthouse \
        --network=gnosis \
        --http \
        --http-port=6061 \
        --metrics \
        --metrics-port=6062 \
        --port=9001 \
        --checkpoint-sync-url=https://checkpoint.gnosischain.com/ \
        --execution-endpoint=http://127.0.0.1:9663 \
        --execution-jwt=/root/.local/share/erigon/jwt.hex

KillSignal=SIGHUP                                                                                                                                                                                          

[Install]
WantedBy=multi-user.target" >> /etc/systemd/system/lighthouse.service
```

### Run Lighthouse

Reload the systemd manager configuration, restart the Lighthouse and Erigon services, and enable the Lighthouse service to start automatically on system boot. This ensures that both services are running with the latest configuration and that the Lighthouse service will be automatically started upon system startup.

```bash
systemctl daemon-reload
systemctl restart lighthouse erigon
sudo systemctl enable lighthouse
```

### Monitor Logs

Use journalctl to display real-time log messages and continuously follow the log output of the Lighthouse service, allowing you to monitor its activity and troubleshoot any issues as they occur.

```bash
sudo journalctl -fu lighthouse
```

## Nimbus ☁️

Official Docs [https://nimbus.guide/index.html](https://nimbus.guide/index.html)

### Install cmake

```bash
sudo apt -y install cmake
```

### Create Nimbus Directory

Create a new directory named nimbus within the "/root/.local/share" directory, providing a location for storing data related to Nimbus.

<pre class="language-bash"><code class="lang-bash"><strong>mkdir /root/.local/share/nimbus
</strong></code></pre>

### Build Nimbus

Clone the Nimbus repository from GitHub, navigate to the nimbus directory, switch to the latest stable release, and then build the project with `make`

```bash
git clone https://github.com/status-im/nimbus-eth2
cd nimbus-eth2
git checkout stable
make -j4 nimbus_beacon_node
```

{% hint style="info" %}
You can increase `-j4` (`-j32`) with higher or lower integer depending on your host machine ram to decrease build time. Omit `-j4` on systems with 4GB of memory or less.
{% endhint %}

### Alternatively, extract the binary from the precompiled docker images:

```bash
mkdir -p /root/nimbus-eth2/build/
sudo docker cp $(sudo docker run -it -d ghcr.io/gnosischain/gnosis-nimbus-eth2:latest):/home/user/nimbus_beacon_node /root/nimbus-eth2/build/
```

### Configure Nimbus

Append a systemd service configuration for the Nimbus Gnosis Service to the "/etc/systemd/system/nimbus.service" file, specifying its description, dependencies, and executable parameters for proper execution and monitoring.

```bash
sudo echo "[Unit]
Description=Nimbus Gnosis Service
After=network.target
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
Type=simple
Restart=on-failure
RestartSec=5
TimeoutSec=900
User=root
Nice=0
LimitNOFILE=200000
WorkingDirectory=/root/nimbus-eth2/
ExecStart=/root/nimbus-eth2/build/nimbus_beacon_node trustedNodeSync \
        --data-dir=/root/.local/share/nimbus \
        --network=gnosis \
        --rest=true \
        --rest-port=6061 \
        --metrics \
        --metrics-port=6062 \
        --tcp-port=9001 --udp-port=9001 \
        --trusted-node-url=https://checkpoint.gnosischain.com/ \
        --web3-url=http://127.0.0.1:9663 \
        --jwt-secret=/root/.local/share/erigon/jwt.hex

KillSignal=SIGHUP                                                                                                                                                                                          

[Install]
WantedBy=multi-user.target" >> /etc/systemd/system/nimbus.service
```

### Run Nimbus

Reload the systemd manager configuration, restart the Nimbus and Erigon services, and enable the Nimbus service to start automatically on system boot. This ensures that both services are running with the latest configuration and that the Nimbus service will be automatically started upon system startup.

```bash
systemctl daemon-reload
systemctl restart nimbus erigon
sudo systemctl enable nimbus
```

### Monitor Logs

Use journalctl to display real-time log messages and continuously follow the log output of the Nimbus service, allowing you to monitor its activity and troubleshoot any issues as they occur.

```bash
sudo journalctl -fu nimbus
```

## Teku 🍷

Official Docs [https://docs.teku.consensys.net/get-started](https://docs.teku.consensys.net/get-started)

### Install the required Java runtime

```bash
sudo apt -y install openjdk-17-jre
```

### Create Teku Directory

Create a new directory named teku within the "/root/.local/share" directory, providing a location for storing data related to Teku.

<pre class="language-bash"><code class="lang-bash"><strong>mkdir /root/.local/share/teku
</strong></code></pre>

### Build Teku

Clone the teku repository from GitHub, navigate to the teku directory, switch to the latest release, and then build the project with with the Gradle wrapper `gradlew`, as follows.

```bash
git clone https://github.com/Consensys/teku.git
cd teku
git checkout <latest release tag>
./gradlew distTar installDist
```

### Configure Teku

Append a systemd service configuration for the Teku Gnosis Service to the "/etc/systemd/system/teku.service" file, specifying its description, dependencies, and executable parameters for proper execution and monitoring.

```bash
sudo echo "[Unit]
Description=Teku Gnosis Service
After=network.target
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
Type=simple
Restart=on-failure
RestartSec=5
TimeoutSec=900
User=root
Nice=0
LimitNOFILE=200000
Environment="JAVA_OPTS=-Xmx5g"
Environment="TEKU_OPTS=-XX:-HeapDumpOnOutOfMemoryError"
WorkingDirectory=/root/teku/
ExecStart=/root/teku/build/install/teku/bin/teku \
        --network=gnosis \
        --data-path=/root/.local/share/teku \
        --rest-api-enabled=true \
        --rest-api-port=6061 \
        --metrics-enabled=true \
        --metrics-port=6062 \
        --p2p-port=9001 \
        --initial-state=https://checkpoint.gnosischain.com \
        --ee-endpoint=http://127.0.0.1:9663 \
        --ee-jwt-secret-file=/root/.local/share/erigon/jwt.hex

KillSignal=SIGHUP                                                                                                                                                                                          

[Install]
WantedBy=multi-user.target" >> /etc/systemd/system/teku.service
```

### Run Teku

Reload the systemd manager configuration, restart the Teku and Erigon services, and enable the Teku service to start automatically on system boot. This ensures that both services are running with the latest configuration and that the Teku service will be automatically started upon system startup.

```bash
systemctl daemon-reload
systemctl restart teku erigon
sudo systemctl enable teku
```

### Monitor Logs

Use journalctl to display real-time log messages and continuously follow the log output of the Teku service, allowing you to monitor its activity and troubleshoot any issues as they occur.

```bash
sudo journalctl -fu teku
```

## Test Erigon RPC 🧪

{% hint style="danger" %}
Erigon and Consensus client must be synced before testing.
{% endhint %}

{% code overflow="wrap" %}
```bash
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["0x1", true],"id":1}' http://localhost:9656
```
{% endcode %}

{% hint style="warning" %}
This should be your result.
{% endhint %}

{% code overflow="wrap" %}
```bash
{"jsonrpc":"2.0","result":{"author":"0xcace5b3c29211740e595850e80478416ee77ca21","difficulty":"0xffffffffffffffffffffffffeda7455a","extraData":"0xde830201018f5061726974792d457468657265756d86312e32392e30826c69","gasLimit":"0x989680","gasUsed":"0x0","hash":"0x96059ccf6d5b78b7f30795cd9661f77a117b94ae458020e9010d5a9968376be4","logsBloom":"0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000","miner":"0xcace5b3c29211740e595850e80478416ee77ca21","number":"0x1","parentHash":"0x4f1dd23188aab3a76b463e4af801b52b1248ef073c648cbdc4c9333d3da79756","receiptsRoot":"0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421","sha3Uncles":"0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347","signature":"0xc35e9f8ac05c69d8c36af2cb8b7bb3beda5945e8367fec758c459d32f9ddc183206bad888ebd7addaa6e3f00593100a628c5e86ca80ebe48d4fc831dd36f825d01","size":"0x249","stateRoot":"0x40cf4430ecaa733787d1a65154a3b9efb560c95d9e324a23b97f0609b539133b","step":307804837,"totalDifficulty":"0xffffffffffffffffffffffffeda9455a","timestamp":"0x5bbba539","transactions":[],"transactionsRoot":"0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421","uncles":[]},"id":1}
```
{% endcode %}
