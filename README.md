# Back Up and Restore via Console

<details>
  <summary>View Back Up and Restore via Console</summary>

## Scheduling automatic backups

ElastiCache supports **automatic daily backups** for the following engines:

* **Redis OSS and Valkey (serverless or cluster mode)**
* **Memcached (serverless)**

### Key Benefits

* **Data Protection:** Helps prevent data loss by keeping daily snapshots.
* **Quick Recovery:** In case of failure, you can restore the cache from the most recent backup.
* **Warm Start:** The restored cache is preloaded with your data and ready for use.

<details>
  <summary>Click to view the steps</summary>


### How It Works

* Backups are created **daily** with **no performance impact** on the live cache.
* **Backup Window:** You can define a preferred time for backups to start. If not set, AWS assigns one.
* **Retention Period:** Determines how many days backups are kept in Amazon S3 (up to 35 days). Setting it to `0` disables automatic backups.

### Configuration Options

You can enable or disable automatic backups during:

* Cache creation
* Cache modification

Configuration tools include:

* AWS Management Console
* AWS CLI
* ElastiCache API

### UI Location:

* **Redis/Valkey:** Under *Advanced Redis OSS Settings* or *Advanced Valkey Settings*
* **Memcached:** Under *Advanced Memcached Settings*

**Example Use Case:**
If your app is running in production, enable automatic backups with a 7-day retention period and schedule the backup window during low-traffic hours (e.g., `23:30–00:30 UTC`) to ensure minimal operational interference.

</details>

---

## Taking manual backups
### Key Features

* **Persistent:** Manual backups are **retained indefinitely** until you choose to delete them.
* **Independent:** Even if the cache is deleted, manual backups remain available.
* **User-Controlled:** Manual backups must be deleted manually—no automatic expiration.

<details>
  <summary>Click to view the steps</summary>


### How to Create Manual Backups

You can create manual backups via:

* **ElastiCache Console**
* **AWS CLI**
* **ElastiCache API**

You can also generate a manual backup by:

* **Copying an existing backup** (manual or automatic)
* **Creating a final backup** before deleting a cache

### Creating Manual Backup via Console

1. Sign in to the [ElastiCache Console](https://console.aws.amazon.com/elasticache/).
2. In the navigation pane, select:

   * *Valkey caches*, *Redis OSS caches*, or *Memcached caches*
3. Select the checkbox next to the cache you want to back up.
4. Click **Backup**.
5. Enter a **Backup Name** in the dialog.
   *Naming rules:*

   * 1–40 characters (letters, numbers, or hyphens)
   * Must start with a letter
   * No two consecutive hyphens
   * Must not end with a hyphen
6. Click **Create Backup**.

> 📌 The cache’s status will temporarily change to **snapshotting** during the backup process.

### Supported Configurations

Manual backups are supported for:

* **Cluster mode enabled** and **disabled** Redis/Valkey
* **All cache types** (Valkey, Redis OSS, Memcached)

</details>

---

## Creating a Final Backup
A **final backup** allows you to preserve your cache data right before deleting a cache or cluster. This ensures you have a snapshot you can restore from later, even after deletion.

<details>
  <summary>Click to view the steps</summary>


### Key Points

* Available for:

  * **Valkey, Redis OSS**, and **Memcached** *serverless caches*
  * **Valkey** and **Redis OSS** *self-designed clusters*
* Can be created using:

  * **ElastiCache Console**
  * **AWS CLI**
  * **ElastiCache API**

### Creating a Final Backup via Console

1. Navigate to the **ElastiCache console**.
2. Select the cache or cluster you wish to delete.
3. Choose **Delete**.
4. In the **Delete dialog box**, under **Create backup**, select **Yes**.
5. Enter a name for the backup.

   * This name should follow standard naming rules (start with a letter, up to 40 characters, etc.).
6. Proceed with the deletion.

> ✅ The final backup will be saved before the cache or cluster is permanently deleted.

</details>

---

## Describing Backups

You can list and inspect your ElastiCache backups using the AWS Management Console.

<details>
  <summary>Click to view the steps</summary>


#### **Steps (Console):**

1. Sign in to the [ElastiCache Console](https://console.aws.amazon.com/elasticache/).
2. From the **navigation pane**, choose **Backups**.
3. To view details of a specific backup:

   * Select the checkbox beside the backup name.
   * The backup details will be displayed.

</details>

---

## Copying Backups

ElastiCache allows you to copy **any backup** — automatic or manual. This is useful for:

* Creating duplicates for testing or region-specific clusters
* Preserving snapshots under different names
* Preparing for export

<details>
  <summary>Click to view the steps</summary>


#### **Steps to Copy a Backup (Console):**

1. Sign in to the [ElastiCache Console](https://console.aws.amazon.com/elasticache/).
2. From the **navigation pane**, choose **Backups**.
3. Select the checkbox beside the backup you want to copy.
4. Choose **Actions** → **Copy**.
5. In the **New backup name** box, enter a name for the new backup.
6. Click **Copy**.

> ✅ The backup copy will appear in the list of backups and can be used like any other snapshot.

</details>

---

## **Exporting ElastiCache Backup to S3** (Console)

### **Pre-requirements**

1. **Backup** exists in your ElastiCache dashboard (manual or automatic).
2. **S3 Bucket** is created in the **same AWS Region** as the backup.
3. **Permissions** are configured to allow ElastiCache to write to the S3 bucket.

<details>
  <summary>Click to view the steps</summary>


### **Step 1: Create an S3 Bucket**

* Go to [Amazon S3 Console](https://console.aws.amazon.com/s3/)
* Click **“Create bucket”**
* Choose:

  * **Bucket name** (DNS-compliant, e.g. `elasticache-backups-myapp`)
  * **Region** same as your Redis backup (e.g. `ap-south-1`)
* Click **“Create”**

### **Step 2: Grant ElastiCache Access to the S3 Bucket**

* In S3 Console, select your bucket → **Permissions tab**
* Under **Access Control List (ACL)**:

  * Click **Edit**
  * Add **grantee** with this Canonical ID:

    ```
    540804c33a284a299d2547575ce1010f2312ef3da9b3a053c8bc45bf233e4353
    ```
  * Allow:

    * **Objects**: List, Write
    * **Bucket ACL**: Read, Write
* Save changes

### OR

Use this **bucket policy** (adjust region/bucket name):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowElastiCacheExport",
      "Effect": "Allow",
      "Principal": {
        "Service": "ap-south-1.elasticache-snapshot.amazonaws.com"
      },
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::elasticache-backups-myapp",
        "arn:aws:s3:::elasticache-backups-myapp/*"
      ]
    }
  ]
}
```

### **Step 3: Export the Backup**

* Go to [ElastiCache Console](https://console.aws.amazon.com/elasticache/)
* In the left menu → choose **Backups**
* Select the backup you want to export
* Click **Actions → Copy**
* In the dialog:

  * Enter a **New backup name**, e.g. `my-exported-backup`
  * Select your **Target S3 Location** from the dropdown
* Click **Copy**

---

### **Notes**

* ElastiCache will append `-0001.rdb` to the filename.
* If export fails, check these permissions are **enabled**:

  * Object **Read/Write**
  * Bucket permissions **Read**
* Export only works for:

  * Redis OSS or Valkey
  * Not supported for **data tiering** or **self-designed Memcached clusters**

</details>

---

## Restore ElastiCache Backup into a New Cache
You can restore:

* A **Redis OSS backup** → into a new **Redis OSS** cache
* A **Valkey backup** → into a new **Valkey** cache
* A **Memcached backup** → into a new **Memcached** serverless cache

<details>
  <summary>Click to view the steps</summary>


### **Option 1: Restore to Serverless Cache (Console)**

> Supports **Valkey 7.2+** and **Redis OSS 5.0+** backup `.rdb` files

#### Steps:

1. Go to [ElastiCache Console](https://console.aws.amazon.com/elasticache/)
2. From the left nav, select **Backups**
3. Select the checkbox next to the backup you want to restore
4. Click **Actions → Restore**
5. In the **Restore dialog**:

   * Enter a name for your new serverless cache
   * Add a description (optional)
6. Click **Create**

> 🔄 AWS will provision a new serverless cache and **import the `.rdb` snapshot** into it.

### **Option 2: Restore to Self-Designed Cluster (Console)**

> Lets you configure everything — instance size, replicas, shards, networking, etc.

#### Steps:

1. Go to [ElastiCache Console](https://console.aws.amazon.com/elasticache/)
2. Navigate to **Backups**
3. Select the backup you'd like to restore
4. Click **Actions → Restore**
5. Choose **Design your own cache**
6. Fill out configuration options:

   * **Cluster Name**
   * **Node type** (e.g. `cache.t4g.medium`)
   * **Number of shards**, **replicas**
   * **Subnet group**, **VPC**, **Security Groups**
7. Click **Create**

> AWS will launch a new Redis/Valkey cluster and restore the snapshot during cluster creation.

</details>

> **_Notes_**:
> Restores **do not** overwrite existing clusters; always create a **new** one.
> After restore, your data is available instantly once the new cache is **available**.
> This works for both **automatic** and **manual** backups.

---

## Deleting an ElastiCache Backup

### Key Concepts:

* **Automatic backups** are deleted **automatically** when:

  * Retention period expires
  * The cache or replication group is deleted
* **Manual backups** are **not** deleted unless you explicitly delete them

  * They persist **even after** the associated cluster is removed

<details>
  <summary>Click to view the steps</summary>


### Deleting a Backup via Console

#### Steps:

1. Go to the [ElastiCache Console](https://console.aws.amazon.com/elasticache/)
2. In the **navigation pane**, click **Backups**
3. From the backup list, check the box next to the backup you want to delete
4. Click **Delete**
5. On the confirmation dialog, click **Delete** again

> 🔁 The status will change to **"deleting"**, and the backup will be removed shortly.

---

### Other Methods:

* **AWS CLI:**

  ```bash
  aws elasticache delete-snapshot --snapshot-name my-backup-name
  ```

* **ElastiCache API:**
  Use the [`DeleteSnapshot`](https://docs.aws.amazon.com/AmazonElastiCache/latest/APIReference/API_DeleteSnapshot.html) API operation

</details>

### Caution:

* Deleted backups **cannot** be recovered.
* Make sure the backup isn't required for future restores before deleting.

---

## Tagging Backups in ElastiCache

### What Are Tags?

Tags are **key-value pairs** that let you add **custom metadata** to ElastiCache backups.

| **Example Tags**            |
| --------------------------- |
| `Key: environment` → `prod` |
| `Key: owner` → `team-x`     |
| `Key: purpose` → `billing`  |

### Why Tag Backups?

* **Organize** backups by purpose, owner, or environment
* **Filter/search** backups more easily
* **Enable cost tracking** with **Cost Allocation Tags**

> Example: You can track all `production` related backups in your AWS bill if you tag them with `environment=prod`.

### How to Manage Tags (Console)

<details>
  <summary>Click to view the steps</summary>
  
#### Add/Modify Tags:

1. Open the [ElastiCache Console](https://console.aws.amazon.com/elasticache/)
2. Go to **Backups** from the left navigation pane
3. Select the desired backup
4. Choose **Manage Tags**
5. Add or update tags as `Key = Value`
6. Save your changes

#### Remove Tags:

* In the **Manage Tags** screen, choose the ❌ next to the tag
* Save the changes

### Cost Allocation Tags

* Special tags used for **billing and usage tracking**
* Must be **activated** in the AWS Billing Console under **Cost Allocation Tags**
* Once activated, they appear in **Cost Explorer**, allowing you to break down expenses by tag

---

### Also Possible Using:

* **AWS CLI**

  ```bash
  aws elasticache add-tags-to-resource \
    --resource-name arn:aws:elasticache:region:account-id:snapshot:snapshot-name \
    --tags Key=environment,Value=dev
  ```

* **ElastiCache API**

  * Use actions like `AddTagsToResource`, `ListTagsForResource`, etc.

</details>

---

## **Seeding a New ElastiCache for Redis OSS Cluster from an External Backup (.rdb)**

### Use Case

Migrate data from an **externally managed Valkey or Redis OSS** instance to a new **ElastiCache for Redis OSS self-designed cluster** using a `.rdb` backup file.

## **Migration Steps**

<details>
  <summary>Click to view the steps</summary>
  
### **Step 1: Create a Valkey or Redis OSS Backup**

* Connect to your Redis OSS or Valkey instance.
* Create a backup using:

  * `BGSAVE` (asynchronous, preferred)
  * `SAVE` (synchronous)
* Locate the resulting `.rdb` file (usually in the data directory).

---

### **Step 2: Create an S3 Bucket & Folder**

* Go to [Amazon S3 Console](https://console.aws.amazon.com/s3/)
* Create a **bucket**:

  * Must be **DNS-compliant**
  * Must be in the **same AWS Region** as your target ElastiCache cluster
* Create a **folder** inside the bucket
* Example bucket/folder path:
  `myBucket/myFolder`

---

### **Step 3: Upload the .rdb File to S3**

* Upload the `.rdb` file into the folder
* Example file path:
  `myBucket/myFolder/redis-backup.rdb`

---

### **Step 4: Grant ElastiCache Access to the File**

Depending on the AWS Region type:

#### 🔸 **For Default AWS Regions**

Use **Canonical ID**:
`540804c33a284a299d2547575ce1010f2312ef3da9b3a053c8bc45bf233e4353`

Via **S3 console**:

* Open `.rdb` file
* Go to **Permissions > Access for other AWS accounts**
* Add grantee with required permissions:

  * List object
  * Read object
  * Read ACL

#### 🔸 **For Opt-in AWS Regions**

Update the **S3 Bucket Policy** with:

```json
{
  "Effect": "Allow",
  "Principal": {
    "Service": "ap-east-1.elasticache-snapshot.amazonaws.com"
  },
  "Action": [
    "s3:GetObject",
    "s3:ListBucket",
    "s3:GetBucketAcl"
  ],
  "Resource": [
    "arn:aws:s3:::myBucket",
    "arn:aws:s3:::myBucket/myFolder/redis-backup.rdb"
  ]
}
```

> Replace `ap-east-1` with your actual AWS Region

---

### **Step 5: Create the ElastiCache Cluster (Seed with Backup)**

#### 🔹 Using the **Console**

* Choose **Restore from backups**
* Under **Backup Source**, select **Other backups**
* Provide S3 path:
  `myBucket/myFolder/redis-backup.rdb`

#### 🔹 Using **AWS CLI**

```bash
aws elasticache create-cache-cluster \
  --cache-cluster-id my-new-cluster \
  --engine redis \
  --snapshot-arns arn:aws:s3:::myBucket/myFolder/redis-backup.rdb \
  --cache-node-type cache.r6g.large \
  --num-cache-nodes 1
```

#### 🔹 Using **ElastiCache API**

Use the `SnapshotArns` parameter in `CreateCacheCluster` or `CreateReplicationGroup` with the `.rdb` file ARN.

</details>

## ⚠**Important Notes**

* You **cannot** seed a **cluster-mode disabled** Redis cluster from a backup created on a **cluster-mode enabled** one.
* Make sure the node type has enough memory for the data in your `.rdb` file.
* If the backup is too large, the cluster will show `restore-failed` status.
* You **must delete and recreate** the cluster in case of restore failure.

---

## Monitoring

* Check restore progress in **ElastiCache Console > Events**
* Or use CLI/API to retrieve status messages

---

## **Where You Can Restore a Redis/Valkey Backup in ElastiCache**

<details>
  <summary>Click to view the Answers</summary>
  
### 🔹 **1. Into a New Cluster**

- **Yes, supported**

* You **can restore a backup into a new Redis OSS or Valkey cluster** (either serverless or self-designed).
* This is the **recommended approach** for safety, testing, and migrations.

**Use cases:**

* Migration to a new node type or AZ.
* Blue/Green deployment.
* Data recovery.

---

### 🔹 **2. Into an Existing Running Cluster**

- **No, not supported directly**

* **You cannot restore a backup into a currently running Redis OSS or Valkey cluster** in-place.
* ElastiCache does **not allow in-place restoration** on an active cluster.
* Restoring wipes existing data, so AWS enforces that this must happen by **creating a new cluster** from the backup.

**Alternatives:**

* Create a new cluster using the backup.
* Redirect traffic to the new cluster after validation.
* Use Route 53, environment variables, or load balancers to update the endpoint.

---

### 🔹 **3. Into a Self-Managed Redis Instance (non-AWS)?**

- **Yes, technically possible (manually)**

* If you **download the `.rdb` file from an ElastiCache backup**, you can:

  * Spin up a local or EC2-hosted Redis.
  * Drop the `.rdb` into the data directory (`/var/lib/redis`), and
  * Restart the Redis server to load it.

⚠️ **Caveat:** Make sure the Redis version matches or is compatible with the RDB file version.

</details>

---

</details>

---

# Back Up and Restore via AWS-CLI and REDIS-CLI

<details>
  <summary>View Back Up and Restore via AWS-CLI and REDIS-CLI</summary>

## ElastiCache Redis OSS (Cluster Mode Disabled) – Backup & Query Guide

## Prerequisites
Ensure you have:

* **ElastiCache Redis** with **Cluster Mode Disabled**
* **EC2 instance** in the **same VPC + subnet**
* EC2 security group allows **port 6379**
* Redis CLI installed on EC2 (`redis-cli` or `redis6-cli`)
* AWS CLI configured with proper permissions


## 📌 PART 1: Connect to ElastiCache and Query Data

<details>
  <summary>Click to view the steps</summary>

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

</details>

---

## 💾 PART 2: Take Backup (Snapshot) using AWS CLI

ElastiCache supports **snapshots** even when **Cluster Mode is Disabled**, using the **cache cluster ID** instead of replication group ID.

<details>
  <summary>Click to view the steps</summary>

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

</details>

---

## 🧪 PART 3: Simulate `.rdb` Backup on EC2 Redis
Since ElastiCache doesn't allow direct `.rdb` downloads, you can simulate a Redis backup via EC2.

<details>
  <summary>Click to view the steps</summary>

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

</details>

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

</details>

---

# Backup and Restore from AWS Elasticache to Redis & Local and Vice Versa

<details>
  <summary>View Backup and Restore from AWS Elasticache to Redis & Local and Vice Versa</summary>

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

### **📁 Step 2: Python Script to Dump All Redis Data From Elasticache**

Save this as `dump_redis_elasticache.py`:

```python
#!/usr/bin/env python3
"""
robust_chunk_redis_backup_all_keys.py

Backs up ALL keys from ALL Redis DBs in an ElastiCache endpoint.

Features:
- Binary-safe: distinguishes UTF-8 strings vs raw bytes (base64)
- Memory-safe: SCAN/HSCAN/SSCAN/ZSCAN/XRANGE pagination
- Strict paginated XRANGE for streams (no duplicates)
- Retries on transient Redis errors
- Atomic chunk writes with metadata
- TTL preserved
"""

import redis
import json
import base64
import os
import time
from functools import wraps
from redis.exceptions import ConnectionError, TimeoutError

# ===== Config =====
REDIS_HOST = "your-elasticache-endpoint.amazonaws.com"
REDIS_PORT = 6379
OUTPUT_DIR = "backup_output"
CHUNK_SIZE = 2500              # keys per JSON chunk file
RETRY_LIMIT = 5
RETRY_DELAY = 1.0              # seconds
XRANGE_PAGE = 1000
# ==================

os.makedirs(OUTPUT_DIR, exist_ok=True)

# --- Retry decorator ---
def retry(attempts=RETRY_LIMIT, delay=RETRY_DELAY, exceptions=(ConnectionError, TimeoutError)):
    def deco(f):
        @wraps(f)
        def wrapped(*a, **kw):
            last_exc = None
            for i in range(attempts):
                try:
                    return f(*a, **kw)
                except exceptions as e:
                    last_exc = e
                    print(f"⚠️ Redis transient error: {e} — retry {i+1}/{attempts} in {delay}s")
                    time.sleep(delay)
            raise last_exc
        return wrapped
    return deco

# --- Serialization ---
def safe_serialize(b):
    """Return wrapper: {"_t":"str","v":...} or {"_t":"b64","v":...}"""
    if b is None:
        return {"_t": "str", "v": ""}
    if isinstance(b, str):
        return {"_t": "str", "v": b}
    try:
        return {"_t": "str", "v": b.decode("utf-8")}
    except Exception:
        return {"_t": "b64", "v": base64.b64encode(b).decode("ascii")}

# --- Atomic JSON write ---
def atomic_write_json(obj, final_path):
    tmp = final_path + ".tmp"
    with open(tmp, "w", encoding="utf-8") as fh:
        json.dump(obj, fh, indent=2, ensure_ascii=False)
        fh.flush()
        os.fsync(fh.fileno())
    os.replace(tmp, final_path)

# --- Redis safe wrappers ---
@retry()
def safe_scan(r, cursor, match=None, count=1000):
    return r.scan(cursor=cursor, match=match, count=count)

@retry()
def safe_type(r, key):
    return r.type(key)

@retry()
def safe_get(r, key):
    return r.get(key)

@retry()
def safe_lrange(r, key, start, end):
    return r.lrange(key, start, end)

@retry()
def safe_hscan(r, key, cursor, count):
    return r.hscan(key, cursor=cursor, count=count)

@retry()
def safe_sscan(r, key, cursor, count):
    return r.sscan(key, cursor=cursor, count=count)

@retry()
def safe_zscan(r, key, cursor, count):
    return r.zscan(key, cursor=cursor, count=count)

@retry()
def safe_xrange(r, key, start_id='-', end_id='+', count=XRANGE_PAGE):
    return r.xrange(key, min=start_id, max=end_id, count=count)

@retry()
def safe_ttl(r, key):
    return r.ttl(key)

# --- Stream collector ---
def collect_stream_strict(r, key, page_size=XRANGE_PAGE):
    entries = []
    start = '-'
    while True:
        batch = safe_xrange(r, key, start_id=start, end_id='+', count=page_size)
        if not batch:
            break
        if start != '-' and batch and batch[0][0].decode() == start:
            batch = batch[1:]
        if not batch:
            break
        for entry_id, fields in batch:
            eid = entry_id.decode()
            fm = {}
            for fk, fv in fields.items():
                try:
                    fk_key = fk.decode("utf-8")
                except Exception:
                    fk_key = "__b64_field__" + base64.b64encode(fk).decode("ascii")
                fm[fk_key] = safe_serialize(fv)
            entries.append([eid, fm])
        last_id = batch[-1][0].decode()
        ms, seq = last_id.split('-')
        start = f"{ms}-{int(seq)+1}"
        if len(batch) < page_size:
            break
    return entries

# --- Key backup dispatcher ---
def backup_key(r, key):
    ktype = safe_type(r, key).decode()
    if ktype == 'string':
        return safe_serialize(safe_get(r, key))
    elif ktype == 'hash':
        out, cursor = {}, 0
        while True:
            cursor, batch = safe_hscan(r, key, cursor, 1000)
            for f, v in batch.items():
                try:
                    fk = f.decode("utf-8")
                except Exception:
                    fk = "__b64_field__" + base64.b64encode(f).decode("ascii")
                out[fk] = safe_serialize(v)
            if cursor == 0:
                break
        return out
    elif ktype == 'set':
        members, cursor = [], 0
        while True:
            cursor, batch = safe_sscan(r, key, cursor, 1000)
            for m in batch:
                members.append(safe_serialize(m))
            if cursor == 0:
                break
        return members
    elif ktype == 'zset':
        items, cursor = [], 0
        while True:
            cursor, batch = safe_zscan(r, key, cursor, 1000)
            for m, score in batch:
                items.append([safe_serialize(m), score])
            if cursor == 0:
                break
        return items
    elif ktype == 'list':
        items, length, BATCH = [], r.llen(key), 1000
        for start in range(0, length, BATCH):
            chunk = safe_lrange(r, key, start, start+BATCH-1)
            items.extend(safe_serialize(e) for e in chunk)
        return items
    elif ktype == 'stream':
        return collect_stream_strict(r, key, XRANGE_PAGE)
    else:
        return None

# --- Save chunk ---
def save_chunk(chunk_obj, db_index, part_num):
    meta = {
        "generated_at": time.strftime("%Y-%m-%dT%H:%M:%SZ", time.gmtime()),
        "db": db_index,
        "chunk_number": part_num,
        "keys_in_chunk": len(chunk_obj)
    }
    out = {"_meta": meta, "data": chunk_obj}
    filename = os.path.join(OUTPUT_DIR, f"redis_backup_db{db_index}_part_{part_num}.json")
    atomic_write_json(out, filename)
    print(f"💾 Saved {len(chunk_obj)} keys to {filename}")

# --- Main ---
def backup_all_dbs():
    r = redis.StrictRedis(host=REDIS_HOST, port=REDIS_PORT, decode_responses=False)
    for db_index in range(0, 16):
        try:
            r.execute_command("SELECT", db_index)
        except Exception:
            break

        print(f"\n🔍 Scanning DB {db_index}...")
        cursor, chunk, part, found = 0, {}, 1, 0
        while True:
            cursor, keys = safe_scan(r, cursor, count=1000)
            for key in keys:
                try:
                    key_str = key.decode("utf-8")
                except Exception:
                    key_str = "__b64_key__" + base64.b64encode(key).decode("ascii")

                found += 1
                try:
                    val = backup_key(r, key)
                    if val is None:
                        print(f"⚠️ Unsupported type for key {key_str}; skipping")
                        continue
                    ttl = safe_ttl(r, key)
                    chunk[key_str] = {"type": safe_type(r, key).decode(), "db": db_index, "value": val, "ttl": ttl}
                except Exception as e:
                    print(f"❌ Error reading key {key_str}: {e}")

                if len(chunk) >= CHUNK_SIZE:
                    save_chunk(chunk, db_index, part)
                    chunk, part = {}, part+1

            if cursor == 0:
                break

        if chunk:
            save_chunk(chunk, db_index, part)
        print(f"✅ DB {db_index} done. Keys backed up: {found}")

    print("\n🎉 All DBs processed.")

if __name__ == "__main__":
    backup_all_dbs()
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
#!/usr/bin/env python3
"""
restore_all_dbs.py

Restores keys into a Redis server from backup JSON files
created by robust_chunk_redis_backup_all_keys.py.
"""

import redis
import json
import base64
import os
import glob
import time

# ===== Config =====
REDIS_HOST = "127.0.0.1"     # local Redis
REDIS_PORT = 6379
INPUT_DIR = "backup_output"  # folder containing JSON backup files
MAX_RETRIES = 5
# ==================

def safe_decode(wrapper):
    """Decode serialized value from backup."""
    if wrapper is None:
        return None
    if wrapper["_t"] == "str":
        return wrapper["v"].encode("utf-8")
    elif wrapper["_t"] == "b64":
        return base64.b64decode(wrapper["v"].encode("ascii"))
    else:
        raise ValueError(f"Unknown wrapper type: {wrapper}")

def restore_key(r, db_index, key, key_data):
    """Restore a single key into Redis."""
    key_type = key_data["type"]
    value = key_data["value"]
    ttl = key_data.get("ttl", -1)

    # Ensure we select the correct DB
    r.execute_command("SELECT", db_index)

    if key_type == "string":
        r.set(key, safe_decode(value))
    elif key_type == "hash":
        decoded = { f: safe_decode(v) for f, v in value.items() }
        if decoded:
            r.hset(key, mapping=decoded)
    elif key_type == "set":
        members = [ safe_decode(v) for v in value ]
        if members:
            r.sadd(key, *members)
    elif key_type == "zset":
        members = { safe_decode(m): score for m, score in value }
        if members:
            r.zadd(key, members)
    elif key_type == "list":
        items = [ safe_decode(v) for v in value ]
        if items:
            r.rpush(key, *items)
    elif key_type == "stream":
        for entry_id, fields in value:
            decoded_fields = { f: safe_decode(v) for f, v in fields.items() }
            r.xadd(key, decoded_fields, id=entry_id)
    else:
        print(f"⚠️ Skipping unknown type {key_type} for key {key}")

    # Restore TTL if present
    if ttl and ttl > 0:
        r.expire(key, ttl)

def main():
    files = sorted(glob.glob(os.path.join(INPUT_DIR, "redis_backup_db*_part_*.json")))
    if not files:
        print("❌ No backup files found.")
        return

    print(f"🔄 Starting restore from {len(files)} files...")

    r = redis.StrictRedis(host=REDIS_HOST, port=REDIS_PORT)

    total_keys = 0
    for file in files:
        print(f"📂 Restoring {file} ...")
        with open(file, "r", encoding="utf-8") as f:
            parsed = json.load(f)

        # Real keys are under "data"
        data = parsed.get("data", {})

        for key, key_data in data.items():
            db_index = key_data.get("db", 0)

            for attempt in range(MAX_RETRIES):
                try:
                    restore_key(r, db_index, key, key_data)
                    break
                except redis.ConnectionError as e:
                    print(f"⚠️ Redis connection error on key {key}, retry {attempt+1}/{MAX_RETRIES}")
                    time.sleep(2 ** attempt)
                except Exception as e:
                    print(f"❌ Error restoring key {key}: {e}")
                    break

            total_keys += 1

        print(f"✅ Restored {len(data)} keys from {file}")

    print(f"🎉 Restore complete. Total keys restored: {total_keys}")

if __name__ == "__main__":
    main()

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

# Taking backup of the entire 16 DB's from Local Redis Local Server

<details>
  <summary>Click to view Step-by-Step Plan</summary>

### ✅ **Changes made:**

1. **Backs up all Redis DBs** (0–6 by default).
2. **Creates a local folder** (e.g., `redis_backups/`) if it doesn't exist.
3. **Stores each DB's JSON file** as `dump_db_<n>.json` inside the backup folder.
4. **Works on local Redis** (`localhost:6379`) by default.

### 🔧 **Modified Script**

```python
import redis
import json
import base64
import os
from datetime import datetime

# CONFIG
HOST = 'localhost'   # Change if needed
PORT = 6379
DB_RANGE = range(0, 7)
BACKUP_DIR = 'redis_backups'

def safe_decode(value):
    try:
        return value.decode('utf-8')
    except Exception:
        return base64.b64encode(value).decode('ascii')

def dump_db(db_index):
    r = redis.StrictRedis(host=HOST, port=PORT, db=db_index)
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
                data[key_decoded] = {'type': 'zset', 'value': [{'member': safe_decode(i[0]), 'score': i[1]} for i in zset_data]}
        except Exception as e:
            print(f"❌ Error dumping key {key}: {e}")

    # Create backup folder if not exists
    os.makedirs(BACKUP_DIR, exist_ok=True)
    filename = os.path.join(BACKUP_DIR, f'dump_db_{db_index}.json')

    with open(filename, 'w') as f:
        json.dump(data, f, indent=2)

if __name__ == "__main__":
    print(f"🔁 Starting Redis backup into '{BACKUP_DIR}' ...")
    for db_index in DB_RANGE:
        print(f'📦 Dumping DB {db_index}...')
        dump_db(db_index)
    print('✅ All Redis databases backed up successfully.')
```

### 📂 Output:

This will create a folder structure like:

```
redis_backups/
  ├── dump_db_0.json
  ├── dump_db_1.json
  ├── ...
  └── dump_db_6.json
```

### 🧪 Optional test

```bash
ls -lh redis_backups/
cat redis_backups/dump_db_0.json | jq '. | keys'
```

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

<details>
  <summary>Step by step To clean all the database</summary>


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


</details>

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

---

# To verify all the keys or the specific keys weather it is present or not

<details>
  <summary>Click here to view step by step guide</summary>

### 🔎 **To verify which DB a key exists in — on *local Redis only* — and report missing ones**


### ✅ **Does**

* Scans all **databases (0–15)** in **local Redis only**
* Reports:

  * Which DB each key exists in (first match)
  * Which keys are **missing entirely**

---

### ✅ Python Script: `verify_keys_in_local.py`

```python
import redis

# ----------- CONFIGURATION -----------
LOCAL_REDIS = {
    "host": "127.0.0.1",
    "port": 6379,
    "decode_responses": True
}

# ----------- List of Keys to Check -----------
keys_to_check = [
    "cabs.9278.live_details",
    "cabs.9274.live_details",
    "cabs.9272.live_details",
    "cabs.9271.live_details",
    "cabs.8954.live_details",
    "cabs.8950.live_details",
    "cabs.8945.live_details"
]

# ----------- Logic -----------
local = redis.Redis(**LOCAL_REDIS)

print("\n🔍 Searching for keys in local Redis (DBs 0–15)...\n")

for key in keys_to_check:
    found = False

    for db in range(16):
        local.select(db)
        if local.exists(key):
            print(f"✅ Found: {key} in DB {db}")
            found = True
            break

    if not found:
        print(f"❌ Missing: {key} (not found in any DB)")

print("\n✅ Done.")
```

---

- Run the script
  ```
  python3 verify_keys_in_local.py
  ```

---
### ✅ Output Example

```bash
✅ Found: cabs.9278.live_details in DB 1
✅ Found: cabs.9274.live_details in DB 0
❌ Missing: cabs.9271.live_details (not found in any DB)
```

---

### 🔎 **To verify which DB a key exists in — on *Elasticache Redis only* — and report missing ones**

### ✅ Assumptions

* Your **ElastiCache Redis endpoint** is available (e.g., `my-cluster.cache.amazonaws.com`)
* You have network access (e.g., from an EC2 instance in the same VPC)
* You want to check **specific keys** across **all DBs (0–15)**

---

### ✅ Python Script: `verify_keys_elasticache.py`

```python
import redis

# -------------------- Configuration --------------------
ELASTICACHE_REDIS = {
    "host": "<your-elasticache-endpoint>",  # Replace with your endpoint
    "port": 6379,
    "decode_responses": True
}

# -------------------- Keys to Check --------------------
keys_to_check = [
    "cabs.9278.live_details",
    "cabs.9274.live_details",
    "cabs.9272.live_details",
    "cabs.9271.live_details",
    "cabs.8954.live_details",
    "cabs.8950.live_details",
    "cabs.8945.live_details"
]

# -------------------- Script --------------------
elasticache = redis.Redis(**ELASTICACHE_REDIS)

print("\n🔍 Searching for keys in ElastiCache Redis (DBs 0–15)...\n")

for key in keys_to_check:
    found = False

    for db in range(16):
        try:
            elasticache.execute_command('SELECT', db)
            if elasticache.exists(key):
                print(f"✅ Found: {key} in DB {db}")
                found = True
                break
        except Exception as e:
            print(f"⚠️ Error checking DB {db}: {e}")

    if not found:
        print(f"❌ Missing: {key} (not found in any DB)")

print("\n✅ Done.")
```

---

### 🔁 Replace `<your-elasticache-endpoint>`:

Use the full host name (e.g.):

```python
"host": "my-elasticache-name.abc123.ng.0001.aps1.cache.amazonaws.com"
```

---

### 🧪 Optional: Check Single Key Only

You can also adapt the code to check a single key like this:

```python
key = "cabs.9278.live_details"
found = False
for db in range(16):
    elasticache.execute_command('SELECT', db)
    if elasticache.exists(key):
        print(f"Found in DB {db}")
        found = True
        break
if not found:
    print("Key not found in any DB")
```

---

### ❗Important Notes

* ElastiCache is single-threaded — avoid heavy scans in production.
* `SELECT` is allowed on ElastiCache **if cluster mode is disabled**.
* For **cluster-mode enabled**: keys are sharded — `SELECT` won't work. Let me know if this is your case.

---


</details>

---

<details>
  <summary>All the scripts to executed in the machine</summary>

```
[root@ip-172-31-35-246 ~]# ls
comparesync.py  dump1  dump_db_safe.py  elasticache_backup  key_dumps  redisenv  redis_key_verifier.py  redis-stable  restore_db_safe.py  roughfolder  sync_all_dbs.py  verify_keys_in_local.py
[root@ip-172-31-35-246 ~]# cat comparesync.py
import redis

# --- Connect to ElastiCache Redis (source) ---
source_redis = redis.Redis(
    host='<your-elasticache-end-point>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com',
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

[root@ip-172-31-35-246 ~]# ls
comparesync.py  dump1  dump_db_safe.py  elasticache_backup  key_dumps  redisenv  redis_key_verifier.py  redis-stable  restore_db_safe.py  roughfolder  sync_all_dbs.py  verify_keys_in_local.py
[root@ip-172-31-35-246 ~]# cat dump_db_safe.py
# dump_db_safe.py
import redis
import json
import base64

HOST = '<your-elasticache-end-point>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com'
PORT = 6379
DB_INDEX = 6  # change this to target a single DB

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
[root@ip-172-31-35-246 ~]# ls
comparesync.py  dump1  dump_db_safe.py  elasticache_backup  key_dumps  redisenv  redis_key_verifier.py  redis-stable  restore_db_safe.py  roughfolder  sync_all_dbs.py  verify_keys_in_local.py
[root@ip-172-31-35-246 ~]# cat redis_key_verifier.py
import redis
from typing import List

# ----------- CONFIGURATION -----------
SOURCE_REDIS = {
    "host": "<your-elasticache-end-point>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com",
    "port": 6379,
    "db": 0,
    "decode_responses": True
}

LOCAL_REDIS = {
    "host": "127.0.0.1",
    "port": 6379,
    "db": 0,
    "decode_responses": True
}

# ---------- INIT CONNECTIONS ----------
source = redis.Redis(**SOURCE_REDIS)
local = redis.Redis(**LOCAL_REDIS)


# ---------- VERIFY ALL KEYS ----------
def verify_all_keys_across_dbs():
    print("\n🔍 Verifying ALL keys across DBs 0–15...\n")
    for db in range(16):
        source.select(db)
        local.select(db)

        source_keys = set(source.scan_iter("*"))
        local_keys = set(local.scan_iter("*"))

        print(f"📂 DB {db}: {len(source_keys)} keys in source, {len(local_keys)} in local")

        missing_keys = source_keys - local_keys
        extra_keys = local_keys - source_keys

        if missing_keys:
            print(f"❌ Missing in LOCAL (DB {db}):")
            for key in missing_keys:
                print(f"   - {key}")
        if extra_keys:
            print(f"⚠️ Extra in LOCAL not in source (DB {db}):")
            for key in extra_keys:
                print(f"   - {key}")
    print("\n✅ Verification complete.")


# ---------- VERIFY SPECIFIC KEYS ----------
def verify_specific_keys(keys: List[str]):
    print(f"\n🔍 Verifying specific keys across DBs 0–15...\n")

    for key in keys:
        found_in_source = False
        found_in_local = False

        for db in range(16):
            source.select(db)
            if source.exists(key):
                print(f"✅ Key '{key}' found in SOURCE Redis DB {db}")
                found_in_source = True
                break

        for db in range(16):
            local.select(db)
            if local.exists(key):
                print(f"✅ Key '{key}' found in LOCAL Redis DB {db}")
                found_in_local = True
                break

        if not found_in_source:
            print(f"❌ Key '{key}' NOT found in SOURCE Redis")
        if not found_in_local:
            print(f"❌ Key '{key}' NOT found in LOCAL Redis")


# ---------- MAIN ENTRY ----------
if __name__ == "__main__":
    import argparse

    parser = argparse.ArgumentParser(description="Redis DB Key Verifier")
    parser.add_argument("--all", action="store_true", help="Check all DBs and keys")
    parser.add_argument("--keys", nargs="+", help="List of keys to verify")

    args = parser.parse_args()

    if args.all:
        verify_all_keys_across_dbs()
    elif args.keys:
        verify_specific_keys(args.keys)
    else:
        parser.print_help()

[root@ip-172-31-35-246 ~]# ls
comparesync.py  dump1  dump_db_safe.py  elasticache_backup  key_dumps  redisenv  redis_key_verifier.py  redis-stable  restore_db_safe.py  roughfolder  sync_all_dbs.py  verify_keys_in_local.py
[root@ip-172-31-35-246 ~]# cat restore_db_safe.py
# restore_db_safe.py
import redis
import json
import base64

DB_INDEX = 6  # same as exported DB

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
[root@ip-172-31-35-246 ~]# ls
comparesync.py  dump1  dump_db_safe.py  elasticache_backup  key_dumps  redisenv  redis_key_verifier.py  redis-stable  restore_db_safe.py  roughfolder  sync_all_dbs.py  verify_keys_in_local.py
[root@ip-172-31-35-246 ~]# cat sync_all_dbs.py
import redis

# --- Connect to ElastiCache Redis (source) ---
source_redis = redis.Redis(
    host='<your-elasticache-end-point>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com',
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

[root@ip-172-31-35-246 ~]# ls
comparesync.py  dump1  dump_db_safe.py  elasticache_backup  key_dumps  redisenv  redis_key_verifier.py  redis-stable  restore_db_safe.py  roughfolder  sync_all_dbs.py  verify_keys_in_local.py
[root@ip-172-31-35-246 ~]# cat verify_keys_in_local.py
import redis

# ----------- CONFIGURATION -----------
LOCAL_REDIS = {
    "host": "127.0.0.1",
    "port": 6379,
    "decode_responses": True
}

# ----------- List of Keys to Check -----------
keys_to_check = [
    "cabs.9278.live_details",
    "cabs.9274.live_details",
    "cabs.9272.live_details",
    "cabs.9271.live_details",
    "cabs.8954.live_details",
    "cabs.8950.live_details",
    "cabs.8945.live_details"
]

# ----------- Logic -----------
local = redis.Redis(**LOCAL_REDIS)

print("\n🔍 Searching for keys in local Redis (DBs 0–15)...\n")

for key in keys_to_check:
    found = False

    for db in range(16):
        local.select(db)
        if local.exists(key):
            print(f"✅ Found: {key} in DB {db}")
            found = True
            break

    if not found:
        print(f"❌ Missing: {key} (not found in any DB)")

print("\n✅ Done.")

[root@ip-172-31-35-246 ~]#
```
  
</details>

---

## 🔍 Workaround to Detect Active Databases
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
for i in {0..15}; do echo "DB $i:"; redis-cli -h <your-end-point>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com -p 6379 -n $i dbsize; done
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


# Take backup of the few keys, delete them in the redis server in local and restore those keys and verify.

<details>
  <summary>Click to view the steps</summary>

### Step 1: Loop through all Redis DBs (0 to 15) to find specified keys and take backup.
- `vi backup_selected_keys.py`
```
import redis
import json
import base64

# Helper to safely decode binary to string
def safe_decode(value):
    if isinstance(value, str):
        return value
    try:
        return value.decode("utf-8")
    except Exception:
        return base64.b64encode(value).decode("ascii")

# Keys to backup
keys_to_backup = [
    "cabs.9278.live_details", "cabs.9274.live_details", "cabs.9272.live_details",
    "cabs.9271.live_details", "cabs.8954.live_details", "cabs.8950.live_details", "cabs.8945.live_details"
]

backup_data = {}

# Loop through Redis DBs 0–15
for db_index in range(16):
    r = redis.StrictRedis(host='localhost', port=6379, db=db_index)

    for key in keys_to_backup:
        # Ensure key is in bytes for Redis operations
        key_bytes = key.encode("utf-8") if isinstance(key, str) else key

        if r.exists(key_bytes):
            key_type = safe_decode(r.type(key_bytes))
            key_decoded = safe_decode(key_bytes)

            if key_type == "string":
                value = safe_decode(r.get(key_bytes))

            elif key_type == "hash":
                raw = r.hgetall(key_bytes)
                value = {safe_decode(k): safe_decode(v) for k, v in raw.items()}

            elif key_type == "set":
                value = [safe_decode(v) for v in r.smembers(key_bytes)]

            elif key_type == "zset":
                value = [(safe_decode(v), score) for v, score in r.zrange(key_bytes, 0, -1, withscores=True)]

            elif key_type == "list":
                value = [safe_decode(v) for v in r.lrange(key_bytes, 0, -1)]

            else:
                value = None

            backup_data[key_decoded] = {
                "type": key_type,
                "value": value,
                "db": db_index
            }

# Save to JSON
with open("redis_backup.json", "w") as f:
    json.dump(backup_data, f, indent=2)

print("✅ Backup complete. Saved to redis_backup.json")

```

- Execute the script
  - `python3 backup_selected_keys.py`

---

### Step 2: Loop through all Redis DBs (0 to 15) to delete specified keys
- `vi delete_selected_keys.py`

```python
import redis

# Connect to local Redis (loop through all DBs)
keys_to_delete = [
    "cabs.9278.live_details", "cabs.9274.live_details", "cabs.9272.live_details",
    "cabs.9271.live_details", "cabs.8954.live_details", "cabs.8950.live_details", "cabs.8945.live_details"
]

# Loop through DBs 0 to 15 (default max)
for db_index in range(16):
    r = redis.StrictRedis(host='localhost', port=6379, db=db_index)
    print(f"\n🔍 Checking DB {db_index}...")
    found_keys = []

    for key in keys_to_delete:
        if r.exists(key):
            found_keys.append(key)

    if found_keys:
        print(f"🗑️ Found {len(found_keys)} key(s) in DB {db_index}. Deleting...")
        r.delete(*found_keys)
        print(f"✅ Deleted keys from DB {db_index}: {found_keys}")
    else:
        print("⚠️ No matching keys found.")
```

- Execute the Script
`python3 delete_selected_keys.py`

- Expected Output
  
<details>
  <summary>Click to view the output</summary>

```

🔍 Checking DB 0...
⚠️ No matching keys found.

🔍 Checking DB 1...
⚠️ No matching keys found.

🔍 Checking DB 2...
⚠️ No matching keys found.

🔍 Checking DB 3...
⚠️ No matching keys found.

🔍 Checking DB 4...
⚠️ No matching keys found.

🔍 Checking DB 5...
🗑️ Found 7 key(s) in DB 5. Deleting...
✅ Deleted keys from DB 5: ['cabs.9278.live_details', 'cabs.9274.live_details', 'cabs.9272.live_details', 'cabs.9271.live_details', 'cabs.8954.live_details', 'cabs.8950.live_details', 'cabs.8945.live_details']

🔍 Checking DB 6...
⚠️ No matching keys found.

🔍 Checking DB 7...
⚠️ No matching keys found.

🔍 Checking DB 8...
⚠️ No matching keys found.

🔍 Checking DB 9...
⚠️ No matching keys found.

🔍 Checking DB 10...
⚠️ No matching keys found.

🔍 Checking DB 11...
⚠️ No matching keys found.

🔍 Checking DB 12...
⚠️ No matching keys found.

🔍 Checking DB 13...
⚠️ No matching keys found.

🔍 Checking DB 14...
⚠️ No matching keys found.

🔍 Checking DB 15...
⚠️ No matching keys found.
```

</details>
  
---

### Step 3: Verify those keys weather its Present or not

### Step by step checking all the keys

```bash
127.0.0.1:6379> select 5
OK
127.0.0.1:6379[5]> exists cabs.9278.live_details
(integer) 0
```

That means:
➡️ `cabs.9278.live_details` does **not** exist in **DB 5** anymore.

### 🔁 To Verify All Deleted Keys (Manually)

Repeat for each of the deleted keys:

```bash
127.0.0.1:6379[5]> exists cabs.9274.live_details
127.0.0.1:6379[5]> exists cabs.9272.live_details
127.0.0.1:6379[5]> exists cabs.9271.live_details
127.0.0.1:6379[5]> exists cabs.8954.live_details
127.0.0.1:6379[5]> exists cabs.8950.live_details
127.0.0.1:6379[5]> exists cabs.8945.live_details
```

Each one should return:

```
(integer) 0
```

### Alternate: Python Script to Confirm

You can use this short verification script:

```python
import redis

keys_to_check = [
    "cabs.9278.live_details", "cabs.9274.live_details", "cabs.9272.live_details",
    "cabs.9271.live_details", "cabs.8954.live_details", "cabs.8950.live_details", "cabs.8945.live_details"
]

for db in range(16):
    r = redis.StrictRedis(host='localhost', port=6379, db=db)
    found = [key for key in keys_to_check if r.exists(key)]
    if found:
        print(f"❌ Found in DB {db}: {found}")
    else:
        print(f"✅ DB {db}: None of the keys exist.")
```

Run:

```bash
python3 verify_deleted_keys.py
```

### ✅ Final Notes

* `exists cabs` returning 1 means there's **another key** just called `cabs` — totally unrelated.
* Your deleted keys are confirmed **gone** when `exists` returns 0.

---

### Step 4: Loop through all Redis DBs (0 to 15) to restore from the specific key backup
- `vi restore_selected_keys.py`
```python
import json
import redis

# Load the backup JSON
with open('redis_backup.json') as f:
    backup_data = json.load(f)

r = redis.StrictRedis(host='127.0.0.1', port=6379)

for key, entry in backup_data.items():
    key_type = entry['type']
    value = entry['value']
    db = entry['db']

    # Select the correct Redis DB
    r.execute_command('SELECT', db)

    if key_type == 'string':
        r.set(key, value)

    elif key_type == 'hash':
        r.hmset(key, value)

    elif key_type == 'list':
        for item in value:
            r.rpush(key, item)

    elif key_type == 'set':
        r.sadd(key, *value)

    elif key_type == 'zset':
        for item in value:
            r.zadd(key, {item['member']: item['score']})

    else:
        print(f"⚠️ Skipping unsupported type for key: {key}")

print("✅ Redis restore completed successfully.")
```

- Run the script
  `python3 restore_selected_keys.py`

---

### Step 5: Verify those keys weather its backedup or not

### Step by step checking all the keys

```bash
127.0.0.1:6379> select 5
OK
127.0.0.1:6379[5]> exists cabs.9278.live_details
(integer) 1
```

That means:
➡️ `cabs.9278.live_details` does exist in **DB 5** anymore.

### 🔁 To Verify All Deleted Keys (Manually)

Repeat for each of the deleted keys:

```bash
127.0.0.1:6379[5]> exists cabs.9274.live_details
127.0.0.1:6379[5]> exists cabs.9272.live_details
127.0.0.1:6379[5]> exists cabs.9271.live_details
127.0.0.1:6379[5]> exists cabs.8954.live_details
127.0.0.1:6379[5]> exists cabs.8950.live_details
127.0.0.1:6379[5]> exists cabs.8945.live_details
```

Each one should return:

```
(integer) 1
```

</details>

---

# To restore specific keys which is more than 20,000

<details>
  <summary>Click to view the steps</summary>

## From Local Redis Server taking Backup
### ✅ **Goal**

You want a script that:

* Reads keys from a file like `keys.txt`(where all the keys are stored which has to backed-up)
* Connects to **all Redis DBs** (0–15)
* Safely backs up data (even binary/unicode issues)
* Preserves `type`, `value`, and `db`
* Outputs to a safe JSON (`redis_backup.json`)
* Handles unknown types cleanly
* Can later be used to restore accurately

---

### ✅ **Final Backup Script (From `keys.txt`)**

```python
import redis
import json
import base64

# Safe decoder to handle non-UTF8 data
def safe_decode(value):
    if isinstance(value, str):
        return value
    try:
        return value.decode("utf-8")
    except Exception:
        return base64.b64encode(value).decode("ascii")

# Read keys from file
with open("keys.txt") as f:
    keys_to_backup = [line.strip() for line in f if line.strip()]

backup_data = {}

# Try DBs 0–15
for db_index in range(16):
    r = redis.StrictRedis(host='localhost', port=6379, db=db_index)

    for key in keys_to_backup:
        key_bytes = key.encode("utf-8") if isinstance(key, str) else key

        if not r.exists(key_bytes):
            continue

        try:
            key_type = r.type(key_bytes).decode()
        except Exception:
            print(f"⚠️ Error detecting type for key: {key}")
            continue

        key_decoded = safe_decode(key_bytes)

        try:
            if key_type == "string":
                value = safe_decode(r.get(key_bytes))

            elif key_type == "hash":
                raw = r.hgetall(key_bytes)
                value = {safe_decode(k): safe_decode(v) for k, v in raw.items()}

            elif key_type == "set":
                value = [safe_decode(v) for v in r.smembers(key_bytes)]

            elif key_type == "zset":
                value = [(safe_decode(v), score) for v, score in r.zrange(key_bytes, 0, -1, withscores=True)]

            elif key_type == "list":
                value = [safe_decode(v) for v in r.lrange(key_bytes, 0, -1)]

            else:
                print(f"⚠️ Unknown type for key: {key} (Type: {key_type})")
                continue

            backup_data[key_decoded] = {
                "type": key_type,
                "value": value,
                "db": db_index
            }

        except Exception as e:
            print(f"❌ Error processing key: {key} — {e}")

# Save to JSON
with open("redis_backup.json", "w") as f:
    json.dump(backup_data, f, indent=2)

print(f"✅ Backup complete. {len(backup_data)} keys saved to redis_backup.json.")
```

---

### 📄 **Your `keys.txt` (example)**

```
cabs.9278.live_details
cabs.9274.live_details
cabs.9272.live_details
cabs.9271.live_details
cabs.8954.live_details
cabs.8950.live_details
cabs.8945.live_details
```

---

### ✅ Benefits

* No hardcoded keys
* Works even if Redis data has binary or emoji characters
* Clear logging of skipped/unknown/corrupt keys
* Can later be restored with type, DB, and structure

---

### ⏭️ Next Step (Optional)

If you want, I can also provide a matching **restore script** that:

* Loads from `redis_backup.json`
* Writes back to the correct DB
* Converts values safely back (even from base64)


---
---

## From Elasticache taking backup
* Use a `keys.txt` file to specify keys to back up.
* Connect to your **ElastiCache endpoint**.
* Save the backup into a uniquely named file inside a separate folder (`backup_output/`).
* Handle **non-UTF8 data** using base64.
* Search across **databases 0–15**, though ElastiCache typically uses only **DB 0** unless cluster mode is enabled.

---

### ✅ Final Python Script: `elasticache_key_backup.py`

```python
import redis
import json
import base64
import os
from datetime import datetime

# Config
REDIS_HOST = "<your-end-point>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com"
REDIS_PORT = 6379
KEY_FILE = "keys.txt"
OUTPUT_DIR = "backup_output"

# Safe decoder to handle non-UTF8 data
def safe_decode(value):
    if isinstance(value, str):
        return value
    try:
        return value.decode("utf-8")
    except Exception:
        return base64.b64encode(value).decode("ascii")

# Read keys from file
with open(KEY_FILE) as f:
    keys_to_backup = [line.strip() for line in f if line.strip()]

backup_data = {}

# Ensure output directory exists
os.makedirs(OUTPUT_DIR, exist_ok=True)

# Generate output filename
timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
output_file = os.path.join(OUTPUT_DIR, f"redis_backup_{timestamp}.json")

# Loop through Redis DBs (0–15)
for db_index in range(16):
    print(f"🔍 Checking DB {db_index}...")
    try:
        r = redis.StrictRedis(host=REDIS_HOST, port=REDIS_PORT, db=db_index, socket_connect_timeout=5)

        for key in keys_to_backup:
            key_bytes = key.encode("utf-8") if isinstance(key, str) else key

            if not r.exists(key_bytes):
                continue

            try:
                key_type = r.type(key_bytes).decode()
            except Exception:
                print(f"⚠️ Error detecting type for key: {key}")
                continue

            key_decoded = safe_decode(key_bytes)

            try:
                if key_type == "string":
                    value = safe_decode(r.get(key_bytes))

                elif key_type == "hash":
                    raw = r.hgetall(key_bytes)
                    value = {safe_decode(k): safe_decode(v) for k, v in raw.items()}

                elif key_type == "set":
                    value = [safe_decode(v) for v in r.smembers(key_bytes)]

                elif key_type == "zset":
                    value = [(safe_decode(v), score) for v, score in r.zrange(key_bytes, 0, -1, withscores=True)]

                elif key_type == "list":
                    value = [safe_decode(v) for v in r.lrange(key_bytes, 0, -1)]

                else:
                    print(f"⚠️ Unknown type for key: {key} (Type: {key_type})")
                    continue

                backup_data[key_decoded] = {
                    "type": key_type,
                    "value": value,
                    "db": db_index
                }

            except Exception as e:
                print(f"❌ Error processing key: {key} — {e}")

    except Exception as conn_err:
        print(f"🚫 Could not connect to DB {db_index}: {conn_err}")
        continue

# Save to JSON
with open(output_file, "w") as f:
    json.dump(backup_data, f, indent=2)

print(f"\n✅ Backup complete. {len(backup_data)} keys saved to {output_file}")
```

---

### ✅ Instructions

1. **Create `keys.txt`** with one Redis key per line, for example:

   ```
   cabs.9278.live_details
   cabs.8954.live_details
   cabs.8950.live_details
   ```

2. **Run the script:**

   ```bash
   python3 elasticache_key_backup.py
   ```

3. **Result:**

   * A file like `backup_output/redis_backup_20250807_164255.json` is created.
   * It contains all found keys with correct types and values.
   * Handles all Redis types: `string`, `hash`, `set`, `zset`, and `list`.

---

### Restoring those particular keys to the Elasticache

```python
import redis
import json
import base64
import os

# Config
REDIS_HOST = "<your-end-point>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com"
REDIS_PORT = 6379
BACKUP_FILE = "backup_output/redis_backup_20250808_120530.json"  # Change to your backup file path

# Safe encoder to handle Base64 or UTF-8 strings
def safe_encode(value):
    if isinstance(value, str):
        try:
            return value.encode("utf-8")
        except Exception:
            pass
    try:
        return base64.b64decode(value)
    except Exception:
        return str(value).encode("utf-8")

# Load backup data
with open(BACKUP_FILE, "r") as f:
    backup_data = json.load(f)

restored_count = 0

# Restore keys
for key, meta in backup_data.items():
    db_index = meta["db"]
    key_type = meta["type"]
    value = meta["value"]

    try:
        r = redis.StrictRedis(
            host=REDIS_HOST,
            port=REDIS_PORT,
            db=db_index,
            socket_connect_timeout=5
        )

        key_bytes = safe_encode(key)

        # Delete existing key if any
        r.delete(key_bytes)

        if key_type == "string":
            r.set(key_bytes, safe_encode(value))

        elif key_type == "hash":
            hash_data = {safe_encode(k): safe_encode(v) for k, v in value.items()}
            if hash_data:
                r.hset(key_bytes, mapping=hash_data)

        elif key_type == "list":
            list_data = [safe_encode(v) for v in value]
            if list_data:
                r.rpush(key_bytes, *list_data)

        elif key_type == "set":
            set_data = [safe_encode(v) for v in value]
            if set_data:
                r.sadd(key_bytes, *set_data)

        elif key_type == "zset":
            zset_data = {safe_encode(v): score for v, score in value}
            if zset_data:
                r.zadd(key_bytes, zset_data)

        else:
            print(f"⚠️ Skipping unsupported type for key: {key_type}")
            continue

        restored_count += 1

    except Exception as e:
        print(f"❌ Error restoring key {key} (DB {db_index}): {e}")

print(f"\n✅ Restore complete. {restored_count} keys restored from {BACKUP_FILE}")

```

---
### ✅ YES — It Can Work

Python + `redis-py` can easily process 30,000 keys **one-by-one** from a `keys.txt` file and fetch their values from ElastiCache.
- Yes, your script **can handle 30,000 keys**, **but** there are **important limitations and optimizations** to keep in mind when backing up that many keys from ElastiCache:

But you must address:

---

## ⚠️ Performance and Limits

| Concern             | Description                                                                                               | Recommendation                                |
| ------------------- | --------------------------------------------------------------------------------------------------------- | --------------------------------------------- |
| **Time**            | 30,000 keys can take **minutes** to hours depending on latency and key size.                              | Use batching or concurrency.                  |
| **Timeouts**        | ElastiCache may **drop idle clients** or large slow queries.                                              | Set proper timeouts; avoid long blocking ops. |
| **Memory**          | If the data volume is large (e.g., 100 MB+), `backup_data` dictionary in memory can cause **RAM spikes**. | Stream output instead of loading all in RAM.  |
| **Throttling**      | AWS may **rate-limit** your ElastiCache connection on high traffic.                                       | Add delays or concurrency control.            |
| **Multi-threading** | Currently the script runs **sequentially**. It will take longer.                                          | Use multiprocessing or threading.             |

---

### ✅ Optimized Solutions

#### 🔁 **Option A – Just Run as Is**

If you're OK waiting for 10–20+ minutes and the memory fits your system, the script will work.
Make sure:

```python
r = redis.StrictRedis(host=..., port=..., socket_connect_timeout=10, socket_timeout=10)
```

#### 🚀 **Option B – Add Progress and Batch Logging**

Use `tqdm` for progress bar and `flush` intermediate backups every 5,000 keys:

```bash
pip install tqdm
```

In script:

```python
from tqdm import tqdm
```

Wrap loop like:

```python
for key in tqdm(keys_to_backup, desc=f"Scanning DB {db_index}"):
```

Every 5,000 keys, write partial output:

```python
if len(backup_data) % 5000 == 0:
    with open(output_file, "w") as f:
        json.dump(backup_data, f, indent=2)
```

#### ⚡ **Option C – Multi-threaded Fetching**

Split `keys_to_backup` into chunks (e.g., 10 threads × 3,000 keys) and run each in a thread with its own connection to ElastiCache.

Let me know if you want me to write that **multithreaded version**.

---

### ✅ Tips for Your Use Case

* If you're doing this from an **EC2 instance in same VPC**, it will be much faster than running from outside AWS.
* Prefer **using DB 0 only** for ElastiCache unless you've customized DB index.
* If your key data is large (e.g., hash/zset with 10,000 items), it might hit memory or performance thresholds — best to test on 1,000 keys first.

---

### ✅ Summary

| ✅ Works for 30k keys? | ✔ Yes                               |
| --------------------- | ----------------------------------- |
| 🐢 May be slow?       | ✔ Yes                               |
| 🧠 High memory usage? | ✔ Possible, depends on key sizes    |
| 🚀 Improvements?      | ✔ Threading or batching recommended |

---

Exploreable Options:

* Add progress bar and partial save?
* Convert to multi-threaded version for faster fetching?
* Add command-line options for easier reuse?

  
</details>

---

# To restore specific keys which is more than 20,000 chunk by chunk

| Total Keys | Chunk Size | Output Files & Keys Count     |
| ---------- | ---------- | ----------------------------- |
| 18,000     | 6,000      | 3 files: 6k + 6k + 6k         |
| 18,001     | 6,000      | 4 files: 6k + 6k + 6k + 1     |
| 19,999     | 6,000      | 4 files: 6k + 6k + 6k + 1,999 |
| 5,000      | 6,000      | 1 file: 5k                    |

<details>
  <summary>Click to view the steps</summary>
  
```
import redis
import json
import base64
import math

# Safe decoder to handle non-UTF8 data
def safe_decode(value):
    if isinstance(value, str):
        return value
    try:
        return value.decode("utf-8")
    except Exception:
        return base64.b64encode(value).decode("ascii")

# Read keys from file
with open("keys.txt") as f:
    keys_to_backup = [line.strip() for line in f if line.strip()]

backup_data = {}

# Try DBs 0–15
for db_index in range(16):
    r = redis.StrictRedis(host='localhost', port=6379, db=db_index)

    for key in keys_to_backup:
        key_bytes = key.encode("utf-8") if isinstance(key, str) else key

        if not r.exists(key_bytes):
            continue

        try:
            key_type = r.type(key_bytes).decode()
        except Exception:
            print(f"⚠️ Error detecting type for key: {key}")
            continue

        key_decoded = safe_decode(key_bytes)

        try:
            if key_type == "string":
                value = safe_decode(r.get(key_bytes))

            elif key_type == "hash":
                raw = r.hgetall(key_bytes)
                value = {safe_decode(k): safe_decode(v) for k, v in raw.items()}

            elif key_type == "set":
                value = [safe_decode(v) for v in r.smembers(key_bytes)]

            elif key_type == "zset":
                value = [(safe_decode(v), score) for v, score in r.zrange(key_bytes, 0, -1, withscores=True)]

            elif key_type == "list":
                value = [safe_decode(v) for v in r.lrange(key_bytes, 0, -1)]

            else:
                print(f"⚠️ Unknown type for key: {key} (Type: {key_type})")
                continue

            backup_data[key_decoded] = {
                "type": key_type,
                "value": value,
                "db": db_index
            }

        except Exception as e:
            print(f"❌ Error processing key: {key} — {e}")

# Save in chunks of 6000 keys
chunk_size = 6000
keys_list = list(backup_data.items())
total_chunks = math.ceil(len(keys_list) / chunk_size)

for i in range(total_chunks):
    chunk_dict = dict(keys_list[i * chunk_size:(i + 1) * chunk_size])
    filename = f"redis_backup_part_{i+1}.json"
    with open(filename, "w") as f:
        json.dump(chunk_dict, f, indent=2)
    print(f"💾 Saved {len(chunk_dict)} keys to {filename}")

print(f"\n✅ Backup complete. Total keys saved: {len(backup_data)} across {total_chunks} files.")
```
  
</details>

---

# To restore specific keys which is more than 20,000 chunk by chunk from particular db index in Elasticache

<details>
  <summary>Click to view step by step</summary>

### Backup Script

```
import redis
import json
import base64
import math

# ===== Config =====
DB_INDEX = 5               # Change this to the DB you want
CHUNK_SIZE = 6000          # Keys per backup file
REDIS_HOST = "localhost"   # Change if remote
REDIS_PORT = 6379
KEYS_FILE = "keys.txt"     # File containing keys to backup
# ==================

# Safe decoder to handle non-UTF8 data
def safe_decode(value):
    if isinstance(value, str):
        return value
    try:
        return value.decode("utf-8")
    except Exception:
        return base64.b64encode(value).decode("ascii")

# Read keys from file
with open(KEYS_FILE) as f:
    keys_to_backup = [line.strip() for line in f if line.strip()]

backup_data = {}

# Connect only to the specified DB
r = redis.StrictRedis(host=REDIS_HOST, port=REDIS_PORT, db=DB_INDEX)

for key in keys_to_backup:
    key_bytes = key.encode("utf-8") if isinstance(key, str) else key

    if not r.exists(key_bytes):
        continue

    try:
        key_type = r.type(key_bytes).decode()
    except Exception:
        print(f"⚠️ Error detecting type for key: {key}")
        continue

    key_decoded = safe_decode(key_bytes)

    try:
        if key_type == "string":
            value = safe_decode(r.get(key_bytes))

        elif key_type == "hash":
            raw = r.hgetall(key_bytes)
            value = {safe_decode(k): safe_decode(v) for k, v in raw.items()}

        elif key_type == "set":
            value = [safe_decode(v) for v in r.smembers(key_bytes)]

        elif key_type == "zset":
            value = [(safe_decode(v), score) for v, score in r.zrange(key_bytes, 0, -1, withscores=True)]

        elif key_type == "list":
            value = [safe_decode(v) for v in r.lrange(key_bytes, 0, -1)]

        else:
            print(f"⚠️ Unknown type for key: {key} (Type: {key_type})")
            continue

        backup_data[key_decoded] = {
            "type": key_type,
            "value": value,
            "db": DB_INDEX
        }

    except Exception as e:
        print(f"❌ Error processing key: {key} — {e}")

# Save in chunks
keys_list = list(backup_data.items())
total_chunks = math.ceil(len(keys_list) / CHUNK_SIZE)

for i in range(total_chunks):
    chunk_dict = dict(keys_list[i * CHUNK_SIZE:(i + 1) * CHUNK_SIZE])
    filename = f"redis_backup_db{DB_INDEX}_part_{i+1}.json"
    with open(filename, "w") as f:
        json.dump(chunk_dict, f, indent=2)
    print(f"💾 Saved {len(chunk_dict)} keys to {filename}")

print(f"\n✅ Backup complete for DB {DB_INDEX}. Total keys: {len(backup_data)} across {total_chunks} files.")

```


### Restore Script

```
import redis
import json
import os
import base64

# ===== Config =====
DB_INDEX = 5               # Target DB index for restore
REDIS_HOST = "localhost"   # Change if remote
REDIS_PORT = 6379
BACKUP_DIR = "."           # Directory containing chunk JSON files
# ==================

# Safe encoder to handle base64-decoded values
def safe_encode(value):
    if isinstance(value, str):
        return value.encode("utf-8")
    try:
        return value
    except Exception:
        return base64.b64decode(value.encode("ascii"))

# Safe decoder for base64-encoded content
def safe_decode_str(value):
    try:
        return base64.b64decode(value).decode("utf-8")
    except Exception:
        return value

# Connect to Redis
r = redis.StrictRedis(host=REDIS_HOST, port=REDIS_PORT, db=DB_INDEX)

# Detect all chunk files for this DB
chunk_files = sorted(
    [f for f in os.listdir(BACKUP_DIR) if f.startswith(f"redis_backup_db{DB_INDEX}_part_") and f.endswith(".json")]
)

if not chunk_files:
    print(f"❌ No chunk files found for DB {DB_INDEX} in {BACKUP_DIR}")
    exit(1)

print(f"🔄 Restoring DB {DB_INDEX} from {len(chunk_files)} chunk files...")

total_keys = 0

for file in chunk_files:
    filepath = os.path.join(BACKUP_DIR, file)
    with open(filepath, "r") as f:
        chunk_data = json.load(f)

    for key, data in chunk_data.items():
        key_bytes = safe_encode(key)
        key_type = data["type"]
        value = data["value"]

        try:
            if key_type == "string":
                r.set(key_bytes, safe_encode(value))

            elif key_type == "hash":
                r.hset(key_bytes, mapping={safe_encode(k): safe_encode(v) for k, v in value.items()})

            elif key_type == "set":
                r.sadd(key_bytes, *[safe_encode(v) for v in value])

            elif key_type == "zset":
                r.zadd(key_bytes, {safe_encode(v): score for v, score in value})

            elif key_type == "list":
                r.rpush(key_bytes, *[safe_encode(v) for v in value])

            else:
                print(f"⚠️ Unknown type for key {key} — skipped")

            total_keys += 1

        except Exception as e:
            print(f"❌ Error restoring key {key}: {e}")

    print(f"✅ Restored {len(chunk_data)} keys from {file}")

print(f"\n🎯 Restore complete for DB {DB_INDEX}. Total keys restored: {total_keys}")
  
```
  
</details>

---

# To restore all keys which is more than 20,000 chunk by chunk from particular db index in Elasticache

<details>
  <summary>Click to view step by step</summary>

### Complete Robust Backup script
```python
#!/usr/bin/env python3
"""
robust_chunk_redis_backup.py

Features:
- Binary-safe: distinguishes utf-8 strings vs raw bytes (base64).
- Memory-safe: scans and iterates large structures (HSCAN/SSCAN/ZSCAN/XRANGE with paging).
- Resilient: retries on transient Redis/network errors.
- Atomic chunk writes: write to temp file then atomically replace.
- Supports: string, hash, set, zset, list, stream
- Produces explicit wrappers for every stored value so restore can reconstruct bytes exactly.

Output JSON schema (per key):
{
  "<key>": {
    "type": "<redis-type>",
    "db": <db_index>,
    "value": <structure depends on type; elements are wrapper objects>
  }
}

Wrapper object (for any value element / field):
{"_t": "str", "v": "<utf8 string>"}
or
{"_t": "b64", "v": "<base64-encoded bytes>"}

"""

import redis
import json
import os
import base64
import math
import time
import errno
from functools import wraps

# ===== Config =====
DB_INDEX = 5
CHUNK_SIZE = 5000               # keys per file
REDIS_HOST = "your-elasticache-endpoint.amazonaws.com"
REDIS_PORT = 6379
OUTPUT_DIR = "backup_output"
KEY_PATTERN = "*"               # pattern to scan
SCAN_COUNT = 1000               # SCAN batch size
RETRY_ATTEMPTS = 3
RETRY_DELAY = 1.0               # seconds between retries
# ==================

os.makedirs(OUTPUT_DIR, exist_ok=True)


def retry(attempts=RETRY_ATTEMPTS, delay=RETRY_DELAY, exceptions=(Exception,)):
    """Simple retry decorator for transient errors."""
    def deco(f):
        @wraps(f)
        def wrapped(*a, **kw):
            last_exc = None
            for i in range(attempts):
                try:
                    return f(*a, **kw)
                except exceptions as e:
                    last_exc = e
                    time.sleep(delay)
            raise last_exc
        return wrapped
    return deco


def wrap_bytes_or_str(b: bytes):
    """Return wrapper dict describing whether the input is UTF-8 string or base64 bytes."""
    if b is None:
        return {"_t": "str", "v": ""}
    if isinstance(b, str):
        return {"_t": "str", "v": b}
    try:
        s = b.decode("utf-8")
        return {"_t": "str", "v": s}
    except Exception:
        b64 = base64.b64encode(b).decode("ascii")
        return {"_t": "b64", "v": b64}


@retry()
def safe_type(r, key): return r.type(key)

@retry()
def safe_get(r, key): return r.get(key)

@retry()
def safe_hscan(r, key, cursor, count): return r.hscan(key, cursor=cursor, count=count)

@retry()
def safe_sscan(r, key, cursor, count): return r.sscan(key, cursor=cursor, count=count)

@retry()
def safe_zscan(r, key, cursor, count): return r.zscan(key, cursor=cursor, count=count)

@retry()
def safe_xrange(r, key, start_id='-', end_id='+', count=1000):
    return r.xrange(key, min=start_id, max=end_id, count=count)

@retry()
def safe_lrange(r, key, start, end): return r.lrange(key, start, end)

@retry()
def safe_zrange_withscores(r, key, start, end, withscores=True):
    return r.zrange(key, start, end, withscores=withscores)

@retry()
def safe_smembers(r, key): return r.smembers(key)

@retry()
def safe_hgetall(r, key): return r.hgetall(key)


def atomic_write_json(obj, final_path):
    """Write JSON atomically: write to temp file then os.replace."""
    tmp_path = final_path + ".tmp"
    with open(tmp_path, "w") as fh:
        json.dump(obj, fh, indent=2)
        fh.flush()
        os.fsync(fh.fileno())
    os.replace(tmp_path, final_path)


def collect_hash(r, key):
    """Iterate hash with HSCAN."""
    cursor = 0
    result = {}
    while True:
        cursor, data = safe_hscan(r, key, cursor, SCAN_COUNT)
        for field, val in data.items():
            fw = wrap_bytes_or_str(field)
            field_key = fw['v'] if fw['_t'] == 'str' else "__b64_field__" + fw['v']
            result[field_key] = wrap_bytes_or_str(val)
        if cursor == 0:
            break
    return result


def collect_set(r, key):
    """Iterate set with SSCAN."""
    cursor = 0
    members = []
    while True:
        cursor, data = safe_sscan(r, key, cursor, SCAN_COUNT)
        for m in data:
            members.append(wrap_bytes_or_str(m))
        if cursor == 0:
            break
    return members


def collect_zset(r, key):
    """Iterate zset with ZSCAN."""
    cursor = 0
    items = []
    while True:
        cursor, data = safe_zscan(r, key, cursor, SCAN_COUNT)
        for member, score in data:
            items.append([wrap_bytes_or_str(member), score])
        if cursor == 0:
            break
    return items


def collect_list(r, key):
    """Read list in batches."""
    length = r.llen(key)
    items = []
    BATCH = 1000
    for start in range(0, length, BATCH):
        end = start + BATCH - 1
        chunk = safe_lrange(r, key, start, end)
        for e in chunk:
            items.append(wrap_bytes_or_str(e))
    return items


def collect_string(r, key):
    """Read a simple string key."""
    v = safe_get(r, key)
    return wrap_bytes_or_str(v)


def collect_stream_strict_paginated(r, key, page_size=1000):
    """
    Collect stream entries using strict pagination (no duplicates).
    Uses XRANGE in chunks until all entries are retrieved.
    """
    entries = []
    last_id = '-'
    while True:
        batch = safe_xrange(r, key, start_id=last_id, end_id='+', count=page_size)
        if not batch:
            break
        if last_id != '-' and batch[0][0].decode() == last_id:
            batch = batch[1:]
        if not batch:
            break
        for entry_id, fields in batch:
            entry_id_str = entry_id.decode() if isinstance(entry_id, bytes) else str(entry_id)
            field_map = {}
            for f_k, f_v in fields.items():
                fk_wrapper = wrap_bytes_or_str(f_k)
                fk_name = fk_wrapper['v'] if fk_wrapper['_t'] == 'str' else "__b64_field__" + fk_wrapper['v']
                field_map[fk_name] = wrap_bytes_or_str(f_v)
            entries.append([entry_id_str, field_map])
        last_id = batch[-1][0].decode() if isinstance(batch[-1][0], bytes) else str(batch[-1][0])
        last_parts = last_id.split('-')
        last_ms = int(last_parts[0])
        last_seq = int(last_parts[1]) if len(last_parts) > 1 else 0
        last_id = f"{last_ms}-{last_seq + 1}"
        if len(batch) < page_size:
            break
    return entries


def backup_db():
    r = redis.StrictRedis(host=REDIS_HOST, port=REDIS_PORT, db=DB_INDEX, socket_timeout=5)

    print(f"🔍 Scanning keys from DB {DB_INDEX}...")
    cursor = 0
    keys_to_backup = []
    while True:
        cursor, keys = r.scan(cursor=cursor, match=KEY_PATTERN, count=SCAN_COUNT)
        keys_to_backup.extend(keys)
        if cursor == 0:
            break

    total_keys_found = len(keys_to_backup)
    print(f"📦 Found {total_keys_found} keys to back up.")

    backup_data = {}
    saved_files = 0
    processed = 0
    start_time = time.time()

    for idx, key_bytes in enumerate(keys_to_backup, start=1):
        processed += 1
        try:
            if not r.exists(key_bytes):
                continue
            key_type = safe_type(r, key_bytes).decode()
        except Exception as e:
            print(f"⚠️ Skipping key: {key_bytes} — {e}")
            continue

        try:
            key_str = key_bytes.decode("utf-8")
            json_key = key_str
        except Exception:
            json_key = "__b64_key__" + base64.b64encode(key_bytes).decode("ascii")

        try:
            if key_type == "string":
                value = collect_string(r, key_bytes)
            elif key_type == "hash":
                value = collect_hash(r, key_bytes)
            elif key_type == "set":
                value = collect_set(r, key_bytes)
            elif key_type == "zset":
                value = collect_zset(r, key_bytes)
            elif key_type == "list":
                value = collect_list(r, key_bytes)
            elif key_type == "stream":
                value = collect_stream_strict_paginated(r, key_bytes, page_size=1000)
            else:
                print(f"⚠️ Unsupported type: {key_type}; skipping {json_key}")
                continue

            backup_data[json_key] = {"type": key_type, "db": DB_INDEX, "value": value}

        except Exception as e:
            print(f"❌ Error reading key {json_key}: {e}")

        if len(backup_data) >= CHUNK_SIZE or idx == total_keys_found:
            part_num = saved_files + 1
            filename = os.path.join(OUTPUT_DIR, f"redis_backup_db{DB_INDEX}_part_{part_num}.json")
            meta = {
                "generated_at": time.strftime("%Y-%m-%dT%H:%M:%SZ", time.gmtime()),
                "db": DB_INDEX,
                "keys_in_chunk": len(backup_data),
                "chunk_number": part_num,
                "total_keys_discovered": total_keys_found
            }
            out_obj = {"_meta": meta, "data": backup_data}
            try:
                atomic_write_json(out_obj, filename)
                print(f"💾 Saved {len(backup_data)} keys to {filename}")
            except Exception as e:
                print(f"❌ Failed to write {filename}: {e}")
            backup_data.clear()
            saved_files += 1

    elapsed = time.time() - start_time
    print(f"\n✅ Backup complete for DB {DB_INDEX}. Keys: {total_keys_found}, Chunks: {saved_files}, Time: {elapsed:.1f}s")


if __name__ == "__main__":
    backup_db()

```

### Restore Script
```python
#!/usr/bin/env python3
"""
robust_chunk_redis_restore.py

- Reads chunked JSON files from the robust backup script
- Restores Redis keys and values exactly as backed up, including binary keys/fields
- Handles types: string, hash, set, zset, list, stream
- Uses pipelining and retries for efficiency and resilience

Usage:
  python3 robust_chunk_redis_restore.py
"""

import redis
import json
import os
import base64
import time
from functools import wraps

# ===== Config =====
DB_INDEX = 5
REDIS_HOST = "your-elasticache-endpoint.amazonaws.com"
REDIS_PORT = 6379
BACKUP_DIR = "backup_output"
CHUNK_FILE_PREFIX = f"redis_backup_db{DB_INDEX}_part_"
PIPELINE_BATCH = 1000
RETRY_ATTEMPTS = 3
RETRY_DELAY = 1.0
# ==================

# --- Retry decorator ---
def retry(attempts=RETRY_ATTEMPTS, delay=RETRY_DELAY, exceptions=(Exception,)):
    def deco(f):
        @wraps(f)
        def wrapped(*a, **kw):
            last_exc = None
            for i in range(attempts):
                try:
                    return f(*a, **kw)
                except exceptions as e:
                    last_exc = e
                    print(f"⚠️ Retry {i+1}/{attempts} after error: {e}")
                    time.sleep(delay)
            raise last_exc
        return wrapped
    return deco

# --- Decode helpers ---
def decode_wrapper(w):
    """
    Convert wrapper object from backup into bytes (for b64) or str.
    """
    if not isinstance(w, dict) or "_t" not in w:
        return w
    if w["_t"] == "str":
        return w.get("v", "")
    elif w["_t"] == "b64":
        return base64.b64decode(w.get("v", ""))
    return w.get("v")

def decode_keyname(json_key):
    """
    Decode a backup key name back to str or bytes.
    """
    if json_key.startswith("__b64_key__"):
        return base64.b64decode(json_key[len("__b64_key__"):])
    return json_key

def decode_fieldname(json_field):
    """
    Decode hash/stream field names from backup format.
    """
    if json_field.startswith("__b64_field__"):
        return base64.b64decode(json_field[len("__b64_field__"):])
    return json_field

@retry()
def safe_execute_pipeline(p):
    return p.execute()

# --- Restore logic ---
def restore_chunk_file(r, filepath):
    with open(filepath, "r") as fh:
        obj = json.load(fh)

    data = obj.get("data", {})
    meta = obj.get("_meta", {})

    print(f"🔁 Restoring chunk {os.path.basename(filepath)} — keys: {len(data)} (meta: {meta})")

    pipe = r.pipeline(transaction=False)
    ops_in_pipeline = 0
    restored_keys = 0

    def flush_pipe(force=False):
        nonlocal ops_in_pipeline, pipe
        if ops_in_pipeline >= PIPELINE_BATCH or force:
            safe_execute_pipeline(pipe)
            pipe = r.pipeline(transaction=False)
            ops_in_pipeline = 0

    for json_key, kdata in data.items():
        restored_keys += 1
        key = decode_keyname(json_key)
        ktype = kdata.get("type")
        val = kdata.get("value")

        try:
            if ktype == "string":
                pipe.set(key, decode_wrapper(val))
                ops_in_pipeline += 1

            elif ktype == "hash":
                mapping = {decode_fieldname(f): decode_wrapper(v) for f, v in val.items()}
                if mapping:
                    pipe.hset(key, mapping=mapping)
                    ops_in_pipeline += 1

            elif ktype == "set":
                members = [decode_wrapper(w) for w in val]
                if members:
                    pipe.sadd(key, *members)
                    ops_in_pipeline += 1

            elif ktype == "zset":
                zmap = {decode_wrapper(m): score for m, score in val}
                if zmap:
                    pipe.zadd(key, zmap)
                    ops_in_pipeline += 1

            elif ktype == "list":
                items = [decode_wrapper(w) for w in val]
                if items:
                    pipe.rpush(key, *items)
                    ops_in_pipeline += 1

            elif ktype == "stream":
                for entry_id, fields in val:
                    fm = {decode_fieldname(f): decode_wrapper(v) for f, v in fields.items()}
                    pipe.xadd(key, fields=fm, id=entry_id)
                    ops_in_pipeline += 1
                    if ops_in_pipeline >= PIPELINE_BATCH:
                        flush_pipe()

            else:
                print(f"⚠️ Unknown/unsupported type '{ktype}' for key {json_key}")

        except Exception as e:
            print(f"❌ Error restoring key {json_key}: {e}")

        if ops_in_pipeline >= PIPELINE_BATCH:
            flush_pipe()

    flush_pipe(force=True)
    print(f"✅ Restored chunk {os.path.basename(filepath)} (keys processed: {restored_keys})")

def main():
    r = redis.StrictRedis(host=REDIS_HOST, port=REDIS_PORT, db=DB_INDEX, socket_timeout=5)

    chunk_files = sorted([
        os.path.join(BACKUP_DIR, f)
        for f in os.listdir(BACKUP_DIR)
        if f.startswith(CHUNK_FILE_PREFIX) and f.endswith(".json")
    ])

    if not chunk_files:
        print(f"❌ No chunk files found in {BACKUP_DIR} with prefix {CHUNK_FILE_PREFIX}")
        return

    print(f"🔎 Found {len(chunk_files)} chunk files; restoring into DB {DB_INDEX}")

    start = time.time()
    for cf in chunk_files:
        try:
            restore_chunk_file(r, cf)
        except Exception as e:
            print(f"❌ Failed to restore chunk {cf}: {e}")
    elapsed = time.time() - start
    print(f"\n🎯 Restore complete in {elapsed:.1f}s")

if __name__ == "__main__":
    main()
```
  
</details>

---


# Restore only particular .json file from the backup if it exists it will skip and if it doesnt exist it will over

<details>
  <summary>To restore the scripts</summary>

```
#!/usr/bin/env python3
"""
robust_single_file_redis_restore.py

- Restores ONLY the specified backup JSON file into Redis.
- Skips keys if they already exist with the same value.
- Overwrites if the key exists but value is missing/empty.
- Handles types: string, hash, set, zset, list, stream
- Uses pipelining and retries for efficiency.
"""

import redis
import json
import os
import base64
import time
from functools import wraps

# ===== Config =====
DB_INDEX = 5
REDIS_HOST = "your-elasticache-endpoint.amazonaws.com"
REDIS_PORT = 6379
BACKUP_FILE = "redis_backup_db5_chunk_42.json"
BACKUP_DIR = "backup_output"
PIPELINE_BATCH = 1000
RETRY_ATTEMPTS = 3
RETRY_DELAY = 1.0
# ==================

# --- Retry decorator ---
def retry(attempts=RETRY_ATTEMPTS, delay=RETRY_DELAY, exceptions=(Exception,)):
    def deco(f):
        @wraps(f)
        def wrapped(*a, **kw):
            last_exc = None
            for i in range(attempts):
                try:
                    return f(*a, **kw)
                except exceptions as e:
                    last_exc = e
                    print(f"⚠️ Retry {i+1}/{attempts} after error: {e}")
                    time.sleep(delay)
            raise last_exc
        return wrapped
    return deco

# --- Decode helpers ---
def decode_wrapper(w):
    if not isinstance(w, dict) or "_t" not in w:
        return w
    if w["_t"] == "str":
        return w.get("v", "")
    elif w["_t"] == "b64":
        return base64.b64decode(w.get("v", ""))
    return w.get("v")

def decode_keyname(json_key):
    if json_key.startswith("__b64_key__"):
        return base64.b64decode(json_key[len("__b64_key__"):])
    return json_key

def decode_fieldname(json_field):
    if json_field.startswith("__b64_field__"):
        return base64.b64decode(json_field[len("__b64_field__"):])
    return json_field

@retry()
def safe_execute_pipeline(p):
    return p.execute()

# --- Check if key already has value ---
def key_has_value(r, key, ktype, val):
    try:
        if not r.exists(key):
            return False  # key missing → overwrite
        if ktype == "string":
            return r.get(key) == decode_wrapper(val)
        elif ktype == "hash":
            current = r.hgetall(key)
            expected = {decode_fieldname(f).encode() if isinstance(decode_fieldname(f), str) else decode_fieldname(f): 
                        decode_wrapper(v) if isinstance(decode_wrapper(v), bytes) else str(decode_wrapper(v)).encode()
                        for f, v in val.items()}
            return current == expected
        elif ktype == "set":
            return r.smembers(key) == set(decode_wrapper(w) if isinstance(decode_wrapper(w), bytes) else str(decode_wrapper(w)).encode() for w in val)
        elif ktype == "zset":
            return dict(r.zrange(key, 0, -1, withscores=True)) == {decode_wrapper(m) if isinstance(decode_wrapper(m), bytes) else str(decode_wrapper(m)).encode(): score for m, score in val}
        elif ktype == "list":
            return r.lrange(key, 0, -1) == [decode_wrapper(w) if isinstance(decode_wrapper(w), bytes) else str(decode_wrapper(w)).encode() for w in val]
        elif ktype == "stream":
            # Streams comparison is more complex; just overwrite if exists but empty
            return r.xlen(key) > 0
    except Exception:
        return False
    return False

# --- Restore logic ---
def restore_single_file(r, filepath):
    with open(filepath, "r") as fh:
        obj = json.load(fh)

    data = obj.get("data", {})
    meta = obj.get("_meta", {})

    print(f"🔁 Restoring file {os.path.basename(filepath)} — keys: {len(data)} (meta: {meta})")

    pipe = r.pipeline(transaction=False)
    ops_in_pipeline = 0
    restored_keys = 0

    def flush_pipe(force=False):
        nonlocal ops_in_pipeline, pipe
        if ops_in_pipeline >= PIPELINE_BATCH or force:
            safe_execute_pipeline(pipe)
            pipe = r.pipeline(transaction=False)
            ops_in_pipeline = 0

    for json_key, kdata in data.items():
        key = decode_keyname(json_key)
        ktype = kdata.get("type")
        val = kdata.get("value")

        # Skip if already has correct value
        if key_has_value(r, key, ktype, val):
            print(f"⏩ Skipped existing key {json_key}")
            continue

        restored_keys += 1
        try:
            if ktype == "string":
                pipe.set(key, decode_wrapper(val))
            elif ktype == "hash":
                mapping = {decode_fieldname(f): decode_wrapper(v) for f, v in val.items()}
                if mapping:
                    pipe.hset(key, mapping=mapping)
            elif ktype == "set":
                members = [decode_wrapper(w) for w in val]
                if members:
                    pipe.sadd(key, *members)
            elif ktype == "zset":
                zmap = {decode_wrapper(m): score for m, score in val}
                if zmap:
                    pipe.zadd(key, zmap)
            elif ktype == "list":
                items = [decode_wrapper(w) for w in val]
                if items:
                    pipe.rpush(key, *items)
            elif ktype == "stream":
                for entry_id, fields in val:
                    fm = {decode_fieldname(f): decode_wrapper(v) for f, v in fields.items()}
                    pipe.xadd(key, fields=fm, id=entry_id)
            else:
                print(f"⚠️ Unknown/unsupported type '{ktype}' for key {json_key}")
            ops_in_pipeline += 1
        except Exception as e:
            print(f"❌ Error restoring key {json_key}: {e}")

        if ops_in_pipeline >= PIPELINE_BATCH:
            flush_pipe()

    flush_pipe(force=True)
    print(f"✅ Restore complete for {os.path.basename(filepath)} — keys processed: {restored_keys}")

def main():
    r = redis.StrictRedis(host=REDIS_HOST, port=REDIS_PORT, db=DB_INDEX, socket_timeout=5)

    filepath = os.path.join(BACKUP_DIR, BACKUP_FILE)
    if not os.path.isfile(filepath):
        print(f"❌ File {filepath} not found.")
        return

    start = time.time()
    restore_single_file(r, filepath)
    elapsed = time.time() - start
    print(f"\n🎯 Restore finished in {elapsed:.1f}s")

if __name__ == "__main__":
    main()
```
  
</details>

---

# To backup and restore the specific keys using `keys.txt` from all the db index in elasticache

Features:

- Binary-safe: distinguishes UTF-8 strings vs raw bytes (base64)
- Memory-safe: SCAN/HSCAN/SSCAN/ZSCAN/XRANGE pagination
- Strict paginated XRANGE for streams (no duplicates)
- Retries on transient Redis errors
- Atomic chunk writes with metadata
  
<details>
  <summary>Click to view the Scripts</summary>

### backup Script
```py
#!/usr/bin/env python3
"""
robust_chunk_redis_backup_all_dbs.py

Backs up keys listed in KEYS_FILE from ALL Redis DBs in an ElastiCache endpoint.

Features:
- Binary-safe: distinguishes UTF-8 strings vs raw bytes (base64)
- Memory-safe: SCAN/HSCAN/SSCAN/ZSCAN/XRANGE pagination
- Strict paginated XRANGE for streams (no duplicates)
- Retries on transient Redis errors
- Atomic chunk writes with metadata
"""

import redis
import json
import base64
import os
import time
from functools import wraps
from redis.exceptions import ConnectionError, TimeoutError

# ===== Config =====
REDIS_HOST = "your-elasticache-endpoint.amazonaws.com"
REDIS_PORT = 6379
KEYS_FILE = "keys.txt"         # required: list of keys to back up (one per line)
OUTPUT_DIR = "backup_output"
CHUNK_SIZE = 2500              # keys per JSON chunk file
RETRY_LIMIT = 5
RETRY_DELAY = 1.0              # seconds
XRANGE_PAGE = 1000
# ==================

# --- Load target keys (must exist) ---
if not os.path.isfile(KEYS_FILE):
    raise SystemExit(f"❌ KEYS_FILE '{KEYS_FILE}' not found. Create it with one key per line.")

with open(KEYS_FILE, "r") as fh:
    TARGET_KEYS = set(k.strip() for k in fh if k.strip())

if not TARGET_KEYS:
    raise SystemExit(f"❌ KEYS_FILE '{KEYS_FILE}' is empty. Put keys (one per line) to back up.")

os.makedirs(OUTPUT_DIR, exist_ok=True)

# --- Retry decorator and retry helper ---
def retry(attempts=RETRY_LIMIT, delay=RETRY_DELAY, exceptions=(ConnectionError, TimeoutError)):
    def deco(f):
        @wraps(f)
        def wrapped(*a, **kw):
            last_exc = None
            for i in range(attempts):
                try:
                    return f(*a, **kw)
                except exceptions as e:
                    last_exc = e
                    print(f"⚠️ Redis transient error: {e} — retry {i+1}/{attempts} in {delay}s")
                    time.sleep(delay)
            raise last_exc
        return wrapped
    return deco

# --- Serialization helpers (match restore format) ---
def safe_serialize(b):
    """Return wrapper: {"_t":"str","v":...} or {"_t":"b64","v":...}"""
    if b is None:
        return {"_t": "str", "v": ""}
    if isinstance(b, str):
        return {"_t": "str", "v": b}
    # bytes
    try:
        s = b.decode("utf-8")
        return {"_t": "str", "v": s}
    except Exception:
        return {"_t": "b64", "v": base64.b64encode(b).decode("ascii")}

# --- Atomic writer ---
def atomic_write_json(obj, final_path):
    tmp = final_path + ".tmp"
    with open(tmp, "w", encoding="utf-8") as fh:
        json.dump(obj, fh, indent=2, ensure_ascii=False)
        fh.flush()
        os.fsync(fh.fileno())
    os.replace(tmp, final_path)

# --- Safe Redis wrappers ---
@retry()
def safe_scan(r, cursor, match=None, count=1000):
    return r.scan(cursor=cursor, match=match, count=count)

@retry()
def safe_type(r, key):
    return r.type(key)

@retry()
def safe_get(r, key):
    return r.get(key)

@retry()
def safe_lrange(r, key, start, end):
    return r.lrange(key, start, end)

@retry()
def safe_hscan(r, key, cursor, count):
    return r.hscan(key, cursor=cursor, count=count)

@retry()
def safe_sscan(r, key, cursor, count):
    return r.sscan(key, cursor=cursor, count=count)

@retry()
def safe_zscan(r, key, cursor, count):
    return r.zscan(key, cursor=cursor, count=count)

@retry()
def safe_zrange_withscores(r, key, start, end):
    return r.zrange(key, start, end, withscores=True)

@retry()
def safe_xrange(r, key, start_id='-', end_id='+', count=XRANGE_PAGE):
    # redis-py: r.xrange(name, min='-', max='+', count=None)
    return r.xrange(key, min=start_id, max=end_id, count=count)

@retry()
def safe_hgetall(r, key):
    return r.hgetall(key)

@retry()
def safe_smembers(r, key):
    return r.smembers(key)

@retry()
def safe_ttl(r, key):
    return r.ttl(key)

# --- Stream collector (strict pagination, no duplicates) ---
def collect_stream_strict(r, key, page_size=XRANGE_PAGE):
    entries = []
    # Start with '-' to indicate beginning; we'll use numeric ids later
    start = '-'
    while True:
        batch = safe_xrange(r, key, start_id=start, end_id='+', count=page_size)
        if not batch:
            break
        # If start != '-' and first entry equals previous start, skip it (safety)
        if start != '-' and batch and isinstance(batch[0][0], (bytes, bytearray)) and batch[0][0].decode() == start:
            batch = batch[1:]
        if not batch:
            break
        for entry_id, fields in batch:
            eid = entry_id.decode() if isinstance(entry_id, (bytes, bytearray)) else str(entry_id)
            fm = {}
            for fk, fv in fields.items():
                # field name -> use string name if utf8 else mark with __b64_field__
                try:
                    fk_s = fk.decode("utf-8")
                    fk_key = fk_s
                except Exception:
                    fk_key = "__b64_field__" + base64.b64encode(fk).decode("ascii")
                fm[fk_key] = safe_serialize(fv)
            entries.append([eid, fm])
        # prepare exclusive start: increment sequence part by 1
        last_raw = batch[-1][0]
        last_id = last_raw.decode() if isinstance(last_raw, (bytes, bytearray)) else str(last_raw)
        parts = last_id.split('-')
        try:
            ms = int(parts[0])
            seq = int(parts[1]) if len(parts) > 1 else 0
            start = f"{ms}-{seq + 1}"
        except Exception:
            # fallback: use last_id and skip duplicates above
            start = last_id
        if len(batch) < page_size:
            break
    return entries

# --- Key backup dispatcher ---
def backup_key(r, key):
    ktype_raw = safe_type(r, key)
    ktype = ktype_raw.decode() if isinstance(ktype_raw, (bytes, bytearray)) else str(ktype_raw)
    if ktype == 'string':
        return safe_serialize(safe_get(r, key))
    elif ktype == 'hash':
        out = {}
        cursor = 0
        while True:
            cursor, batch = safe_hscan(r, key, cursor, 1000)
            for f, v in batch.items():
                try:
                    f_s = f.decode('utf-8')
                    fk = f_s
                except Exception:
                    fk = "__b64_field__" + base64.b64encode(f).decode("ascii")
                out[fk] = safe_serialize(v)
            if cursor == 0:
                break
        return out
    elif ktype == 'set':
        members = []
        cursor = 0
        while True:
            cursor, batch = safe_sscan(r, key, cursor, 1000)
            for m in batch:
                members.append(safe_serialize(m))
            if cursor == 0:
                break
        return members
    elif ktype == 'zset':
        items = []
        cursor = 0
        while True:
            cursor, batch = safe_zscan(r, key, cursor, 1000)
            for member, score in batch:
                items.append([safe_serialize(member), score])
            if cursor == 0:
                break
        return items
    elif ktype == 'list':
        length = r.llen(key)
        items = []
        BATCH = 1000
        for start in range(0, length, BATCH):
            end = start + BATCH - 1
            chunk = safe_lrange(r, key, start, end)
            for e in chunk:
                items.append(safe_serialize(e))
        return items
    elif ktype == 'stream':
        return collect_stream_strict(r, key, page_size=XRANGE_PAGE)
    else:
        return None

# --- Save chunk with metadata ---
def save_chunk(chunk_obj, db_index, part_num):
    meta = {
        "generated_at": time.strftime("%Y-%m-%dT%H:%M:%SZ", time.gmtime()),
        "db": db_index,
        "chunk_number": part_num,
        "keys_in_chunk": len(chunk_obj)
    }
    out = {"_meta": meta, "data": chunk_obj}
    filename = os.path.join(OUTPUT_DIR, f"redis_backup_db{db_index}_part_{part_num}.json")
    atomic_write_json(out, filename)
    print(f"💾 Saved {len(chunk_obj)} keys to {filename}")

# --- Main: iterate all DBs and back up TARGET_KEYS ---
def backup_all_dbs():
    r = redis.StrictRedis(host=REDIS_HOST, port=REDIS_PORT, decode_responses=False)
    # Redis default max DB is 16 (0..15) but some servers have fewer; try sequentially and stop on ResponseError
    for db_index in range(0, 16):
        try:
            retry_cmd = retry()
            # select DB (use r.execute_command wrapped in retry)
            retry_cmd(r.execute_command)("SELECT", db_index)
        except Exception:
            # no more DBs
            break

        print(f"\n🔍 Scanning DB {db_index} ...")
        cursor = 0
        chunk = {}
        part = 1
        found = 0

        while True:
            cursor, keys = safe_scan(r, cursor, count=1000)
            for key in keys:
                # key is bytes
                try:
                    key_str = key.decode("utf-8")
                except Exception:
                    # if key name not utf8, use __b64_key__ prefix
                    key_str = "__b64_key__" + base64.b64encode(key).decode("ascii")

                if key_str not in TARGET_KEYS:
                    continue

                found += 1
                try:
                    val = backup_key(r, key)
                    if val is None:
                        print(f"⚠️ Unsupported/unknown type for key {key_str}; skipping")
                        continue
                    ttl = safe_ttl(r, key)
                    chunk[key_str] = {"type": safe_type(r, key).decode(), "db": db_index, "value": val, "ttl": ttl}
                except Exception as e:
                    print(f"❌ Error reading key {key_str}: {e}")

                # flush chunk
                if len(chunk) >= CHUNK_SIZE:
                    save_chunk(chunk, db_index, part)
                    chunk.clear()
                    part += 1

            if cursor == 0:
                break

        if chunk:
            save_chunk(chunk, db_index, part)

        print(f"✅ DB {db_index} done. Keys backed up from keys.txt: {found}")

    print("\n🎉 All DBs processed.")

if __name__ == "__main__":
    backup_all_dbs()

```

### Restore Script

```py
#!/usr/bin/env python3
"""
robust_restore_from_backups.py with --merge mode

- Default: delete existing keys before restoring (replace mode).
- With --merge: do not delete keys; add missing fields/members/entries only.
- Streams: in merge mode, add only entries with IDs not present.
- Uses same features as before.
"""

import os
import json
import base64
import time
import glob
import argparse
from functools import wraps
from redis import StrictRedis
from redis.exceptions import ConnectionError, TimeoutError, ResponseError

# ===== Config =====
REDIS_HOST = "your-elasticache-endpoint.amazonaws.com"
REDIS_PORT = 6379
OUTPUT_DIR = "backup_output"
RETRY_LIMIT = 5
RETRY_DELAY = 1.0
LIST_BATCH = 1000
PIPELINE_BATCH = 500
# ==================

if not os.path.isdir(OUTPUT_DIR):
    raise SystemExit(f"❌ OUTPUT_DIR '{OUTPUT_DIR}' not found.")

# --- Retry decorator ---
def retry(attempts=RETRY_LIMIT, delay=RETRY_DELAY, exceptions=(ConnectionError, TimeoutError)):
    def deco(f):
        @wraps(f)
        def wrapped(*a, **kw):
            last_exc = None
            for i in range(attempts):
                try:
                    return f(*a, **kw)
                except exceptions as e:
                    last_exc = e
                    print(f"⚠️ Transient Redis error: {e} — retry {i+1}/{attempts} in {delay}s")
                    time.sleep(delay)
            raise last_exc
        return wrapped
    return deco

# --- Decode helpers ---
def decode_value(vdict):
    if vdict is None:
        return b""
    t = vdict.get("_t")
    v = vdict.get("v", "")
    if t == "b64":
        return base64.b64decode(v)
    return v

def decode_key(key_str):
    prefix = "__b64_key__"
    if key_str.startswith(prefix):
        b64 = key_str[len(prefix):]
        return base64.b64decode(b64)
    return key_str

def decode_field_name(fk):
    prefix = "__b64_field__"
    if fk.startswith(prefix):
        b64 = fk[len(prefix):]
        return base64.b64decode(b64)
    return fk

# --- Safe Redis wrappers ---
@retry()
def safe_select(r, db):
    return r.execute_command("SELECT", db)

@retry()
def safe_set(r, key, value):
    return r.set(key, value)

@retry()
def safe_delete(r, key):
    return r.delete(key)

@retry()
def safe_expire(r, key, ttl):
    return r.expire(key, ttl)

@retry()
def safe_hset(r, key, mapping):
    return r.hset(key, mapping=mapping)

@retry()
def safe_hgetall(r, key):
    return r.hgetall(key)

@retry()
def safe_sadd(r, key, *members):
    return r.sadd(key, *members)

@retry()
def safe_smembers(r, key):
    return r.smembers(key)

@retry()
def safe_zadd(r, key, mapping):
    return r.zadd(key, mapping)

@retry()
def safe_zrange_withscores(r, key, start, end):
    return r.zrange(key, start, end, withscores=True)

@retry()
def safe_rpush(r, key, *values):
    return r.rpush(key, *values)

@retry()
def safe_llen(r, key):
    return r.llen(key)

@retry()
def safe_xadd(r, key, fields, id='*'):
    return r.xadd(key, fields=fields, id=id)

@retry()
def safe_xrange(r, key, start_id='-', end_id='+', count=1000):
    return r.xrange(key, min=start_id, max=end_id, count=count)

@retry()
def safe_exists(r, key):
    return r.exists(key)

# --- Helpers ---
def exists_stream_id(r, key, eid):
    # Check if stream entry with ID eid exists
    # XREVRANGE key id id COUNT 1 returns one entry if exists
    # Or XRANGE key id id COUNT 1
    res = safe_xrange(r, key, start_id=eid, end_id=eid, count=1)
    return len(res) > 0

# --- Restore functions ---

def restore_string(r, key, entry, merge):
    val = decode_value(entry["value"])
    if merge:
        # In merge mode, only set if key does not exist
        if not safe_exists(r, key):
            safe_set(r, key, val)
    else:
        safe_delete(r, key)
        safe_set(r, key, val)

def restore_hash(r, key, entry, merge):
    mapping = {}
    for fk, fv in entry["value"].items():
        field = decode_field_name(fk)
        val = decode_value(fv)
        mapping[field] = val
    if merge:
        # Add missing fields only
        existing = safe_hgetall(r, key)
        # existing keys/fields may be bytes, convert to set of bytes for faster lookup
        existing_fields = set(existing.keys()) if existing else set()
        to_add = {k: v for k, v in mapping.items() if k not in existing_fields}
        if to_add:
            safe_hset(r, key, to_add)
    else:
        safe_delete(r, key)
        safe_hset(r, key, mapping)

def restore_set(r, key, entry, merge):
    members = [decode_value(m) for m in entry["value"]]
    if merge:
        existing = safe_smembers(r, key)
        existing_set = set(existing) if existing else set()
        to_add = [m for m in members if m not in existing_set]
        if to_add:
            # chunk add
            for i in range(0, len(to_add), PIPELINE_BATCH):
                safe_sadd(r, key, *to_add[i:i+PIPELINE_BATCH])
    else:
        safe_delete(r, key)
        for i in range(0, len(members), PIPELINE_BATCH):
            safe_sadd(r, key, *members[i:i+PIPELINE_BATCH])

def restore_zset(r, key, entry, merge):
    mapping = {}
    for mem_ser, score in entry["value"]:
        member = decode_value(mem_ser)
        mapping[member] = score
    if merge:
        # get existing members to skip duplicates
        existing = safe_zrange_withscores(r, key, 0, -1)
        existing_members = set(mem for mem, _ in existing) if existing else set()
        to_add = {k: v for k, v in mapping.items() if k not in existing_members}
        if to_add:
            for i in range(0, len(to_add), PIPELINE_BATCH):
                chunk = dict(list(to_add.items())[i:i+PIPELINE_BATCH])
                safe_zadd(r, key, chunk)
    else:
        safe_delete(r, key)
        items = list(mapping.items())
        for i in range(0, len(items), PIPELINE_BATCH):
            chunk = dict(items[i:i+PIPELINE_BATCH])
            safe_zadd(r, key, chunk)

def restore_list(r, key, entry, merge):
    items = [decode_value(it) for it in entry["value"]]
    if merge:
        # append only items that don't exist
        existing_len = safe_llen(r, key)
        # check tail items, if identical we can skip? Since Redis lists have no direct contains()
        # We'll just RPUSH all for safety (merge mode means append)
        if items:
            safe_rpush(r, key, *items)
    else:
        safe_delete(r, key)
        for i in range(0, len(items), LIST_BATCH):
            safe_rpush(r, key, *items[i:i+LIST_BATCH])

def restore_stream(r, key, entry, merge):
    entries = entry["value"]
    if not entries:
        return
    if not merge:
        safe_delete(r, key)
    pipe = r.pipeline()
    count = 0
    for eid, fields in entries:
        fx = {}
        for fk, fv in fields.items():
            fname = decode_field_name(fk)
            fval = decode_value(fv)
            fx[fname] = fval
        if merge:
            # Only add if id doesn't exist
            if exists_stream_id(r, key, eid):
                continue
        try:
            pipe.xadd(key, fields=fx, id=eid)
            count += 1
        except ResponseError as re:
            if "ID already exists" in str(re) or "entry ID already exists" in str(re):
                pass
            else:
                print(f"❌ Error xadd {key} {eid}: {re}")
        if count >= PIPELINE_BATCH:
            try:
                pipe.execute()
            except Exception as e:
                print(f"❌ Pipeline error during stream restore: {e}")
            pipe = r.pipeline()
            count = 0
    if count > 0:
        try:
            pipe.execute()
        except Exception as e:
            print(f"❌ Final pipeline error during stream restore: {e}")

# --- Restore one chunk ---
def restore_chunk(r, chunk_obj, src_filename, merge):
    data = chunk_obj.get("data", {})
    meta = chunk_obj.get("_meta", {})
    total = len(data)
    print(f"🔁 Restoring {total} keys from {src_filename} (meta: {meta}), merge={merge}")
    current_db = None
    for k_str, entry in data.items():
        db_index = entry.get("db", 0)
        if current_db != db_index:
            safe_select(r, db_index)
            current_db = db_index
            print(f"➡️ Selected DB {db_index}")

        key = decode_key(k_str)
        ktype = entry.get("type")
        ttl = entry.get("ttl", -1)
        try:
            if ktype == "string":
                restore_string(r, key, entry, merge)
            elif ktype == "hash":
                restore_hash(r, key, entry, merge)
            elif ktype == "set":
                restore_set(r, key, entry, merge)
            elif ktype == "zset":
                restore_zset(r, key, entry, merge)
            elif ktype == "list":
                restore_list(r, key, entry, merge)
            elif ktype == "stream":
                restore_stream(r, key, entry, merge)
            else:
                print(f"⚠️ Unknown type '{ktype}' for key {k_str}; skipping")
                continue
            if not merge and isinstance(ttl, int) and ttl > 0:
                safe_expire(r, key, ttl)
            elif merge and ttl > 0:
                # In merge mode, update TTL only if key exists
                if safe_exists(r, key):
                    safe_expire(r, key, ttl)
        except Exception as e:
            print(f"❌ Error restoring key {k_str} (db {db_index}): {e}")

# --- Main ---
def restore_all_from_dir(merge=False):
    r = StrictRedis(host=REDIS_HOST, port=REDIS_PORT, decode_responses=False)
    files = sorted(glob.glob(os.path.join(OUTPUT_DIR, "redis_backup_db*_part_*.json")))
    if not files:
        print("⚠️ No backup files found.")
        return
    for fpath in files:
        print(f"\n📂 Loading {fpath} ...")
        try:
            with open(fpath, "r", encoding="utf-8") as fh:
                obj = json.load(fh)
        except Exception as e:
            print(f"❌ Failed to parse {fpath}: {e}")
            continue
        try:
            restore_chunk(r, obj, os.path.basename(fpath), merge)
            print(f"✅ Restored {fpath}")
        except Exception as e:
            print(f"❌ Error restoring from {fpath}: {e}")
    print("\n🎉 Restore complete.")

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Restore Redis backup JSON files.")
    parser.add_argument(
        "--merge", action="store_true",
        help="Merge mode: add missing keys/fields/members only, do NOT delete existing keys."
    )
    args = parser.parse_args()
    restore_all_from_dir(merge=args.merge)
```
  
</details>

---

# To backup and restore the specific keys using `keys.txt` from single db index in elasticache

Features:

- Binary-safe: distinguishes UTF-8 strings vs raw bytes (base64)
- Memory-safe: SCAN/HSCAN/SSCAN/ZSCAN/XRANGE pagination
- Strict paginated XRANGE for streams (no duplicates)
- Retries on transient Redis errors
- Atomic chunk writes with metadata

<details>
  <summary>Click to view the scripts</summary>

### Backup Script

```py
#!/usr/bin/env python3
"""
robust_chunk_redis_backup_single_db.py

Backs up keys listed in KEYS_FILE from a single Redis DB (DB_INDEX).

Features:
- Binary-safe (UTF-8 or base64)
- SCAN/HSCAN/SSCAN/ZSCAN/XRANGE pagination
- Strict paginated XRANGE for streams (no duplicates)
- Retries on transient errors
- Atomic chunk writes with metadata
"""

import redis
import json
import base64
import os
import time
from functools import wraps
from redis.exceptions import ConnectionError, TimeoutError

# ===== Config =====
REDIS_HOST = "your-elasticache-endpoint.amazonaws.com"
REDIS_PORT = 6379
DB_INDEX = 5                   # target DB
KEYS_FILE = "keys.txt"         # required: list of keys to back up
OUTPUT_DIR = "backup_output"
CHUNK_SIZE = 2500              # keys per JSON chunk
RETRY_LIMIT = 5
RETRY_DELAY = 1.0
XRANGE_PAGE = 1000
# ==================

# --- load keys ---
if not os.path.isfile(KEYS_FILE):
    raise SystemExit(f"❌ KEYS_FILE '{KEYS_FILE}' not found. Create it with one key per line.")

with open(KEYS_FILE, "r") as fh:
    TARGET_KEYS = set(k.strip() for k in fh if k.strip())

if not TARGET_KEYS:
    raise SystemExit(f"❌ KEYS_FILE '{KEYS_FILE}' is empty. Put keys (one per line) to back up.")

os.makedirs(OUTPUT_DIR, exist_ok=True)

# --- retry decorator ---
def retry(attempts=RETRY_LIMIT, delay=RETRY_DELAY, exceptions=(ConnectionError, TimeoutError)):
    def deco(f):
        @wraps(f)
        def wrapped(*a, **kw):
            last_exc = None
            for i in range(attempts):
                try:
                    return f(*a, **kw)
                except exceptions as e:
                    last_exc = e
                    print(f"⚠️ Redis transient error: {e} — retry {i+1}/{attempts} in {delay}s")
                    time.sleep(delay)
            raise last_exc
        return wrapped
    return deco

# --- Safe serialization & write helpers ---
def safe_serialize(b):
    if b is None:
        return {"_t": "str", "v": ""}
    if isinstance(b, str):
        return {"_t": "str", "v": b}
    try:
        s = b.decode("utf-8")
        return {"_t": "str", "v": s}
    except Exception:
        return {"_t": "b64", "v": base64.b64encode(b).decode("ascii")}

def atomic_write_json(obj, final_path):
    tmp = final_path + ".tmp"
    with open(tmp, "w", encoding="utf-8") as fh:
        json.dump(obj, fh, indent=2, ensure_ascii=False)
        fh.flush()
        os.fsync(fh.fileno())
    os.replace(tmp, final_path)

# --- safe redis wrappers ---
@retry()
def safe_scan(r, cursor, match=None, count=1000):
    return r.scan(cursor=cursor, match=match, count=count)

@retry()
def safe_type(r, key):
    return r.type(key)

@retry()
def safe_get(r, key):
    return r.get(key)

@retry()
def safe_lrange(r, key, start, end):
    return r.lrange(key, start, end)

@retry()
def safe_hscan(r, key, cursor, count):
    return r.hscan(key, cursor=cursor, count=count)

@retry()
def safe_sscan(r, key, cursor, count):
    return r.sscan(key, cursor=cursor, count=count)

@retry()
def safe_zscan(r, key, cursor, count):
    return r.zscan(key, cursor=cursor, count=count)

@retry()
def safe_zrange_withscores(r, key, start, end):
    return r.zrange(key, start, end, withscores=True)

@retry()
def safe_xrange(r, key, start_id='-', end_id='+', count=XRANGE_PAGE):
    return r.xrange(key, min=start_id, max=end_id, count=count)

@retry()
def safe_hgetall(r, key):
    return r.hgetall(key)

@retry()
def safe_smembers(r, key):
    return r.smembers(key)

@retry()
def safe_ttl(r, key):
    return r.ttl(key)

# --- stream collector ---
def collect_stream_strict(r, key, page_size=XRANGE_PAGE):
    entries = []
    start = '-'
    while True:
        batch = safe_xrange(r, key, start_id=start, end_id='+', count=page_size)
        if not batch:
            break
        if start != '-' and batch and isinstance(batch[0][0], (bytes, bytearray)) and batch[0][0].decode() == start:
            batch = batch[1:]
        if not batch:
            break
        for entry_id, fields in batch:
            eid = entry_id.decode() if isinstance(entry_id, (bytes, bytearray)) else str(entry_id)
            fm = {}
            for fk, fv in fields.items():
                try:
                    fk_s = fk.decode("utf-8")
                    fk_key = fk_s
                except Exception:
                    fk_key = "__b64_field__" + base64.b64encode(fk).decode("ascii")
                fm[fk_key] = safe_serialize(fv)
            entries.append([eid, fm])
        last_raw = batch[-1][0]
        last_id = last_raw.decode() if isinstance(last_raw, (bytes, bytearray)) else str(last_raw)
        parts = last_id.split('-')
        try:
            ms = int(parts[0])
            seq = int(parts[1]) if len(parts) > 1 else 0
            start = f"{ms}-{seq + 1}"
        except Exception:
            start = last_id
        if len(batch) < page_size:
            break
    return entries

# --- key backup dispatcher ---
def backup_key(r, key):
    ktype_raw = safe_type(r, key)
    ktype = ktype_raw.decode() if isinstance(ktype_raw, (bytes, bytearray)) else str(ktype_raw)
    if ktype == 'string':
        return safe_serialize(safe_get(r, key))
    elif ktype == 'hash':
        out = {}
        cursor = 0
        while True:
            cursor, batch = safe_hscan(r, key, cursor, 1000)
            for f, v in batch.items():
                try:
                    fk = f.decode('utf-8')
                except Exception:
                    fk = "__b64_field__" + base64.b64encode(f).decode("ascii")
                out[fk] = safe_serialize(v)
            if cursor == 0:
                break
        return out
    elif ktype == 'set':
        members = []
        cursor = 0
        while True:
            cursor, batch = safe_sscan(r, key, cursor, 1000)
            for m in batch:
                members.append(safe_serialize(m))
            if cursor == 0:
                break
        return members
    elif ktype == 'zset':
        items = []
        cursor = 0
        while True:
            cursor, batch = safe_zscan(r, key, cursor, 1000)
            for member, score in batch:
                items.append([safe_serialize(member), score])
            if cursor == 0:
                break
        return items
    elif ktype == 'list':
        length = r.llen(key)
        items = []
        BATCH = 1000
        for start in range(0, length, BATCH):
            end = start + BATCH - 1
            chunk = safe_lrange(r, key, start, end)
            for e in chunk:
                items.append(safe_serialize(e))
        return items
    elif ktype == 'stream':
        return collect_stream_strict(r, key, page_size=XRANGE_PAGE)
    else:
        return None

def save_chunk(chunk_obj, part_num):
    meta = {
        "generated_at": time.strftime("%Y-%m-%dT%H:%M:%SZ", time.gmtime()),
        "db": DB_INDEX,
        "chunk_number": part_num,
        "keys_in_chunk": len(chunk_obj)
    }
    out = {"_meta": meta, "data": chunk_obj}
    filename = os.path.join(OUTPUT_DIR, f"redis_backup_db{DB_INDEX}_part_{part_num}.json")
    atomic_write_json(out, filename)
    print(f"💾 Saved {len(chunk_obj)} keys to {filename}")

def backup_single_db():
    r = redis.StrictRedis(host=REDIS_HOST, port=REDIS_PORT, db=DB_INDEX, decode_responses=False)

    cursor = 0
    chunk = {}
    part = 1
    found = 0
    total_scanned = 0

    while True:
        cursor, keys = safe_scan(r, cursor, count=1000)
        total_scanned += len(keys)
        for key in keys:
            try:
                key_str = key.decode('utf-8')
            except Exception:
                key_str = "__b64_key__" + base64.b64encode(key).decode("ascii")

            if key_str not in TARGET_KEYS:
                continue

            found += 1
            try:
                val = backup_key(r, key)
                if val is None:
                    print(f"⚠️ Unsupported type for key {key_str}; skipping")
                    continue
                ttl = safe_ttl(r, key)
                chunk[key_str] = {"type": safe_type(r, key).decode(), "db": DB_INDEX, "value": val, "ttl": ttl}
            except Exception as e:
                print(f"❌ Error reading key {key_str}: {e}")

            if len(chunk) >= CHUNK_SIZE:
                save_chunk(chunk, part)
                chunk.clear()
                part += 1

        if cursor == 0:
            break

    if chunk:
        save_chunk(chunk, part)

    print(f"\n✅ DB {DB_INDEX} backup complete. Keys found in keys.txt: {found} (scanned {total_scanned} keys).")

if __name__ == "__main__":
    backup_single_db()

```

### Restore Script

```py
#!/usr/bin/env python3
"""
robust_restore_single_db.py

Restore Redis backup chunks for a single DB index.

Features:
- Restore to original DB index from backup metadata.
- Binary-safe (utf8/base64).
- SCAN/HSCAN/SSCAN/ZSCAN/XRANGE pagination safe.
- Strict paginated XRANGE for streams (no duplicates).
- Retry on transient Redis errors.
- Atomic operations.
- Merge mode is DEFAULT: skip deletes and only add missing keys/fields/members.
- Use --force to overwrite (delete and restore all keys).
"""

import os
import json
import base64
import time
import glob
import argparse
from functools import wraps
from redis import StrictRedis
from redis.exceptions import ConnectionError, TimeoutError, ResponseError

# ===== Config =====
REDIS_HOST = "your-elasticache-endpoint.amazonaws.com"
REDIS_PORT = 6379
OUTPUT_DIR = "backup_output"
RETRY_LIMIT = 5
RETRY_DELAY = 1.0
LIST_BATCH = 1000
PIPELINE_BATCH = 500
# ==================

if not os.path.isdir(OUTPUT_DIR):
    raise SystemExit(f"❌ OUTPUT_DIR '{OUTPUT_DIR}' not found.")

# --- Retry decorator ---
def retry(attempts=RETRY_LIMIT, delay=RETRY_DELAY, exceptions=(ConnectionError, TimeoutError)):
    def deco(f):
        @wraps(f)
        def wrapped(*a, **kw):
            last_exc = None
            for i in range(attempts):
                try:
                    return f(*a, **kw)
                except exceptions as e:
                    last_exc = e
                    print(f"⚠️ Transient Redis error: {e} — retry {i+1}/{attempts} in {delay}s")
                    time.sleep(delay)
            raise last_exc
        return wrapped
    return deco

# --- Decode helpers ---
def decode_value(vdict):
    if vdict is None:
        return b""
    t = vdict.get("_t")
    v = vdict.get("v", "")
    if t == "b64":
        return base64.b64decode(v)
    return v

def decode_key(key_str):
    prefix = "__b64_key__"
    if key_str.startswith(prefix):
        b64 = key_str[len(prefix):]
        return base64.b64decode(b64)
    return key_str

def decode_field_name(fk):
    prefix = "__b64_field__"
    if fk.startswith(prefix):
        b64 = fk[len(prefix):]
        return base64.b64decode(b64)
    return fk

# --- Safe Redis wrappers ---
@retry()
def safe_select(r, db):
    return r.execute_command("SELECT", db)

@retry()
def safe_set(r, key, value):
    return r.set(key, value)

@retry()
def safe_delete(r, key):
    return r.delete(key)

@retry()
def safe_expire(r, key, ttl):
    return r.expire(key, ttl)

@retry()
def safe_hset(r, key, mapping):
    return r.hset(key, mapping=mapping)

@retry()
def safe_hgetall(r, key):
    return r.hgetall(key)

@retry()
def safe_sadd(r, key, *members):
    return r.sadd(key, *members)

@retry()
def safe_smembers(r, key):
    return r.smembers(key)

@retry()
def safe_zadd(r, key, mapping):
    return r.zadd(key, mapping)

@retry()
def safe_zrange_withscores(r, key, start, end):
    return r.zrange(key, start, end, withscores=True)

@retry()
def safe_rpush(r, key, *values):
    return r.rpush(key, *values)

@retry()
def safe_llen(r, key):
    return r.llen(key)

@retry()
def safe_xadd(r, key, fields, id='*'):
    return r.xadd(key, fields=fields, id=id)

@retry()
def safe_xrange(r, key, start_id='-', end_id='+', count=1000):
    return r.xrange(key, min=start_id, max=end_id, count=count)

@retry()
def safe_exists(r, key):
    return r.exists(key)

# --- Helpers ---
def exists_stream_id(r, key, eid):
    res = safe_xrange(r, key, start_id=eid, end_id=eid, count=1)
    return len(res) > 0

# --- Restore functions ---
def restore_string(r, key, entry, merge):
    val = decode_value(entry["value"])
    if merge:
        if not safe_exists(r, key):
            safe_set(r, key, val)
    else:
        safe_delete(r, key)
        safe_set(r, key, val)

def restore_hash(r, key, entry, merge):
    mapping = {}
    for fk, fv in entry["value"].items():
        field = decode_field_name(fk)
        val = decode_value(fv)
        mapping[field] = val
    if merge:
        existing = safe_hgetall(r, key)
        existing_fields = set(existing.keys()) if existing else set()
        to_add = {k: v for k, v in mapping.items() if k not in existing_fields}
        if to_add:
            safe_hset(r, key, to_add)
    else:
        safe_delete(r, key)
        safe_hset(r, key, mapping)

def restore_set(r, key, entry, merge):
    members = [decode_value(m) for m in entry["value"]]
    if merge:
        existing = safe_smembers(r, key)
        existing_set = set(existing) if existing else set()
        to_add = [m for m in members if m not in existing_set]
        if to_add:
            for i in range(0, len(to_add), PIPELINE_BATCH):
                safe_sadd(r, key, *to_add[i:i+PIPELINE_BATCH])
    else:
        safe_delete(r, key)
        for i in range(0, len(members), PIPELINE_BATCH):
            safe_sadd(r, key, *members[i:i+PIPELINE_BATCH])

def restore_zset(r, key, entry, merge):
    mapping = {}
    for mem_ser, score in entry["value"]:
        member = decode_value(mem_ser)
        mapping[member] = score
    if merge:
        existing = safe_zrange_withscores(r, key, 0, -1)
        existing_members = set(mem for mem, _ in existing) if existing else set()
        to_add = {k: v for k, v in mapping.items() if k not in existing_members}
        if to_add:
            items = list(to_add.items())
            for i in range(0, len(items), PIPELINE_BATCH):
                chunk = dict(items[i:i+PIPELINE_BATCH])
                safe_zadd(r, key, chunk)
    else:
        safe_delete(r, key)
        items = list(mapping.items())
        for i in range(0, len(items), PIPELINE_BATCH):
            chunk = dict(items[i:i+PIPELINE_BATCH])
            safe_zadd(r, key, chunk)

def restore_list(r, key, entry, merge):
    items = [decode_value(it) for it in entry["value"]]
    if merge:
        # Lists cannot check membership; just append
        if items:
            safe_rpush(r, key, *items)
    else:
        safe_delete(r, key)
        for i in range(0, len(items), LIST_BATCH):
            safe_rpush(r, key, *items[i:i+LIST_BATCH])

def restore_stream(r, key, entry, merge):
    entries = entry["value"]
    if not entries:
        return
    if not merge:
        safe_delete(r, key)
    pipe = r.pipeline()
    count = 0
    for eid, fields in entries:
        fx = {}
        for fk, fv in fields.items():
            fname = decode_field_name(fk)
            fval = decode_value(fv)
            fx[fname] = fval
        if merge and exists_stream_id(r, key, eid):
            continue
        try:
            pipe.xadd(key, fields=fx, id=eid)
            count += 1
        except ResponseError as re:
            if "ID already exists" in str(re):
                pass
            else:
                print(f"❌ Error xadd {key} {eid}: {re}")
        if count >= PIPELINE_BATCH:
            try:
                pipe.execute()
            except Exception as e:
                print(f"❌ Pipeline error during stream restore: {e}")
            pipe = r.pipeline()
            count = 0
    if count > 0:
        try:
            pipe.execute()
        except Exception as e:
            print(f"❌ Final pipeline error during stream restore: {e}")

# --- Restore one chunk ---
def restore_chunk(r, chunk_obj, src_filename, merge):
    data = chunk_obj.get("data", {})
    meta = chunk_obj.get("_meta", {})
    total = len(data)
    db_index = meta.get("db", 0)
    print(f"🔁 Restoring {total} keys from {src_filename} (DB {db_index}), merge={merge}")
    safe_select(r, db_index)
    print(f"➡️ Selected DB {db_index}")
    for k_str, entry in data.items():
        key = decode_key(k_str)
        ktype = entry.get("type")
        ttl = entry.get("ttl", -1)
        try:
            if ktype == "string":
                restore_string(r, key, entry, merge)
            elif ktype == "hash":
                restore_hash(r, key, entry, merge)
            elif ktype == "set":
                restore_set(r, key, entry, merge)
            elif ktype == "zset":
                restore_zset(r, key, entry, merge)
            elif ktype == "list":
                restore_list(r, key, entry, merge)
            elif ktype == "stream":
                restore_stream(r, key, entry, merge)
            else:
                print(f"⚠️ Unknown type '{ktype}' for key {k_str}; skipping")
                continue
            if not merge and isinstance(ttl, int) and ttl > 0:
                safe_expire(r, key, ttl)
            elif merge and ttl > 0 and safe_exists(r, key):
                safe_expire(r, key, ttl)
        except Exception as e:
            print(f"❌ Error restoring key {k_str} (db {db_index}): {e}")

# --- Main ---
def restore_all_from_dir(merge=True):
    r = StrictRedis(host=REDIS_HOST, port=REDIS_PORT, decode_responses=False)
    files = sorted(glob.glob(os.path.join(OUTPUT_DIR, f"redis_backup_db*_part_*.json")))
    if not files:
        print("⚠️ No backup files found.")
        return

    for fpath in files:
        print(f"\n📂 Loading {fpath} ...")
        try:
            with open(fpath, "r", encoding="utf-8") as fh:
                obj = json.load(fh)
        except Exception as e:
            print(f"❌ Failed to parse {fpath}: {e}")
            continue
        try:
            restore_chunk(r, obj, os.path.basename(fpath), merge)
            print(f"✅ Restored {fpath}")
        except Exception as e:
            print(f"❌ Error restoring from {fpath}: {e}")

    print("\n🎉 Restore complete.")

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Restore Redis single DB backup JSON files.")
    parser.add_argument(
        "--force", action="store_true",
        help="Force overwrite mode: delete and restore keys even if they exist."
    )
    args = parser.parse_args()
    merge = not args.force
    restore_all_from_dir(merge=merge)
```
  
</details>

---

# To backup and restore the specific key pattern using `cabs.*.live_detail` from single db index in elasticache


<details>
  <summary>Click to view the scripts</summary>
  
### Backup Script

```py
#!/usr/bin/env python3
"""
robust_chunk_redis_backup_single_db_filter.py

Backs up keys from a single Redis DB (DB_INDEX) using a MATCH pattern.

Features:
- Binary-safe (UTF-8 or base64)
- SCAN/HSCAN/SSCAN/ZSCAN/XRANGE pagination
- Strict paginated XRANGE for streams (no duplicates)
- Retries on transient errors
- Atomic chunk writes with metadata
"""

import redis
import json
import base64
import os
import time
from functools import wraps
from redis.exceptions import ConnectionError, TimeoutError

# ===== Config =====
REDIS_HOST = "<your-end-point>.bp8cjs.ng.0001.aps1.cache.amazonaws.com"
REDIS_PORT = 6379
DB_INDEX = 1                          # target DB
KEY_PATTERN = "cabs.*.live_details"   # SCAN match filter
OUTPUT_DIR = "backup_specific_keys_db1"
CHUNK_SIZE = 2500                     # keys per JSON chunk
RETRY_LIMIT = 5
RETRY_DELAY = 1.0
XRANGE_PAGE = 1000
# ==================

os.makedirs(OUTPUT_DIR, exist_ok=True)

# --- retry decorator ---
def retry(attempts=RETRY_LIMIT, delay=RETRY_DELAY, exceptions=(ConnectionError, TimeoutError)):
    def deco(f):
        @wraps(f)
        def wrapped(*a, **kw):
            last_exc = None
            for i in range(attempts):
                try:
                    return f(*a, **kw)
                except exceptions as e:
                    last_exc = e
                    print(f"⚠️ Redis transient error: {e} — retry {i+1}/{attempts} in {delay}s")
                    time.sleep(delay)
            raise last_exc
        return wrapped
    return deco

# --- Safe serialization & write helpers ---
def safe_serialize(b):
    if b is None:
        return {"_t": "str", "v": ""}
    if isinstance(b, str):
        return {"_t": "str", "v": b}
    try:
        s = b.decode("utf-8")
        return {"_t": "str", "v": s}
    except Exception:
        return {"_t": "b64", "v": base64.b64encode(b).decode("ascii")}

def atomic_write_json(obj, final_path):
    tmp = final_path + ".tmp"
    with open(tmp, "w", encoding="utf-8") as fh:
        json.dump(obj, fh, indent=2, ensure_ascii=False)
        fh.flush()
        os.fsync(fh.fileno())
    os.replace(tmp, final_path)

# --- safe redis wrappers ---
@retry()
def safe_scan(r, cursor, match=None, count=1000):
    return r.scan(cursor=cursor, match=match, count=count)

@retry()
def safe_type(r, key):
    return r.type(key)

@retry()
def safe_get(r, key):
    return r.get(key)

@retry()
def safe_lrange(r, key, start, end):
    return r.lrange(key, start, end)

@retry()
def safe_hscan(r, key, cursor, count):
    return r.hscan(key, cursor=cursor, count=count)

@retry()
def safe_sscan(r, key, cursor, count):
    return r.sscan(key, cursor=cursor, count=count)

@retry()
def safe_zscan(r, key, cursor, count):
    return r.zscan(key, cursor=cursor, count=count)

@retry()
def safe_zrange_withscores(r, key, start, end):
    return r.zrange(key, start, end, withscores=True)

@retry()
def safe_xrange(r, key, start_id='-', end_id='+', count=XRANGE_PAGE):
    return r.xrange(key, min=start_id, max=end_id, count=count)

@retry()
def safe_hgetall(r, key):
    return r.hgetall(key)

@retry()
def safe_smembers(r, key):
    return r.smembers(key)

@retry()
def safe_ttl(r, key):
    return r.ttl(key)

# --- stream collector ---
def collect_stream_strict(r, key, page_size=XRANGE_PAGE):
    entries = []
    start = '-'
    while True:
        batch = safe_xrange(r, key, start_id=start, end_id='+', count=page_size)
        if not batch:
            break
        if start != '-' and batch and isinstance(batch[0][0], (bytes, bytearray)) and batch[0][0].decode() == start:
            batch = batch[1:]
        if not batch:
            break
        for entry_id, fields in batch:
            eid = entry_id.decode() if isinstance(entry_id, (bytes, bytearray)) else str(entry_id)
            fm = {}
            for fk, fv in fields.items():
                try:
                    fk_s = fk.decode("utf-8")
                    fk_key = fk_s
                except Exception:
                    fk_key = "__b64_field__" + base64.b64encode(fk).decode("ascii")
                fm[fk_key] = safe_serialize(fv)
            entries.append([eid, fm])
        last_raw = batch[-1][0]
        last_id = last_raw.decode() if isinstance(last_raw, (bytes, bytearray)) else str(last_raw)
        parts = last_id.split('-')
        try:
            ms = int(parts[0])
            seq = int(parts[1]) if len(parts) > 1 else 0
            start = f"{ms}-{seq + 1}"
        except Exception:
            start = last_id
        if len(batch) < page_size:
            break
    return entries

# --- key backup dispatcher ---
def backup_key(r, key):
    ktype_raw = safe_type(r, key)
    ktype = ktype_raw.decode() if isinstance(ktype_raw, (bytes, bytearray)) else str(ktype_raw)
    if ktype == 'string':
        return safe_serialize(safe_get(r, key))
    elif ktype == 'hash':
        out = {}
        cursor = 0
        while True:
            cursor, batch = safe_hscan(r, key, cursor, 1000)
            for f, v in batch.items():
                try:
                    fk = f.decode('utf-8')
                except Exception:
                    fk = "__b64_field__" + base64.b64encode(f).decode("ascii")
                out[fk] = safe_serialize(v)
            if cursor == 0:
                break
        return out
    elif ktype == 'set':
        members = []
        cursor = 0
        while True:
            cursor, batch = safe_sscan(r, key, cursor, 1000)
            for m in batch:
                members.append(safe_serialize(m))
            if cursor == 0:
                break
        return members
    elif ktype == 'zset':
        items = []
        cursor = 0
        while True:
            cursor, batch = safe_zscan(r, key, cursor, 1000)
            for member, score in batch:
                items.append([safe_serialize(member), score])
            if cursor == 0:
                break
        return items
    elif ktype == 'list':
        length = r.llen(key)
        items = []
        BATCH = 1000
        for start in range(0, length, BATCH):
            end = start + BATCH - 1
            chunk = safe_lrange(r, key, start, end)
            for e in chunk:
                items.append(safe_serialize(e))
        return items
    elif ktype == 'stream':
        return collect_stream_strict(r, key, page_size=XRANGE_PAGE)
    else:
        return None

def save_chunk(chunk_obj, part_num):
    meta = {
        "generated_at": time.strftime("%Y-%m-%dT%H:%M:%SZ", time.gmtime()),
        "db": DB_INDEX,
        "chunk_number": part_num,
        "keys_in_chunk": len(chunk_obj)
    }
    out = {"_meta": meta, "data": chunk_obj}
    filename = os.path.join(OUTPUT_DIR, f"redis_backup_db{DB_INDEX}_part_{part_num}.json")
    atomic_write_json(out, filename)
    print(f"💾 Saved {len(chunk_obj)} keys to {filename}")

def backup_single_db():
    r = redis.StrictRedis(host=REDIS_HOST, port=REDIS_PORT, db=DB_INDEX, decode_responses=False)

    cursor = 0
    chunk = {}
    part = 1
    found = 0
    total_scanned = 0

    while True:
        cursor, keys = safe_scan(r, cursor, match=KEY_PATTERN, count=1000)
        total_scanned += len(keys)
        for key in keys:
            try:
                key_str = key.decode('utf-8')
            except Exception:
                key_str = "__b64_key__" + base64.b64encode(key).decode("ascii")

            found += 1
            try:
                val = backup_key(r, key)
                if val is None:
                    print(f"⚠️ Unsupported type for key {key_str}; skipping")
                    continue
                ttl = safe_ttl(r, key)
                chunk[key_str] = {"type": safe_type(r, key).decode(), "db": DB_INDEX, "value": val, "ttl": ttl}
            except Exception as e:
                print(f"❌ Error reading key {key_str}: {e}")

            if len(chunk) >= CHUNK_SIZE:
                save_chunk(chunk, part)
                chunk.clear()
                part += 1

        if cursor == 0:
            break

    if chunk:
        save_chunk(chunk, part)

    print(f"\n✅ DB {DB_INDEX} backup complete. Keys matched by pattern '{KEY_PATTERN}': {found} (scanned {total_scanned} keys).")

if __name__ == "__main__":
    backup_single_db()
```

### Restore Script

```py
#!/usr/bin/env python3
"""
robust_restore_single_db.py

Restore Redis backup chunks for a single DB index.

Features:
- Restore to original DB index from backup metadata.
- Binary-safe (utf8/base64).
- SCAN/HSCAN/SSCAN/ZSCAN/XRANGE pagination safe.
- Strict paginated XRANGE for streams (no duplicates).
- Retry on transient Redis errors.
- Atomic operations.
- Merge mode is DEFAULT: skip deletes and only add missing keys/fields/members.
- Use --force to overwrite (delete and restore all keys).
"""

import os
import json
import base64
import time
import glob
import argparse
from functools import wraps
from redis import StrictRedis
from redis.exceptions import ConnectionError, TimeoutError, ResponseError

# ===== Config =====
REDIS_HOST = "your-elasticache-endpoint.amazonaws.com"
REDIS_PORT = 6379
OUTPUT_DIR = "backup_output"
RETRY_LIMIT = 5
RETRY_DELAY = 1.0
LIST_BATCH = 1000
PIPELINE_BATCH = 500
# ==================

if not os.path.isdir(OUTPUT_DIR):
    raise SystemExit(f"❌ OUTPUT_DIR '{OUTPUT_DIR}' not found.")

# --- Retry decorator ---
def retry(attempts=RETRY_LIMIT, delay=RETRY_DELAY, exceptions=(ConnectionError, TimeoutError)):
    def deco(f):
        @wraps(f)
        def wrapped(*a, **kw):
            last_exc = None
            for i in range(attempts):
                try:
                    return f(*a, **kw)
                except exceptions as e:
                    last_exc = e
                    print(f"⚠️ Transient Redis error: {e} — retry {i+1}/{attempts} in {delay}s")
                    time.sleep(delay)
            raise last_exc
        return wrapped
    return deco

# --- Decode helpers ---
def decode_value(vdict):
    if vdict is None:
        return b""
    t = vdict.get("_t")
    v = vdict.get("v", "")
    if t == "b64":
        return base64.b64decode(v)
    return v

def decode_key(key_str):
    prefix = "__b64_key__"
    if key_str.startswith(prefix):
        b64 = key_str[len(prefix):]
        return base64.b64decode(b64)
    return key_str

def decode_field_name(fk):
    prefix = "__b64_field__"
    if fk.startswith(prefix):
        b64 = fk[len(prefix):]
        return base64.b64decode(b64)
    return fk

# --- Safe Redis wrappers ---
@retry()
def safe_select(r, db):
    return r.execute_command("SELECT", db)

@retry()
def safe_set(r, key, value):
    return r.set(key, value)

@retry()
def safe_delete(r, key):
    return r.delete(key)

@retry()
def safe_expire(r, key, ttl):
    return r.expire(key, ttl)

@retry()
def safe_hset(r, key, mapping):
    return r.hset(key, mapping=mapping)

@retry()
def safe_hgetall(r, key):
    return r.hgetall(key)

@retry()
def safe_sadd(r, key, *members):
    return r.sadd(key, *members)

@retry()
def safe_smembers(r, key):
    return r.smembers(key)

@retry()
def safe_zadd(r, key, mapping):
    return r.zadd(key, mapping)

@retry()
def safe_zrange_withscores(r, key, start, end):
    return r.zrange(key, start, end, withscores=True)

@retry()
def safe_rpush(r, key, *values):
    return r.rpush(key, *values)

@retry()
def safe_llen(r, key):
    return r.llen(key)

@retry()
def safe_xadd(r, key, fields, id='*'):
    return r.xadd(key, fields=fields, id=id)

@retry()
def safe_xrange(r, key, start_id='-', end_id='+', count=1000):
    return r.xrange(key, min=start_id, max=end_id, count=count)

@retry()
def safe_exists(r, key):
    return r.exists(key)

# --- Helpers ---
def exists_stream_id(r, key, eid):
    res = safe_xrange(r, key, start_id=eid, end_id=eid, count=1)
    return len(res) > 0

# --- Restore functions ---
def restore_string(r, key, entry, merge):
    val = decode_value(entry["value"])
    if merge:
        if not safe_exists(r, key):
            safe_set(r, key, val)
    else:
        safe_delete(r, key)
        safe_set(r, key, val)

def restore_hash(r, key, entry, merge):
    mapping = {}
    for fk, fv in entry["value"].items():
        field = decode_field_name(fk)
        val = decode_value(fv)
        mapping[field] = val
    if merge:
        existing = safe_hgetall(r, key)
        existing_fields = set(existing.keys()) if existing else set()
        to_add = {k: v for k, v in mapping.items() if k not in existing_fields}
        if to_add:
            safe_hset(r, key, to_add)
    else:
        safe_delete(r, key)
        safe_hset(r, key, mapping)

def restore_set(r, key, entry, merge):
    members = [decode_value(m) for m in entry["value"]]
    if merge:
        existing = safe_smembers(r, key)
        existing_set = set(existing) if existing else set()
        to_add = [m for m in members if m not in existing_set]
        if to_add:
            for i in range(0, len(to_add), PIPELINE_BATCH):
                safe_sadd(r, key, *to_add[i:i+PIPELINE_BATCH])
    else:
        safe_delete(r, key)
        for i in range(0, len(members), PIPELINE_BATCH):
            safe_sadd(r, key, *members[i:i+PIPELINE_BATCH])

def restore_zset(r, key, entry, merge):
    mapping = {}
    for mem_ser, score in entry["value"]:
        member = decode_value(mem_ser)
        mapping[member] = score
    if merge:
        existing = safe_zrange_withscores(r, key, 0, -1)
        existing_members = set(mem for mem, _ in existing) if existing else set()
        to_add = {k: v for k, v in mapping.items() if k not in existing_members}
        if to_add:
            items = list(to_add.items())
            for i in range(0, len(items), PIPELINE_BATCH):
                chunk = dict(items[i:i+PIPELINE_BATCH])
                safe_zadd(r, key, chunk)
    else:
        safe_delete(r, key)
        items = list(mapping.items())
        for i in range(0, len(items), PIPELINE_BATCH):
            chunk = dict(items[i:i+PIPELINE_BATCH])
            safe_zadd(r, key, chunk)

def restore_list(r, key, entry, merge):
    items = [decode_value(it) for it in entry["value"]]
    if merge:
        # Lists cannot check membership; just append
        if items:
            safe_rpush(r, key, *items)
    else:
        safe_delete(r, key)
        for i in range(0, len(items), LIST_BATCH):
            safe_rpush(r, key, *items[i:i+LIST_BATCH])

def restore_stream(r, key, entry, merge):
    entries = entry["value"]
    if not entries:
        return
    if not merge:
        safe_delete(r, key)
    pipe = r.pipeline()
    count = 0
    for eid, fields in entries:
        fx = {}
        for fk, fv in fields.items():
            fname = decode_field_name(fk)
            fval = decode_value(fv)
            fx[fname] = fval
        if merge and exists_stream_id(r, key, eid):
            continue
        try:
            pipe.xadd(key, fields=fx, id=eid)
            count += 1
        except ResponseError as re:
            if "ID already exists" in str(re):
                pass
            else:
                print(f"❌ Error xadd {key} {eid}: {re}")
        if count >= PIPELINE_BATCH:
            try:
                pipe.execute()
            except Exception as e:
                print(f"❌ Pipeline error during stream restore: {e}")
            pipe = r.pipeline()
            count = 0
    if count > 0:
        try:
            pipe.execute()
        except Exception as e:
            print(f"❌ Final pipeline error during stream restore: {e}")

# --- Restore one chunk ---
def restore_chunk(r, chunk_obj, src_filename, merge):
    data = chunk_obj.get("data", {})
    meta = chunk_obj.get("_meta", {})
    total = len(data)
    db_index = meta.get("db", 0)
    print(f"🔁 Restoring {total} keys from {src_filename} (DB {db_index}), merge={merge}")
    safe_select(r, db_index)
    print(f"➡️ Selected DB {db_index}")
    for k_str, entry in data.items():
        key = decode_key(k_str)
        ktype = entry.get("type")
        ttl = entry.get("ttl", -1)
        try:
            if ktype == "string":
                restore_string(r, key, entry, merge)
            elif ktype == "hash":
                restore_hash(r, key, entry, merge)
            elif ktype == "set":
                restore_set(r, key, entry, merge)
            elif ktype == "zset":
                restore_zset(r, key, entry, merge)
            elif ktype == "list":
                restore_list(r, key, entry, merge)
            elif ktype == "stream":
                restore_stream(r, key, entry, merge)
            else:
                print(f"⚠️ Unknown type '{ktype}' for key {k_str}; skipping")
                continue
            if not merge and isinstance(ttl, int) and ttl > 0:
                safe_expire(r, key, ttl)
            elif merge and ttl > 0 and safe_exists(r, key):
                safe_expire(r, key, ttl)
        except Exception as e:
            print(f"❌ Error restoring key {k_str} (db {db_index}): {e}")

# --- Main ---
def restore_all_from_dir(merge=True):
    r = StrictRedis(host=REDIS_HOST, port=REDIS_PORT, decode_responses=False)
    files = sorted(glob.glob(os.path.join(OUTPUT_DIR, f"redis_backup_db*_part_*.json")))
    if not files:
        print("⚠️ No backup files found.")
        return

    for fpath in files:
        print(f"\n📂 Loading {fpath} ...")
        try:
            with open(fpath, "r", encoding="utf-8") as fh:
                obj = json.load(fh)
        except Exception as e:
            print(f"❌ Failed to parse {fpath}: {e}")
            continue
        try:
            restore_chunk(r, obj, os.path.basename(fpath), merge)
            print(f"✅ Restored {fpath}")
        except Exception as e:
            print(f"❌ Error restoring from {fpath}: {e}")

    print("\n🎉 Restore complete.")

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Restore Redis single DB backup JSON files.")
    parser.add_argument(
        "--force", action="store_true",
        help="Force overwrite mode: delete and restore keys even if they exist."
    )
    args = parser.parse_args()
    merge = not args.force
    restore_all_from_dir(merge=merge)
```

</details>


---


  
</details>

---

# To write in Elasticache from Local using Scripts

### To write the missing keys in the specific db, Existing keys it will skip.

```python
#!/usr/bin/env python3
"""
robust_recreate_cabs.py

Safely recreates missing cab_id hashes in Redis with robust handling.

Features:
- UTF-8 & binary safe
- Retry on transient Redis errors
- Pipelines for batch inserts
- Skips existing keys (merge mode)
- Structured logging and summary
"""

import redis
import time
import logging
from functools import wraps
from redis.exceptions import ConnectionError, TimeoutError

# ===== Config =====
REDIS_HOST = "<your-end-point>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com"
REDIS_PORT = 6379
REDIS_DB = 1
RETRY_LIMIT = 5
RETRY_DELAY = 1.0
PIPELINE_BATCH = 100
# ==================

MISSING_CAB_IDS = [
    3521, 5528, 5529, 5530, 5687, 5689, 5690, 5693,
    5729, 5734, 5735, 5737, 5738, 5740, 5749, 5804,
    5806, 5807, 5906, 5908, 5909, 5928, 5929, 5967,
    6130, 6209, 6210, 6211, 6212, 6214, 6218, 6594,
    6752, 6754, 6755, 6756, 6814, 6815, 6818, 6820,
    6821, 6855, 6856, 6857, 6858, 6860, 6962, 7013,
    60174, 60179, 60186, 60190, 60195, 60198, 60202, 60205,
    60209, 60212, 60220, 60224, 60229, 60234
]

DEFAULT_HASH = {
    "cab_id": "",  # will be filled dynamically
    "latitude": "",
    "longitude": "",
    "recently_connected_at": "",
    "speed": "",
    "kms_since_last_login": "",
    "bearing": "",
    "total_login_hours_of_current_month": "0",
    "total_trip_fare_of_current_month": "0",
    "target_revenue_amount": "0",
    "achieved_average_target_revenue": "0",
    "total_login_hours_for_current_day": "0",
    "target_revenue_amount_for_current_day": "0",
    "achieved_average_target_revenue_for_current_day": "0",
    "total_daily_collection_for_cab": "0",
    "total_monthly_collection_for_cab": "0",
    "achieved_total_daily_collection_for_cab": "0",
    "achieved_total_monthly_collection_for_cab": "0",
    "city_id": "",
    "device_id": "",
    "driver_id": "",
    "driver_app_version": "",
    "total_trip_fare_of_current_day": "0",
    "total_rental_trip_fare_of_current_month": "0",
    "total_outstation_trip_fare_of_current_month": "0",
}

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")

# --- Retry decorator ---
def retry(attempts=RETRY_LIMIT, delay=RETRY_DELAY, exceptions=(ConnectionError, TimeoutError)):
    def deco(f):
        @wraps(f)
        def wrapped(*a, **kw):
            last_exc = None
            for i in range(1, attempts + 1):
                try:
                    return f(*a, **kw)
                except exceptions as e:
                    last_exc = e
                    logging.warning(f"⚠️ Transient Redis error: {e} — retry {i}/{attempts} in {delay}s")
                    time.sleep(delay)
            raise last_exc
        return wrapped
    return deco

@retry()
def get_redis():
    return redis.StrictRedis(
        host=REDIS_HOST,
        port=REDIS_PORT,
        db=REDIS_DB,
        decode_responses=True,  # utf-8 safe
    )

def recreate_missing_keys():
    r = get_redis()
    created, skipped, failed = 0, 0, 0
    pipe = r.pipeline()

    for idx, cab_id in enumerate(MISSING_CAB_IDS, 1):
        key = f"cabs.{cab_id}.live_details"
        try:
            if r.exists(key):
                logging.info(f"✅ Skipping existing key: {key}")
                skipped += 1
                continue

            new_hash = DEFAULT_HASH.copy()
            new_hash["cab_id"] = str(cab_id)
            pipe.hset(key, mapping=new_hash)
            created += 1

            # execute batch
            if idx % PIPELINE_BATCH == 0:
                pipe.execute()
                pipe = r.pipeline()

        except Exception as e:
            logging.error(f"❌ Failed to create {key}: {e}")
            failed += 1

    # flush remaining
    try:
        pipe.execute()
    except Exception as e:
        logging.error(f"❌ Final pipeline execution failed: {e}")

    logging.info(f"Summary → Created: {created}, Skipped: {skipped}, Failed: {failed}")

if __name__ == "__main__":
    recreate_missing_keys()
```

### To overwrite the existing keys, It will write the values which are not present in the keys and skip the values and keys which is already present
- It will not delete the keys and write new values instead It will write the values which is not present.
```python
#!/usr/bin/env python3
"""
robust_overwrite_cabs.py

Overwrites cab_id hashes in Redis with the provided default structure.

Features:
- UTF-8 & binary safe
- Retry on transient Redis errors
- Pipelines for batch inserts
- Always overwrites existing keys (no skipping)
- Structured logging and summary
"""

import redis
import time
import logging
from functools import wraps
from redis.exceptions import ConnectionError, TimeoutError

# ===== Config =====
REDIS_HOST = "<your-end-point>-dev.bp8cjs.ng.0001.aps1.cache.amazonaws.com"
REDIS_PORT = 6379
REDIS_DB = 1
RETRY_LIMIT = 5
RETRY_DELAY = 1.0
PIPELINE_BATCH = 100
# ==================

CAB_IDS = [
    3521, 5528, 5529, 5530, 5687, 5689, 5690, 5693,
    5729, 5734, 5735, 5737, 5738, 5740, 5749, 5804,
    5806, 5807, 5906, 5908, 5909, 5928, 5929, 5967,
    6130, 6209, 6210, 6211, 6212, 6214, 6218, 6594,
    6752, 6754, 6755, 6756, 6814, 6815, 6818, 6820,
    6821, 6855, 6856, 6857, 6858, 6860, 6962, 7013,
    60174, 60179, 60186, 60190, 60195, 60198, 60202, 60205,
    60209, 60212, 60220, 60224, 60229, 60234
]

DEFAULT_HASH = {
    "cab_id": "",  # will be filled dynamically
    "latitude": "",
    "longitude": "",
    "recently_connected_at": "",
    "speed": "",
    "kms_since_last_login": "",
    "bearing": "",
    "total_login_hours_of_current_month": "0",
    "total_trip_fare_of_current_month": "0",
    "target_revenue_amount": "0",
    "achieved_average_target_revenue": "0",
    "total_login_hours_for_current_day": "0",
    "target_revenue_amount_for_current_day": "0",
    "achieved_average_target_revenue_for_current_day": "0",
    "total_daily_collection_for_cab": "0",
    "total_monthly_collection_for_cab": "0",
    "achieved_total_daily_collection_for_cab": "0",
    "achieved_total_monthly_collection_for_cab": "0",
    "city_id": "",
    "device_id": "",
    "driver_id": "",
    "driver_app_version": "",
    "total_trip_fare_of_current_day": "0",
    "total_rental_trip_fare_of_current_month": "0",
    "total_outstation_trip_fare_of_current_month": "0",
}

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")

# --- Retry decorator ---
def retry(attempts=RETRY_LIMIT, delay=RETRY_DELAY, exceptions=(ConnectionError, TimeoutError)):
    def deco(f):
        @wraps(f)
        def wrapped(*a, **kw):
            last_exc = None
            for i in range(1, attempts + 1):
                try:
                    return f(*a, **kw)
                except exceptions as e:
                    last_exc = e
                    logging.warning(f"⚠️ Redis error: {e} — retry {i}/{attempts} in {delay}s")
                    time.sleep(delay)
            raise last_exc
        return wrapped
    return deco

@retry()
def get_redis():
    return redis.StrictRedis(
        host=REDIS_HOST,
        port=REDIS_PORT,
        db=REDIS_DB,
        decode_responses=True,  # utf-8 safe
    )

def overwrite_cabs():
    r = get_redis()
    created, failed = 0, 0
    pipe = r.pipeline()

    for idx, cab_id in enumerate(CAB_IDS, 1):
        key = f"cabs.{cab_id}.live_details"
        try:
            new_hash = DEFAULT_HASH.copy()
            new_hash["cab_id"] = str(cab_id)

            pipe.delete(key)  # ensure we wipe old/bad fields
            pipe.hset(key, mapping=new_hash)
            created += 1

            if idx % PIPELINE_BATCH == 0:
                pipe.execute()
                pipe = r.pipeline()

        except Exception as e:
            logging.error(f"❌ Failed to overwrite {key}: {e}")
            failed += 1

    # flush any remaining commands
    try:
        pipe.execute()
    except Exception as e:
        logging.error(f"❌ Final pipeline execution failed: {e}")

    logging.info(f"Summary → Overwritten: {created}, Failed: {failed}")

if __name__ == "__main__":
    overwrite_cabs()
```

### To overwrite the existing keys, It will write the values which are not present in the keys and skip the values and keys which is already present
