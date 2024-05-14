# 💻 Baremetal

Authors: \[man4ela | catapulta.eth]

### System Requirements <a href="#system-requirements" id="system-requirements"></a>

| CPU                      | OS           | RAM       | DISK |
| ------------------------ | ------------ | --------- | ---- |
| A fast CPU with 4+ cores | Ubuntu 22.04 | 16GB+ RAM | 1TB+ |

{% hint style="info" %}
The Linea archive node was 1.2 TB on April 02.2024
{% endhint %}

## 🔲 Linea

* [Official Docs](https://docs.linea.build/build-on-linea/run-a-node#step-3-1)

## Pre-Requisites

<pre class="language-bash"><code class="lang-bash">sudo apt update -y &#x26;&#x26; sudo apt upgrade -y &#x26;&#x26; sudo apt autoremove -y

<strong>sudo apt install -y build-essential bsdmainutils aria2 dtrx screen clang cmake curl httpie jq nano wget
</strong>
</code></pre>

### Setting up Firewall

Set explicit default UFW rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Allow SSH

```bash
sudo ufw allow 22/tcp
```

Allow remote RPC connections with Linea Node

```bash
sudo ufw allow from ${REMOTE.HOST.IP} to any port 8545
```

{% hint style="warning" %}
Not advised to allow all or unknown IP address to RPC port
{% endhint %}

Enable Firewall

```bash
sudo ufw enable
```

<pre class="language-bash"><code class="lang-bash">#Download the Go programming language distribution archive

<strong>wget https://golang.org/dl/go1.21.6.linux-amd64.tar.gz
</strong><strong>
</strong>#Extract it to the /usr/local directory and install Go v1.21.6 on the system

sudo tar -C /usr/local -xzf go1.21.6.linux-amd64.tar.gz
<strong>
</strong><strong>#add the Go executable path to your system's PATH environment variable, 
</strong><strong>
</strong><strong>echo 'export PATH=/usr/local/go/bin:$PATH' >> ~/.bashrc
</strong>
source ~/.bashrc

#test to ensure that Go is working correctly

go version
</code></pre>

### Setup the Geth client to run Linea

```bash
git clone https://github.com/ethereum/go-ethereum.git

cd go-ethereum

#Reportedly most stable version to run Linea node 1.13.4-stable-3f907d6a

git checkout 3f907d6a

make geth

cd

#Create directory for database

mkdir linea-datadir

cd linea-datadir

#Download Genesis file

wget https://docs.linea.build/files/genesis.json

#Bootstrap the node:

cd /root/go-ethereum

./build/bin/geth --datadir /root/linea-datadir/ init /root/linea-datadir/genesis.json
```

### Create service to run Linea Node

<pre class="language-bash"><code class="lang-bash"><strong>sudo echo "[Unit]
</strong>Description=Linea Node
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
WorkingDirectory=/root/go-ethereum
ExecStart=/root/go-ethereum/build/bin/geth \
--datadir /root/linea-datadir \
--networkid 59144 \
--rpc.allow-unprotected-txs \
--txpool.accountqueue 50000 \
--txpool.globalqueue 50000 \
--txpool.globalslots 50000 \
--txpool.pricelimit 1000000 \
--txpool.pricebump 1 \
--txpool.nolocals \
--http --http.addr '0.0.0.0' --http.port 8545 --http.corsdomain '*' --http.api 'web3,eth,txpool,net' --http.vhosts='*' \
--ws --ws.addr '0.0.0.0' --ws.port 8545 --ws.origins '*' --ws.api 'web3,eth,txpool,net' \
--bootnodes "enode://ca2f06aa93728e2883ff02b0c2076329e475fe667a48035b4f77711ea41a73cf6cb2ff232804c49538ad77794185d83295b57ddd2be79eefc50a9dd5c48bbb2e@3.23.106.165:30303,enode://eef91d714494a1ceb6e06e5ce96fe5d7d25d3701b2d2e68c042b33d5fa0e4bf134116e06947b3f40b0f22db08f104504dd2e5c790d8bcbb6bfb1b7f4f85313ec@3.133.179.213:30303,enode://cfd472842582c422c7c98b0f2d04c6bf21d1afb2c767f72b032f7ea89c03a7abdaf4855b7cb2dc9ae7509836064ba8d817572cf7421ba106ac87857836fa1d1b@3.145.12.13:30303" \
--syncmode full \
--metrics \
--metrics.addr '0.0.0.0' \
--verbosity 3 \
--gcmode archive
KillSignal=SIGHUP

[Install]
WantedBy=multi-user.target" >> /etc/systemd/system/linea-node.service
</code></pre>

{% hint style="info" %}
To check or modify linea-node.service parameters simply run&#x20;

`sudo nano /etc/systemd/system/linea-node.service`

Ctrl+X and Y to save changes
{% endhint %}

### Run Linea

```bash
sudo systemctl daemon-reload
sudo systemctl start linea-node
sudo systemctl enable linea-node
```

## Monitor logs

```bash
sudo journalctl -fu linea-node
```

## References

[Run A Linea Node](https://docs.linea.build/build-on-linea/run-a-node#step-3-1)
