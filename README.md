# ✅ **Accessing and Querying the Restored Data**

Once your Redis instance (ElastiCache or other) has successfully loaded the `.rdb` file, the keys and values from that snapshot will be available in memory.

You can now **connect** to the Redis cache and use standard **Redis CLI commands** to query or explore your data.

For example, assuming the backup contained a key called `user:1001:name` with value `"Gyan"`:

---

### 🔍 Querying Existing Keys (Step-by-step):

1. **List keys** (if you don’t know the key names):

   ```bash
   KEYS *
   ```

   > ⚠️ Not recommended in production with large datasets. Use patterns like:

   ```bash
   KEYS user:*
   ```

2. **Check if a specific key exists**:

   ```bash
   EXISTS mykey
   ```

3. **Get the value of a known key**:

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

4. **Get all fields from a hash** (if data was stored in hash format):

   ```bash
   HGETALL user:1001
   ```

---

### 🧪 Sample: Use the Provided Example in Your Redis

From the AWS guide:

```bash
SET mykey "Hello, ElastiCache!"
GET mykey
```

This means:

* Key: `mykey`
* Value: `"Hello, ElastiCache!"`

---

### 🛠️ Example: Working with Restored Data

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

---

### 🧠 Tip: Verify RDB Load

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
