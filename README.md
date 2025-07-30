# Elasticache Redis Parameters

## Cluster Details
### Cluster Name

* **What it is:** Unique identifier for the cluster.
* **Why used:** Distinguishes this Redis cluster from others; important for management, automation, and monitoring.
* **Advantages:** Easy to reference; clearly identifies the purpose/environment.
* **Disadvantages:** Must conform to AWS naming rules.
* **Example:** `<redis_cluster_name>` used in scripts and monitoring tools.

### Description

* **What it is:** Human-readable description of the cluster.
* **Why used:** Helps provide clarity about cluster usage or environment.
* **Advantages:** Improves documentation and operations.
* **Disadvantages:** Not used programmatically.
* **Example:** `<redis_cluster_name>` used for internal reference.

### Node Type (`cache.t4g.micro`)

* **What it is:** Defines hardware configuration (vCPU, memory, networking) for nodes.
* **Why used:** Controls performance, throughput, and cost.
* **Advantages:** Allows choosing from low-cost dev nodes to high-throughput production types.
* **Disadvantages:** Small instances may cause latency or memory pressure.
* **Example:** `cache.t4g.micro` is cost-efficient for development but not suited for production.

### Status

* **What it is:** Indicates current state of the cluster (e.g., Available, Modifying).
* **Why used:** Lets you monitor lifecycle and health.
* **Advantages:** Provides real-time status.
* **Disadvantages:** None.
* **Example:** "Available" means ready for reads/writes.

### Engine

* **What it is:** The caching engine used—Redis or Memcached.
* **Why used:** Determines feature set, compatibility, and behavior.
* **Advantages:** Choose Redis for persistence and rich features.
* **Disadvantages:** Redis is more complex than Memcached.
* **Example:** Redis 7.1.0 supports streams, ACLs, etc.

### Engine Version

* **What it is:** Specific version of the Redis engine.
* **Why used:** Select version for security, feature availability.
* **Advantages:** Newer versions have security patches and performance improvements.
* **Disadvantages:** Compatibility risks with older apps.
* **Example:** 7.1.0 supports modern Redis features.

### Global Datastore

* **What it is:** Cross-region replication capability.
* **Why used:** Disaster recovery, global application performance.
* **Advantages:** Multi-region availability.
* **Disadvantages:** Higher cost, latency in replication.
* **Example:** Not used in the cluster shown.

### Global Datastore Role

* **What it is:** Defines whether the cluster is a primary or secondary in a global datastore.
* **Why used:** Manages replication topology.
* **Advantages:** Control over write and read behavior across regions.
* **Disadvantages:** Configuration complexity.

### Update Status

* **What it is:** Indicates if updates are available for Redis engine.
* **Why used:** Helps apply critical patches.
* **Advantages:** Improves performance and security.
* **Disadvantages:** Updates may cause momentary downtime.

### Cluster Mode

* **What it is:** Enables Redis Cluster mode for sharding.
* **Why used:** Scale out Redis horizontally with shards.
* **Advantages:** Higher scalability.
* **Disadvantages:** Requires partitioning data.
* **Example:** Disabled = simpler single-node architecture.

### Shards

* **What it is:** Logical partitions in cluster mode.
* **Why used:** Each shard can contain multiple nodes.
* **Advantages:** Scale by splitting datasets.
* **Disadvantages:** Only applies if cluster mode enabled.

### Number of Nodes

* **What it is:** Total nodes in the cluster (primary + replicas).
* **Why used:** Affects fault tolerance and read throughput.
* **Advantages:** Add replicas for HA and reads.
* **Disadvantages:** Cost increases.

### Data Tiering

* **What it is:** Moves infrequently accessed data to SSDs.
* **Why used:** Reduces memory costs for large datasets.
* **Advantages:** Efficient storage usage.
* **Disadvantages:** Latency increase for cold data.

### Multi-AZ

* **What it is:** Distributes nodes across AZs for HA.
* **Why used:** Survive AZ failures.
* **Advantages:** Fault-tolerant.
* **Disadvantages:** Requires replicas; costs more.

### Auto-Failover

* **What it is:** Automatically promotes replica if primary fails.
* **Why used:** Ensures continuity.
* **Advantages:** Reduces downtime.
* **Disadvantages:** Only works with Multi-AZ and replicas.

### Encryption in Transit

* **What it is:** TLS encryption for data in motion.
* **Why used:** Prevents eavesdropping.
* **Advantages:** Secure.
* **Disadvantages:** Slight CPU overhead.

### Encryption at Rest

* **What it is:** Encrypts Redis data on disk.
* **Why used:** Meets compliance like PCI/HIPAA.
* **Advantages:** Secure.
* **Disadvantages:** Minimal performance hit.

### Parameter Group

* **What it is:** Defines Redis settings like eviction policy.
* **Why used:** Tweak Redis behavior.
* **Advantages:** Tunable.
* **Disadvantages:** Wrong config = instability.
* **Example:** Change `maxmemory-policy` to LFU.

### Outpost ARN

* **What it is:** Indicates Outpost integration.
* **Why used:** Hybrid cloud setups.
* **Advantages:** On-prem AWS services.
* **Disadvantages:** Enterprise-only feature.

### Transit Encryption Mode

* **What it is:** Specifies TLS behavior (`preferred`, `required`).
* **Why used:** Compatibility and security.
* **Advantages:** Flexible.
* **Disadvantages:** Potential unencrypted access if not required.

### Primary Endpoint

* **What it is:** DNS name for writing data.
* **Why used:** Clients connect for writes.
* **Advantages:** Simplifies connection logic.

### Reader Endpoint

* **What it is:** DNS name for read-only access.
* **Why used:** Load balance read traffic.
* **Advantages:** Scalable reads.

### ARN

* **What it is:** AWS identifier for the cluster.
* **Why used:** IAM permissions and automation.
* **Advantages:** Uniquely identifies resource.

---

## Connectivity and Security
### Connect to Your Cache

* **What it is:** A set of methods to interact with your Redis cluster from AWS tools or external clients.
* **Why used:** Allows developers and operators to test, debug, and operate cache instances securely.
* **Advantages:** Fast connection setup, supports various environments (CLI, browser, code).
* **Disadvantages:** Requires correct auth config; CLI access may be limited by IP or VPC.

<details>
  <summary>Click to view in details How to connect to your cache</summary>

### Connect to Cache Button

* **What it is:** A console feature to auto-initiate a connection via CloudShell or CLI.
* **Why used:** Simplifies connecting without manually entering commands.
* **Example:** Clicking 'Connect to cache' opens CloudShell with pre-installed Redis/Valkey CLI.

### AWS CloudShell

* **What it is:** A browser-based, pre-authenticated shell environment.
* **Why used:** Lets you securely manage your ElastiCache cluster directly from the AWS Console.
* **Advantages:** No local installation, secure, immediate access.
* **Disadvantages:** Not ideal for automation or large-scale scripting.
* **Example Command:**

```bash
redis6-cli --tls -h master.<redis_cluster_name>.bp8cjs.aps1.cache.amazonaws.com -p 6379
```

### Valkey GLIDE (Recommended)

* **What it is:** An open-source client for Redis/Valkey designed for modern environments.
* **Why used:** Preferred over the older Redis CLI due to better support and performance.
* **Advantages:** Updated command set, supports TLS, integrates well with modern Redis features.
* **Example Usage:**

```bash
glide-cli --tls -h master.<redis_cluster_name>.bp8cjs.aps1.cache.amazonaws.com -p 6379
```

### Client Libraries

* **What it is:** SDKs and libraries in languages like Python, JavaScript, Java, Go, etc.
* **Why used:** Integrates Redis cache into application code.
* **Advantages:** Seamless caching in real-time applications.
* **Disadvantages:** Requires connection pooling, error handling.
* **Example:**

```python
# Python (redis-py)
import redis
r = redis.StrictRedis(
    host='master.<redis_cluster_name>.bp8cjs.aps1.cache.amazonaws.com',
    port=6379,
    ssl=True
)
r.set('mykey', 'Hello, ElastiCache!')
print(r.get('mykey'))
```

### Basic Commands After Connecting

1. **Set a key:**

```bash
SET mykey "Hello, ElastiCache!"
```

2. **Get a key:**

```bash
GET mykey
```

3. **Ping the server:**

```bash
PING
```

4. **Get info:**

```bash
INFO
```

These commands help validate that the connection works and the cache is operational.

</details>

### Network Type

* **What it is:** IP version (IPv4).
* **Why used:** Routing and client compatibility.
* **Advantages:** Standard.

### Subnet Group Name

* **What it is:** Group of subnets across AZs.
* **Why used:** Controls where nodes are placed.
* **Advantages:** AZ diversity.
* **Disadvantages:** Must manage subnet CIDRs.

### VPC ID

* **What it is:** The VPC where the cluster resides.
* **Why used:** Network isolation.
* **Advantages:** Control traffic with VPC routing and security.

## Security Features

### AUTH Default User Access

* **What it is:** Enables password protection for default user.
* **Why used:** Basic Redis security.
* **Advantages:** Prevents anonymous access.
* **Disadvantages:** Disabled by default.

### Encryption Key

* **What it is:** KMS key used for encryption at rest.
* **Why used:** Data security.
* **Advantages:** AWS or customer managed.

### User Group / Association Status

* **What it is:** Controls Redis ACLs.
* **Why used:** Role-based access control.
* **Advantages:** Granular security.
* **Disadvantages:** Requires config.

## Connected Compute Resources

### Connected Compute Resources

* **What it is:** EC2 or Lambda with direct integration.
* **Why used:** Easier auth + monitoring.
* **Advantages:** Secure and simple.
* **Disadvantages:** Only shows auto-linked resources.

## Security Groups
### Security Group

* **What it is:** Controls inbound/outbound access.
* **Why used:** Acts as firewall.
* **Advantages:** Secure.
* **Disadvantages:** Misconfig = blocked access.

---

## Nodes
* **What it is:** Nodes are individual cache instances in your Redis cluster.
* **Why used:** Redis clusters use nodes to distribute data, provide high availability, and enable failover.
* **Advantages:** Scalable, fault-tolerant, can promote replicas to primary.
* **Disadvantages:** Manual failover for non-replicated nodes, limited control in managed clusters.

### Node Management Options

* **Manage tags:** Add metadata for billing or identification.
* **Failover primary:** Force a replica to take over if primary is failing.
* **Promote:** Promote a replica to become a new primary manually.
* **Reboot node:** Restart the node (e.g., for maintenance or recovery).
* **Delete node:** Remove the node from the cluster.
* **Add node:** Scale the cluster by adding a replica or shard.

<details>
  <summary>Click to view Examples</summary>

### Example Node Detail

| Property                   | Description                                                                                                                      |
| -------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| **Node name**              | `<redis_cluster_name>`                                                                                                 |
| **Status**                 | `Available` — Node is running and ready to serve requests.                                                                       |
| **Current role**           | `primary` — This node is handling writes and read replicas sync from it.                                                         |
| **Endpoint**               | `<redis_cluster_name>-001.<redis_cluster_name>.bp8cjs.aps1.cache.amazonaws.com:6379` — Used to connect from clients. |
| **ARN**                    | `arn:aws:elasticache:ap-south-1:<account_id>:cluster:<redis_cluster_name>001` — Unique AWS identifier.                    |
| **Parameter group status** | `In-sync` — Using the latest parameter group settings.                                                                           |
| **Zone**                   | `ap-south-1a` — Indicates which AZ the node resides in.                                                                          |
| **Created date**           | `December 18, 2024, 10:44:58 (UTC+05:30)` — Timestamp when the node was provisioned.                                             |

### Example Use Case:

* A read-heavy application can promote a replica to reduce latency.
* Reboot can help if the node becomes unresponsive without data loss (in clustered mode).

This information helps you monitor, maintain, and scale your Redis deployment efficiently.

</details>

---

## Metrics
* **What it is:** Metrics are quantitative measurements collected from your Redis nodes to monitor performance, health, and activity.
* **Why used:** To track memory usage, CPU load, cache hit rate, connections, and latency.
* **Advantages:** Enables proactive monitoring, automated scaling, and troubleshooting.
* **Disadvantages:** Some metrics are delayed (every 60 seconds), limited history without CloudWatch retention customization.

### Metric Selection & Comparison

* **Selected nodes:** Choose individual nodes (e.g., `<redis_cluster_name>-001`) to view detailed metrics.
* **Compare nodes:** Allows side-by-side comparison of performance across multiple nodes.

<details>
  <summary>Click to view the Metrics Shown and Explained</summary>

### Example Metrics (Sample 16 out of 39)

| Metric Name              | Description                                                                         |
| ------------------------ | ----------------------------------------------------------------------------------- |
| **CPUUtilization**       | Percentage of CPU used by the node. High usage may indicate processing bottlenecks. |
| **FreeableMemory**       | Amount of unused memory available. Low value signals memory pressure.               |
| **BytesUsedForCache**    | Total memory currently being used for cached data.                                  |
| **CurrConnections**      | Number of client connections currently open.                                        |
| **CacheHits**            | Number of successful key retrievals. High value indicates effective caching.        |
| **CacheMisses**          | Number of failed retrievals. High value suggests caching inefficiency.              |
| **Evictions**            | Count of keys evicted due to memory limits. Should be minimized.                    |
| **ReplicationLag**       | Delay between primary and replica sync. High values could lead to stale reads.      |
| **NetworkBytesIn/Out**   | Volume of data transferred in and out of the node.                                  |
| **CurrItems**            | Number of items currently in the cache.                                             |
| **SwapUsage**            | Amount of disk swap used. Should be ideally zero for in-memory databases.           |
| **Latency**              | Time taken to respond to client requests.                                           |
| **EngineCPUUtilization** | CPU usage specifically by the Redis engine.                                         |
| **DatabaseMemoryUsage**  | RAM used by Redis database.                                                         |
| **ClientConnections**    | Active connections to the cache from applications.                                  |
| **CommandRate**          | Rate of Redis commands processed per second.                                        |

### Example Use Case:

* You observe high `CacheMisses` and low `CacheHits`: You might need to increase your cache size or adjust TTLs.
* Increasing `Evictions` may indicate your working set is too large for the current node size.
* Monitoring `ReplicationLag` is critical in read-replica environments to avoid stale data.

Metrics can be visualized in CloudWatch dashboards or used with alarms and AWS Lambda automation for self-healing workflows.

</details>

---

## Logs
Logs in ElastiCache help with monitoring slow-performing commands (Slow Logs) and engine-level activity (Engine Logs). These logs are crucial for identifying performance bottlenecks and internal errors.

### Slow Logs
* **What it is:** Records Redis commands that exceed a certain execution time threshold.
* **Why used:** Helps detect slow-performing operations that may degrade performance.
* **Advantages:** Useful for profiling and optimizing command usage.
* **Disadvantages:** Disabled by default. Requires enabling and configuring.

<details>
  <summary>Click to view the Properties Shown and Explained</summary>
  
| Property                 | Description                                                                                        |
| ------------------------ | -------------------------------------------------------------------------------------------------- |
| **Enabled**              | No (Disabled by default)                                                                           |
| **Log destination type** | Type of log service where logs are sent. Example: `cloudwatch-logs`.                               |
| **Log format**           | Format used to structure the log data. Example: `text` or `json`.                                  |
| **Log destination**      | Target where the logs will be stored. Example: CloudWatch Log Group ARN.                           |
| **Log status**           | Displays whether the log delivery is working correctly. Example: `active`.                         |
| **Log error message**    | Describes any errors encountered during log delivery. Example: `Access denied to CloudWatch Logs.` |

**Example Use Case:**
Enable slow logs and set the threshold to `1000` microseconds to log all commands taking over 1ms. This helps pinpoint inefficient queries like `SMEMBERS large-set`.

</details>

### Engine Logs
* **What it is:** Captures engine-level events like restarts, connection failures, and internal errors.
* **Why used:** Useful for operational monitoring and debugging.
* **Advantages:** Can help detect crashes, restart causes, and memory failures.
* **Disadvantages:** Requires manual enabling and integration with CloudWatch or another logging destination.

<details>
  <summary>Click to view the Properties Shown and Explained</summary>
  
| Property                 | Description                                                                                        |
| ------------------------ | -------------------------------------------------------------------------------------------------- |
| **Enabled**              | No (Disabled by default)                                                                           |
| **Log destination type** | Type of log service where logs are sent. Example: `cloudwatch-logs`.                               |
| **Log format**           | Format used to structure the log data. Example: `text` or `json`.                                  |
| **Log destination**      | Target where the logs will be stored. Example: CloudWatch Log Group ARN.                           |
| **Log status**           | Displays whether the log delivery is working correctly. Example: `active`.                         |
| **Log error message**    | Describes any errors encountered during log delivery. Example: `Access denied to CloudWatch Logs.` |

**Example Use Case:**
Enable engine logs and route them to CloudWatch Logs to monitor for out-of-memory errors or failed snapshot saves (`RDB save failed`), and configure CloudWatch Alarms to alert operations teams.

These logs are essential for maintaining visibility into cache performance and health. Enable them via the AWS Console or CLI depending on your operational requirements.

</details>

---

## Maintenance and Backups

### Maintenance

* **Maintenance Window:** Scheduled weekly maintenance is set for `Sunday 18:30 - Sunday 19:30 UTC`. During this window, AWS may apply critical updates.
* **Auto Upgrade Minor Versions:** *Disabled*. This means that patch-level updates (e.g., from 7.0.1 to 7.0.2) won't be automatically applied.
* **Notification ARN:** *Disabled*. No Amazon SNS notification target is configured for maintenance-related alerts.

**Example Use Case:**
If you want to be alerted before any maintenance actions, set up an SNS topic and subscribe your email to receive notifications by configuring the Notification ARN.

### Backup

* **Automatic Backups:** *Disabled*. No daily backups are currently configured.
* **Backup Retention Period:** `0 days`. Since automated backups are disabled, retention is not applicable.
* **Backup Window:** `23:30-00:30 UTC`. If enabled, automated backups would be taken during this window.

### Backups Table

| Column          | Description                                                                  |
| --------------- | ---------------------------------------------------------------------------- |
| **Backup Name** | The name assigned to the backup. E.g., `rt-redis-backup-2025-07-30-23-30`.   |
| **Backup Type** | Indicates if the backup is automatic or manual. E.g., `automated`, `manual`. |
| **Status**      | Current state of the backup. E.g., `available`, `creating`, `failed`.        |
| **Cache Size**  | The total memory used by the cache at the time of backup. E.g., `1.5 GB`.    |
| **Shards**      | Number of shards included in the backup (relevant for cluster mode enabled). |

| Backup Name | Backup Type | Status | Cache Size | Shards |
| ----------- | ----------- | ------ | ---------- | ------ |
| *(None)*    | -           | -      | -          | -      |

**Example Use Case:**
To enable point-in-time recovery, you can enable automated backups with a retention period (e.g., `7 days`) and ensure the backup window avoids peak traffic.

These features are critical for operational resilience and disaster recovery. Maintenance windows allow safe patching, and backups ensure recoverability in case of data loss.

---

## Service Updates
Service updates are AWS-initiated patches or enhancements to the ElastiCache engine. These updates can be related to engine features, security, or operational improvements. You can apply them manually or wait for the auto-update deadline.

| Field                       | Description                                                                                         | Example                                |
| --------------------------- | --------------------------------------------------------------------------------------------------- | -------------------------------------- |
| **Service Update Name**     | Unique name of the update for tracking.                                                             | `elasticache-july-patch-update-202507` |
| **Cluster Update Status**   | Indicates if the update has been applied. Values include `Complete`, `Not-applied`, or `Available`. | `Not-applied`                          |
| **Update Type**             | Nature of the update. Typically `engine-update`, `security-update`, etc.                            | `engine-update`                        |
| **Apply-by Date**           | Deadline before which the update should be manually applied.                                        | `August 5, 2025, 17:29:59 (UTC+05:30)` |
| **Release Date**            | The date the update was released.                                                                   | `July 6, 2025, 17:30:00 (UTC+05:30)`   |
| **Update Severity**         | Indicates importance: `Low`, `Medium`, `Important`, `Critical`.                                     | `Important`                            |
| **Status**                  | General availability of the update.                                                                 | `Available`                            |
| **Cluster Update Modified** | When the status was last changed.                                                                   | `July 6, 2025, 23:11:18 (UTC+05:30)`   |
| **Scheduled Auto Update**   | If applicable, the date when AWS will automatically apply the update.                               | `-`                                    |


<details>
  <summary>Click to view the Sample Service Updates Table</summary>
  
**Sample Service Updates Table:**

| Service Update Name                  | Cluster Update Status | Update Type     | Apply-by Date                        | Release Date                        | Update Severity | Status    | Cluster Update Modified | Scheduled Auto Update |
| ------------------------------------ | --------------------- | --------------- | ------------------------------------ | ----------------------------------- | --------------- | --------- | ----------------------- | --------------------- |
| elasticache-july-patch-update-202507 | Not-applied           | engine-update   | August 5, 2025, 17:29:59 (UTC+05:30) | July 6, 2025, 17:30:00 (UTC+05:30)  | Important       | Available | July 6, 2025, 23:11:18  | -                     |
| elasticache-20250603-arm             | Complete              | security-update | July 24, 2025, 23:29:59 (UTC+05:30)  | June 24, 2025, 23:30:00 (UTC+05:30) | Medium          | Available | June 25, 2025, 02:35:12 | -                     |

</details>

**Example Use Case:**

* Regularly review this section to apply non-critical updates before the deadline to avoid unplanned auto-updates.
* Critical updates may be auto-applied even if not manually triggered.

This ensures stability, performance, and security compliance of your ElastiCache deployments.

---

## Tags

Tags help identify, organize, and manage AWS resources. Each tag is a key-value pair.

| Key          | Value             |
| ------------ | ----------------- |
| `testing` | `corporate-redis` |

**Example Use Case:**
Tags can be used for cost allocation, automation, or organizing resources. For instance, you can filter resources tagged with `corporate-redis` to track environment-specific metrics or billing.

---
