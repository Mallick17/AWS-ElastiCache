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

<details>
   <summary>Click to view the commands</summary>

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

</details>

---

## Creating and Storing New Keys

<details>
   <summary>Click to view the commands</summary>


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

</details>

---

## Accessing Stored Keys


<details>
   <summary>Click to view the commands</summary>
   
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

</details>

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

<details>
   <summary>Click to view the steps of execution</summary>

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

</details>

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

# Redis Installation, Setup, Key Insert & Backup, and Key Sync (Local ↔ ElastiCache)

<details>
    <summary>Clean Documentation without Error</summary>

Here is a clean, detailed step-by-step documentation based on your setup for **exporting Redis keys from AWS ElastiCache**, **navigating multiple Redis databases**, and using `redis-cli` effectively.

---

# 🧾 Redis ElastiCache Access & Export Guide (RHEL/YUM-based Linux)

## 📌 Objective

* Install and use `redis-cli` to access **AWS ElastiCache for Redis**
* Export all keys and values to a local JSON file via Python
* Handle multiple Redis databases (Redis supports 16 DBs by default: `0–15`)
* Restore/export data safely by identifying key types

---

## 1. 🧱 Prerequisites

### ✅ AWS ElastiCache Redis

* Confirm endpoint (e.g., `<your-endpoint>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com`)
* Port (default: `6379`)
* Make sure **security groups** allow access from your EC2 or local instance

### ✅ Linux EC2 (RHEL/CentOS/Amazon Linux)

Update the system and install dependencies:

```bash
sudo yum update -y
sudo yum install -y gcc jemalloc-devel tcl
sudo yum groupinstall -y "Development Tools"
```

---

## 2. 📥 Install Redis CLI Only (No Server Daemon)

### A. Download & Compile Redis

```bash
curl -O http://download.redis.io/redis-stable.tar.gz
tar xzvf redis-stable.tar.gz
cd redis-stable
make redis-cli
sudo cp src/redis-cli /usr/local/bin/
cd ..
rm -rf redis-stable redis-stable.tar.gz
```

### B. Verify Installation

```bash
redis-cli --version
# Output: redis-cli 7.x.x
```

---

## 3. 🔌 Connect to AWS ElastiCache

```bash
redis-cli -h <your-endpoint>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com -p 6379
```

### ✅ Redis CLI Connected Output:

```bash
<your-endpoint>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com:6379>
```

---

## 4. 🧭 Working with Multiple Redis Databases

### A. By default, you're in DB 0:

Check keys:

```redis
dbsize
keys *
```

### B. Switch to another DB:

```redis
select 1
```

Check for key existence:

```redis
keys *
```

> output
> 13411) "cabs.5114.live_details"
> 13412) "cabs.9161.live_details"
> 13413) "cabs.8788.live_details"
> 13414) "cabs.8163.live_details"
> 13415) "cabs.12103.live_details"
> 13416) "cabs.4001.live_details"
> 13417) "cabs.12496.live_details"
> 13418) "cabs.10590.live_details"
> 13419) "cabs.9175.live_details"
> 13420) "cabs.5916.live_details"
> 13421) "cabs.369.live_details"
> (0.57s)

### C. Multiple Switches to another-another DB:
```
<your-endpoint>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com:6379[3]> select 4
OK
<your-endpoint>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com:6379[4]> select 5
OK
<your-endpoint>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com:6379[5]> select 6
OK
<your-endpoint>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com:6379[6]> keys *
1) "user_check_fare_booking_counts"
2) "user_app_search_session_details:2025-08-06"
<your-endpoint>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com:6379[6]> 
```

Repeat `select N` from 0 to 15 to check each DB:

```bash
for i in {0..15}; do
  echo "DB $i:"
  redis-cli -h <your-endpoint>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com -p 6379 -n $i dbsize
done
```

### D. Admin Commands Blocked in ElastiCache

The following do **not** work in AWS-managed Redis:

```redis
config get databases     # ❌ Blocked
monitor                  # ❌ Blocked
```

---

## 5. 📤 Export All Redis Keys with Python

### A. Install Required Python Modules

```bash
sudo yum install -y python3-pip
pip3 install redis
```

### B. Save the following script as `sync_redis_keys.py`:

```python
import redis
import json

source_redis = redis.Redis(
    host='<your-endpoint>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com',
    port=6379,
    decode_responses=True,
    db=1  # Change DB number as needed
)

export_data = []

print("📤 Exporting keys from Redis...\n")
cursor = '0'
while True:
    cursor, keys = source_redis.scan(cursor=cursor, count=100)
    for key in keys:
        key_type = source_redis.type(key)
        ttl = source_redis.ttl(key)
        entry = {"key": key, "type": key_type, "ttl": ttl, "value": None}
        try:
            if key_type == 'string':
                entry["value"] = source_redis.get(key)
            elif key_type == 'hash':
                entry["value"] = source_redis.hgetall(key)
            elif key_type == 'set':
                entry["value"] = list(source_redis.smembers(key))
            elif key_type == 'list':
                entry["value"] = source_redis.lrange(key, 0, -1)
            elif key_type == 'zset':
                entry["value"] = source_redis.zrange(key, 0, -1, withscores=True)
            else:
                print(f"⚠️ Unknown type '{key_type}' for key '{key}' — skipping")
                continue
            export_data.append(entry)
        except Exception as e:
            print(f"❌ Failed to export key '{key}': {e}")
    if cursor == '0':
        break

with open("redis_export.json", "w") as f:
    json.dump(export_data, f, indent=2)

print(f"\n✅ Exported {len(export_data)} keys to redis_export.json")
```

### C. Run the Export

```bash
python3 sync_redis_keys.py
```

✅ Output:

```bash
📤 Exporting keys from Redis...

✅ Exported 45 keys to redis_export.json
```

---

## 6. 🧪 Testing Redis Keys in CLI

Inside the correct DB (`select 1` for example):

```redis
exists cabs.7204.live_details
# (integer) 1

ttl cabs.7204.live_details
# (integer) 1450 (or -1 if no expiry)

type cabs.7204.live_details
# hash

hgetall cabs.7204.live_details
```

---

### 🔍 Workaround to Detect Active Databases
- By default, Redis is configured with 16 logical databases (0–15). This is controlled by the `databases` setting in `redis.conf`.
- However, in AWS ElastiCache, you cannot run `CONFIG GET databases` because administrative commands like `CONFIG`, `MONITOR`, etc. are blocked for security reasons.
- Since CONFIG GET databases doesn't work, the best method is to probe each DB using a script:

```
for db in {0..15}; do
  echo -n "DB $db: "
  redis-cli -h <your-endpoint> -p 6379 -n $db dbsize
done
```

or

```
for i in {0..15}; do echo "DB $i:"; redis-cli -h redtaxi-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com -p 6379 -n $i dbsize; done
```

- Output Example
  
```
[root@ip-172-31-35-246 ~]# for db in {0..15}; do   echo -n "DB $db: ";   redis-cli -h <your-endpoint>.bp8cjs.ng.0001.aps1.cache.amazonaws.com -p 6379 -n $db dbsize; done
DB 0: (integer) 637
DB 1: (integer) 502
DB 2: (integer) 1325
DB 3: (integer) 3650
DB 4: (integer) 394
DB 5: (integer) 13418
DB 6: (integer) 2
DB 7: (integer) 0
DB 8: (integer) 0
DB 9: (integer) 0
DB 10: (integer) 0
DB 11: (integer) 0
DB 12: (integer) 0
DB 13: (integer) 0
DB 14: (integer) 0
DB 15: (integer) 0
```

> Replace <your-endpoint> with your actual ElastiCache endpoint (without redis:// prefix).

---

## 🧠 Redis Data Types Recap

| Redis Type | CLI Command                  | Python Access via redis-py            |
| ---------- | ---------------------------- | ------------------------------------- |
| string     | `get key`                    | `get(key)`                            |
| hash       | `hgetall key`                | `hgetall(key)`                        |
| list       | `lrange key 0 -1`            | `lrange(key, 0, -1)`                  |
| set        | `smembers key`               | `smembers(key)`                       |
| zset       | `zrange key 0 -1 withscores` | `zrange(key, 0, -1, withscores=True)` |

---

## 7. ✅ Summary

* You successfully installed and compiled `redis-cli`
* Connected to AWS ElastiCache and selected the correct DB
* Exported key data to a JSON file with types and TTLs preserved
* Avoided admin commands (disabled in AWS)
* Used `php artisan tinker` and Laravel to cross-check which DB holds your keys

---

</details>

## 1. Install Redis on Amazon Linux (with build from source)

```bash
# Install Redis via DNF (optional, if you just want the binary)
sudo dnf install redis -y
```

### Alternatively, build Redis from source:

```bash
# Install required build tools
sudo yum install -y gcc gcc-c++ make wget

# Download and extract Redis
wget http://download.redis.io/redis-stable.tar.gz
tar xvzf redis-stable.tar.gz
cd redis-stable

# Compile Redis
make distclean   # optional: clean previous builds
make             # builds all components including redis-cli and redis-server

# Verify CLI tool
./src/redis-cli --version

# (Optional) Install Redis binaries globally
sudo make install
```

---

## 2. Start Redis Server Locally

```bash
# From inside the redis-stable directory
./src/redis-server ./redis.conf
```

In another terminal:

```bash
cd redis-stable/src/
./redis-cli
```

---

## 3. Create and View Redis Keys

```redis
127.0.0.1:6379> SET mykey "adith"
127.0.0.1:6379> GET mykey
"adith"

127.0.0.1:6379> SET mykey1 "abishek"
127.0.0.1:6379> SET mykey2 "big boss"
127.0.0.1:6379> SET mykey3 "abi"
127.0.0.1:6379> SET mykey4 "abishekboss"
127.0.0.1:6379> SET mykey5 "Jeeva"
127.0.0.1:6379> SET mykey6 "santhosh"
127.0.0.1:6379> SET mykey7 "santosh"
127.0.0.1:6379> SET mykey8 "Thrisan"
127.0.0.1:6379> SET mykey9 "Mallick"
127.0.0.1:6379> SET mykey10 "Gyan"

# Save snapshot to dump.rdb
127.0.0.1:6379> SAVE
```

---

## 4. Backup dump.rdb File

```bash
# Ensure you're in redis-stable/src
cd /root/redis-stable/src

# Copy dump.rdb to root directory
cp dump.rdb /root/
cd ~
mv dump.rdb dump31072025.rdb

# To restore later:
mv dump31072025.rdb dump.rdb
mv dump.rdb /root/redis-stable/src/
```

---

## 5. Verify Key Persistence

```bash
cd /root/redis-stable/src/
./redis-cli

127.0.0.1:6379> KEYS *
127.0.0.1:6379> GET mykey
"abishek pre married boss"
```

---

## 6. Sync Keys from AWS ElastiCache to Local Redis

### Install Redis Python Client

```bash
sudo dnf install -y python3-pip
pip3 install redis
```

### Create sync script: `sync_redis_keys.py`

```python
import redis
import warnings

# --- Connect to ElastiCache Redis (source) ---
source_redis = redis.Redis(
    host='<name>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com',
    port=6379,
    decode_responses=True
)

# --- Connect to Local Redis (destination) ---
destination_redis = redis.Redis(
    host='127.0.0.1',
    port=6379,
    decode_responses=True
)

# --- Fetch and Sync Keys ---
keys = source_redis.keys('*')
print(f"Found {len(keys)} keys. Starting copy...\n")

for key in keys:
    key_type = source_redis.type(key)
    ttl = source_redis.ttl(key)

    try:
        if key_type == 'string':
            value = source_redis.get(key)
            destination_redis.set(key, value)

        elif key_type == 'hash':
            hash_data = source_redis.hgetall(key)
            if hash_data:
                destination_redis.hset(key, mapping=hash_data)

        elif key_type == 'set':
            members = source_redis.smembers(key)
            if members:
                destination_redis.sadd(key, *members)

        else:
            print(f"Skipping unsupported key type: {key_type} for key: {key}")
            continue

        if ttl and ttl > 0:
            destination_redis.expire(key, ttl)

    except Exception as e:
        print(f"❌ Error copying key '{key}': {e}")

print("\n✅ Copy complete.")
```

### Run the script:

```bash
python3 sync_redis_keys.py
```

You will see logs like:

```
Found 605 keys. Starting copy...
Skipping unsupported key type: set for key: laravel:...
✅ Copy complete.
```

### Cross-check:

```bash
# ElastiCache
redis-cli -h <name>.bp8cjs.ng.0001.aps1.cache.amazonaws.com -p 6379

# Local
./redis-cli
127.0.0.1:6379> KEYS *
```

---

## Optional Cleanup

```bash
# Remove the sync script if no longer needed
rm sync_redis_keys.py
```

---

## Summary

| Step | Description                                            |
| ---- | ------------------------------------------------------ |
| 1    | Installed Redis and/or compiled from source            |
| 2    | Started the Redis server locally                       |
| 3    | Inserted and saved keys using `redis-cli`              |
| 4    | Backed up and restored `dump.rdb`                      |
| 5    | Synced keys from AWS ElastiCache to local using Python |
| 6    | Verified keys were successfully copied                 |

---


# Restore Redis Keys from Local `dump.rdb` to AWS ElastiCache

## Why `dump.rdb` Cannot Be Directly Restored to ElastiCache

ElastiCache Redis is a **managed service** and does **not support direct file access** like a self-hosted Redis server.

> ❌ You cannot upload `dump.rdb` directly to ElastiCache.

## Solution: Use Key-Level Sync via Scripts

Instead of direct file-level operations, you must:

* Load the `dump.rdb` into a **local Redis instance**
* Use **scripts (e.g., Python)** to read keys from the local Redis and write them to ElastiCache

---

# Full Restore: All Keys from Local to ElastiCache

### Step 1: Load `dump.rdb` into Local Redis

```bash
# Stop running Redis
sudo systemctl stop redis

# Replace dump file
sudo cp /path/to/your/dump.rdb /var/lib/redis/dump.rdb
sudo chown redis:redis /var/lib/redis/dump.rdb

# Restart Redis
sudo systemctl start redis
```

This will load all keys from the `dump.rdb` into your local Redis server.

### Step 2: Sync All Keys to ElastiCache via Python

#### Install the Redis client

```bash
pip3 install redis
```

#### Python script: `push_to_elasticache.py`

```python
import redis

# Connect to local Redis (where dump.rdb was loaded)
source = redis.StrictRedis(host='localhost', port=6379, decode_responses=True)

# Connect to ElastiCache
destination = redis.StrictRedis(
    host='<name>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com',
    port=6379,
    decode_responses=True
)

# Copy all keys
for key in source.keys('*'):
    key_type = source.type(key)

    if key_type == 'string':
        destination.set(key, source.get(key))
    elif key_type == 'list':
        destination.delete(key)
        destination.rpush(key, *source.lrange(key, 0, -1))
    elif key_type == 'set':
        destination.delete(key)
        destination.sadd(key, *source.smembers(key))
    elif key_type == 'zset':
        destination.delete(key)
        members = source.zrange(key, 0, -1, withscores=True)
        destination.zadd(key, dict(members))
    elif key_type == 'hash':
        destination.delete(key)
        destination.hset(key, mapping=source.hgetall(key))
    else:
        print(f"❌ Skipping unsupported type {key_type} for key: {key}")

print("✅ Restore from local Redis to ElastiCache complete.")
```

#### Run the script:

```bash
python3 push_to_elasticache.py
```

### Verify Keys in ElastiCache

```bash
redis-cli -h <name>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com -p 6379
> keys *
```

---

# Delete and Restore a Specific Key

### Step 1: Delete the Key in ElastiCache

```bash
redis-cli -h <name>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com -p 6379 DEL "your_key_name"
```

> 💡 Use `\` to escape spaces or wrap in quotes:

```bash
DEL "rate_limit_summary_user:2025-07-31 17:03"
```

### Step 2: Restore One Key via Python

#### Minimal restore script: `restore_single_key.py`

```python
import redis

KEY_TO_RESTORE = 'rate_limit_summary_user:2025-07-31 17:03'

source = redis.StrictRedis(host='localhost', port=6379, decode_responses=True)
destination = redis.StrictRedis(
    host='<name>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com',
    port=6379,
    decode_responses=True
)

key_type = source.type(KEY_TO_RESTORE)

if key_type == 'string':
    destination.set(KEY_TO_RESTORE, source.get(KEY_TO_RESTORE))
elif key_type == 'list':
    destination.delete(KEY_TO_RESTORE)
    destination.rpush(KEY_TO_RESTORE, *source.lrange(KEY_TO_RESTORE, 0, -1))
elif key_type == 'set':
    destination.delete(KEY_TO_RESTORE)
    destination.sadd(KEY_TO_RESTORE, *source.smembers(KEY_TO_RESTORE))
elif key_type == 'zset':
    destination.delete(KEY_TO_RESTORE)
    members = source.zrange(KEY_TO_RESTORE, 0, -1, withscores=True)
    destination.zadd(KEY_TO_RESTORE, dict(members))
elif key_type == 'hash':
    destination.delete(KEY_TO_RESTORE)
    destination.hset(KEY_TO_RESTORE, mapping=source.hgetall(KEY_TO_RESTORE))
else:
    print(f"❌ Unsupported key type: {key_type}")

print(f"✅ Key restored to ElastiCache: {KEY_TO_RESTORE}")
```

### Step 3: Verify the Key in ElastiCache

```bash
# For strings
redis-cli -h <name>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com -p 6379 GET "your_key_name"

# For hashes
redis-cli -h <name>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com -p 6379 HGETALL "your_key_name"
```

---

# Backup One Specific Key from ElastiCache to Local Redis

### Step 1: Copy Key from ElastiCache → Local Redis

Create script `copy_key_to_local.py`:

```python
import redis

KEY = 'your_key_name_here'

source = redis.StrictRedis(
    host='<name>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com',
    port=6379,
    decode_responses=True
)

destination = redis.StrictRedis(host='localhost', port=6379, decode_responses=True)

key_type = source.type(KEY)

if key_type == 'none':
    print(f"❌ Key not found: {KEY}")
elif key_type == 'string':
    destination.set(KEY, source.get(KEY))
elif key_type == 'list':
    destination.delete(KEY)
    destination.rpush(KEY, *source.lrange(KEY, 0, -1))
elif key_type == 'set':
    destination.delete(KEY)
    destination.sadd(KEY, *source.smembers(KEY))
elif key_type == 'zset':
    destination.delete(KEY)
    z_members = source.zrange(KEY, 0, -1, withscores=True)
    destination.zadd(KEY, dict(z_members))
elif key_type == 'hash':
    destination.delete(KEY)
    destination.hset(KEY, mapping=source.hgetall(KEY))
else:
    print(f"❌ Unsupported key type: {key_type}")

print(f"✅ Key '{KEY}' copied to local Redis.")
```

### Step 2: Save `dump.rdb` from Local Redis

```bash
redis-cli SAVE
```

This will save the updated key to `/var/lib/redis/dump.rdb`.

---

## Important Notes

* ElastiCache **does NOT support** commands like:

  * `SAVE`, `RESTORE`, `BGSAVE`, `SYNC`, or loading from `dump.rdb`
* You **must** use **key-level logic** to migrate or restore keys.
* Scripts like the ones shown are the **only reliable way** to move data between Redis clusters.

---

## Optional Enhancements

* Support for TTLs (expire times)
* Logging or dry-run mode
* Filtering by key pattern (e.g., only `prefix:*`)

---


<details>
    <summary>Clean Method for mallow</summary>

---

## ✅ Option 1: Export keys + values + types to JSON

Instead of syncing directly to Redis, you export everything to a structured `.json` file like this:

```json
[
  {
    "key": "user:123",
    "type": "hash",
    "ttl": 3600,
    "value": {
      "name": "John",
      "email": "john@example.com"
    }
  },
  {
    "key": "online_users",
    "type": "set",
    "ttl": -1,
    "value": ["user1", "user2"]
  },
  ...
]
```

Then you can restore each key **accurately**, based on its saved type.

---

## ✅ Here's the Updated Python Script to **Export to JSON**

```python
import redis
import json

source_redis = redis.Redis(
    host='<your-endpoint>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com',
    port=6379,
    decode_responses=True
)

export_data = []

print("📤 Exporting keys from Redis...\n")
cursor = '0'
while cursor != 0:
    cursor, keys = source_redis.scan(cursor=cursor, count=100)

    for key in keys:
        key_type = source_redis.type(key)
        ttl = source_redis.ttl(key)
        entry = {"key": key, "type": key_type, "ttl": ttl, "value": None}

        try:
            if key_type == 'string':
                entry["value"] = source_redis.get(key)

            elif key_type == 'hash':
                entry["value"] = source_redis.hgetall(key)

            elif key_type == 'set':
                entry["value"] = list(source_redis.smembers(key))

            elif key_type == 'list':
                entry["value"] = source_redis.lrange(key, 0, -1)

            elif key_type == 'zset':
                entry["value"] = source_redis.zrange(key, 0, -1, withscores=True)

            else:
                print(f"⚠️ Unknown type '{key_type}' for key '{key}' — skipping")
                continue

            export_data.append(entry)

        except Exception as e:
            print(f"❌ Failed to export key '{key}': {e}")

# Write to file
with open("redis_export.json", "w") as f:
    json.dump(export_data, f, indent=2)

print(f"\n✅ Exported {len(export_data)} keys to redis_export.json")
```

---

## ✅ Restoration Script (from `redis_export.json`)

You can later run this to **restore to any Redis**:

```python
import redis
import json

destination_redis = redis.Redis(
    host='127.0.0.1',
    port=6379,
    decode_responses=True
)

with open("redis_export.json") as f:
    data = json.load(f)

for item in data:
    key = item["key"]
    key_type = item["type"]
    ttl = item["ttl"]
    value = item["value"]

    try:
        if key_type == "string":
            destination_redis.set(key, value)

        elif key_type == "hash":
            destination_redis.hset(key, mapping=value)

        elif key_type == "set":
            destination_redis.sadd(key, *value)

        elif key_type == "list":
            destination_redis.rpush(key, *value)

        elif key_type == "zset":
            destination_redis.zadd(key, dict(value))

        if ttl and ttl > 0:
            destination_redis.expire(key, ttl)

    except Exception as e:
        print(f"❌ Failed to restore key '{key}': {e}")

print("✅ Restore complete.")
```

---

## 🔐 Pro Tips

* Track keys by `type` always when doing migrations.
* Save `ttl` so time-sensitive keys behave correctly.
* If you want to include timestamps or namespaces, we can enhance the schema.

---

Let me know if you want:

* This to work with **multiple Redis clusters**
* Output in **CSV** or a different structure
* To export **only keys of a certain pattern or prefix**

Ready when you are.

---

# After 1st Error

## ✅ Step to Fix

Run this to install the Redis client library for Python:

```bash
pip3 install redis
```

> If `pip3` is not available, install it with:

```bash
sudo yum install -y python3-pip
```

Then try your script again:

```bash
python3 sync_redis_keys.py
```

---

### 🔎 Confirm it works:

After `pip3 install redis`, verify with:

```bash
python3 -c "import redis; print(redis.__version__)"
```

You should see something like:

```
4.6.0
```

---
    
</details>
