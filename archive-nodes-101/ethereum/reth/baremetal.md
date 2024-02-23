---
description: 'Authors: [Vince | Nodeify]'
---

# 💻 Baremetal

## System Requirements

<table data-full-width="false"><thead><tr><th align="center">CPU</th><th width="140" align="center">OS</th><th align="center">RAM</th><th align="center">DISK</th></tr></thead><tbody><tr><td align="center">Higher clock speed over core count</td><td align="center">Ubuntu 22.04</td><td align="center">8GB+</td><td align="center">At least 2.1TB (TLC NVMe recommended)</td></tr></tbody></table>

## Reth 🦀

Official Docs [https://paradigmxyz.github.io/reth/intro.html](https://paradigmxyz.github.io/reth/intro.html)

### Pre-requisites

Update, upgrade, and clean the system, and then install essential development tools (build-essential), firewall management (ufw), and the Git version control system.

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt-get install -y build-essential ufw git libclang-dev pkg-config
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

Allow P2P connections with Reth, Consensus peers and SSH

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

```bash
sudo ufw enable
```

### Install rust

The rustup installer provides an easy way to update the Rust compiler

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Please add the Cargo path to your system's `PATH` environment variable, and then test to ensure that Cargo is working correctly.

```bash
echo "export PATH="$PATH:/root/.local/bin:/root/.cargo/env"" >> /root/.bashrc
source /root/.bashrc
cargo --version #test
```

### Create reth directory

Create a new directory named reth within the "/root/.local/share" directory, providing a location for storing data related to Reth.

```
mkdir /root/.local/share/reth
```

### Generate JWT token

Generate a 32-byte random hexadecimal value using OpenSSL, removes any newline characters, and then save the result into the file "jwt.hex" located in the "/root/.local/share/reth/" directory.

```bash
openssl rand -hex 32 | sudo tee /root/.local/share/reth/jwt.hex > /dev/null
```

### Build Reth

Clone the reth repository from GitHub, navigate to the reth directory, then build the project with `cargo build`.

```
git clone https://github.com/paradigmxyz/reth
cd reth
RUSTFLAGS="-C target-cpu=native" cargo build --profile maxperf
```

### Configure Reth

```
sudo echo "[Unit]
Description=Reth Mainnet Service
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
WorkingDirectory=/root/.local/share/reth/
ExecStart=/root/reth/target/maxperf/reth \
        --datadir=/root/.local/share/reth/datadir \
        --chain mainnet \
        --authrpc.jwtsecret=/root/.local/share/reth/jwt.hex \
        --authrpc.port=9663 \
        --http \
        --http.addr=0.0.0.0 \
        --http.port=9656 \
        --http.corsdomain=* \
        --http.api=eth,debug,net,trace,web3,erigon \
        --ws \
        --metrics=6060
        
KillSignal=SIGHUP

[Install]
WantedBy=multi-user.target" >> /etc/systemd/system/reth.service
```

### Run Reth

Reload the systemd manager configuration, start the Reth service, and enable it to start automatically on system boot, ensuring that the Reth Mainnet Service is active and will be automatically started upon system startup.

```bash
sudo systemctl daemon-reload
sudo systemctl start reth
sudo systemctl enable reth
```

### Monitor Logs

Use journalctl to display real-time log messages and continuously follow the log output of the Reth service, allowing you to monitor its activity and troubleshoot any issues as they occur.

```bash
sudo journalctl -fu reth
```

## Consensus Clients&#x20;

{% hint style="danger" %}
For Reth to operate, it requires the use of a consensus client. It is crucial to consider client diversity while doing so. -> [https://clientdiversity.org/](https://clientdiversity.org/)
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

Append a systemd service configuration for the Lodestar Mainnet Service to the "/etc/systemd/system/lodestar.service" file, specifying its description, dependencies, and executable parameters for proper execution and monitoring.

```bash
sudo echo "[Unit]
Description=Lodestar Mainnet Service
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
  --network=mainnet \
  --rest \
  --rest.port=6061 \
  --metrics \
  --metrics.port=6062 \
  --port 9001 \
  --checkpointSyncUrl=https://sync-mainnet.beaconcha.in \
  --execution.urls=http://127.0.0.1:9663 \
  --jwt-secret=/root/.local/share/reth/jwt.hex

KillSignal=SIGHUP  

[Install]
WantedBy=multi-user.target" >> /etc/systemd/system/lodestar.service
```

### Run Lodestar

Reload the systemd manager configuration, restart the Lodestar and Reth services, and enable the Lodestar service to start automatically on system boot. This ensures that both services are running with the latest configuration and that the Lodestar service will be automatically started upon system startup.

```bash
systemctl daemon-reload
systemctl restart lodestar reth
sudo systemctl enable lodestar
```

### Monitor Logs

Use journalctl to display real-time log messages and continuously follow the log output of the Lodestar service, allowing you to monitor its activity and troubleshoot any issues as they occur.

```bash
sudo journalctl -fu lodestar
```

## Lighthouse 🕯️

Official Docs [https://lighthouse-book.sigmaprime.io/](https://lighthouse-book.sigmaprime.io/)

### Create Lighthouse Directory

Create a new directory named lighthouse within the "/root/.local/share" directory, providing a location for storing data related to Lighthouse.

<pre class="language-bash"><code class="lang-bash"><strong>mkdir /root/.local/share/lighthouse
</strong></code></pre>

### Build Lighthouse

Clone the lighthouse repository from GitHub, navigate to the lighthouse directory, switch to the stable branch, and then build the project with make.

```bash
git clone https://github.com/sigp/lighthouse.git
cd lighthouse
git checkout stable
make
```

### Configure Lighthouse

Append a systemd service configuration for the Lighthouse Mainnet Service to the "/etc/systemd/system/lighthouse.service" file, specifying its description, dependencies, and executable parameters for proper execution and monitoring.

```bash
sudo echo "[Unit]
Description=LightHouse Mainnet Service
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
        --network=mainnet \
        --http \
        --http-port=6061 \
        --metrics \
        --metrics-port=6062 \
        --port=9001 \
        --checkpoint-sync-url=https://sync-mainnet.beaconcha.in \
        --execution-endpoint=http://127.0.0.1:9663 \
        --execution-jwt=/root/.local/share/reth/jwt.hex

KillSignal=SIGHUP                                                                                                                                                                                          

[Install]
WantedBy=multi-user.target" >> /etc/systemd/system/lighthouse.service
```

### Run Lighthouse

Reload the systemd manager configuration, restart the Lighthouse and Reth services, and enable the Lighthouse service to start automatically on system boot. This ensures that both services are running with the latest configuration and that the Lighthouse service will be automatically started upon system startup.

```bash
systemctl daemon-reload
systemctl restart lighthouse reth
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

### Configure Nimbus

Append a systemd service configuration for the Nimbus Mainnet Service to the "/etc/systemd/system/nimbus.service" file, specifying its description, dependencies, and executable parameters for proper execution and monitoring.

```bash
sudo echo "[Unit]
Description=Nimbus Mainnet Service
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
        --network=mainnet \
        --rest=true \
        --rest-port=6061 \
        --metrics \
        --metrics-port=6062 \
        --tcp-port=9001 --udp-port=9001 \
        --trusted-node-url=https://sync-mainnet.beaconcha.in \
        --web3-url=http://127.0.0.1:9663 \
        --jwt-secret=/root/.local/share/reth/jwt.hex

KillSignal=SIGHUP                                                                                                                                                                                          

[Install]
WantedBy=multi-user.target" >> /etc/systemd/system/nimbus.service
```

### Run Nimbus

Reload the systemd manager configuration, restart the Nimbus and Reth services, and enable the Nimbus service to start automatically on system boot. This ensures that both services are running with the latest configuration and that the Nimbus service will be automatically started upon system startup.

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

### Install the required Java runtime&#x20;

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

Append a systemd service configuration for the Teku Mainnet Service to the "/etc/systemd/system/teku.service" file, specifying its description, dependencies, and executable parameters for proper execution and monitoring.

```bash
sudo echo "[Unit]
Description=Teku Mainnet Service
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
        --network=mainnet \
        --data-path=/root/.local/share/teku \
        --rest-api-enabled=true \
        --rest-api-port=6061 \
        --metrics-enabled=true \
        --metrics-port=6062 \
        --p2p-port=9001 \
        --initial-state=https://sync-mainnet.beaconcha.in \
        --ee-endpoint=http://127.0.0.1:9663 \
        --ee-jwt-secret-file=/root/.local/share/reth/jwt.hex

KillSignal=SIGHUP                                                                                                                                                                                          

[Install]
WantedBy=multi-user.target" >> /etc/systemd/system/teku.service
```

### Run Teku

Reload the systemd manager configuration, restart the Teku and Reth services, and enable the Teku service to start automatically on system boot. This ensures that both services are running with the latest configuration and that the Teku service will be automatically started upon system startup.

```bash
systemctl daemon-reload
systemctl restart teku reth
sudo systemctl enable teku
```

### Monitor Logs

Use journalctl to display real-time log messages and continuously follow the log output of the Teku service, allowing you to monitor its activity and troubleshoot any issues as they occur.

```bash
sudo journalctl -fu teku
```

## Test Reth RPC 🧪

{% hint style="danger" %}
Reth and Consensus client must be synced before testing.
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
{"jsonrpc":"2.0","id":1,"result":{"difficulty":"0x3ff800000","extraData":"0x476574682f76312e302e302f6c696e75782f676f312e342e32","gasLimit":"0x1388","gasUsed":"0x0","hash":"0x88e96d4537bea4d9c05d12549907b32561d3bf31f45aae734cdc119f13406cb6","logsBloom":"0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000","miner":"0x05a56e2d52c817161883f50c441c3228cfe54d9f","mixHash":"0x969b900de27b6ac6a67742365dd65f55a0526c41fd18e1b16f1a1215c2e66f59","nonce":"0x539bd4979fef1ec4","number":"0x1","parentHash":"0xd4e56740f876aef8c010b86a40d5f56745a118d0906a34e69aec8c0db1cb8fa3","receiptsRoot":"0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421","sha3Uncles":"0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347","size":"0x219","stateRoot":"0xd67e4d450343046425ae4271474353857ab860dbc0a1dde64b41b5cd3a532bf3","timestamp":"0x55ba4224","totalDifficulty":"0x7ff800000","transactions":[],"transactionsRoot":"0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421","uncles":[]}}
```
{% endcode %}
