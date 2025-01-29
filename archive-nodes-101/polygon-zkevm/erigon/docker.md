# 🐳 Docker

Authors: \[ Ankur | Dapplooker]

## System Requirements

<table data-full-width="false"><thead><tr><th>CPU</th><th>OS</th><th>RAM</th><th>DISK</th></tr></thead><tbody><tr><td>4 vCPU</td><td>Ubuntu 22.04</td><td>64 GB</td><td>1TB (SSD)</td></tr></tbody></table>

{% hint style="info" %}
_The CDK-Erigon archival node has a size of 104GB on January 29, 2025._
{% endhint %}

## Pre-requisite

Before starting, clean the setup then update and upgrade. Install following:

* Docker
* Docker Compose
* Git
* Go v1.19 +
* L1 Ethereum node RPC&#x20;

### **Commands:**

{% code overflow="wrap" %}
```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt install docker.io docker-compose git ufw -y 
```
{% endcode %}

## Firewall Settings:

### Set explicit default UFW rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### Allow SSH, HTTP, and HTTPS

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80
sudo ufw allow 443
```

### Allow Remote connection
