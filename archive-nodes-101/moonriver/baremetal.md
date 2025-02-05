# 💻 Baremetal

Authors: \[man4ela | catapulta.eth]

### System Requirements <a href="#system-requirements" id="system-requirements"></a>

Comment

| CPU                              | OS                     | RAM   | DISK       |
| -------------------------------- | ---------------------- | ----- | ---------- |
| 8 Cores (Fastest per core speed) | Debian 12/Ubuntu 22.04 | 16 GB | 2TB+ (SSD) |

{% hint style="info" %}
_The Moonriver tracing node has a size of 1.8TB on January 30, 2025_
{% endhint %}

### Run a tracing node <a href="#run-a-tracing-node" id="run-a-tracing-node"></a>

{% hint style="success" %}
Geth's `debug` and `txpool` APIs and OpenEthereum's `trace` module provide non-standard RPC methods for getting a deeper insight into transaction processing. Supporting these RPC methods is important because many projects, such as [The Graph](https://thegraph.com/), rely on them to index blockchain data.



To use the supported RPC methods, you need to run a **tracing node.** This guide covers the steps on how to setup and sync a tracing node on Moonriver.
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

Allow remote RPC connections with Moonriver node (_The default port for parachains is `9944` and `9945` for the embedded relay chain_)

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

## Building a Node on Moonriver

Download the Latest Release Binary

Check [release binary](https://github.com/moonbeam-foundation/moonbeam/releases) page and take the following steps to download the latest version:

Create a directory to store the binary and chain data (you might need `sudo`)

```bash
mkdir /var/lib/moonriver-data
```

Use `wget` to grab the latest [release binary](https://github.com/moonbeam-foundation/moonbeam/releases) and output it to the directory created in the previous step:

```bash
wget https://github.com/moonbeam-foundation/moonbeam/releases/download/v0.42.1/moonbeam \
-O /var/lib/moonriver-data/moonbeam
```

To verify that you have downloaded the correct version, you can run the following command in your terminal

```bash
sha256sum /var/lib/moonriver-data/moonbeam
```

You should receive the following output:

9b645b2fd9e575b26ea727e96dc2a81486a73461bb961ba33ea2136e4c9060d8

### Setup the Wasm Overrides (required for a tracing node) <a href="#setup-the-wasm-overrides" id="setup-the-wasm-overrides"></a>

You'll need to create a directory for the Wasm runtime overrides and obtain them from the [Moonbeam Runtime Overrides repository](https://github.com/moonbeam-foundation/moonbeam-runtime-overrides) on GitHub

```bash
git clone https://github.com/moonbeam-foundation/moonbeam-runtime-overrides.git
```

Move the Wasm overrides into your on-chain data directory:

```bash
mv moonbeam-runtime-overrides/wasm /var/lib/moonriver-data
```

Delete the override files for the networks that you aren't running

```bash
rm /var/lib/moonriver-data/wasm/moonbeam-runtime-* &&  rm /var/lib/moonriver-data/wasm/moonbase-runtime-*
```

Set permissions for the overrides

```bash
chmod +x /var/lib/moonriver-data/wasm/*
```

### Create the Configuration File <a href="#create-the-configuration-file" id="create-the-configuration-file"></a>

The next step is to create the systemd configuration file, you'll need to:

* Replace`INSERT_YOUR_NODE_NAME` in two different places with the preffered name (it specifies a human-readable name for the node, which can be seen on [telemetry](https://telemetry.polkadot.io/), if enabled)
* **`--db-cache`** specifies the memory the database cache is limited to use. It is recommended to set it to 50% of the actual RAM your server has. For example, for 128 GB RAM, the value must be set to `64000`. The minimum value is `2000`, but it is below the recommended specs
* Double-check that the binary is in the proper path as described below (_ExecStart_)
* Double-check the base path if you've used a different directory
* Name the file `/etc/systemd/system/moonriver.service`

Ensure that you grant execute permission to the binary file

```bash
sudo chmod +x /var/lib/moonriver-data/moonbeam
```

```bash
sudo nano /etc/systemd/system/moonriver.service
```

Copy/Paste and edit `INSERT_YOUR_NODE_NAME` and `--db-cache` according to your parameters:

```bash
[Unit]
Description="Moonriver service"
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
ExecStart=/var/lib/moonriver-data/moonbeam \
--rpc-port 9944 \
--execution wasm \
--wasm-execution compiled \
--state-pruning archive \
--trie-cache-size 1073741824 \
--runtime-cache-size 64 \
--ethapi debug,trace,txpool \
--wasm-runtime-overrides /var/lib/moonriver-data/wasm \
--unsafe-rpc-external \
--rpc-cors all \
--prometheus-port 6060 \
--db-cache 64000 \
--base-path /var/lib/moonriver-data \
--chain moonriver \
--name "infradao-moonriver" \
-- \
--name "infradao-moonriver (Embedded Relay)"
```

_**Ctrl+X and Y to save changes**_

{% hint style="info" %}
_`--rpc-port`_ sets the unified port for both HTTP and WS connections. The default port for parachains is `9944` and `9945` for the embedded relay chain
{% endhint %}

{% hint style="info" %}
We run an RPC endpoint so we must use the `--unsafe-rpc-external` flag to run the Moonbeam node with external access to the RPC ports
{% endhint %}

```bash
systemctl enable moonriver.service #enable moonriver service at system startup

sudo systemctl daemon-reload #refresh systemd configuration when changes made

sudo systemctl start moonriver.service #start moonriver

sudo systemctl stop moonriver.service #stop moonriver
```

{% hint style="info" %}
To check or modify moonriver`.service` parameters simply run&#x20;

`sudo nano /etc/systemd/system/`moonriver`.service`

Ctrl+X and Y to save changes
{% endhint %}

```bash
journalctl -f -u moonriver.service  #follow logs of moonriver service
```

{% hint style="success" %}
```
The logs should look like below and indicate that your node syncs and is expected to reach chainhead in about a week

Syncing 27.0 bps, target=#6051603 (30 peers), best: #3053702 (0xe669…1876), finalized #918931 (0x6587…f763), ⬇ 484.9kiB/s ⬆ 0.5kiB/s
```
{% endhint %}

### Maintain Your Node <a href="#maintain-your-node" id="maintain-your-node"></a>

As Moonbeam development continues, it will sometimes be necessary to upgrade your node software.

To update moonriver client, you can keep your existing chain data in tact, and only update the binary by following these steps:

1. _Stop the systemd service_

```bash
sudo systemctl stop moonriver.service
```

2. _Remove the old binary file_

```bash
rm /var/lib/moonriver-data/moonbeam
```

3. _Get the latest version of the_ [_Moonbeam release binary on GitHub_](https://github.com/moonbeam-foundation/moonbeam/releases/) _and run the following command to update to that version (ensure to replace `INSERT_NEW_VERSION_TAG`with actual version)_

```bash
wget https://github.com/moonbeam-foundation/moonbeam/releases/download/INSERT_NEW_VERSION_TAG/moonbeam \
-O /var/lib/moonriver-data/moonbeam
```

4. _Update permissions_

```bash
chmod +x /var/lib/moonriver-data/moonbeam
```

5. _Start_ moonriver _service_

```bash
systemctl start moonriver.service
```

## References

{% embed url="https://docs.moonbeam.network/node-operators/networks/run-a-node/systemd/#introduction" %}

{% embed url="https://docs.moonbeam.network/node-operators/networks/tracing-node/#introduction" %}
