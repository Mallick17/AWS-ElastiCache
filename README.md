# Redis Installation, Setup, Key Insert & Backup, and Key Sync (Local ↔ ElastiCache)

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


Great point — you **absolutely should track the key type** when exporting, so you can **restore each key correctly** without guessing.

Below are two options:

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
    host='redtaxi-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com',
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

    
</details>
