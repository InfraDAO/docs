# 🐳 Docker

Author: \[ jLeopoldA ]

### System Requirements <a href="#system-requirements" id="system-requirements"></a>

Comment

| CPU    | OS                  | RAM  | DISK  |
| ------ | ------------------- | ---- | ----- |
| 4 Core | Ubunutu 24.04.1 LTS | 16GB | 128GB |

{% hint style="info" %}
The Polygon zkEVM archive node has a size of 103GB as of 1/15/2025
{% endhint %}

### Pre-Requisites <a href="#pre-requisites" id="pre-requisites"></a>

{% hint style="info" %}
CDK-Erigon requires the installation of Go.
{% endhint %}

### Update System

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -ye
```

### Set up Firewall <a href="#set-up-firewall" id="set-up-firewall"></a>

**Set Explicit Default Firewall Rules**

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

**Allow SSH**

\
