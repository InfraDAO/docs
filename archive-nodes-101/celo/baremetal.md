# 💻 Baremetal

Authors: Godwin

## System Requirements

|     CPU    |           OS           |  RAM  |    DISK    |
| :--------: | :--------------------: | :---: | :--------: |
| 4-8 Cores  | Debian 12/Ubuntu 22.04 | 16 GB | 4TB+ (SSD) |

{% hint style="info" %}
_The celo l2 node has a size of 2.1TB on June 24, 2025_
{% endhint %}

### Pre-Requisites

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y

sudo apt install -y git make wget gcc pkg-config libusb-1.0-0-dev libudev-dev jq gcc g++ curl libssl-dev screen apache2-utils build-essential pkg-config
```

### Setting up Firewall

Set explicit default UFW rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Allow SSH

```bash
sudo ufw allow 22/tcp
sudo ufw allow 8546
sudo ufw allow 8547
```

{% hint style="warning" %}
Not advised to allow all or unknown IP address to RPC port
{% endhint %}

Enable Firewall

```bash
sudo ufw enable
```

### Download pre-migrated L1 data and run the migration script

The script below uses screen to create a session called archive, and download the Celo L1 data.

```bash
screen -S archive 
aria2c --file-allocation=none -c -x 10 -s 10 https://storage.googleapis.com/cel2-rollup-files/celo/celo-mainnet-migrated-chaindata.tar.zst

# you can use ctrl + A + D to go back to your main terminal session
# screen -r archive to return to the screen window.
```

Once the data has been download, you need to extract the data into a directory of your chosen.

```bash
mkdir -p /opt/celo-l1
mkdir -p /opt/celo
mkdir -p /opt/celo-migrated-l2/

tar --zstd -xvf celo-mainnet-migrated-chaindata.tar.zst -C /opt/celo-l1/

cd /opt/

# the l2 migration script attachs the directory /celo/chaindata to the source-dir argument. 
mv /opt/celo-l1/chaindata/ /opt/celo/

cd
```

Clone the celo l2 docker setup and perform the migration

```bash
git clone git@github.com:celo-org/celo-l2-node-docker-compose.git
cd celo-l2-node-docker-compose


chmod +x migrate.sh

./migrate.sh pre mainnet /opt/ /opt/celo-migrated-l2/
```

This will perform a migration of the L1 data to the Celo L2.

### Set up Celo L2 Archive Node

#### Install go

Download the Go programming language distribution archive, extracts it to the "/usr/local" directory, and then removes the downloaded archive, effectively installing Go version 1.23.5 on the system.

```bash
wget https://go.dev/dl/go1.24.4.linux-amd64.tar.gz
ls /usr/local/
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.24.4.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
go version
```

#### Build Celo L2 OP-GETH

```bash
git clone https://github.com/celo-org/op-geth.git
cd op-geth/
make geth
sudo cp ./build/bin/geth /usr/local/bin/cl2-geth
sudo chmod +x /usr/local/bin/cl2-geth

/usr/local/bin/cl2-geth version
```

### Build Celo L2 OP-NODE

Install just

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
$HOME/.cargo/env
source $HOME/.cargo/env
cargo install just
```

```bash
git clone https://github.com/celo-org/optimism.git
cd optimism/
make build

sudo cp ./op-node/bin/op-node /usr/local/bin/op-node

op-node --version
```

Create JWT Secret

```bash
cd /opt/
mkdir chainconfig
cd chainconfig/
wget https://storage.googleapis.com/cel2-rollup-files/celo/rollup.json
sudo openssl rand -hex 32 > jwt.hex
```

### Create OP-GETH Service

```
sudo nano /etc/systemd/system/celol2-op-geth.service
```

<pre class="language-bash"><code class="lang-bash">[Unit]
Description=Celo L2 op-geth
After=network.target
Wants=network.target

[Service]
Type=exec
User=root
ExecStart=/usr/local/bin/cl2-geth \
    --datadir=/opt/celo-migrated-l2/ \
    --http \
    --http.addr=0.0.0.0 \
    --http.vhosts="*" \
    --http.port=8545 \
    --http.api=eth,net,web3,debug,admin,txpool,engine \
    --ws \
    --ws.addr=0.0.0.0 \
    --ws.port=8546 \
    --ws.api=eth,net,web3,debug,admin,txpool,engine \
    --syncmode=full \
    --gcmode=archive \
    --port=30303 \
    --discovery.port=30303 \
    --authrpc.jwtsecret=/opt/celo/chainconfig/jwt.txt \
    --authrpc.addr=0.0.0.0 \
    --authrpc.port=8551 \
    --authrpc.vhosts="*" \
    --metrics \
    --metrics.addr=0.0.0.0 \
    --metrics.port=6060 \
    --rollup.sequencerhttp="https://cel2-sequencer.celo.org" \
    --rollup.disabletxpoolgossip=true \
    --nat=any \
    --snapshot=true \
    --verbosity=3 \
    --history.transactions=0 \
    --bootnodes=enode://28f4fcb7f38c1b012087f7aef25dcb0a1257ccf1cdc4caa88584dc25416129069b514908c8cead5d0105cb0041dd65cd4ee185ae0d379a586fb07b1447e9de38@34.169.39.223:30303,enode://a9077c3e030206954c5c7f22cc16a32cb5013112aa8985e3575fadda7884a508384e1e63c077b7d9fcb4a15c716465d8585567f047c564ada2e823145591e444@34.169.212.31:30303,enode://029b007a7a56acbaa8ea50ec62cda279484bf3843fae1646f690566f784aca50e7d732a9a0530f0541e5ed82ba9bf2a4e21b9021559c5b8b527b91c9c7a38579@34.82.139.199:30303,enode://f3c96b73a5772c5efb48d5a33bf193e58080d826ba7f03e9d5bdef20c0634a4f83475add92ab6313b7a24aa4f729689efb36f5093e5d527bb25e823f8a377224@34.82.84.247:30303,enode://daa5ad65d16bcb0967cf478d9f20544bf1b6de617634e452dff7b947279f41f408b548261d62483f2034d237f61cbcf92a83fc992dbae884156f28ce68533205@34.168.45.168:30303,enode://c79d596d77268387e599695d23e941c14c220745052ea6642a71ef7df31a13874cb7f2ce2ecf5a8a458cfc9b5d9219ce3e8bc6e5c279656177579605a5533c4f@35.247.32.229:30303,enode://4151336075dd08eb6c75bfd63855e8a4bd6fd0f91ae4a81b14930f2671e16aee55495c139380c16e1094a49691875e69e40a3a5e2b4960c7859e7eb5745f9387@35.205.149.224:30303,enode://ab999db751265c714b171344de1972ed74348162de465a0444f56e50b8cfd048725b213ba1fe48c15e3dfb0638e685ea9a21b8447a54eb2962c6768f43018e5c@34.79.3.199:30303,enode://9d86d92fb38a429330546fe1aefce264e1f55c5d40249b63153e7df744005fa3c1e2da295e307041fd30ab1c618715f362c932c28715bc20bed7ae4fc76dea81@34.77.144.164:30303,enode://c82c31f21dd5bbb8dc35686ff67a4353382b4017c9ec7660a383ccb5b8e3b04c6d7aefe71203e550382f6f892795728570f8190afd885efcb7b78fa398608699@34.76.202.74:30303,enode://3bad5f57ad8de6541f02e36d806b87e7e9ca6d533c956e89a56b3054ae85d608784f2cd948dc685f7d6bbd5a2f6dd1a23cc03e529ea370dd72d880864a2af6a3@104.199.93.87:30303,enode://1decf3b8b9a0d0b8332d15218f3bf0ceb9606b0efe18f352c51effc14bbf1f4f3f46711e1d460230cb361302ceaad2be48b5b187ad946e50d729b34e463268d2@35.240.26.148:30303


Restart=always
RestartSec=10
StandardOutput=journal
<strong>StandardError=journal
</strong>SyslogIdentifier=celo-op-geth

# Resource limits
LimitNOFILE=65536


[Install]
WantedBy=multi-user.target
</code></pre>

### Create OP-NODE Service

```bash
sudo nano /etc/systemd/system/celol2-op-node.service
```

```bash
[Unit]
Description=Celo L2 op-node
After=network.target

[Service]
Type=exec
User=root
ExecStart=/usr/local/bin/op-node \
    --l1=https://ethereum-rpc.publicnode.com \
    --l2=http://localhost:8551 \
    --rpc.addr=0.0.0.0 \
    --rpc.port=9545 \
    --l2.jwt-secret=/opt/celo/chainconfig/jwt.txt \
    --l1.trustrpc \
    --l1.rpckind=basic \
    --l1.beacon=https://ethereum-beacon-api.publicnode.com \
    --metrics.enabled \
    --metrics.addr=0.0.0.0 \
    --metrics.port=7300 \
    --syncmode=execution-layer \
    --rollup.config=/opt/celo/chainconfig/rollup.json \
    --p2p.bootnodes=enr:-J64QJipvmFhMq6DVh6RR4HvIiiBtyy1NUg_QlnAAbf18SMqCxCPZtLgUiWED5p0HRVPv69Wth4YPsvdKXSUyh57mWuGAZXRp6HjgmlkgnY0gmlwhCJTtG-Hb3BzdGFja4TsyQIAiXNlY3AyNTZrMaECKPT8t_OMGwEgh_eu8l3LChJXzPHNxMqohYTcJUFhKQaDdGNwgiQGg3VkcIIkBg,enr:-J64QCxBGS49IQbkbwsUuVWt9CkMctMCRe0b-4dqRsLr4QJ1S52urWPUk2uhBU5uerRGpxWTZZW5FtJC-9gSBHN3cSiGAZXRp4rbgmlkgnY0gmlwhCKph0CHb3BzdGFja4TsyQIAiXNlY3AyNTZrMaECqQd8PgMCBpVMXH8izBajLLUBMRKqiYXjV1-t2niEpQiDdGNwgiQGg3VkcIIkBg,enr:-J64QLG71bmmljNbLFx3qim6zXohKA3jbK_4C4d1cwixI-7VMoBIlnM6kWZVvvdWcbjTQ6QXB1LAO39eZWC4Heztj1-GAZXRpzUGgmlkgnY0gmlwhCKpySSHb3BzdGFja4TsyQIAiXNlY3AyNTZrMaEDApsAenpWrLqo6lDsYs2ieUhL84Q_rhZG9pBWb3hKylCDdGNwgiQGg3VkcIIkBg,enr:-J64QKFU-u1x1gt3WmNP88EDUMQ316ymbzdGy83QjkBDqVSsJBn6-nipuqYQDeHYoLBLVJUMdyAiwxVbbDm14qQSf5qGAZXRppmIgmlkgnY0gmlwhCJTfzOHb3BzdGFja4TsyQIAiXNlY3AyNTZrMaEC88lrc6V3LF77SNWjO_GT5YCA2Ca6fwPp1b3vIMBjSk-DdGNwgiQGg3VkcIIkBg,enr:-J64QIXTVl0Opbdn20TSrkzpIZ4xQ54bERRlTmSeZ05dFLdlSbuRY7yn5tJeTPzsSldTw5V5E0qjEQcsfr20vMjTUDyGAZXRpiWygmlkgnY0gmlwhCPjrx6Hb3BzdGFja4TsyQIAiXNlY3AyNTZrMaED2qWtZdFrywlnz0eNnyBUS_G23mF2NORS3_e5RyefQfSDdGNwgiQGg3VkcIIkBg,enr:-J64QFAsbeR4xRSyVyQOk7bILUCoMjI2EnbZvo4UAK3842HMYw41-UZXdnQJH8lwvzWn7qsY3Vu73NuxzxWKn4XB5wiGAZXRpYPAgmlkgnY0gmlwhCJSxmKHb3BzdGFja4TsyQIAiXNlY3AyNTZrMaEDx51ZbXcmg4flmWldI-lBwUwiB0UFLqZkKnHvffMaE4eDdGNwgiQGg3VkcIIkBg,enr:-J64QFQSrL3mfG-i64T-5DgVE5V9dGKC5A0JrEvD6CRpZvuLK3feg4bPaqFWfqXyNN_6IgY2z1Jkr4Mf2Zx-GdWlWquGAZXQkMdSgmlkgnY0gmlwhCImtd-Hb3BzdGFja4TsyQIAiXNlY3AyNTZrMaEDQVEzYHXdCOtsdb_WOFXopL1v0Pka5KgbFJMPJnHhau6DdGNwgiQGg3VkcIIkBg,enr:-J64QAp3g1m-5uX-_mBXWyo6ZQqAlnRcAt11Xwy0-ZzqaSrDSlg4adyOz6v9flzLgxYkVvXI50nJGs8GjLgT5bwDLtyGAZXQrD69gmlkgnY0gmlwhCJMJgaHb3BzdGFja4TsyQIAiXNlY3AyNTZrMaECq5mdt1EmXHFLFxNE3hly7XQ0gWLeRloERPVuULjP0EiDdGNwgiQGg3VkcIIkBg,enr:-J64QFCZs1ePThNEsRxIIzbfDxYfap1nEyuPPpSUeeWOoPFWOp0zSEPwLEtXhG1eH-ipsB5CgtaVzcXOyT9hKeAeVVaGAZXQkaZ3gmlkgnY0gmlwhCO7ajaHb3BzdGFja4TsyQIAiXNlY3AyNTZrMaEDnYbZL7OKQpMwVG_hrvziZOH1XF1AJJtjFT5990QAX6ODdGNwgiQGg3VkcIIkBg,enr:-J64QJ9LY8m9AjNgujuVT0juX8T6PHKojZEIqd-7_vhBasfiT2xUUJoUfWga_xVJGFECFcN6hPKB4TjihmYFxHXelwOGAZXQkclrgmlkgnY0gmlwhCJMELeHb3BzdGFja4TsyQIAiXNlY3AyNTZrMaEDyCwx8h3Vu7jcNWhv9npDUzgrQBfJ7HZgo4PMtbjjsEyDdGNwgiQGg3VkcIIkBg,enr:-J64QGJFPZzLj2GLFgB4JhTde7rXChMNFERNbzrwYYTG7CY2SCSggFrU3VXczzWBvOoJWdbOMOzPuCI2klknGjruUxeGAZXQkf1LgmlkgnY0gmlwhGjHJzuHb3BzdGFja4TsyQIAiXNlY3AyNTZrMaEDO61fV62N5lQfAuNtgGuH5-nKbVM8lW6JpWswVK6F1giDdGNwgiQGg3VkcIIkBg,enr:-J64QEXleDl25w0qEG__wmDgwnzB0F5zapu00D_jM4qkCbA3WIcLC8rXPm8dcrKdZNBuNXJOtNE6c2_ZDkuQMvIuhjCGAZXQwDjFgmlkgnY0gmlwhCKMdU-Hb3BzdGFja4TsyQIAiXNlY3AyNTZrMaECHezzuLmg0LgzLRUhjzvwzrlgaw7-GPNSxR7_wUu_H0-DdGNwgiQGg3VkcIIkBg


Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal
SyslogIdentifier=celo-op-node

# Resource limits
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

_**Enable and Start the Services**_

```bash
# Reload systemd daemon
sudo systemctl daemon-reload

# Enable services to start on boot
sudo systemctl enable celo-op-geth
sudo systemctl enable celo-op-node


# Start op-geth and op-node first
sudo systemctl start celo-op-geth
sudo systemctl start celo-op-node
```

### Check Service Status

```bash
# Check all Celo services
sudo systemctl status celo-op-geth
sudo systemctl status celo-op-node

# View logs 
sudo journalctl -f -u celo-op-geth-n 100
sudo journalctl -f -u celo-op-node -n 100
```

```bash
curl -X POST -H "Content-Type: application/json" \ --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' \ http://localhost:8545
```

Result

```bash
# if the sync is complete, the above command will return this
{"jsonrpc":"2.0","id":1,"result":false}
```

Check the current block number

```bash
curl -X POST -H "Content-Type: application/json" \ --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \ http://localhost:8545
```

Result

```bash
{"jsonrpc":"2.0","id":1,"result":"0x24ffbfb"}
```

## References

{% embed url="https://docs.celo.org/cel2/operators/migrate-node#troubleshooting" %}

{% embed url="https://docs.celo.org/cel2/notices/l2-migration" %}

{% embed url="https://docs.celo.org/cel2/operators/run-node#mainnet-1" %}

{% embed url="https://github.com/celo-org/celo-l2-node-docker-compose" %}
