# Installing Redis and taking backup of the DB from Elasticache Redis to Redis Local Server

---
### **🧰 Prerequisites**

Ensure the following are ready on your Linux system:

* `redis-cli` installed
* Python 3.7+ installed
* AWS ElastiCache Redis **endpoint** accessible from your instance (i.e., no VPC access restriction)
* Local Redis server installed (default: single node, 16 DBs)
  
### ✅ Option 1: Install Redis via Amazon Linux 2 Extra Repository

```bash
sudo amazon-linux-extras enable redis6
sudo yum clean metadata
sudo yum install -y redis
```

Then start and enable it:

```bash
sudo systemctl start redis
sudo systemctl enable redis
```

Verify it's running:

```bash
redis-cli ping
# Output: PONG
```

<details>
    <summary>Option 2: Use Redis via Docker (no system install required)</summary>

### ✅ Option 2: Use Redis via Docker (no system install required)

If Docker is available, you can spin up Redis in a container:

```bash
docker run --name redis-local -p 6379:6379 -d redis
```

Then test:

```bash
redis-cli ping
# PONG
```

And run your restore script — it will connect to `localhost:6379` by default.

</details>

---

# Installing Redis and taking backup of the entire 16 DB's from Elasticache Redis to Redis Local Server

<details>
    <summary>Click to view Step-by-Step Plan</summary>


## ✅ Step-by-Step Plan

---

### **🧰 Prerequisites**

Ensure the following are ready on your Linux system:

* `redis-cli` installed
* Python 3.7+ installed
* AWS ElastiCache Redis **endpoint** accessible from your instance (i.e., no VPC access restriction)
* Local Redis server installed (default: single node, 16 DBs)

---

### **📦 Step 1: Install Required Tools**

```bash
# Install redis-cli (if not installed)
sudo yum install redis -y

# Install required Python tools
python3 -m venv redisenv
source redisenv/bin/activate
pip install redis
```

---

### **📁 Step 2: Python Script to Dump All Redis Data by DB**

Save this as `dump_redis_elasticache.py`:

```python
import redis
import json
import base64

# CONFIG
HOST = '<your-end-point>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com'
PORT = 6379
DB_RANGE = range(0, 7)

def safe_decode(value):
    try:
        return value.decode('utf-8')
    except Exception:
        return base64.b64encode(value).decode('ascii')  # fallback to base64 string

def dump_db(db_index):
    r = redis.StrictRedis(host=HOST, port=PORT, db=db_index)  # no decode_responses!
    keys = r.keys('*')
    data = {}
    for key in keys:
        try:
            key_decoded = safe_decode(key)
            key_type = r.type(key)
            if key_type == b'string':
                val = r.get(key)
                data[key_decoded] = {'type': 'string', 'value': safe_decode(val)}
            elif key_type == b'hash':
                hash_data = r.hgetall(key)
                data[key_decoded] = {
                    'type': 'hash',
                    'value': {safe_decode(k): safe_decode(v) for k, v in hash_data.items()}
                }
            elif key_type == b'list':
                list_data = r.lrange(key, 0, -1)
                data[key_decoded] = {'type': 'list', 'value': [safe_decode(i) for i in list_data]}
            elif key_type == b'set':
                set_data = r.smembers(key)
                data[key_decoded] = {'type': 'set', 'value': [safe_decode(i) for i in set_data]}
            elif key_type == b'zset':
                zset_data = r.zrange(key, 0, -1, withscores=True)
                data[key_decoded] = {'type': 'zset', 'value': [[safe_decode(i[0]), i[1]] for i in zset_data]}
        except Exception as e:
            print(f"❌ Error dumping key {key}: {e}")
    with open(f'dump_db_{db_index}.json', 'w') as f:
        json.dump(data, f, indent=2)

if __name__ == "__main__":
    for db_index in DB_RANGE:
        print(f'Dumping DB {db_index}...')
        dump_db(db_index)
    print('✅ Done dumping all DBs.')

```

Run:

```bash
python dump_redis_elasticache.py
```

This will create 7 files: `dump_db_0.json` to `dump_db_6.json`.

---

### **🔁 Step 3: Load Dumped Data into Local Redis**

Save this as `restore_to_local_redis.py`:

```python
import redis
import json
import base64

DB_RANGE = range(0, 7)

def try_base64_decode(val):
    try:
        # Try decoding assuming it's base64
        return base64.b64decode(val.encode('ascii'))
    except Exception:
        # If decoding fails, assume it's plain text
        return val.encode('utf-8')

def restore_db(db_index):
    with open(f'dump_db_{db_index}.json') as f:
        data = json.load(f)
    r = redis.StrictRedis(host='localhost', port=6379, db=db_index)

    for key, item in data.items():
        t = item['type']
        v = item['value']
        key_b = try_base64_decode(key)

        if t == 'string':
            r.set(key_b, try_base64_decode(v))

        elif t == 'hash':
            decoded_hash = {try_base64_decode(k): try_base64_decode(val) for k, val in v.items()}
            r.hset(key_b, mapping=decoded_hash)

        elif t == 'list':
            r.rpush(key_b, *[try_base64_decode(i) for i in v])

        elif t == 'set':
            r.sadd(key_b, *[try_base64_decode(i) for i in v])

        elif t == 'zset':
            r.zadd(key_b, {try_base64_decode(k): s for k, s in v})

    print(f"✅ DB {db_index} restored.")

if __name__ == "__main__":
    for db_index in DB_RANGE:
        restore_db(db_index)
    print("🎉 All Redis DBs restored successfully.")

```

Run:

```bash
python restore_to_local_redis.py
```

---

### **🧪 Step 4: Verify Locally**

To check that DBs are correctly loaded:

```bash
redis-cli

127.0.0.1:6379> SELECT 0
127.0.0.1:6379[0]> DBSIZE

127.0.0.1:6379> SELECT 1
127.0.0.1:6379[1]> DBSIZE

...repeat for DB 2 to 6
```

You can also query specific keys:

```bash
127.0.0.1:6379[0]> KEYS *
127.0.0.1:6379[0]> HGETALL cabs.7204.live_details
```

---

## 📌 Notes

* ElastiCache does **not allow CONFIG GET**, `BGSAVE`, or `DUMP`, hence we use key-based extraction.
* This method works **without downtime** or special Redis permissions.

---

The error you're seeing means that **your script is trying to connect to a Redis server on `localhost:6379`**, but **no Redis server is currently running there**, so the connection is refused:

```
redis.exceptions.ConnectionError: Error 111 connecting to localhost:6379. Connection refused.
```

---

### ✅ Step-by-step fix:

#### **1. Start a local Redis server**

If Redis is not running, start it:

```bash
redis-server --daemonize yes
```

> This will start Redis in the background on port 6379.

#### **2. Check if Redis is now running**

Use:

```bash
redis-cli ping
```

Expected output:

```bash
PONG
```

If not, check logs:

```bash
cat /var/log/redis/redis.log
```

Or if Redis was installed manually:

```bash
cat /tmp/redis.log
```

---

### ✅ Optional: Bind Redis to all interfaces (for remote access, not recommended on production)

If needed, edit your config (usually `/etc/redis/redis.conf` or `/etc/redis.conf`) and ensure:

```ini
bind 127.0.0.1
```

Or allow external access (risky unless firewalled):

```ini
bind 0.0.0.0
```

And:

```ini
protected-mode no
```

Then restart:

```bash
redis-server /etc/redis.conf
```

---

### ✅ Once Redis is running

You can rerun your script:

```bash
python restore_to_local_redis.py
```

---

### 🧪 Bonus: Check if local Redis has correct DBs restored

Run:

```bash
for db in {0..6}; do
    echo -n "DB $db: ";
    redis-cli -n $db dbsize;
done
```

You should see non-zero values confirming restore.

</details>

---

# Installing Redis and taking backup of the single-single DB from Elasticache Redis to Redis Local Server

<details>
  <summary>Click to view Step-by-Step Plan</summary>

Got it — since you're losing some keys when exporting entire DBs in bulk, the best approach is to **isolate and export/restore one Redis DB at a time**, with full type safety and encoding support (e.g., base64 when needed).

---

### ✅ Step-by-Step Plan (Single DB at a Time)

---

#### **1. Export a Single Redis DB Safely (ElastiCache → JSON)**

Here’s a script that dumps a single DB with proper key type handling and base64 fallback for binary/unicode-safe export:

```python
# dump_db_safe.py
import redis
import json
import base64

HOST = 'your-elasticache-endpoint'
PORT = 6379
DB_INDEX = 0  # change this to target a single DB

def safe_encode(value):
    try:
        json.dumps(value)
        return {'encoding': 'plain', 'data': value}
    except (TypeError, UnicodeDecodeError):
        return {'encoding': 'base64', 'data': base64.b64encode(value.encode() if isinstance(value, str) else value).decode()}

def dump_db(db_index):
    r = redis.StrictRedis(host=HOST, port=PORT, db=db_index, decode_responses=False)
    keys = r.keys('*')
    data = {}
    for key in keys:
        key_str = key.decode('utf-8', errors='replace')
        key_type = r.type(key).decode()
        if key_type == 'string':
            val = r.get(key)
            data[key_str] = {'type': 'string', 'value': safe_encode(val)}
        elif key_type == 'hash':
            val = {k.decode(): v.decode(errors='replace') for k, v in r.hgetall(key).items()}
            data[key_str] = {'type': 'hash', 'value': val}
        elif key_type == 'list':
            val = [v.decode(errors='replace') for v in r.lrange(key, 0, -1)]
            data[key_str] = {'type': 'list', 'value': val}
        elif key_type == 'set':
            val = [v.decode(errors='replace') for v in r.smembers(key)]
            data[key_str] = {'type': 'set', 'value': val}
        elif key_type == 'zset':
            val = [(v.decode(errors='replace'), s) for v, s in r.zrange(key, 0, -1, withscores=True)]
            data[key_str] = {'type': 'zset', 'value': val}
    with open(f'dump_db_{db_index}.json', 'w') as f:
        json.dump(data, f, indent=2)

if __name__ == "__main__":
    print(f'Dumping DB {DB_INDEX}...')
    dump_db(DB_INDEX)
    print('✅ Dumped dump_db_{DB_INDEX}.json')
```

---

#### **2. Restore to Local Redis (One DB at a Time)**

This matches the dump format and decodes base64 if needed:

```python
# restore_db_safe.py
import redis
import json
import base64

DB_INDEX = 0  # same as exported DB

def decode_value(val_obj):
    if isinstance(val_obj, dict) and val_obj.get('encoding') == 'base64':
        return base64.b64decode(val_obj['data']).decode(errors='replace')
    return val_obj['data']

def restore_db(db_index):
    with open(f'dump_db_{db_index}.json') as f:
        data = json.load(f)
    r = redis.StrictRedis(host='localhost', port=6379, db=db_index, decode_responses=True)
    for key, item in data.items():
        t = item['type']
        v = item['value']
        if t == 'string':
            r.set(key, decode_value(v))
        elif t == 'hash':
            r.hset(key, mapping=v)
        elif t == 'list':
            r.rpush(key, *v)
        elif t == 'set':
            r.sadd(key, *v)
        elif t == 'zset':
            r.zadd(key, {k: s for k, s in v})
    print(f'✅ Restored DB {db_index}')

if __name__ == "__main__":
    restore_db(DB_INDEX)
```

---

### ✅ How to Run for Specific DB

1. To export DB 3, update:

```python
DB_INDEX = 3
```

Run:

```bash
python dump_db_safe.py
```

2. To restore to local:

```bash
python restore_db_safe.py
```

---

Let me know if you want to:

* Export/restore only specific **keys** inside the DB (rather than full DB),
* Do a **batch script** for multiple DBs one by one, or
* Have a `.sh` wrapper to automate this process.

  
</details>


---

# To **clean (flush) the entire Redis database** on your local machine, you have two main options depending on whether you want to:


### 🧹 1. **Flush a Single Database (e.g. DB 0)**

```bash
redis-cli FLUSHDB
```

This clears **only the currently selected database** (default is DB 0).

To flush a specific DB, like DB 5:

```bash
redis-cli -n 5 FLUSHDB
```

---

### 💣 2. **Flush All Databases (DB 0 to DB 15)**

```bash
redis-cli FLUSHALL
```

This **removes all keys from all databases**. Be very careful with this one.

---

### ✅ Optional: Confirm Redis is Clean

After flushing, you can verify:

```bash
for i in {0..15}; do echo "DB $i:"; redis-cli -n $i dbsize; done
```

This will show `DB x: 0` for each DB if the flush was successful.

---

# How to compare and sync all the dbs from elasticache to local redis

<details>
  <summary>Click to view step by step guide</summary>

To **find which Redis keys are missing** during migration from ElastiCache to local Redis and ensure **no data loss**, you need to compare the keys in each DB *before and after* transfer and sync them.

```python
import redis

# --- Connect to ElastiCache Redis (source) ---
source_redis = redis.Redis(
    host='<your-end-point>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com',
    port=6379,
    decode_responses=True
)

# --- Connect to Local Redis (destination) ---
destination_redis = redis.Redis(
    host='127.0.0.1',
    port=6379,
    decode_responses=True
)

total_missing = 0
total_copied = 0

for db in range(16):
    print(f"\n📂 Processing DB {db}...")

    source_redis.select(db)
    destination_redis.select(db)

    source_keys = set(source_redis.scan_iter('*'))
    destination_keys = set(destination_redis.scan_iter('*'))

    missing_keys = sorted(source_keys - destination_keys)

    print(f"   🔍 Source keys     : {len(source_keys)}")
    print(f"   💾 Local keys      : {len(destination_keys)}")
    print(f"   ❌ Missing to copy : {len(missing_keys)}")

    total_missing += len(missing_keys)
    copied = 0

    for key in missing_keys:
        try:
            key_type = source_redis.type(key)
            ttl = source_redis.ttl(key)

            if key_type == 'string':
                value = source_redis.get(key)
                destination_redis.set(key, value)

            elif key_type == 'hash':
                value = source_redis.hgetall(key)
                if value:
                    destination_redis.hset(key, mapping=value)

            elif key_type == 'set':
                members = source_redis.smembers(key)
                if members:
                    destination_redis.sadd(key, *members)

            elif key_type == 'zset':
                zitems = source_redis.zrange(key, 0, -1, withscores=True)
                if zitems:
                    destination_redis.zadd(key, dict(zitems))

            elif key_type == 'list':
                items = source_redis.lrange(key, 0, -1)
                if items:
                    destination_redis.rpush(key, *items)

            else:
                print(f"   ⚠️ Skipping unsupported type '{key_type}' for key: {key}")
                continue

            if ttl and ttl > 0:
                destination_redis.expire(key, ttl)

            copied += 1

        except Exception as e:
            print(f"   ❌ Error copying '{key}': {e}")

    total_copied += copied
    print(f"   ✅ Copied {copied} keys from DB {db}.")

print(f"\n🎯 Done. Total missing keys: {total_missing}, total copied: {total_copied}")
```
### Another kind of script to compare the single db's

```python
import redis

# --- Connect to ElastiCache Redis (source) ---
source_redis = redis.Redis(
    host='<your-end-point>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com',
    port=6379,
    decode_responses=True
)

# --- Connect to Local Redis (destination) ---
destination_redis = redis.Redis(
    host='127.0.0.1',
    port=6379,
    decode_responses=True
)

# --- Fetch all keys from both Redis ---
print("🔄 Fetching keys from both Redis instances...")
source_keys = set(source_redis.scan_iter('*'))
destination_keys = set(destination_redis.scan_iter('*'))

# --- Calculate missing keys ---
missing_keys = sorted(source_keys - destination_keys)

print(f"\n📦 Total source keys      : {len(source_keys)}")
print(f"💾 Total local keys       : {len(destination_keys)}")
print(f"❌ Missing keys to sync   : {len(missing_keys)}")

# --- Copy missing keys ---
copied = 0
for key in missing_keys:
    try:
        key_type = source_redis.type(key)
        ttl = source_redis.ttl(key)

        if key_type == 'string':
            value = source_redis.get(key)
            destination_redis.set(key, value)

        elif key_type == 'hash':
            value = source_redis.hgetall(key)
            if value:
                destination_redis.hset(key, mapping=value)

        elif key_type == 'set':
            members = source_redis.smembers(key)
            if members:
                destination_redis.sadd(key, *members)

        elif key_type == 'zset':
            zitems = source_redis.zrange(key, 0, -1, withscores=True)
            if zitems:
                destination_redis.zadd(key, dict(zitems))

        elif key_type == 'list':
            items = source_redis.lrange(key, 0, -1)
            if items:
                destination_redis.rpush(key, *items)

        else:
            print(f"⚠️ Skipping unsupported type '{key_type}' for key: {key}")
            continue

        # Set TTL if exists
        if ttl and ttl > 0:
            destination_redis.expire(key, ttl)

        copied += 1

    except Exception as e:
        print(f"❌ Error copying '{key}': {e}")

print(f"\n✅ Copied {copied} missing keys.")

```
  
</details>
