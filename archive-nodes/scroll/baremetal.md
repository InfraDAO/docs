# 💻 Baremetal

\[Author: Dood]

### System Requirements <a href="#system-requirements" id="system-requirements"></a>

| CPU   | OS        | RAM    | DISK            |
| ----- | --------- | ------ | --------------- |
| 4c/8t | Debian 12 | >=16GB | >= 1TB SSD/NVME |

{% hint style="info" %}
The archive node size is 300G as of 04.03.2024&#x20;
{% endhint %}

{% hint style="warning" %}
All commands in this guide are supposed to be executed as `root`
{% endhint %}

## 📜 Scroll

**Unofficial Docs & Support**: &#x20;

* [Running a Node](https://scrollzkp.notion.site/Running-a-Scroll-L2geth-Node-Scroll-Mainnet-9d7b8aa810fc4cc4ae4add8b707a392d#6d5d8f157b6243128dbe2742a2bc272c)&#x20;
* [Namespace](https://scrollzkp.notion.site/Scroll-RPCs-scroll-namespace-e756b0df98fe42cda8a707083486f9e8)
* [Discord Server](https://discord.gg/99ERMfPC)
* [Github](https://github.com/scroll-tech/)

## Pre-Requisites

```
apt update -y && apt upgrade -y && apt autoremove -y
apt install -y build-essential make gcc musl-dev git curl
```

## Setting up Firewall

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
sudo ufw allow from ${REMOTE.HOST.IP} to any port 8546
```

{% hint style="warning" %}
Not advised to allow all or unknown IP address to RPC port
{% endhint %}

Enable Firewall

```bash
sudo ufw enable
```

```
wget https://golang.org/dl/go1.21.6.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.21.6.linux-amd64.tar.gz
echo 'export PATH=/usr/local/go/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

#check Go version

go version
```

## Setup scroll client

#### Compile Binaries

```
git clone https://github.com/scroll-tech/go-ethereum /root/l2geth-source
cd /root/l2geth-source && make nccc_geth
mkdir /root/scroll-datadir
```

#### Create service to run Scroll Node

If possible, replace '--l1.endpoint "https://eth.llamarpc.com"'  with your own L1 ethereum endpoint.

```
echo "[Unit]
Description=Scroll Node
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
WorkingDirectory=/root/l2geth-source
ExecStart=/root/l2geth-source/build/bin/geth \
--scroll \
--datadir '/root/scroll-datadir' \
--gcmode archive \
--syncmode full \
--cache.noprefetch \
--http \
--http.corsdomain '*' \
--http.vhosts '*' \
--http.addr '0.0.0.0' \
--http.port 8546 \
--http.api 'eth,net,web3,debug,scroll' \
--l1.endpoint "https://eth.llamarpc.com" \
--rollup.verify \
--l1.confirmations finalized \
--verbosity 3 \
--metrics \
--metrics.addr '0.0.0.0' \
--metrics.port 6060 \
--maxpeers 100 \
--port 30303
KillSignal=SIGHUP

[Install]
WantedBy=multi-user.target" >> /etc/systemd/system/scroll-node.service
```

```
systemctl daemon-reload
systemctl start scroll-node
systemctl enable scroll-node
```

## Logging

```
journalctl -fu scroll-node
```

#### Get Sync status via curl (replace $YOURIP$ with the IP of your server)

```
curl -H "Content-type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' $YOURIP$:8546
curl -H "Content-type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"scroll_syncStatus","params":[],"id":1}' $YOURIP$:8546
```

#### References

* [Running a Node](https://scrollzkp.notion.site/Running-a-Scroll-L2geth-Node-Scroll-Mainnet-9d7b8aa810fc4cc4ae4add8b707a392d#6d5d8f157b6243128dbe2742a2bc272c)&#x20;
* [Namespace](https://scrollzkp.notion.site/Scroll-RPCs-scroll-namespace-e756b0df98fe42cda8a707083486f9e8)
* [Discord Server](https://discord.gg/99ERMfPC)
* [Github](https://github.com/scroll-tech/)
