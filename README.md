Here’s a clean and well-structured documentation of your Redis installation, data handling, and key synchronization process. The commands and flow have been preserved closely, while unnecessary noise (like typos, retries, redundant steps) has been removed for clarity.

---

# 📘 Redis Installation, Setup, Key Insert & Backup, and Key Sync (Local ↔ ElastiCache)

## 📦 1. Install Redis on Amazon Linux (with build from source)

```bash
# Install Redis via DNF (optional, if you just want the binary)
sudo dnf install redis -y
```

### 🔧 Alternatively, build Redis from source:

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

## 🚀 2. Start Redis Server Locally

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

## 📝 3. Create and View Redis Keys

```redis
127.0.0.1:6379> SET mykey "adith"
127.0.0.1:6379> GET mykey
"adith"

127.0.0.1:6379> SET mykey1 "abishek"
127.0.0.1:6379> SET mykey2 "abishek big boss"
127.0.0.1:6379> SET mykey3 "abi"
127.0.0.1:6379> SET mykey4 "abishek married boss"
127.0.0.1:6379> SET mykey5 "abishek married"
127.0.0.1:6379> SET mykey6 "santhosh"
127.0.0.1:6379> SET mykey7 "kutty boss santhosh"
127.0.0.1:6379> SET mykey8 "kutty bass clk"
127.0.0.1:6379> SET mykey9 "Mallick"
127.0.0.1:6379> SET mykey10 "No boss Mallick"

# Save snapshot to dump.rdb
127.0.0.1:6379> SAVE
```

---

## 💾 4. Backup dump.rdb File

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

## ✅ 5. Verify Key Persistence

```bash
cd /root/redis-stable/src/
./redis-cli

127.0.0.1:6379> KEYS *
127.0.0.1:6379> GET mykey
"abishek pre married boss"
```

---

## 🔁 6. Sync Keys from AWS ElastiCache to Local Redis

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

## 🧹 Optional Cleanup

```bash
# Remove the sync script if no longer needed
rm sync_redis_keys.py
```

---

## ✅ Summary

| Step | Description                                            |
| ---- | ------------------------------------------------------ |
| 1    | Installed Redis and/or compiled from source            |
| 2    | Started the Redis server locally                       |
| 3    | Inserted and saved keys using `redis-cli`              |
| 4    | Backed up and restored `dump.rdb`                      |
| 5    | Synced keys from AWS ElastiCache to local using Python |
| 6    | Verified keys were successfully copied                 |

---
