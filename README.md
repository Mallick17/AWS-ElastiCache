# ElastiCache Redis OSS (Cluster Mode Disabled) – Backup & Query Guide
## Prerequisites
Ensure you have:

* **ElastiCache Redis** with **Cluster Mode Disabled**
* **EC2 instance** in the **same VPC + subnet**
* EC2 security group allows **port 6379**
* Redis CLI installed on EC2 (`redis-cli` or `redis6-cli`)
* AWS CLI configured with proper permissions

---

## 📌 PART 1: Connect to ElastiCache and Query Data
### 🔧 Step 1: Install Redis CLI on EC2

For Amazon Linux 2023:

```bash
sudo dnf install redis -y
```

Manual build option:

```bash
curl -O http://download.redis.io/releases/redis-6.2.6.tar.gz
tar xzvf redis-6.2.6.tar.gz && cd redis-6.2.6
make
sudo cp src/redis-cli /usr/local/bin/
```

### 🔗 Step 2: Get Redis Primary Endpoint (for Cluster Mode Disabled)

```bash
aws elasticache describe-cache-clusters \
  --cache-cluster-id <your-cache-id> \
  --show-cache-node-info
```

Look for:

```
"Endpoint": {
  "Address": "your-primary-endpoint",
  "Port": 6379
}
```

### 🔌 Step 3: Connect Using Redis CLI

```bash
redis-cli -h <primary-endpoint> -p 6379
```

Example:

```bash
redis-cli -h redtaxi-noncluster.abc123.aps1.cache.amazonaws.com -p 6379
```

### 🧪 Step 4: Query Redis

```bash
PING
SET user:1:name "Alice"
GET user:1:name
KEYS *
INFO
```

---

## 💾 PART 2: Take Backup (Snapshot) using AWS CLI

ElastiCache supports **snapshots** even when **Cluster Mode is Disabled**, using the **cache cluster ID** instead of replication group ID.
### 📥 Step 5: Create Snapshot

```bash
aws elasticache create-snapshot \
  --snapshot-name redtaxi-backup-20250731 \
  --cache-cluster-id redtaxi-noncluster
```

### 🔁 Step 6: Verify Snapshot Status

```bash
aws elasticache describe-snapshots \
  --snapshot-name redtaxi-backup-20250731
```

Wait until:

```
"SnapshotStatus": "available"
```

---

## 🧪 PART 3: Simulate `.rdb` Backup on EC2 Redis
Since ElastiCache doesn't allow direct `.rdb` downloads, you can simulate a Redis backup via EC2.

### 🏗️ Step 7: Setup Redis Server on EC2

```bash
sudo dnf install redis -y
sudo systemctl start redis
```

---

### 📦 Step 8: Simulate Data + Trigger Save

```bash
redis-cli
SET product:101:name "Shoes"
SET product:101:price "3999"
SAVE
```

This creates a file at:

```bash
/var/lib/redis/dump.rdb
```

You can now copy this `.rdb` for offline or staging analysis.

---

### ♻️ Step 9: Restore `.rdb` to Redis

If needed:

```bash
sudo systemctl stop redis
sudo cp dump.rdb /var/lib/redis/dump.rdb
sudo chown redis:redis /var/lib/redis/dump.rdb
sudo systemctl start redis
```

Then query:

```bash
redis-cli
GET product:101:name
GET product:101:price
```

---

## 🧠 Summary

| Step          | Cluster Mode Disabled                        |
| ------------- | -------------------------------------------- |
| Connect       | `redis-cli -h <primary-endpoint> -p 6379`    |
| Backup        | `create-snapshot` using `--cache-cluster-id` |
| Restore       | Only to new cluster or simulate via EC2      |
| Query         | Use standard Redis CLI commands              |
| Export `.rdb` | Simulate via EC2 Redis `SAVE`                |

---

