# 💻 Baremetal

Author: \[ jleopoldA ]

### System Requirements <a href="#system-requirements" id="system-requirements"></a>

Comment

| CPU     | OS                    | RAM     | DISK  |
| ------- | --------------------- | ------- | ----- |
| 2 Cores | Debian / Ubuntu 22.04 | 8Gb RAM | 128GB |

{% hint style="info" %}
Rootstock has a size of 118GB on October 9, 2024.
{% endhint %}

{% hint style="danger" %}
Using a version of Rootstock that is below 6.3.1 will result in issues that prevent successful syncing.
{% endhint %}

## Pre-Requisites

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y
```

### Setting up Firewall

#### Set explicit default UFW rules

```bash
# Set explicit default UFW rules
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

#### Allow SSH

```bash
sudo ufw allow 22/tcp
```

#### Allow remote RPC connections with Rootstock node

```bash
sudo ufw allow 4444
sudo ufw allow 4445
```

#### Allow P2P Connections

```bash
sudo ufw allow 30303/tcp
sudo ufw allow 30303/udp
```

#### Enable Firewall

```bash
sudo ufw enable
```

#### To check status of UFW and see the current rules

```bash
sudo ufw status verbose
```

## Building a Node on Rootstock

### Dependencies

{% hint style="info" %}
Rootstock uses Java 8
{% endhint %}

#### Install Java

```bash
# If you have a previous version of Java - remove it.
sudo apt purge openjdk-*

# Remove unused packages
sudo apt autoremove

# Install Java 8
sudo apt install openjdk-8-jdk

# Verify Installation
java -version
```

#### Create Directories for Rootstock

```bash
# Create directory for Rootstock
mkdir /root/rootstock 

# Create a folder for configuration
mkdir /root/rootstock/config
```

#### Create Configuration File

```bash
# Create Configuration File
nano /root/rootstock/config/node.conf
```

#### Paste the below Configuration into the file:

```bash
blockchain.config.name = "main"

database.dir = /root/rootstock/database/mainnet
rpc {
    providers: {
        web: {
            cors = "*"
                http: {
                    enabled = true
                    bind_address = 0.0.0.0
                    port = 4444
                    hosts = ["*"]
                }
                ws: {
                    enabled = true
                    bind_address = 0.0.0.0
                    port = 4445
                }
        }
    }
    modules = {
        eth { version: "1.0", enabled: "true"},
        net { version: "1.0", enabled: "true"},
        rpc { version: "1.0", enabled: "true"},
        web3 { version: "1.0", enabled: "true"},
        evm { version: "1.0", enabled: "true"},
        sco { version: "1.0", enabled: "false"},
        txpool { version: "1.0", enabled: "true"},
        debug { version:"1.0", enabled: "true"},
        personal { version: "1.0", enabled: "false"}
    }
}
```

> `Ctrl + X and Y` to exit and confirm saving changes to a file.

#### Create Data Directory to store chain data for Rootstock blockchain.

```bash
mkdir /root/rootstock/database/mainnet/
```

#### Download Rootstock

```bash
# Download Rootstock within your "rootstock" directory.
cd ./root/rootstock/
git clone --recursive https://github.com/rsksmart/rskj.git
cd rskj
git checkout tags/ARROWHEAD-6.3.1 -b ARROWHEAD-6.3.1
```

#### Ensure the Security Chain

{% hint style="info" %}
Rootstock advises to ensure the security chain. Follow the verification steps provided here: [**Verify security chain of RSKj source code**](https://dev.rootstock.io/node-operators/setup/security-chain/)
{% endhint %}

#### Get External Dependencies

```bash
# From within the root of your "rskj" directory, run the following.
./configure.sh
# This will download and set important components (ex. Gradle Wrapper)
```

#### Compile the node

```bash
# From within the root of your "rskj" directory, run the following.
./gradlew build -x test
```

### Create systemd service for Rootstock node

```bash
# Copy and paste the code below and run it within your terminal.
sudo echo "[Unit]
Description=Rootstock Node
After=network.target
StartLimitIntervalSec=200
StartLimitBurst=5

[Service]
Type=simple
Restart=on-failure
RestartSec=5
TimeoutSec=900
User=root
Nice=0
LimitNOFILE=200000
WorkingDirectory=/root/rootstock/rskj/rskj-core/build/libs/
ExecStart=/usr/bin/java -Drsk.conf.file=/root/rootstock/config/node.conf -jar /root/rootstock/rskj/rskj-core/build/libs/rskj-core-6.3.1-ARROWHEAD-all.jar co.rsk.Start

KillSignal=SIGTERM

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/rootstock.service
```

### Start Rootstock Node

```bash
sudo systemctl daemon-reload # refresh for systemd configuration changes

sudo systemctl enable rootstock.service # enable rootstock.service at start up

sudo systemctl start rootstock.service # start rootstock.service
```

#### View Logs for Debugging

{% hint style="info" %}
This method of installing does not allow you to view sync progress.
{% endhint %}

```bash
journalctl -fu rootstock.service
```

### Query Rootstock Node

```bash
curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' http://localhost:4444

# The response should resemble the follow
{"jsonrpc":"2.0","id":1,"result":"0xcab5ab"}
```

### References

{% embed url="https://dev.rootstock.io/node-operators/setup/node-runner/linux/" %}

{% embed url="https://dev.rootstock.io/node-operators/setup/configuration/preferences/" %}
