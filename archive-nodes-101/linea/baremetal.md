# 💻 Baremetal

Authors: \[man4ela | catapulta.eth]

### System Requirements <a href="#system-requirements" id="system-requirements"></a>

| CPU                      | OS           | RAM       | DISK   |
| ------------------------ | ------------ | --------- | ------ |
| A fast CPU with 4+ cores | Ubuntu 22.04 | 16GB+ RAM | 2.5TB+ |

{% hint style="info" %}
The Linea archive node was 2.5 TB on September 11.2024
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

<pre class="language-bash"><code class="lang-bash">git clone https://github.com/ethereum/go-ethereum.git

cd go-ethereum

#Reportedly most stable version to run Linea node 1.13.4-stable-3f907d6a

git checkout v1.13.15
make geth

cd

#Create directory for database

mkdir linea-datadir

cd linea-datadir

#Download Genesis file

wget https://docs.linea.build/files/geth/mainnet/genesis.json

#Bootstrap the node:

cd /root/go-ethereum

<strong>./build/bin/geth --datadir /root/linea-datadir/ init /root/linea-datadir/genesis.json
</strong></code></pre>

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
--bootnodes "enode://069800db9e6e0ec9cadca670994ef1aea2cfd3d88133e63ecadbc1cdbd1a5847b09838ee08d8b5f02a9c32ee13abeb4d4104bb5514e5322c9d7ee19f41ff3e51@3.132.73.210:31002,enode://a8e03a71eab12ec4b47bb6e19169d8e4dc7a58373a2476969bbe463f2dded6003037fa4dd5f71e15027f7fc8d7340956fbbefed67ddd116ac19a7f74da034b61@3.132.73.210:31003,enode://97706526cf79df9d930003644f9156805f6c8bd964fc79e083444f7014ce10c9bdd2c5049e63b58040dca1d4c82ebef970822198cf0714de830cff4111534ff1@18.223.198.165:31004,enode://24e1c654a801975a96b7f54ebd7452ab15777fc635c1db25bdbd4425fdb04e7f4768e9e838a87ab724320a765e41631d5d37758c933ad0e8668693558125c8aa@18.223.198.165:31000,enode://27010891d960f73d272a553f72b6336c6698db3ade98d631f09c764e57674a797be5ebc6829ddbb65ab564f439ebc75215d20aa98b6f351d12ea623e7d139ac3@3.132.73.210:31001,enode://228e1b8a4931e46f383e30721dac21fb8fb4e5e1b32c870e13b25478c82db3dc1cd9e7ceb93d302a766466b55638cc9c5cbfc43aa48fa41ced19baf365951f76@3.1.142.64:31002,enode://c22eb0d40fc3ad5ea710aeddea906567778166bfe18c157955e8c39b23a46c45db18a0fa2ba07f2b64c81178a8c796aec2a29151533920ead06fcdfc6d8d03c6@47.128.192.57:31004,enode://8ce733abe39fd7ae0a278b9893f85c1193c611a3886168690dd843435460f22cc4d61f9e8d0ace7f5905836a665319a31cccdaacdada2acc69972c382ecce7db@3.1.142.64:31003,enode://b7c1b2bed65a855f7a2104aac9a14674dfdf018fdac763415b373b29ce18cdb81d36328ba4e5c9f12629f3a50c3e8f9ee048f22dbdbe93a82813da89c6b81334@51.20.235.126:31004,enode://95270e0550848a72fb141cf27f1c4ea10714edde365b411dc0fa06c81c0f282ce155eb9fa472b6b8bb9ee98395eeaf4c5a7b02a01fe58b37ea98ba152eda4c37@13.50.94.193:31000,enode://72013391755f24f08567b932feeeec4c893c06e0b1fb480890c83bf87fd277ad86a5ab9cb586db9ae9970371a2f8cb0c96f6c9f69045abca0fb801db7f047138@51.20.235.126:31001" \
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

List of actual bootnodes can be found here [https://docs.linea.build/developers/guides/run-a-node/enodes](https://docs.linea.build/developers/guides/run-a-node/enodes)

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
