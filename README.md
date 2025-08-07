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

---
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

---

</details>

# Installing Redis and taking backup of the entire DB from Elasticache Redis to Redis Local Server

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
HOST = 'redtaxi-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com'
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
