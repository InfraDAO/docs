# 💻 Baremetal

Authors: \[man4ela | catapulta.eth]

## System Requirements

|   CPU  |           OS           |                  RAM                  |     DISK     |
| :----: | :--------------------: | :-----------------------------------: | :----------: |
| 4 vCPU | Debian 12/Ubuntu 22.04 | <p>8GB min</p><p>16GB Recommended</p> | 2.5TB+ (SSD) |

{% hint style="info" %}
_The Fuse archival node has a size of 2.5TB on Septmber 3rd, 2024_
{% endhint %}

### Pre-Requisites

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y

sudo apt install -y git make wget gcc pkg-config libusb-1.0-0-dev libudev-dev jq gcc g++ curl libssl-dev screen apache2-utils build-essential pkg-config unzip
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
```

Allow remote RPC connections with Fuse node

```bash
sudo ufw allow from ${REMOTE.HOST.IP} to any port 8545 8546
```

{% hint style="warning" %}
Not advised to allow all or unknown IP address to RPC port
{% endhint %}

Allow P2P Connections

```bash
sudo ufw allow 30303/tcp
sudo ufw allow 30303/udp
```

Enable Firewall

```bash
sudo ufw enable
```

To check the status of UFW and see the current rules

```bash
sudo ufw status verbose
```

## Building a Node on Fuse with Nethermind client

{% hint style="success" %}
Since **08.2022** Fuse is moving from OE client to [Nethermind](https://nethermind.io/). To bootstrap Fuse archive node this guide covers the steps on how to build Nethermind from source and configure it to run for Fuse Network
{% endhint %}

### Install .NET SDK

```bash
wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt update
sudo apt install -y apt-transport-https
sudo apt update
sudo apt install -y dotnet-sdk-8.0
```

### Download the Latest Nethermind Release Binary <a href="#the-release-binary" id="the-release-binary"></a>

Check [release binary](https://github.com/NethermindEth/nethermind/releases) page and take the following steps to download the latest Nethermind version:

Create a directory to store the binary and chain data (you might need `sudo`)

```bash
mkdir fuse-archive
```

Use `wget` to grab the latest [release binary](https://github.com/NethermindEth/nethermind/releases) archive and output it to the directory created in the previous step:

```bash
wget https://github.com/NethermindEth/nethermind/releases/download/1.27.0/nethermind-1.27.0-220b5b85-linux-x64.zip \
-O /root/fuse-archive/ nethermind-1.27.0-220b5b85-linux-x64.zip
```

Use `unzip` to extract downloaded archive

```bash
unzip nethermind-1.27.0-220b5b85-linux-x64.zip
```

### Configuing Nethermind client

#### Increase the maximum number of open files

```bash
sudo bash -c 'echo "nethermind soft nofile 100000" > /etc/security/limits.d/nethermind.conf' 
sudo bash -c 'echo "nethermind hard nofile 100000" >> /etc/security/limits.d/nethermind.conf'
```

#### Create chainspec file for Fuse

```bash
mkdir -p /root/fuse-archive/chainspec

nano /root/fuse-archive/chainspec/fuse.json
```

Copy/Paste the following contents into the file:

```bash
{
  "name": "FuseNetwork",
  "engine": {
    "authorityRound": {
      "params": {
        "stepDuration": "5",
        "blockReward": "0x0",
        "blockRewardContractAddress": "0x63D4efeD2e3dA070247bea3073BCaB896dFF6C9B",
        "blockRewardContractTransition": 100,
        "validators": {
          "multi": {
            "0": {
              "list": ["0xd9176e84898a0054680aec3f7c056b200c3d96c3"]
            },
            "100": {
              "safeContract": "0x3014ca10b91cb3D0AD85fEf7A3Cb95BCAc9c0f79"
            }
          }
        }
      }
    }
  },
  "params": {
    "gasLimitBoundDivisor": "0x400",
    "maximumExtraDataSize": "0x20",
    "minGasLimit": "0x1388",
    "networkID" : "0x07a",
    "eip155Transition": 0,
    "validateChainIdTransition": 0,
    "eip140Transition": 0,
    "eip211Transition": 0,
    "eip214Transition": 0,
    "eip658Transition": 0,
    "eip150Transition": "0x0",
    "eip160Transition": "0x0",
    "eip161abcTransition": "0x0",
    "eip161dTransition": "0x0",
    "eip98Transition": "0x7fffffffffffff",
    "eip145Transition": "0x38ada7",
    "eip1014Transition": "0x38ada7",
    "eip1052Transition": "0x38ada7",
    "eip1283Transition": "0xd29240",
    "eip1344Transition": "0xd29240",
    "eip1706Transition": "0xd29240",
    "eip1884Transition": "0xd29240",
    "eip2028Transition": "0xd29240",
    "eip2929Transition": "0xd29240",
    "eip2930Transition": "0xd29240",
    "maxCodeSize": 24576,
    "maxCodeSizeTransition": "0x0"
  },
  "genesis": {
    "seal": {
      "authorityRound": {
        "step": "0x0",
        "signature": "0x0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"
      }
    },
    "difficulty": "0x20000",
    "gasLimit": "0x989680"
  },
  "nodes": [
    "enode://60ca021ca7a60c5fedeb39344d6ef282c6c8574c87bf492dcef7ed8dd9611c5a33da2b286f12eb81554c403718565a749d5028c9bcfc1d5b90b8d105ac04da4b@35.205.73.124:30303",
    "enode://550041c1883866ee537ddf220c0ea84b614bce27e9adb8de85b3b86bd745d7ed9575043a78fabe192f1f0ceee71a343f1d5b35f09e6bb41f24ac69bfe214f414@34.76.228.61:30303"
  ],
  "accounts": {
    "0x0000000000000000000000000000000000000001": { "balance": "1", "builtin": { "name": "ecrecover", "pricing": { "linear": { "base": 3000, "word": 0 } } } },
    "0x0000000000000000000000000000000000000002": { "balance": "1", "builtin": { "name": "sha256", "pricing": { "linear": { "base": 60, "word": 12 } } } },
    "0x0000000000000000000000000000000000000003": { "balance": "1", "builtin": { "name": "ripemd160", "pricing": { "linear": { "base": 600, "word": 120 } } } },
    "0x0000000000000000000000000000000000000004": { "balance": "1", "builtin": { "name": "identity", "pricing": { "linear": { "base": 15, "word": 3 } } } },
    "0x0000000000000000000000000000000000000005": { "builtin": { "name": "modexp", "pricing": { "0": { "price": { "modexp": { "divisor": 20 } } }, "0xd29240": { "info": "EIP-2565: ModExp Gas Cost.", "price": { "modexp2565": {} } } } } },
    "0x0000000000000000000000000000000000000006": { "builtin": { "name": "alt_bn128_add", "pricing": { "0": { "price": { "alt_bn128_const_operations": { "price": 500 } } }, "0xd29240": { "info": "EIP-1108 Istanbul HF", "price": { "alt_bn128_const_operations": { "price": 150 } } } } } },
    "0x0000000000000000000000000000000000000007": { "builtin": { "name": "alt_bn128_mul", "pricing": { "0": { "price": { "alt_bn128_const_operations": { "price": 4000 } } }, "0xd29240": { "info": "EIP-1108 Istanbul HF", "price": { "alt_bn128_const_operations": { "price": 6000 } } } } } },
    "0x0000000000000000000000000000000000000008": { "builtin": { "name": "alt_bn128_pairing", "pricing": { "0": { "price": { "alt_bn128_pairing": { "base": 100000, "pair": 80000 } } }, "0xd29240": { "info": "EIP-1108 Istanbul HF", "price": { "alt_bn128_pairing": { "base": 45000, "pair": 34000 } } } } } },
    "0x0000000000000000000000000000000000000009": { "builtin": { "name": "blake2_f", "pricing": { "0xd29240": { "info": "EIP-152 Istanbul HF", "price": { "blake2_f": { "gas_per_round": 1 } } } } } },
    "0xd9176e84898a0054680aec3f7c056b200c3d96c3": { "balance": "300000000000000000000000000" }
  }
}
```

`Ctrl + X and Y` to exit and confirm saving changes to a file

#### Create the Configuration File (`fuse_archive.cfg`)

Create Configuration Directory and File:

```bash
mkdir -p /root/fuse-archive/configs

nano /root/fuse-archive/configs/fuse_archive.cfg
```

Paste the following configuration into the file:

```bash
{
  "Init": {
    "DiscoveryEnabled": true,
    "WebSocketsEnabled": true,
    "StoreReceipts" : true,
    "ChainSpecPath": "chainspec/fuse.json",
    "BaseDbPath": "nethermind_db/fuse_archive",
    "LogFileName": "fuse_archive.logs.txt"
  },
  "Network": {
    "DiscoveryPort": 30303,
    "P2PPort": 30303,
    "LocalIp": "0.0.0.0",
    "ExternalIp": "0.0.0.0"
  },
  "JsonRpc": {
        "Enabled": true,
        "Timeout": 20000,
        "Host": "0.0.0.0",
        "Port": 8545,
        "WebSocketsPort": 8546,
        "UseMinGasPriceInEstimates": true
   },
  "Metrics": {
    "NodeName": "Fuse_archive"
  },
  "Bloom": {
    "IndexLevelBucketSizes": [ 16, 16, 16, 16 ]
  },
  "Pruning": {
    "Mode": "None"
  },
  "Mining": {
    "MinGasPrice": "1000000000"
  },
  "Merge": {
    "Enabled": false
  }
}
```

`Ctrl + X and Y` to exit and confirm saving changes to a file

#### Create Data Directory to store chain data for Fuse blockchain

```bash
mkdir /root/fuse-archive/fuse_data
```

#### Configure systemd Service

Ensure that you grant execute permission to the binary file:

```bash
sudo chmod +x /root/fuse-archive/Nethermind.Runner
```

Create a systemd service file:

```bash
sudo nano /etc/systemd/system/nethermind.service
```

Add the following content:

```bash
[Unit]
Description=Nethermind Node
Documentation=https://docs.nethermind.io
After=network.target

[Service]
User=root
ExecStart=/root/fuse-archive/Nethermind.Runner \
  --config /root/fuse-archive/configs/fuse_archive.cfg \
  --datadir /root/fuse-archive/fuse_data \
  --TraceStore.Enabled true \
  --TraceStore.BlocksToKeep 0 \
  --TraceStore.TraceTypes Trace,Rewards \
  --Sync.FastSync=false
Restart=on-failure
LimitNOFILE=1000000

[Install]
WantedBy=multi-user.target
```

`Ctrl + X and Y` to exit and confirm saving changes to a file

#### Reload systemd and Enable the Service

```bash
systemctl enable nethermind #enable nethermind service at system startup

sudo systemctl daemon-reload #refresh systemd configuration when changes made

sudo systemctl start nethermind #start nethermind

sudo systemctl stop nethermind #stop nethermind
```

{% hint style="info" %}
To check or modify `nethermind.service` parameters simply run&#x20;

`sudo nano /etc/systemd/system/nethermind.service`

Ctrl+X and Y to save changes
{% endhint %}

### View Logs for Debugging

```bash
journalctl -f -u nethermind  #follow logs of nethermind service
```

_The logs should look like below and indicate that your node syncs and is expected to reach a chainhead in \~2-3 days_

{% hint style="success" %}
`28 Jun 01:13:08 | Finalizing validators for transition signalled within contract at block 473632 after block 473633 (0xba9913...ff8838).`

`28 Jun 01:13:08 | Applying validator set change before block 473634 (0xae6e4c...881217).`

`28 Jun 01:13:08 | Downloaded 473,680 / 30,284,955 ( 1.56 %) | current 102 Blk/s | total 156 Blk/s`

`28 Jun 01:13:09 | Processed 473428... 473681 | 193.72 ms | slot 2,151 ms |⛽ Gas gwei: 0.00 .. 0.00 (0.00) .. 0.00`

`28 Jun 01:13:09 | - Blocks 254 0.14 MGas | 6 txs | calls 1,835 ( 0) | sload 4,835 | sstore 735 | create 0`

`28 Jun 01:13:09 | - Block throughput 0.74 MGas/s | 30.97 t/s | 1311.17 Blk/s | recover 0 | process 7`

`28 Jun 01:13:09 | Signal for transition within contract at block 473732 (0x89197a...464a90). New list of 2 : [0xc736793ff31e04807cbf20b39d50ac7a04a4bdad, 0xd9176e84898a0054680aec3f7c056b200c3d96c3].`
{% endhint %}

## References

{% embed url="https://docs.nethermind.io/get-started/installing-nethermind/" %}

{% embed url="https://github.com/fuseio/fuse-network/blob/master/README.md#archival-node" %}

{% embed url="https://docs.fuse.io/developers/run-or-access-fuse-nodes" %}

{% embed url="https://github.com/fuseio/nethermind-client/tree/production/src/Nethermind/Nethermind.Runner/configs" %}
