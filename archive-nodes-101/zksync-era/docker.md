---
description: 'Authors: [ Ankur | Dapplooker]'
---

# 🐳 Docker

## System Requirements

<table data-full-width="false"><thead><tr><th>CPU</th><th>OS</th><th>RAM</th><th>DISK</th></tr></thead><tbody><tr><td>8 vCPU</td><td>Ubuntu 22.04</td><td>16 GB</td><td>10+ TB  (SSD)</td></tr></tbody></table>

{% hint style="success" %}
_The ZkSync node has a size of  6.5 TB on February 28, 2025._
{% endhint %}

## Pre-requisite

Before starting, clean the setup then update and upgrade. Install following:

* Docker
* Git
* Go v1.23+
* aria2 (optional)

### **Commands**

{% code overflow="wrap" %}
```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt install docker.io git ufw -y jq -y aria2 -y
```
{% endcode %}

## Firewall Settings

### Check status & enable UFW&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>sudo ufw enable
</strong>sudo ufw status verbose
</code></pre>

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
sudo ufw allow 3322 && sudo ufw allow 3061  # Allow for P2P & Metrics
```

### Allow Remote connection

```bash
sudo ufw allow from ${REMOTE.HOST.IP} to any port 3060
```

## Setup Instructions&#x20;

**Clone the ZkSync Era Repository**

<pre class="language-bash"><code class="lang-bash"><strong>git clone https://github.com/matter-labs/zksync-era.git
</strong>cd zksync-era/docs/src/guides/external-node/docker-compose-examples/
</code></pre>

### Download Snapshot

{% hint style="info" %}
To download the Latest Snapshot visit [https://en-backups.matterlabs.dev](https://en-backups.matterlabs.dev) and copy the desired link.\
Run the below command in `screen` as the download may take hours/day depending upon internet speed & snapshot size.
{% endhint %}

```bash
aria2c -c -x 16 -s 16 --dir=/path/to/download --out=latest_backup.sql.gz --file-allocation=none --timeout=600 --retry-wait=30 --max-tries=0 <snapshot_link>
# Example download link
mkdir ~/zksync-era/backup/
aria2c -c -x 16 -s 16 --dir=~/zksync-era/backup/ --out=latest_backup.sql.gz --file-allocation=none --timeout=600 --retry-wait=30 --max-tries=0 http://37.27.135.10:8080/ipfs/QmVWVRjw5DEuG5noaD7Ltcc4BGGCKEcjvL8CYFw5GcAzWD/external_node_latest.pgdump.sql.gz
```

### **Start ZkSync Node**&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>docker compose --file mainnet-external-node-docker-compose.yml up -d 
</strong></code></pre>

### Example docker-compose file

```yaml
name: "mainnet-node"
services:
  prometheus:
    image: prom/prometheus:v2.35.0
    volumes:
      - prometheus-data:/prometheus
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
    expose:
      - 9090
  grafana:
    image: grafana/grafana:9.3.6
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    environment:
      GF_AUTH_ANONYMOUS_ORG_ROLE: "Admin"
      GF_AUTH_ANONYMOUS_ENABLED: "true"
      GF_AUTH_DISABLE_LOGIN_FORM: "true"
    ports:
      - "127.0.0.1:3000:3000"
  postgres:
    image: "postgres:14"
    command: >
      postgres
      -c max_connections=200
      -c log_error_verbosity=terse
      -c shared_buffers=2GB
      -c effective_cache_size=4GB
      -c maintenance_work_mem=1GB
      -c checkpoint_completion_target=0.9
      -c random_page_cost=1.1
      -c effective_io_concurrency=200
      -c min_wal_size=4GB
      -c max_wal_size=16GB
      -c max_worker_processes=16
      -c checkpoint_timeout=1800
    expose:
      - 5430
    volumes:
      - postgres:/var/lib/postgresql/data
      - ~/zksync-era/backup/latest_backup.sql.gz:/usr/src/en/pg_backups/latest_backup.sql.gz
    healthcheck:
      interval: 1s
      timeout: 3s
      test:
        [
          "CMD-SHELL",
          'psql -U postgres -c "select exists (select * from pg_stat_activity where datname = ''{{ database_name }}'' and application_name = ''pg_restore'')" | grep -e ".f$$"',
        ]
    environment:
      - POSTGRES_PASSWORD=notsecurepassword
      - PGPORT=5430
  # Generation of consensus secrets.
  # The secrets are generated iff the secrets file doesn't already exist.
  generate-secrets:
    image: "matterlabs/external-node:2.0-v26.2.1"
    entrypoint:
      [
      "/configs/generate_secrets.sh",
      "/configs/mainnet_consensus_secrets.yaml",
      ]
    volumes:
      - ./configs:/configs
  external-node:
    image: "matterlabs/external-node:2.0-v26.2.1"
    entrypoint:
      [
      "/usr/bin/entrypoint.sh",
      "--enable-consensus",
      ]
    restart: always
    depends_on:
      postgres:
        condition: service_healthy
      generate-secrets:
        condition: service_completed_successfully
    ports:
      - "0.0.0.0:3054:3054" # consensus public port
      - "0.0.0.0:3060:3060"
      - "127.0.0.1:3061:3061"
      - "127.0.0.1:3081:3081"
      - "127.0.0.1:5000:5000"
    volumes:
      - rocksdb:/db
      - ./configs:/configs
    expose:
      - 3322
    environment:
      DATABASE_URL: "postgres://postgres:notsecurepassword@postgres:5430/zksync_local_ext_node"
      DATABASE_POOL_SIZE: 10

      EN_HTTP_PORT: 3060
      EN_WS_PORT: 3061
      EN_HEALTHCHECK_PORT: 3081
      EN_PROMETHEUS_PORT: 3322
      EN_ETH_CLIENT_URL: https://ethereum-rpc.publicnode.com
      EN_MAIN_NODE_URL: https://zksync2-mainnet.zksync.io
      EN_L1_CHAIN_ID: 1
      EN_L2_CHAIN_ID: 324
      EN_PRUNING_ENABLED: false

      EN_STATE_CACHE_PATH: "./db/ext-node/state_keeper"
      EN_MERKLE_TREE_PATH: "./db/ext-node/lightweight"
      RUST_LOG: "warn,zksync=info,zksync_core::metadata_calculator=debug,zksync_state=debug,zksync_utils=debug,zksync_web3_decl::client=error"

      EN_CONSENSUS_CONFIG_PATH: "/configs/mainnet_consensus_config.yaml"
      EN_CONSENSUS_SECRETS_PATH: "/configs/mainnet_consensus_secrets.yaml"

volumes:
  postgres: {}
  rocksdb: {}
  prometheus-data: {}
  grafana-data: {}
```

### Import Database

```bash
docker exec -it mainnet-node-postgres-1 bash
zcat /usr/src/en/pg_backups/latest_backup.sql.gz | nohup psql -U postgres -d zksync_local_ext_node & disown

# Check Database Size
docker exec -it mainnet-node-postgres-1 psql -U postgres -c "SELECT pg_size_pretty
```

{% hint style="warning" %}
Ensure that your database has finished importing before proceeding to the next step.
{% endhint %}

### Perform Database Migration

```bash
cd /root/zksync/repo/usr/bin

# Set migration folder as source and specify database
nohup ./sqlx migrate info --source /root/FINAL/repo/migrations --database-url postgres://postgres:password@localhost/zksync_local_ext_node & disown

# Run Database Migration
nohup ./sqlx migrate run --source /root/FINAL/repo/migrations --database-url postgres://postgres:password@localhost/zksync_local_ext_node & disown
```

### Restart the Containers

```bash
docker compose --file ~/zksync-era/docs/src/guides/external-node/docker-compose-examples/mainnet-external-node-docker-compose.yml up -d --force-recreate
```

## Monitoring

### Monitor Logs of Docker Container&#x20;

```bash
docker ps 
docker logs mainnet-node-external-node-1
```

## Sync Status

* **Syncing status** :&#x20;

```bash
curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0", "method":"eth_syncing", "params":[], "id":1}' http://localhost:3060
```

&#x20;_Response should look like:_

<pre class="language-json"><code class="lang-json"><strong>{"jsonrpc":"2.0","id":1,"result":{"startingBlock":"0x0","currentBlock":"0x3219750","highestBlock":"0x35fc3d7"}}
</strong><strong>                                            
</strong><strong>                                            # or
</strong><strong>                                            
</strong><strong> {"jsonrpc":"2.0","id":1,"result":false} # If block is synced or &#x3C; 11 block behind
</strong></code></pre>

* **Latest synchronized L2 block** :

```bash
curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber", "params":[],"id":1}' http://localhost:3060
```

_Response should look like:_

```json
{"jsonrpc":"2.0","id":83,"result":"0x2112b2d"}
```

## References

{% embed url="https://github.com/matter-labs/zksync-era" %}

{% embed url="https://explorer.zksync.io/" %}
