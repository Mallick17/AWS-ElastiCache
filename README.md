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

## Optional: Restore Backup

To restore the data later:

```php
$json = file_get_contents('/var/www/html/storage/app/cabs_8825_live_details.json');
$data = json_decode($json, true);
$elastiCache->setAll('cabs.8825.live_details', $data);
```

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

## Objective

We will:

1. **Delete** the key `cabs.8825.live_details` from Redis.
2. **Restore** it using a previously backed-up JSON file (`cabs_8825_live_details.json`).

---

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

## Step 2: Restore the Redis Key from JSON

Assuming you already have a JSON file named `cabs_8825_live_details.json`.

---

### 🔹 2.1 (Optional) If the file is on your local machine, copy it into the container:

```bash
docker cp cabs_8825_live_details.json 467a2ae897cf:/var/www/html/storage/app/cabs_8825_live_details.json
```

---

### 🔹 2.2 Enter Laravel Tinker again (if not already)

```bash
php artisan tinker
```

---

### 🔹 2.3 Read and decode the JSON

```php
$json = file_get_contents('/var/www/html/storage/app/cabs_8825_live_details.json');
$data = json_decode($json, true);
```

✅ `$data` now contains the associative array that was saved in the backup.

---

### 🔹 2.4 Set the Redis key back

```php
$elastiCache->setAll($key, $data);
```

✅ This restores the original Redis key and all its fields.

---

### 🔹 2.5 Confirm restoration

```php
$elastiCache->getAll($key);
```

You should now see the full restored structure like:

```php
=> [
     "city_id" => "1",
     "device_id" => "",
     "driver_id" => "23414",
     ...
   ]
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

## Best Practices

* Always **backup before deletion**.
* Validate data integrity after restoration.
* Use JSON format for portability and readability.

---


