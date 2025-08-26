# Back Up and Restore
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

---

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

---

### Why Tag Backups?

* **Organize** backups by purpose, owner, or environment
* **Filter/search** backups more easily
* **Enable cost tracking** with **Cost Allocation Tags**

> Example: You can track all `production` related backups in your AWS bill if you tag them with `environment=prod`.

---

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

</details>

---

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

---

# **Seeding a New ElastiCache for Redis OSS Cluster from an External Backup (.rdb)**

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

---

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

# **Where You Can Restore a Redis/Valkey Backup in ElastiCache**

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

---
