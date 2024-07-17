# 💻 Baremetal

Authors: \[man4ela | catapulta.eth]

## System Requirements

|                CPU               |           OS           |  RAM  |    DISK    |
| :------------------------------: | :--------------------: | :---: | :--------: |
| 8 Cores (Fastest per core speed) | Debian 12/Ubuntu 22.04 | 16 GB | 1TB+ (SSD) |

{% hint style="info" %}
_The Moonbeam tracing node has a size of 1.5TB on May 2, 2024_
{% endhint %}

## Run a tracing node

{% hint style="success" %}
Geth's `debug` and `txpool` APIs and OpenEthereum's `trace` module provide non-standard RPC methods for getting a deeper insight into transaction processing. Supporting these RPC methods is important because many protocols, such as [The Graph](https://thegraph.com/), rely on them to index blockchain data.



To use the supported RPC methods, you need to run a **tracing node.** This guide covers the steps of how to set up and sync a tracing node on Moonbeam.
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
```

Allow remote RPC connections with Moonbeam node (_The default port for parachains is `9944` and `9945` for the embedded relay chain_)

```bash
sudo ufw allow from ${REMOTE.HOST.IP} to any port 9944 9945
```

{% hint style="warning" %}
Not advised to allow all or unknown IP address to RPC port
{% endhint %}

Enable Firewall

```bash
sudo ufw enable
```

## Building a Node on Moonbeam

Download the Latest Release Binary

Check [release binary](https://github.com/moonbeam-foundation/moonbeam/releases) page and take the following steps to download the latest version:

Create a directory to store the binary and chain data (you might need `sudo`)

```
mkdir /var/lib/moonbeam-data
```

Use `wget` to grab the latest [release binary](https://github.com/moonbeam-foundation/moonbeam/releases) and output it to the directory created in the previous step:

```bash
wget https://github.com/moonbeam-foundation/moonbeam/releases/download/v0.37.2/moonbeam \
-O /var/lib/moonbeam-data/moonbeam
```

To verify that you have downloaded the correct version, you can run the following command in your terminal

```bash
sha256sum /var/lib/moonbeam-data/moonbeam
```

You should receive the following output:

`71242b8cdf2b97fba4aadb7d6050e0b46eaacafae6bcde9b5f5534aff620d144`

### Setup the Wasm Overrides (required for a tracing node) <a href="#setup-the-wasm-overrides" id="setup-the-wasm-overrides"></a>

You'll need to create a directory for the Wasm runtime overrides and obtain them from the [Moonbeam Runtime Overrides repository](https://github.com/moonbeam-foundation/moonbeam-runtime-overrides) on GitHub

```bash
git clone https://github.com/moonbeam-foundation/moonbeam-runtime-overrides.git
```

Move the Wasm overrides into your on-chain data directory:

```bash
mv moonbeam-runtime-overrides/wasm /var/lib/moonbeam-data
```

Delete the override files for the networks that you aren't running

```bash
rm /var/lib/moonbeam-data/wasm/moonriver-runtime-* &&  rm /var/lib/moonbeam-data/wasm/moonbase-runtime-*
```

Set permissions for the overrides

```bash
chmod +x /var/lib/moonbeam-data/wasm/*
```

### Create the Configuration File <a href="#create-the-configuration-file" id="create-the-configuration-file"></a>

The next step is to create the systemd configuration file, you'll need to:

* Replace`INSERT_YOUR_NODE_NAME` in two different places with the preffered name (it specifies a human-readable name for the node, which can be seen on [telemetry](https://telemetry.polkadot.io/), if enabled)
* **`--db-cache`** specifies the memory the database cache is limited to use. It is recommended to set it to 50% of the actual RAM your server has. For example, for 128 GB RAM, the value must be set to `64000`. The minimum value is `2000`, but it is below the recommended specs
* Double-check that the binary is in the proper path as described below (_ExecStart_)
* Double-check the base path if you've used a different directory
* Name the file `/etc/systemd/system/moonbeam.service`

Ensure that you grant execute permission to the binary file

```bash
sudo chmod +x /var/lib/moonbeam-data/moonbeam
```

```bash
sudo nano /etc/systemd/system/moonbeam.service
```

Copy/Paste and edit `INSERT_YOUR_NODE_NAME` and `--db-cache` according to your parameters:

```bash
[Unit]
Description="Moonbeam systemd service"
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=on-failure
RestartSec=10
User=root
SyslogIdentifier=moonbeam
SyslogFacility=local7
KillSignal=SIGHUP
ExecStart=/var/lib/moonbeam-data/moonbeam \
     --state-pruning=archive \
     --trie-cache-size 1073741824 \
     --db-cache 64000 \
     --base-path /var/lib/moonbeam-data \
     --ethapi=debug,trace,txpool \
     --wasm-runtime-overrides=/var/lib/moonbeam-data/wasm \
     --runtime-cache-size 64 \
     --chain moonbeam \
     --unsafe-rpc-external \
     --rpc-cors="*" \
     --name "INSERT_YOUR_NODE_NAME" \
     -- \
     --name="INSERT_YOUR_NODE_NAME (Embedded Relay)"

[Install]
WantedBy=multi-user.target
```

_**Ctrl+X and Y to save changes**_

{% hint style="info" %}
_`--rpc-port`_ sets the unified port for both HTTP and WS connections. The default port for parachains is `9944` and `9945` for the embedded relay chain
{% endhint %}

{% hint style="info" %}
We run an RPC endpoint so we must use the `--unsafe-rpc-external` flag to run the Moonbeam node with external access to the RPC ports
{% endhint %}

```bash
systemctl enable moonbeam.service #enable moonbeam service at system startup

sudo systemctl daemon-reload #refresh systemd configuration when changes made

sudo systemctl start moonbeam.service #start moonbeam

sudo systemctl stop moonbeam.service #stop moonbeam
```

{% hint style="info" %}
To check or modify `moonbeam.service` parameters simply run&#x20;

`sudo nano /etc/systemd/system/moonbeam.service`

Ctrl+X and Y to save changes
{% endhint %}

```bash
journalctl -f -u moonbeam.service  #follow logs of moonbeam service
```

{% hint style="success" %}
```
The logs should look like the below example and indicate that your node syncs and is expected to reach chainhead in 3-4 days

Syncing 27.0 bps, target=#6051603 (30 peers), best: #3053702 (0xe669…1876), finalized #918931 (0x6587…f763), ⬇ 484.9kiB/s ⬆ 0.5kiB/s
```
{% endhint %}

### Maintain Your Node <a href="#maintain-your-node" id="maintain-your-node"></a>

As Moonbeam development continues, it will sometimes be necessary to upgrade your node software.

To update moonbeam client, you can keep your existing chain data in tact, and only update the binary by following these steps:

1. _Stop the systemd service_

```bash
sudo systemctl stop moonbeam.service
```

2. _Remove the old binary file_

```bash
rm /var/lib/moonbeam-data/moonbeam
```

3. _Get the latest version of the_ [_Moonbeam release binary on GitHub_](https://github.com/moonbeam-foundation/moonbeam/releases/) _and run the following command to update to that version (ensure to replace `INSERT_NEW_VERSION_TAG`with actual version)_

```bash
wget https://github.com/moonbeam-foundation/moonbeam/releases/download/INSERT_NEW_VERSION_TAG/moonbeam \
-O /var/lib/moonbeam-data/moonbeam
```

4. _Update permissions_

```bash
chmod +x /var/lib/moonbeam-data/moonbeam
```

5. _Start moonbeam service_

```bash
systemctl start moonbeam.service
```

## References

{% embed url="https://docs.moonbeam.network/node-operators/networks/run-a-node/systemd/#introduction" %}

{% embed url="https://docs.moonbeam.network/node-operators/networks/tracing-node/#introduction" %}
