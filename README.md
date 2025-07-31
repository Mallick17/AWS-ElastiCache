# **Accessing and Querying the Restored Data**

> Note: Without these steps we cant query the cache data

> 1: **_[Set up a compute connection to automatically configure connectivity between your EC2 instance and cache, both of which must reside in the same VPC.](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/compute-connection.html)_**

> 2: **_[Install the Redis OSS CLI  on your EC2 instance](https://redis.io/docs/latest/operate/rs/references/cli-utilities/redis-cli/)_**

> 3: **_Connect to your Redis OSS cache using the following command which is shown in the console_**

## Working with Keys in Redis OSS (ElastiCache)
Redis uses key-value data structures. Each key maps to a value that can be a string, list, set, hash, etc. In ElastiCache Redis (Cluster Mode Disabled), all keys are stored in a **single logical database** and can be queried, created, and managed using `redis-cli` or any Redis client.
- Once your Redis instance (ElastiCache or other) has successfully loaded the `.rdb` file, the keys and values from that snapshot will be available in memory.
- You can now **connect** to the Redis cache and use standard **Redis CLI commands** to query or explore your data.
- For example, assuming the backup contained a key called `user:1001:name` with value `"Gyan"`:

---

## Querying Existing Keys (Step-by-step):
### 1. **List All Keys** (Not recommended in production with many keys)

```bash
KEYS *
```

### 2. **Find Keys by Pattern**

```bash
KEYS user:*
KEYS session:*
```

### 3. **Check if Key Exists**

```bash
EXISTS user:1:name
```
### 4. **Get Value of a Key**
**Get the value of a known key**:

```bash
GET user:1:name
```

   ```bash
   GET mykey
   ```

   Example:

   ```bash
   GET user:1001:name
   ```

   Output:

   ```
   "Gyan"
   ```

### 5. **Check TTL (Time to Live)**

```bash
TTL user:1:name
```

### 6. **Get all fields from a hash** (if data was stored in hash format):

   ```bash
   HGETALL user:1001
   ```

<details>
   <summary>Clcik to view Examples</summary>

### Sample: Use the Provided Example in Your Redis

From the AWS guide:

```bash
SET mykey "Hello, ElastiCache!"
GET mykey
```

This means:

* Key: `mykey`
* Value: `"Hello, ElastiCache!"`

---

### Example: Working with Restored Data

Let's say your restored backup had the following:

| Redis Key        | Value                 |
| ---------------- | --------------------- |
| `session:abc123` | `{"user_id": 42}`     |
| `user:42:name`   | `"Alice"`             |
| `user:42:email`  | `"alice@example.com"` |

You can now query like:

```bash
GET session:abc123
GET user:42:name
GET user:42:email
```

Or, if stored as hash:

```bash
HGETALL user:42
```

</details>

---

### Tip: Verify RDB Load

If you're unsure whether the RDB file was successfully loaded:

```bash
INFO Persistence
```

Look for:

```
loading:0
rdb_last_load_time_sec:...
```

Also useful:

```bash
INFO Keyspace
```

Shows how many keys exist in each database.

---

## Creating and Storing New Keys

### 1. **Simple String Key**

```bash
SET user:1:name "Alice"
```

### 2. **Add Expiry to a Key**

```bash
SETEX otp:123456 300 "abc789"  # expires in 300 seconds
```

### 3. **Hash (similar to objects/dictionaries)**

```bash
HMSET user:1 name "Alice" email "alice@example.com"
HGETALL user:1
```

### 4. **List**

```bash
LPUSH tasks "task1"
LPUSH tasks "task2"
LRANGE tasks 0 -1
```

---

## Accessing Stored Keys

### 1. **Access String Value**

```bash
GET user:1:name
```

### 2. **Access Hash Field**

```bash
HGET user:1 email
HGETALL user:1
```

### 3. **Access List Items**

```bash
LRANGE tasks 0 -1
```

### 4. **Access Expiring Key**

If you try to access a key after it expires:

```bash
GET otp:123456
# (nil)
```

---

## Useful Commands for Managing Keys

| Command              | Description                              |
| -------------------- | ---------------------------------------- |
| `DEL key`            | Delete a key                             |
| `EXPIRE key seconds` | Set expiration                           |
| `TTL key`            | Show time to live                        |
| `PERSIST key`        | Remove expiration                        |
| `TYPE key`           | Show key type (string, list, hash, etc.) |

---

## Best Practices for Keys

* Use **namespaces** with colons: `user:123:name`, `session:abc123`
* Avoid `KEYS *` in production — use **SCAN** instead:

  ```bash
  SCAN 0 MATCH user:* COUNT 10
  ```
* Set **expiration** for temporary keys like OTPs or sessions
* Monitor memory usage with:

  ```bash
  INFO memory
  ```

---

# Redis OSS Backup & Restore on ElastiCache – Full Guide

## Overview

This guide explains how to:

* Connect to your ElastiCache Redis cluster
* Run Redis CLI commands
* Backup data (RDB snapshots) using AWS CLI
* Restore and query Redis data
* Use `.rdb` files with EC2 for inspection/testing
* Best practices for serverless/fine-grained node-based environments

---

## Architecture Assumptions

* You are using **Amazon ElastiCache for Redis OSS**
* Your setup is **serverless or dynamic**, and Redis nodes are **fine-grained/scaled**
* You want to **create backups (snapshots)** and **query/restored data**
* You have access to an **EC2 instance** in the same **VPC** as ElastiCache

---

## Step 1: Connect to ElastiCache Cluster

### Prerequisites

* ElastiCache Redis cluster must be **in the same VPC** as the EC2 instance
* Port **6379** must be open in the **Security Group**
* Redis CLI installed (you can use `redis6-cli` or compile from source)

### 💻 Connect from EC2 using Redis CLI

```bash
redis6-cli -h <primary-endpoint> -p 6379
```

Example:

```bash
redis6-cli -h redtaxi-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com -p 6379
```

---

## Step 2: Basic Redis Commands

Once connected, test Redis functionality:

```bash
SET mykey "Hello, ElastiCache!"
GET mykey
PING
INFO
```

Example:

```bash
> SET user:42:name "Alice"
OK
> GET user:42:name
"Alice"
```

To explore all keys:

```bash
KEYS *
```

**Note:** Avoid `KEYS *` in production — use patterns:

```bash
KEYS user:*
```

---

## Step 3: Take a Backup (Snapshot) Using AWS CLI

### Prerequisites

* You must have **AWS CLI** configured with permissions:

  * `elasticache:CreateSnapshot`
  * `elasticache:DescribeSnapshots`
  * `elasticache:CopySnapshot` (optional)
* Your cluster must be in a **running** and **available** state

### Command to Create Snapshot

```bash
aws elasticache create-snapshot \
  --snapshot-name my-cluster-snapshot-20250731 \
  --replication-group-id my-redis-cluster-id
```

**Example:**

```bash
aws elasticache create-snapshot \
  --snapshot-name redtaxi-prod-backup-20250731 \
  --replication-group-id redtaxi-prod-cluster
```

### Verify Snapshot

```bash
aws elasticache describe-snapshots \
  --snapshot-name redtaxi-prod-backup-20250731
```

Snapshot statuses:

* `creating`
* `available`
* `failed`

---

## Step 4: Restore Backup from Snapshot

### Restoring to a New Cluster

```bash
aws elasticache restore-replication-group-from-snapshot \
  --replication-group-id restored-cluster-20250731 \
  --snapshot-name redtaxi-prod-backup-20250731 \
  --cache-node-type cache.t4g.small \
  --engine redis \
  --cache-subnet-group-name my-subnet-group
```

Replace `cache.t4g.small` with your desired node type.

---

## Step 5: Query the Restored Data

After the cluster is restored:

1. Connect again via EC2 using:

   ```bash
   redis6-cli -h <restored-cluster-endpoint> -p 6379
   ```

2. Query known keys from the snapshot:

   ```bash
   GET user:42:name
   HGETALL user:42
   ```

3. Check if data was restored:

   ```bash
   INFO Keyspace
   ```

---

## Optional: Load `.rdb` into a Local Redis (via EC2)

If you want to inspect an RDB file manually:

### 🔧 Steps

1. **Install Redis** on EC2:

   ```bash
   sudo yum install redis -y
   ```

2. **Stop Redis**:

   ```bash
   sudo systemctl stop redis
   ```

3. **Copy your `.rdb`** file to:

   ```bash
   sudo mv dump.rdb /var/lib/redis/dump.rdb
   ```

4. **Restart Redis**:

   ```bash
   sudo systemctl start redis
   ```

5. **Connect** using:

   ```bash
   redis-cli
   ```

---

## Best Practices for Serverless/Fine-Grained Node Setups

* Use **automated snapshot schedules** via ElastiCache to avoid manual operations
* Store backups in **Amazon S3** by exporting snapshots
* Enable **Encryption at Rest** and **in-transit** for production data
* Consider using **Redis AUTH** or **RBAC** for secure access
* For low-memory fine-tuned nodes, **monitor snapshot frequency** and **memory consumption** before snapshotting
* Automate snapshot and restore through **CloudWatch Events + Lambda** if in a serverless architecture

---

## Resources

* [ElastiCache Redis Snapshots Docs](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/backups-snapshots.html)
* [AWS CLI ElastiCache Reference](https://docs.aws.amazon.com/cli/latest/reference/elasticache/index.html)
* [Redis CLI Commands](https://redis.io/commands)

---

