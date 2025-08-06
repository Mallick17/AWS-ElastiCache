# Using PHP ARTISAN TINKER
## Documentation: Backing up a Redis Key from Laravel Inside a Docker Container

### Objective

- To **export a specific Redis key** (`cabs.8825.live_details`) from a Laravel application running in a Docker container, save its contents as a `.json` file, and **copy that file to the local machine**.

---

## Why This Approach?

| Step                        | Why                                                                            |
| --------------------------- | ------------------------------------------------------------------------------ |
| Using `php artisan tinker`  | It provides an interactive shell to access Laravel classes and Redis bindings. |
| Using JSON format           | JSON is a universal format that's portable, readable, and easy to restore.     |
| Using `file_put_contents()` | This saves the data directly to the Laravel container's filesystem.            |
| Using `docker cp`           | This command copies the file from the container to your local system.          |

---

## Step-by-Step Guide

---

### **Step 1: Enter Laravel Tinker inside the Docker container**

Use Laravel’s interactive shell to interact with Redis.

```bash
php artisan tinker
```

---

### **Step 2: Load the Redis Wrapper Class**

```php
use App\Packages\Cache\ElastiCache;
```

> This loads your custom Redis wrapper class that simplifies Redis operations (like `getAll()`).

---

### **Step 3: Initialize the Redis connection**

```php
$elastiCache = new ElastiCache();
```

> Creates an instance that allows you to read/write to Redis via Laravel.

---

### **Step 4: Define the cab ID and Redis key**

```php
$cabId = 8825;
$key = 'cabs.' . $cabId . '.live_details';
```

> Constructs the exact Redis key you want to back up.

---

### **Step 5: Fetch the data from Redis**

```php
$data = $elastiCache->getAll($key);
```

> This retrieves the Redis hash (associative array) stored under the key.

---

### **Step 6: Convert data to JSON format**

```php
$json = json_encode($data, JSON_PRETTY_PRINT);
```

> Converts the PHP array into a human-readable `.json` format.

---

### **Step 7: Save the JSON file inside the container**

```php
file_put_contents('/var/www/html/storage/app/cabs_8825_live_details.json', $json);
```

> This saves the JSON string into a file in the Laravel `storage/app/` directory.

You’ll get an output like:

```php
=> 1111
```

> Which means `1111` bytes were written to the file.

---

### **Step 8: Exit tinker**

```php
q
```

> Or just use `exit`.

---

### **Step 9: Copy the file from the Docker container to your local system**

Assuming your container ID is `467a2ae897cf`:

```bash
docker cp 467a2ae897cf:/var/www/html/storage/app/cabs_8825_live_details.json .
```

> This copies the file to the current local directory (`.`).

---

## Final Output

You will now have a file on your local system named:

```
cabs_8825_live_details.json
```

Containing the backed-up Redis data in JSON format.

---

<details>
 <summary>Click to view Mallow Technology Method</summary>

<details>
 <summary>Error Faced and the difference in error</summary>


The error you're seeing:

```
PHP Error: Call to undefined method App\Packages\Cache\ElastiCache::keys()
```

means that the `ElastiCache` class does **not** have a `keys()` method defined. You're likely trying to use a method similar to the Redis `KEYS` command, but through your application's `ElastiCache` wrapper.

### What’s going wrong?

You're using:

```php
$keys = ElastiCache::keys('*');
```

But the method `keys()` doesn’t exist on your `ElastiCache` class.

---

### How to fix this?

#### ✅ Step 1: Check your ElastiCache wrapper

If `App\Packages\Cache\ElastiCache` is a wrapper around Redis, then you need to access the underlying Redis client (like Predis or PhpRedis) to call `keys()`.

Example (if it wraps Laravel’s cache or Redis facade):

```php
$keys = ElastiCache::getClient()->keys('*');
```

But this depends entirely on how `ElastiCache` is implemented.

---

#### ✅ Step 2: If you're using Laravel

If your `ElastiCache` is using Laravel’s Redis, you can directly use:

```php
$keys = Redis::keys('*');
```

Make sure you have `use Illuminate\Support\Facades\Redis;` at the top.

Or, if you're using a Redis connection instance:

```php
$keys = app('redis')->keys('*');
```

---

### ⚠️ Warning about `KEYS *`

Using `keys('*')` in production is dangerous for performance, especially on large datasets. Consider using `SCAN` for safer iteration:

```php
$cursor = null;
$keys = [];

do {
    [$cursor, $results] = Redis::scan($cursor, ['MATCH' => '*', 'COUNT' => 100]);
    $keys = array_merge($keys, $results);
} while ($cursor !== 0);
```

---

### ✅ Summary

* Your error means `keys()` is not defined in `ElastiCache`.
* Try accessing the Redis client via a method like `getClient()` or similar.
* Alternatively, use Laravel's `Redis::keys('*')`.
* Avoid using `KEYS` in production — prefer `SCAN`.
* `keys()` method (Redis command wrapper), or
* `getClient()` method (to expose the underlying Redis instance).

---

## ✅ What You Should Do Next

### 🔎 Step 1: Inspect the `ElastiCache` class

Open the file:

```bash
app/Packages/Cache/ElastiCache.php
```

And check:

* Does it internally use `Redis::`?
* Does it expose any raw access to the Redis client?
* Which public methods does it expose?

Look for anything like:

```php
use Illuminate\Support\Facades\Redis;

class ElastiCache {
    public function get($key) {
        return Redis::get($key);
    }

    public function hgetall($key) {
        return Redis::hgetall($key);
    }

    // Do you see something like this?
    public function keys($pattern) {
        return Redis::keys($pattern);
    }

    // or
    public function getClient() {
        return Redis::connection();
    }
}
```

---

## 🛠️ Option 1: Add a `keys()` method

If it's your custom class, add the following to `ElastiCache.php`:

```php
public function keys(string $pattern = '*') {
    return \Illuminate\Support\Facades\Redis::keys($pattern);
}
```

You can now do in `tinker`:

```php
use App\Packages\Cache\ElastiCache;
$elastiCache = new ElastiCache();
$keys = $elastiCache->keys('*');
```

---

## 🛠️ Option 2: Use Redis directly for keys

Since this is just for backup, **you can mix in** the Laravel Redis facade directly:

```php
use App\Packages\Cache\ElastiCache;
use Illuminate\Support\Facades\Redis;

$elastiCache = new ElastiCache();
$keys = Redis::keys('*');  // this works even if ElastiCache doesn’t expose keys()

$allData = [];

foreach ($keys as $key) {
    $type = Redis::type($key);
    if ($type === 'hash') {
        $allData[$key] = $elastiCache->hgetall($key); // still use wrapper
    } else {
        $allData[$key] = $elastiCache->get($key);
    }
}

file_put_contents(storage_path('app/redis_backup.json'), json_encode($allData, JSON_PRETTY_PRINT));
```

---

## ✅ Summary

| What you want to do       | Recommendation                                |
| ------------------------- | --------------------------------------------- |
| List all Redis keys       | Use `Redis::keys('*')` directly               |
| Use only ElastiCache      | Add a `keys()` method inside the class        |
| Wrap into a command later | Use this logic inside a `php artisan` command |


```bash
php artisan redis:backup
```

</details>

Yes — if you **cannot modify the `ElastiCache` class**, you can **access the underlying Redis connection directly using Laravel's Redis facade** and bypass the wrapper.

---

### ✅ Workaround Using Laravel's Redis Facade

Run this in `php artisan tinker` or your custom command:

```php
use Illuminate\Support\Facades\Redis;

$keys = Redis::connection('custom')->keys('*');
$allData = [];

foreach ($keys as $key) {
    $type = Redis::connection('custom')->type($key);
    $value = null;

    switch ($type) {
        case 'hash':
            $value = Redis::connection('custom')->hgetall($key);
            break;
        case 'string':
            $value = Redis::connection('custom')->get($key);
            break;
        case 'zset':
            // include scores too
            $zsetMembers = Redis::connection('custom')->zrange($key, 0, -1, ['withscores' => true]);
            $value = $zsetMembers;
            break;
        case 'list':
            $value = Redis::connection('custom')->lrange($key, 0, -1);
            break;
        case 'set':
            $value = Redis::connection('custom')->smembers($key);
            break;
        default:
            $value = "Unsupported type: $type";
    }

    $allData[$key] = [
        'type' => $type,
        'value' => $value,
    ];
}

file_put_contents(storage_path('app/redis_backup.json'), json_encode($allData, JSON_PRETTY_PRINT));

```

---

### 🧠 Notes

* This uses Laravel’s Redis Facade (`Illuminate\Support\Facades\Redis`) and the `'custom'` connection — adjust if needed.
* `keys('*')` works for smaller datasets. For production, prefer `scan()` to avoid blocking Redis.

---

### ✅ To Restore a Particular Key

Once you’ve verified the key is missing (you already checked earlier using `Redis::exists()`), restore like this:

```php
$backup = json_decode(file_get_contents(storage_path('app/redis_backup.json')), true);

$key = 'cabs.8825.live_details';

if (isset($backup[$key])) {
    Redis::connection('custom')->hmset($key, $backup[$key]);
}
```

</details>

---

## Redis Key Deletion via Laravel Tinker

We will:

1. **Delete** the key `cabs.8825.live_details` from Redis.

## Prerequisites

* Laravel is running inside a Docker container.
* Redis is connected and accessible via Laravel (e.g., using your `App\Packages\Cache\ElastiCache` class).
* You already have the backup file (JSON format), either:

  * Inside the container at: `/var/www/html/storage/app/cabs_8825_live_details.json`
  * Or on your local machine and copied into the container.

---

## Step 1: Delete the Redis Key

### 🔹 1.1 Enter Laravel Tinker

```bash
php artisan tinker
```

---

### 🔹 1.2 Use the Redis wrapper class

```php
use App\Packages\Cache\ElastiCache;
```

---

### 🔹 1.3 Initialize the ElastiCache class

```php
$elastiCache = new ElastiCache();
```

---

### 🔹 1.4 Define the key and delete it

```php
$cabId = 8825;
$key = 'cabs.' . $cabId . '.live_details';
$elastiCache->delete($key);
```

✅ This command removes the Redis key from the database.

---

### 🔹 1.5 Verify deletion (optional)

```php
$elastiCache->getAll($key);
```

You should get:

```php
=> []
```

or

```php
=> null
```

---


## Summary Table

| Action           | Command                              |
| ---------------- | ------------------------------------ |
| Delete key       | `$elastiCache->delete($key);`        |
| Backup file read | `file_get_contents('path_to_file')`  |
| JSON decode      | `json_decode($json, true)`           |
| Restore key      | `$elastiCache->setAll($key, $data);` |


---

# 📘 Redis Key Restoration via Laravel Tinker
## Use Case

Restore the Redis key:
**`cabs.8825.live_details`**
from the backup file:
**`/var/www/html/storage/app/cabs_8825_live_details.json`**

---

## Prerequisites

* Laravel application running inside the Docker container.
* Redis is configured and accessible via `Illuminate\Support\Facades\Redis`.
* JSON backup exists at the expected path.
* PHP and Laravel Artisan installed inside the container.

---

## Step 1: Check if the Redis Key Exists

### Command:

```bash
php artisan tinker
```

### In Tinker:

```php
use Illuminate\Support\Facades\Redis;

$key = 'cabs.8825.live_details';

// Check key existence
Redis::exists($key);       // Returns 0 if not exists

// Try fetching the data
Redis::hgetall($key);      // Returns empty array if not exists
Redis::get($key);          // Returns null if not exists
```

### Expected Output (if key doesn't exist):

```php
=> 0
=> []
=> null
```

---

## Step 2: Restore Key from JSON Backup

### Load the JSON and Decode:

```php
$json = file_get_contents('/var/www/html/storage/app/cabs_8825_live_details.json');
$data = json_decode($json, true);
```

### Set Redis Key Based on Data Type:

```php
if (is_array($data)) {
    Redis::hmset($key, $data);   // Store as a hash
} else {
    Redis::set($key, $data);     // Store as a string
}
```

---

## Step 3: Verify Restoration

### Run:

```php
Redis::exists($key);      // Should return 1
Redis::hgetall($key);     // Should return the restored data
```

### Example Output:

```php
=> 1

=> [
  "city_id" => "1",
  "driver_id" => "23414",
  "driver_app_version" => "3.6.83",
  "cab_id" => "8825",
  "status" => "On Local Trip Assigned",
  ...
]
```

---

## Notes

* `hmset()` is used for associative arrays (hashes).
* Always validate your JSON before decoding.
* If key already exists and needs to be overwritten, Redis will automatically overwrite it during `hmset()` or `set()`.

---

> This will push the backed-up data back to Redis under the same key.

---

## Summary

| Task                 | Tool                         |
| -------------------- | ---------------------------- |
| Redis interaction    | Laravel `php artisan tinker` |
| Serialization format | JSON                         |
| File creation        | `file_put_contents()`        |
| File transfer        | `docker cp`                  |

---

## Best Practices

* Always **backup before deletion**.
* Validate data integrity after restoration.
* Use JSON format for portability and readability.

---

# Taking entire Redis Elasticache Backup

## ✅ 1. View All Redis Keys

### 🔹 If using Laravel's Redis facade:

php
use Illuminate\Support\Facades\Redis;

Redis::keys('*');


> 📝 This returns all Redis keys matching * (i.e., all keys). Be cautious: In production Redis with many keys, this can be expensive.
The error you're seeing:

```
Predis\Response\ServerException with message 'WRONGTYPE Operation against a key holding the wrong kind of value'
```

means that you tried to perform a `GET` or `HGETALL` on a key that has a Redis **data type different** from what the code expects.

---

## ❌ Why It Happens

The code only checks:

* If the key is a `hash`, use `hgetall`
* Otherwise, assume it’s a `string` and use `get`

But Redis has **other data types**:

* `list`
* `set`
* `zset`
* `stream`

If your Redis contains any of these, calling `get()` or `hgetall()` on them will throw the error you're seeing.

---

## ✅ Solution: Add a Full Type Check Before Reading

Update your backup code to handle all key types **or skip unknown ones safely**.

---

### ✅ **Safe & Flexible Backup Script**

```php
use Illuminate\Support\Facades\Redis;

$keys = Redis::keys('*');
$allData = [];

foreach ($keys as $key) {
    $type = Redis::type($key);

    switch ($type) {
        case 'string':
            $allData[$key] = [
                'type' => 'string',
                'value' => Redis::get($key),
            ];
            break;

        case 'hash':
            $allData[$key] = [
                'type' => 'hash',
                'value' => Redis::hgetall($key),
            ];
            break;

        case 'list':
            $allData[$key] = [
                'type' => 'list',
                'value' => Redis::lrange($key, 0, -1),
            ];
            break;

        case 'set':
            $allData[$key] = [
                'type' => 'set',
                'value' => Redis::smembers($key),
            ];
            break;

        case 'zset':
            $allData[$key] = [
                'type' => 'zset',
                'value' => Redis::zrange($key, 0, -1, 'WITHSCORES'),
            ];
            break;

        default:
            // Skip unsupported or unknown types
            continue;
    }
}

file_put_contents('/var/www/html/storage/app/redis_backup.json', json_encode($allData, JSON_PRETTY_PRINT));
```

---

### ✅ Restore Script for Multiple Types

```php
use Illuminate\Support\Facades\Redis;

$json = file_get_contents('/var/www/html/storage/app/redis_backup.json');
$data = json_decode($json, true);

foreach ($data as $key => $entry) {
    $type = $entry['type'];
    $value = $entry['value'];

    switch ($type) {
        case 'string':
            Redis::set($key, $value);
            break;

        case 'hash':
            Redis::hmset($key, $value);
            break;

        case 'list':
            foreach ($value as $item) {
                Redis::rpush($key, $item);
            }
            break;

        case 'set':
            foreach ($value as $item) {
                Redis::sadd($key, $item);
            }
            break;

        case 'zset':
            foreach ($value as $member => $score) {
                Redis::zadd($key, $score, $member);
            }
            break;
    }
}
```

---

## ✅ Result

With this improved script:

* You **won’t get type errors**
* You **can back up and restore all key types**, not just strings and hashes

---


