# Amazon ElastiCache
Amazon ElastiCache is a fully managed, scalable, and secure in-memory caching service provided by AWS (Amazon Web Services). It simplifies the process of setting up, operating, and scaling an in-memory data store or cache in the cloud. It is widely used to improve application performance by reducing the latency of accessing data and reducing the load on databases or backend services.

## ElastiCache Workflow of an Web Application
### Scenario:
- Imagine you're running a **web application** in a **containerized environment** (using Amazon ECS or EKS) and you're distributing content globally via **CloudFront** to reduce latency for users worldwide.
- Here’s how **ElastiCache** fits into this architecture:
  > **Visualizing It:**
  > 
  > **User Request → CloudFront (caches static content) → Backend (ECS/EKS) → ElastiCache (in-memory) → RDS/DynamoDB (if cache miss)**
- This architecture provides **faster data access** and **reduces database load**, leading to better performance for your application.

### 1. **Initial Request Flow** (Without ElastiCache):

<details>
   <summary>Click to View Initial Request flow without ElastiCache</summary>

* A user from a distant geographical location makes a request to your application (e.g., querying a product page).
* The request first hits **CloudFront** (a Content Delivery Network, CDN), which caches static content (like images, CSS, and JavaScript) close to the user, reducing latency.
* For dynamic content (like user data, product details, etc.), CloudFront forwards the request to your **backend container cluster** (ECS/EKS), where your application is running.
* The backend processes the request and fetches the data from your **primary database** (say, Amazon RDS, DynamoDB, or an external DB).
* The backend then returns the processed data to CloudFront, which caches it temporarily for future requests.
* The user receives the response.

**Problem:** Every time a user makes a request, your backend is fetching data from the database. If many users request the same data (e.g., product details), the database can become a bottleneck, leading to slower response times and higher costs from read-heavy operations.

</details>

### 2. **How ElastiCache Helps (with Caching):**
Now, let’s introduce **Amazon ElastiCache** into the system to improve this process.

<details>
   <summary>Click to View Request flow with ElastiCache</summary>
   
#### **How ElastiCache Works in this Scenario:**

1. **Setting Up ElastiCache**:

   * You create an **ElastiCache cluster** using **Redis** (or **Memcached** depending on your needs) in AWS.
   * You configure your backend containers (ECS/EKS) to use ElastiCache as an in-memory caching layer.

2. **Modified Request Flow (With ElastiCache):**

   * **Step 1:** A user makes a request for dynamic content (e.g., a product page).
   * **Step 2:** The request goes through **CloudFront**. CloudFront will cache any static content, as before. However, for dynamic content, CloudFront checks if that content has been cached in the CDN **edge location**.

     * If **cached**, CloudFront serves the response directly to the user with low latency.
     * If **not cached**, CloudFront forwards the request to your backend containers.
   * **Step 3:** Your backend container (ECS/EKS) first checks **ElastiCache** to see if the requested data (e.g., product info) is already stored in the cache.

     * If the data is **cached** in ElastiCache (cache hit), the backend retrieves the data **instantly** from the in-memory cache, greatly reducing latency.
     * If the data is **not cached** (cache miss), your backend queries the **primary database** (e.g., RDS), retrieves the data, and then stores it in **ElastiCache** for future use.
   * **Step 4:** The backend processes the data (if needed) and sends it to CloudFront, which then serves the response to the user.

3. **Subsequent Requests (Cache Hits):**

   * When subsequent requests for the same product or dynamic content come in, **CloudFront** first checks its cache for static content.
   * If CloudFront doesn’t have the cached dynamic content, it goes to the backend, but now the **ElastiCache** layer will return the data quickly (since it’s stored in memory) without hitting the database.
   * As more requests are handled, ElastiCache keeps serving the data much faster than querying the database repeatedly.

</details>

---
#### **How This Improves Performance and Reduces Latency:**
1. Fast Data Retrieval with ElastiCache
2. Reduced Database Load
3. Global Caching
4. Low Latency for Users
5. Cost Efficiency

<details>
   <summary>Click to View Detailed Insights on Performance and Reduced Latency</summary>

1. **Fast Data Retrieval with ElastiCache**:

   * **ElastiCache (Redis or Memcached)** stores frequently accessed data in memory. Data retrieval from memory is orders of magnitude faster than querying a database or disk storage.
   * Instead of hitting your primary database for every request, the backend gets data from **ElastiCache** nearly instantly.

2. **Reduced Database Load**:

   * By offloading frequent requests to ElastiCache, your database isn't constantly bombarded with read-heavy operations. This reduces load, preventing database bottlenecks and allowing your database to focus on more critical tasks like write operations and complex queries.

3. **Global Caching**:

   * **CloudFront** caches static content globally at its edge locations, but when combined with **ElastiCache**, CloudFront can also cache dynamic content at the **origin** level (from your backend containers).
   * This means both **static** and **dynamic** content can be served faster, based on what's stored in **ElastiCache** and **CloudFront**.

4. **Low Latency for Users**:

   * As the content is cached in **ElastiCache**, subsequent requests from users (even from faraway regions) will be served quickly because the data is **already in memory** and does not need to be recalculated or fetched from the database.
   * This ensures users get a **faster response** and an overall smoother experience.

5. **Cost Efficiency**:

   * Using ElastiCache reduces the need for more powerful, costly database instances since it offloads frequent reads.
   * Your **CloudFront** and **ElastiCache** setup can serve requests at a fraction of the cost compared to constantly scaling the database to handle read traffic.

</details>

<details>
   <summary>Key Features & Use Cases of Amazon ElastiCache</summary>

### Key Features of Amazon ElastiCache:
1. Fully managed services
2. Scalable
3. High Availability
4. Low Latency
5. Supports Redis and Memcached
6. Security
7. Cost-Effective
   
<details>
   <summary>Click to View Detailed Features of Amazon ElastiCache</summary>

1. **Fully Managed Service**: ElastiCache is fully managed, meaning AWS handles the infrastructure management, such as hardware provisioning, setup, configuration, patching, and scaling, leaving you to focus on building your applications.

2. **Scalable**: You can scale ElastiCache horizontally and vertically. You can increase capacity easily to meet growing demand and scale down when you no longer need as much capacity.

3. **High Availability**: ElastiCache supports replication, automatic failover, and backup, ensuring high availability for your applications and providing redundancy in case of failures.

4. **Low Latency**: ElastiCache stores data in-memory, which leads to very fast data retrieval times (often in microseconds), reducing the latency that would be experienced when accessing data from traditional databases.

5. **Supports Redis and Memcached**: ElastiCache supports two popular in-memory caching engines, **Redis** and **Memcached**. Both of these engines are designed to provide fast access to data and are widely used in caching and high-performance scenarios.

6. **Security**: You can control access to your ElastiCache clusters using AWS Identity and Access Management (IAM), VPC (Virtual Private Cloud), and encryption.

7. **Cost-Effective**: ElastiCache is designed to be cost-effective by improving application performance through caching, reducing the load on your backend systems, and lowering infrastructure costs.

</details>

### Use Cases for Amazon ElastiCache:
1. Caching
2. DB Performance Enhancement
3. Real Time Analytics
4. Message Queuing
5. Geospatial Indexing
6. Distributed Caching
7. Search Index Caching

<details>
   <summary>Click to View Detailed Use Cases of Amazon ElastiCache</summary>
   
1. **Caching**:

   - **Content Caching**: Store frequently requested content like web pages, API responses, or media files in an in-memory cache to quickly serve them to users, reducing backend load and improving user experience.
   - **Session Caching**: Store session state for web applications in memory, so that session data can be accessed quickly without making frequent calls to the database.
   - **Query Result Caching**: Cache database query results or computationally expensive operations to avoid performing the same computation or database query repeatedly.

2. **Database Performance Enhancement**:

   - **Reduce Database Load**: Offload frequent read requests from your database to the cache, allowing the database to focus on more complex queries or transactional operations, improving overall performance and response time.
   - **Write-Through/Write-Behind Caching**: ElastiCache can be configured to automatically update the cache when data is written to the database. This ensures consistency between the cache and the backend data store.

3. **Real-Time Analytics**:

   - **Leaderboard and Counting**: ElastiCache (especially with Redis) is used for applications that require real-time analytics, such as leaderboards or tracking user activities where quick updates and reads are essential.
   - **Session Tracking and Real-Time User Data**: Cache user session information, behavior, or analytics data that can be updated and retrieved in real time.

4. **Message Queuing**:

   - **Pub/Sub Systems**: Redis supports the "publish/subscribe" messaging paradigm, which can be used for real-time messaging systems, event notifications, or message queues in distributed systems.

5. **Geospatial Indexing**:

   - **Location-based Services**: Redis in ElastiCache provides support for geospatial data and operations, making it suitable for applications involving location-based queries (e.g., finding the nearest stores, users, or items).

6. **Distributed Caching**:

   - **Global Caching for Multi-Region Applications**: ElastiCache supports replication across regions, allowing a distributed cache for applications that require a fast, globally accessible cache layer.

7. **Search Index Caching**:

   - Caching search results, such as complex search queries or recommendations, to avoid repetitive and slow database lookups.

### **ElastiCache and Redis:**

**Redis** is an open-source, advanced key-value store that supports a variety of data structures such as strings, hashes, lists, sets, sorted sets, bitmaps, hyperloglogs, and geospatial indexes. It is commonly used as a caching layer, message broker, and for real-time analytics due to its low-latency, high-throughput capabilities.

**ElastiCache for Redis** is Amazon's managed service for deploying, managing, and scaling Redis in the cloud. Redis provides several key features that make it a great choice for ElastiCache, including:

- **Persistence**: Redis allows you to persist data to disk, which can be useful in use cases that require both high availability and durability. It offers snapshotting and append-only file persistence modes.
- **Replication and Clustering**: Redis supports replication and sharding (with clustering), which is supported in ElastiCache to provide high availability and scalability.
- **Pub/Sub**: Redis supports publish/subscribe messaging, which is ideal for event-driven architectures, and ElastiCache for Redis offers this functionality as part of its service.
- **Advanced Data Structures**: Redis's advanced data structures like lists, sets, sorted sets, and hashes are widely used in real-time applications such as gaming leaderboards, recommendation engines, and more.

When you use **Amazon ElastiCache for Redis**, AWS takes care of managing Redis clusters, scaling, monitoring, and availability. You only need to focus on the application logic, without worrying about the underlying infrastructure.

</details>

</details>

---

### Example Flow with Caching and CloudFront:

Let’s break down a practical flow of this architecture:

1. **User Request**: A user visits the website to view a product (e.g., Product A).
2. **CloudFront Cache Check**:

   * If **Product A** info is cached in CloudFront (static content like images), it’s served directly from the CDN.
   * If **Product A** info is **not cached** in CloudFront, it sends the request to the backend containers (ECS/EKS).
3. **Backend with ElastiCache**:

   * The backend queries **ElastiCache** to see if **Product A** data is in memory.
   * If **cached in ElastiCache**, it returns **instantly**.
   * If **not cached**, it fetches data from **RDS**, stores it in **ElastiCache**, and returns the response to CloudFront.
4. **Future Requests**:

   * Any future requests for **Product A** (from any region) will benefit from CloudFront caching for static content and ElastiCache caching for dynamic content, making the whole process much faster.

> 🛡 **Summary:**
> 
> * **ElastiCache** reduces the load on your **database** by caching frequently requested data, ensuring faster data retrieval.
> * It **improves application performance** by storing and serving data from memory rather than having to query the database for every request.
> * It works **seamlessly with CloudFront** to reduce latency both at the **edge (CloudFront)** and at the **backend (ElastiCache)**, delivering a highly responsive user experience.

---

### Key Differences Between ElastiCache with Redis and Memcached:

1. **Data Structures**:

   - **Redis**: Provides advanced data structures (strings, lists, sets, hashes, etc.).
   - **Memcached**: Primarily a simple key-value store.

2. **Persistence**:

   - **Redis**: Supports persistence and durability options (RDB snapshots, AOF).
   - **Memcached**: Does not offer built-in persistence, only volatile storage.

3. **Replication & Clustering**:

   - **Redis**: Supports replication and clustering for high availability and scalability.
   - **Memcached**: Does not support native clustering, though it can be configured in a distributed system.

4. **Use Case Suitability**:

   - **Redis**: Best for use cases requiring complex data operations, pub/sub messaging, persistent data, and high availability.
   - **Memcached**: Best for simple, high-speed caching requirements with a focus on key-value pairs and simplicity.

### Example: Redis Use Cases with ElastiCache

- **Leaderboard for a Gaming App**: Redis provides an efficient way to track real-time scores and rankings using sorted sets. ElastiCache for Redis can be used to store these rankings in-memory, ensuring that the game can update and retrieve rankings quickly.
- **Shopping Cart**: Use Redis to store session data, cart information, and user preferences in real time. This helps in quickly retrieving the data without hitting a database every time.

### **Good Use Cases for Redis**  
1. **Session Storage**
2. **Web Page Caching**  
3. **Pub/Sub Messaging** 
4. **Application State in Serverless Environments**  
5. **S3 Object Lookup Optimization**  
6. **Persistence Considerations** 

<details>
  <summary>Explaination of Use Cases Mentioned Above</summary>
   
1. **Session Storage**  
   - Primary use case for Redis.  
   - Early web apps had single servers with state, but scaling for high availability and performance required distributed session storage.  
   - Redis is a popular solution for this.  

2. **Web Page Caching**  
   - Store pre-rendered server-side content in Redis.  
   - Acts as a database cache to reduce latency and query load on relational databases.  
   - Popularized in the Rails community for cost optimization by minimizing database hits.  

3. **Pub/Sub Messaging**  
   - Supports low-latency microservice communication.  
   - Useful for real-time messaging between services.  

4. **Application State in Serverless Environments**  
   - Lambda functions need shared state with low latency.  
   - Redis serves as a fast state store instead of point-to-point communication.  

5. **S3 Object Lookup Optimization**  
   - S3 is an object store, not a file system, making key lookups expensive.  
   - Solution:  
     - Capture S3 object creation events via EventBridge.  
     - Store object metadata in Redis for fast prefix-based lookups.  
     - Avoids slow S3 `ListObjects` pagination for large buckets.  
   - Trade-offs:  
     - Requires sufficient memory.  
     - Must handle high write throughput with proper scaling.  

6. **Persistence Considerations**  
   - Redis’s non-serverless nature may require additional scaling strategies.  
   - Further discussion needed on persistence mechanisms.

</details>

### How ElastiCache Works:

- **Setting up ElastiCache**: You start by creating a cache cluster in the AWS Management Console. You choose either Redis or Memcached as the engine, configure the cluster (choose instance types, enable security features, etc.), and deploy it in your Virtual Private Cloud (VPC).

- **Data Access**: Applications access ElastiCache by connecting to the cluster's endpoint. ElastiCache will serve data from memory, providing low-latency responses. You can also configure it to automatically failover to a replica node in case of a failure.

- **Monitoring & Maintenance**: AWS CloudWatch is integrated with ElastiCache, allowing you to monitor the performance and health of your clusters. ElastiCache also supports automatic backups and software patching.

### Conclusion:

Amazon ElastiCache is an excellent solution for applications that require low-latency data access, high throughput, and the ability to scale dynamically. Its support for Redis and Memcached makes it a versatile tool for caching, real-time analytics, session management, and more. Redis, in particular, offers a rich set of features like persistence, complex data structures, and pub/sub messaging that makes it ideal for a wide range of use cases, from gaming leaderboards to session management and real-time notifications.

---

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
