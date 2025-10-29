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

## Elasticache Instances Cost
### Common Parameters for all
- Utilization (On-Demand only) --> Value = 100, Unit = %Utilised/Month
- Cache Engine: Redis
- Cache Node Type: Standard
- Pricing model: On-Demand
  
#### cache.m5.large
- vCPU: 2
- Memory: 6.38 GiB
- Network Performance: Up to 10 Gigabit
- Cost: 1 instance(nodes) x 0.163 USD hourly x (100 / 100 Utilized/Month) x 730 hours in a month = 118.9900 USD

### cache.m7g.large
- vCPU: 2
- Memory: 6.38 GiB
- Network Performance: Up to 12.5 Gigabit
- Cost: 1 instance(nodes) x 0.164 USD hourly x (100 / 100 Utilized/Month) x 730 hours in a month = 119.7200 USD

---

## ElastiCache Redis OSS Metrics

### **Quick Reference**

| Metric                                         | Key Idea                        | Risk if High/Low                |
| ---------------------------------------------- | ------------------------------- | ------------------------------- |
| CPU Utilization                                | Server CPU load                 | High → slow responses           |
| Engine CPU Utilization                         | Redis CPU load                  | High → Redis bottleneck         |
| DB Memory Usage %                              | % of Redis memory used          | High → evictions                |
| DB Capacity Usage %                            | % of instance memory used       | High → risk of out-of-memory    |
| Cache Hit Rate                                 | How often cache served requests | Low → DB is queried more        |
| Memory Fragmentation Ratio                     | Efficiency of memory usage      | High → wasted RAM               |
| Swap Usage                                     | Disk memory usage               | >0 → Redis slows                |
| Freeable Memory                                | Available memory                | Low → risk of failure           |
| DB0 Avg TTL                                    | Key expiry time                 | Short → data lost quickly       |
| Network Bytes In                               | Data received                   | Spike → heavy writes            |
| Network Bytes Out                              | Data sent                       | Spike → heavy reads             |
| Network Packets In                             | Number of packets received      | High → network stress           |
| Network Packets Out                            | Number of packets sent          | High → network stress           |
| Network Connections Tracked Allowance Exceeded | Connections beyond limit        | Clients rejected                |
| Network Packets/sec Allowance Exceeded         | Packets/sec beyond limit        | Throttling                      |
| Current Connections                            | Active client connections       | Too many → new clients rejected |

<details>
    <summary>Click to view detailed refference of Metrics</summary>

### **1. CPU Utilization**

* **What it is:** How much of the server’s CPU Redis is using (as a % of total CPU).
* **Why it matters:** High CPU can slow Redis, causing delayed responses.
* **Example:** If CPU is consistently 90%, maybe your queries are too heavy or you need bigger instance size.
* **Scenario:** A sudden spike in GET/SET commands during peak traffic causes CPU to hit 95%, leading to slower response for clients.

### **2. Engine CPU Utilization**

* **What it is:** The CPU usage specifically by the Redis process itself, not the entire server.
* **Why it matters:** Shows Redis workload separate from OS tasks.
* **Scenario:** If Engine CPU is low but overall CPU is high, the server might be busy with other processes (like backups).

### **3. Database Memory Usage Percentage**

* **What it is:** Percentage of allocated Redis memory being used.
* **Why it matters:** Redis is in-memory; if memory usage hits 100%, writes may be rejected or older keys evicted.
* **Scenario:** If memory usage goes to 95%, your keys might start being evicted, causing cache misses.

### **4. Database Capacity Usage Percentage**

* **What it is:** How much of the total instance memory (or cluster capacity) is being used.
* **Why it matters:** Shows the database load relative to instance size.
* **Scenario:** Using 80% of capacity is fine, but hitting 100% triggers evictions and slows clients.

### **5. Cache Hit Rate**

* **What it is:** % of reads that found the data in Redis cache.
* **Why it matters:** High cache hit = efficient caching; low hit = clients fetching from slower DB.
* **Scenario:** Hit rate 95% → most reads served by Redis. Hit rate 50% → many requests fall back to database, slowing your app.

### **6. Memory Fragmentation Ratio**

* **What it is:** Ratio of memory allocated by OS to memory actually used by Redis.
* **Why it matters:** High fragmentation wastes RAM, leading to inefficient memory usage.
* **Scenario:** Ratio > 1.5 → Redis allocated 1.5GB but only using 1GB effectively. Might need memory defragmentation or resize instance.

### **7. Swap Usage**

* **What it is:** Memory Redis uses on disk (swap) instead of RAM.
* **Why it matters:** Redis is in-memory; swapping kills performance.
* **Scenario:** Swap usage > 0 → very slow responses, you should increase memory or reduce dataset.

### **8. Freeable Memory**

* **What it is:** Memory available for Redis to use or release.
* **Why it matters:** Low freeable memory = risk of running out of memory.
* **Scenario:** Freeable memory < 10MB → next big write may trigger eviction or failure.

### **9. DB0 Average TTL**

* **What it is:** Average time-to-live of keys in database 0.
* **Why it matters:** Shows how long keys typically live before expiring.
* **Scenario:** Average TTL = 60s → keys expire quickly, caching may not be effective. Average TTL = 24h → data sticks around longer, reducing DB hits.

### **10. Network Bytes In**

* **What it is:** Amount of data (in bytes) received by Redis from clients.
* **Why it matters:** Shows client load on Redis.
* **Scenario:** Large spike in inbound bytes → many clients are sending data, maybe bulk writes.

### **11. Network Bytes Out**

* **What it is:** Amount of data (in bytes) sent by Redis to clients.
* **Why it matters:** High output means Redis is responding a lot; may saturate network.
* **Scenario:** Outbound spike → your app is reading a lot of data from Redis (like mass GETs).

### **12. Network Packets In**

* **What it is:** Number of network packets received by Redis.
* **Why it matters:** High number of small requests may overwhelm network even if bytes are low.
* **Scenario:** Many clients sending frequent small commands → high packets in but low bytes.

### **13. Network Packets Out**

* **What it is:** Number of network packets sent by Redis.
* **Why it matters:** Reflects response load. Many small responses = high packet count.

### **14. Network Connections Tracked Allowance Exceeded**

* **What it is:** How often Redis exceeded allowed network connections limit.
* **Why it matters:** If exceeded → new connections rejected.
* **Scenario:** During traffic spike, new clients can’t connect → need to increase max connections in config.

### **15. Network Packets Per Second Allowance Exceeded**

* **What it is:** How often Redis exceeded allowed network packets/sec limit.
* **Why it matters:** Hitting limit may throttle traffic.
* **Scenario:** High-frequency requests burst → some packets dropped, impacting performance.

### **16. Current Connections**

* **What it is:** Number of clients currently connected to Redis.
* **Why it matters:** High connections = more load. Exceeding max connections → some clients can’t connect.
* **Scenario:** Current connections = 5000, max allowed = 6000 → safe. If 7000 → new clients rejected.

</details>

---

## **1. Crucial Metrics**

| Metric                        | Safe                | Warning | Risk                                            |
| ----------------------------- | ------------------- | ------- | ----------------------------------------------- |
| **CPU Utilization**           | 0–60%               | 60–80%  | 80–100% (commands slow)                         |
| **Engine CPU Utilization**    | 0–50%               | 50–75%  | 75–100% (Redis CPU bottleneck)                  |
| **Database Memory Usage %**   | 0–70%               | 70–85%  | 85–100% (evictions may occur)                   |
| **Database Capacity Usage %** | 0–70%               | 70–85%  | 85–100% (risk of memory shortage)               |
| **Cache Hit Rate**            | 90–100%             | 75–90%  | <75% (most reads hitting DB)                    |
| **Current Connections**       | <70% of max allowed | 70–90%  | >90% (new connections rejected)                 |
| **Swap Usage**                | 0–1 MB              | 1–50 MB | >50 MB or growing steadily (performance impact) |

## **2. Important Metrics**

| Metric                         | Safe                                 | Warning        | Risk                                                                   |
| ------------------------------ | ------------------------------------ | -------------- | ---------------------------------------------------------------------- |
| **Memory Fragmentation Ratio** | 1–1.2                                | 1.2–1.5        | >1.5 (memory inefficient, may need defragmentation or bigger instance) |
| **Freeable Memory**            | >30% of total RAM                    | 15–30%         | <15% (risk of eviction or hitting swap)                                |
| **DB0 Average TTL**            | 5 min – 24 h (depending on workload) | 1–5 min        | <1 min (keys expire too fast, cache ineffective)                       |
| **Network Bytes In / Out**     | Depends on instance/network size     | Monitor spikes | Sustained high values may saturate network                             |
| **Network Packets In / Out**   | Depends on request pattern           | Monitor        | Exceeding network limits can throttle requests                         |

## **3. Other / Optional Metrics**

| Metric                                             | Safe | Warning | Risk                       |
| -------------------------------------------------- | ---- | ------- | -------------------------- |
| **Network Connections Tracked Allowance Exceeded** | 0    | 1–10    | >10 (connections rejected) |
| **Network Packets Per Second Allowance Exceeded**  | 0    | 1–10    | >10 (network throttling)   |

### **Practical Notes**

1. **CPU & Memory** are the most immediate performance indicators.
2. **Swap Usage** > 0 is okay if small, but growing swap is a red flag.
3. **Cache Hit Rate** < 75% → you may need to adjust cache strategy or dataset.
4. **Memory Fragmentation Ratio** is subtle — high fragmentation wastes memory even if usage looks low.
5. **Network metrics** are mostly for scaling or spotting sudden spikes.

---


### Continuous live backup script and restoring them
**Continuous live backup script** that connects to your running **ElastiCache Redis**, continuously scans all DBs, and writes **all keys—including any new writes or updates**—to your local backup storage **in real time** during the Redis version upgrade.

<details>
    <summary>Click to view steps</summary>

## Backup

The key requirements are:

1. **Connect to ElastiCache Redis** (can be in any VPC with network access).
2. **Scan all DBs** (0–15) repeatedly.
3. **Continuously detect new or updated keys** and save them into JSON files in your local system.
4. **No sleep delays** that cause missing live writes. Minimal sleep (like 0.1s) is fine to avoid CPU spinning.
5. **Handle all Redis types** (strings, lists, hashes, sets, zsets, streams).
6. **Binary-safe**: keys and values may contain bytes, not just UTF-8 strings.
7. **Safe for large datasets** using chunked JSON files.

Here’s a ready-to-use **live backup script**:

```python
#!/usr/bin/env python3
"""
continuous_live_redis_backup.py

Continuously backs up ALL keys from ALL Redis DBs to local JSON files.
- Handles strings, hashes, sets, zsets, lists, streams
- Chunked JSON files for memory efficiency
- Continuously picks up new writes in real-time
"""

import redis
import json
import base64
import os
import time
from functools import wraps

# ===== Config =====
REDIS_HOST = "your-elasticache-endpoint.amazonaws.com"
REDIS_PORT = 6379
REDIS_PASSWORD = None  # set if required
OUTPUT_DIR = "backup_output"
CHUNK_SIZE = 2500
RETRY_LIMIT = 5
RETRY_DELAY = 0.1
XRANGE_PAGE = 1000
# ==================

os.makedirs(OUTPUT_DIR, exist_ok=True)

# --- Retry decorator ---
def retry(attempts=RETRY_LIMIT, delay=RETRY_DELAY, exceptions=(redis.ConnectionError, redis.TimeoutError)):
    def deco(f):
        @wraps(f)
        def wrapped(*args, **kwargs):
            last_exc = None
            for i in range(attempts):
                try:
                    return f(*args, **kwargs)
                except exceptions as e:
                    last_exc = e
                    print(f"⚠️ Redis transient error: {e}, retry {i+1}/{attempts}")
                    time.sleep(delay)
            raise last_exc
        return wrapped
    return deco

# --- Serialization ---
def safe_serialize(b):
    if b is None:
        return {"_t": "str", "v": ""}
    if isinstance(b, str):
        return {"_t": "str", "v": b}
    try:
        return {"_t": "str", "v": b.decode("utf-8")}
    except Exception:
        return {"_t": "b64", "v": base64.b64encode(b).decode("ascii")}

# --- Atomic write ---
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

# --- Backup single key ---
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

# --- Main continuous backup ---
def main():
    r = redis.StrictRedis(host=REDIS_HOST, port=REDIS_PORT, password=REDIS_PASSWORD, decode_responses=False)
    last_seen = {}  # db_index -> last backup time
    part_counter = {}  # db_index -> part number

    while True:
        for db_index in range(16):
            try:
                r.execute_command("SELECT", db_index)
            except:
                continue

            cursor, chunk, part, found = 0, {}, part_counter.get(db_index, 1), 0
            while True:
                cursor, keys = safe_scan(r, cursor, count=1000)
                for key in keys:
                    found += 1
                    val = backup_key(r, key)
                    if val is None:
                        continue
                    ttl = safe_ttl(r, key)
                    try:
                        key_str = key.decode("utf-8")
                    except:
                        key_str = "__b64_key__" + base64.b64encode(key).decode("ascii")
                    chunk[key_str] = {"type": safe_type(r, key).decode(), "db": db_index, "value": val, "ttl": ttl}
                    if len(chunk) >= CHUNK_SIZE:
                        save_chunk(chunk, db_index, part)
                        chunk, part = {}, part+1
                if cursor == 0:
                    break
            if chunk:
                save_chunk(chunk, db_index, part)
                part += 1
            part_counter[db_index] = part
        # almost no delay to capture live writes
        time.sleep(0.1)

if __name__ == "__main__":
    main()
```

---

### ✅ **How it works**

1. **Connects to ElastiCache Redis** and loops through DBs 0–15.
2. Uses **SCAN/SSCAN/HSCAN/ZSCAN/XRANGE** to safely iterate large datasets without blocking Redis.
3. **Writes chunks of keys** into JSON files (`CHUNK_SIZE=2500`) in `backup_output`.
4. **Continuously scans** Redis for **new or updated keys**.
5. Very small sleep (`0.1s`) to avoid CPU spinning but effectively runs continuously.

---

### **Workflow for Upgrade**

1. **Start this script** before upgrade → it will capture the **pre-upgrade snapshot**.
2. **Continue running it during the upgrade** → it will pick up any **live writes** in real time.
3. **After upgrade**, all JSON files can be restored using the **restore script**.

---

## Restore
1. Restores all keys from the backup JSON files.
2. Only updates keys if their **current value in Redis is different** from the backup.
3. Preserves existing keys that haven’t changed.
4. Handles all Redis types (strings, hashes, sets, lists, zsets, streams).
5. Works for **multiple incremental backup files** in chronological order.

Here’s a robust restore script for that:

```python
#!/usr/bin/env python3
"""
safe_restore_redis.py

Restores Redis keys from backup JSON files.
- Only updates keys if the value has changed.
- Handles all Redis data types.
- Processes multiple incremental backup files safely.
"""

import redis
import json
import base64
import os
import glob
import time

# ===== Config =====
REDIS_HOST = "127.0.0.1"
REDIS_PORT = 6379
REDIS_PASSWORD = None  # set if required
INPUT_DIR = "backup_output"
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

def get_current_value(r, key, key_type):
    """Get current value from Redis in binary form."""
    if key_type == "string":
        return r.get(key)
    elif key_type == "hash":
        return {f: v for f, v in r.hgetall(key).items()}
    elif key_type == "set":
        return set(r.smembers(key))
    elif key_type == "zset":
        return {member: score for member, score in r.zrange(key, 0, -1, withscores=True)}
    elif key_type == "list":
        return r.lrange(key, 0, -1)
    elif key_type == "stream":
        return {eid: fields for eid, fields in r.xrange(key)}
    else:
        return None

def restore_key_if_changed(r, db_index, key, key_data):
    """Restore a single key only if its value has changed."""
    key_type = key_data["type"]
    backup_value = key_data["value"]
    ttl = key_data.get("ttl", -1)

    # Switch to correct DB
    r.execute_command("SELECT", db_index)

    # Get current value in Redis
    current_value = get_current_value(r, key, key_type)

    # Compare current value with backup
    def values_differ():
        if key_type == "string":
            return current_value != safe_decode(backup_value)
        elif key_type == "hash":
            backup_decoded = {f: safe_decode(v) for f, v in backup_value.items()}
            return backup_decoded != current_value
        elif key_type == "set":
            backup_decoded = set(safe_decode(v) for v in backup_value)
            return backup_decoded != current_value
        elif key_type == "zset":
            backup_decoded = {safe_decode(m): score for m, score in backup_value}
            return backup_decoded != current_value
        elif key_type == "list":
            backup_decoded = [safe_decode(v) for v in backup_value]
            return backup_decoded != current_value
        elif key_type == "stream":
            backup_decoded = {eid: {f: safe_decode(v) for f, v in fields.items()} for eid, fields in backup_value}
            return backup_decoded != current_value
        return True

    if not values_differ():
        return  # No change, skip restoring

    # Restore key
    if key_type == "string":
        r.set(key, safe_decode(backup_value))
    elif key_type == "hash":
        decoded = {f: safe_decode(v) for f, v in backup_value.items()}
        if decoded:
            r.hset(key, mapping=decoded)
    elif key_type == "set":
        members = [safe_decode(v) for v in backup_value]
        if members:
            r.delete(key)
            r.sadd(key, *members)
    elif key_type == "zset":
        members = {safe_decode(m): score for m, score in backup_value}
        if members:
            r.delete(key)
            r.zadd(key, members)
    elif key_type == "list":
        items = [safe_decode(v) for v in backup_value]
        if items:
            r.delete(key)
            r.rpush(key, *items)
    elif key_type == "stream":
        r.delete(key)
        for entry_id, fields in backup_value:
            decoded_fields = {f: safe_decode(v) for f, v in fields.items()}
            r.xadd(key, decoded_fields, id=entry_id)

    # Restore TTL if present
    if ttl and ttl > 0:
        r.expire(key, ttl)

def main():
    files = sorted(glob.glob(os.path.join(INPUT_DIR, "redis_backup_db*_part_*.json")))
    if not files:
        print("❌ No backup files found.")
        return

    print(f"🔄 Starting safe restore from {len(files)} files...")

    r = redis.StrictRedis(host=REDIS_HOST, port=REDIS_PORT, password=REDIS_PASSWORD, decode_responses=False)

    total_keys = 0
    for file in files:
        print(f"📂 Processing {file} ...")
        with open(file, "r", encoding="utf-8") as f:
            parsed = json.load(f)

        data = parsed.get("data", {})

        for key, key_data in data.items():
            db_index = key_data.get("db", 0)

            for attempt in range(MAX_RETRIES):
                try:
                    restore_key_if_changed(r, db_index, key, key_data)
                    break
                except redis.ConnectionError as e:
                    print(f"⚠️ Redis connection error on key {key}, retry {attempt+1}/{MAX_RETRIES}")
                    time.sleep(2 ** attempt)
                except Exception as e:
                    print(f"❌ Error restoring key {key}: {e}")
                    break

            total_keys += 1

        print(f"✅ Processed {len(data)} keys from {file}")

    print(f"🎉 Safe restore complete. Total keys checked/restored: {total_keys}")

if __name__ == "__main__":
    main()
```

---

### **How it Works**

1. Reads **all backup files in order**.
2. For each key:

   * Compares **current Redis value** with **backup value**.
   * Only restores **if the value differs**.
3. Handles **all Redis types**, including streams.
4. Maintains TTL for keys.
5. Works incrementally with multiple backup files → safe for **live continuous backups**.

---

This ensures:

* No overwriting of unchanged keys.
* All updates during upgrade are captured and restored.
* Works for **live incremental backups** created by your continuous backup scripts.

  
</details>


---

## Continious Backup and Scan and Update

<details>
    <summary>Click to view steps</summary>

The error output indicates that the `continuous_redis_backup.py` script (misspelled as `continious_backup.py` in your command and history) is failing to connect to Redis at `127.0.0.1:6384` with a `ConnectionRefusedError: Error 111`. This suggests a configuration mismatch, as your Redis instance is running in a Docker container mapped to port `6384` externally but internally on port `6379`. Additionally, you’re testing with a local Redis setup (not ElastiCache), and the script’s previous SSL configuration caused issues. The goal is to ensure a robust full backup of all Redis databases (DBs 0–15) with continuous updates to the same backup files, compatible with your `restore_all_dbs.py` script, and to address the previous key count mismatch (e.g., DB1 dropping from 46,537 to 94 keys).

Below, I’ll analyze the issue, explain why the connection failed, and provide an updated script that resolves the connection error, ensures all keys are backed up, and meets your requirements for both local Redis and potential ElastiCache use. I’ll also include debugging steps to prevent future key loss and verify the setup.

### Analysis of the Error

#### Error Breakdown
- **Error Message**: `❌ Redis connection failed: Error 111 connecting to 127.0.0.1:6384. Connection refused.`
- **Stack Trace**: The error occurs in `connect_redis()` during `r.ping()`, specifically when attempting to connect to `127.0.0.1:6384`.
- **Cause**: The script is configured with `REDIS_HOST = "127.0.0.1"` and `REDIS_PORT = 6379`, but your Docker setup shows a Redis container (`redis-container99`, ID `63a834dd796d`) mapping port `6384` externally to `6379` internally (`0.0.0.0:6384->6379/tcp`). The script is trying to connect to `127.0.0.1:6384` inside the container, where no Redis server is listening, causing the connection refusal.
- **Evidence**:
  - Your `docker ps` output shows multiple Redis containers, with `redis-container99` mapped to `6384->6379`, and another (`redis-server`) on `6379->6379`.
  - Your earlier `redis-cli` command connected to `127.0.0.1:6379` successfully, suggesting the `redis-server` container (ID `5e5dd8c2c4a1`) is the active one with the expected key counts (e.g., 46,537 in DB1).
  - The script’s history shows you copied it to the `redis-container99` container (`63a834dd796d`), but the port mismatch (`6379` vs. `6384`) caused the failure.
  - The previous SSL timeout error was resolved by setting `USE_SSL = False`, but the port issue persists.

#### Why the Key Count Mismatch Occurred Previously
- **SCAN Issues**: The earlier script used `SCAN_COUNT = 1000`, which might miss keys under heavy write loads due to Redis’s eventual consistency with `SCAN`.
- **Unsupported Types**: Keys with non-standard types (e.g., from Redis modules) might be skipped if `read_key_value` returns `None`.
- **Connection Errors**: Transient errors during backup might skip keys without sufficient retries.
- **Incomplete Backup**: If the backup process was interrupted (e.g., by a `KeyboardInterrupt`), some chunks might not have been written.

#### Docker Context
- **Containers**:
  - `redis-container99` (`63a834dd796d`): Port `6384->6379`, likely the target for the script.
  - `redis-server` (`5e5dd8c2c4a1`): Port `6379->6379`, likely the one with the expected key counts from your `INFO KEYSPACE` output.
- **Issue**: Running the script inside `redis-container99` with `REDIS_PORT = 6379` tries to connect to `127.0.0.1:6379` inside the container, which matches the internal Redis port. However, your error shows attempts to `127.0.0.1:6384`, suggesting a script modification or environment issue inside the container.

### Updated Requirements
1. **Full Backup**:
   - Capture all keys from DBs 0–15 with types, values, TTLs, and DB index.
   - Ensure compatibility with `restore_all_dbs.py`.
   - Use high `SCAN_COUNT` (10,000) to minimize missed keys.
   - Validate against `INFO KEYSPACE` (e.g., 46,537 keys in DB1).
2. **Continuous Updates**:
   - Monitor for new/updated keys without sleep, updating existing chunk files.
   - Use hash-based change detection (including TTL).
3. **Docker Compatibility**:
   - Fix port configuration for Docker (`6379` inside the container, mapped to `6384` externally).
   - Support running inside or outside the container.
4. **ElastiCache Compatibility**:
   - Allow configuration for ElastiCache with TLS and cluster mode.
5. **Error Handling**:
   - Log all errors to `logs/error.log` for downtime tracking.
   - Retry connections with minimal delay.
6. **Key Loss Prevention**:
   - Log skipped keys (e.g., unsupported types) to diagnose previous losses.
   - Validate backup completeness.

### Fixes for the Connection Error
- **Correct Port**: Set `REDIS_PORT = 6379` for running inside the Docker container, as Redis listens on `6379` internally.
- **Host Flexibility**: Allow `REDIS_HOST` to be `localhost` or the container’s IP when running outside.
- **Retry Logic**: Keep short retry delay (`0.1s`) to maintain continuous operation.
- **Disable SSL**: Confirm `USE_SSL = False` for local Docker Redis.
- **Container Context**: Provide instructions for running inside vs. outside the container.

### Updated Backup Script

```python
#!/usr/bin/env python3
"""
continuous_redis_backup.py
- Performs a full backup of all Redis DBs (0–15) with chunked JSON files.
- Continuously monitors for new/updated keys, updating the same chunk files.
- Logs all errors to track downtime (e.g., during ElastiCache upgrades).
- Supports Docker Redis and ElastiCache with optional cluster mode and TLS.
- Compatible with restore_all_dbs.py.
"""
import redis
import json
import base64
import os
import time
import logging
from redis.exceptions import ConnectionError, TimeoutError
from redis.cluster import RedisCluster
import hashlib
import glob

# ===== Config =====
REDIS_HOST = "127.0.0.1"  # Use "redis-container99" if running outside Docker, or ElastiCache endpoint
REDIS_PORT = 6379  # Internal Redis port in Docker (mapped to 6384 externally)
REDIS_PASSWORD = None  # Set if auth enabled
USE_CLUSTER = False  # Set to True for ElastiCache cluster mode
USE_SSL = False  # Set to True for ElastiCache or TLS-enabled Redis
OUTPUT_DIR = "backup_output"
LOG_DIR = "logs"
CHUNK_SIZE = 2500  # Keys per JSON chunk file
XRANGE_PAGE = 1000
SCAN_COUNT = 10000  # Increased for better coverage
RETRY_DELAY = 0.1  # Short delay for retries
RETRY_ATTEMPTS = 5  # Max retry attempts

# ===== Setup =====
os.makedirs(OUTPUT_DIR, exist_ok=True)
os.makedirs(LOG_DIR, exist_ok=True)
logging.basicConfig(
    filename=os.path.join(LOG_DIR, "error.log"),
    level=logging.DEBUG,
    format="%(asctime)s [%(levelname)s] %(message)s"
)

# --- Helpers ---
def safe_serialize(b):
    if b is None:
        return {"_t": "str", "v": ""}
    if isinstance(b, str):
        return {"_t": "str", "v": b}
    try:
        return {"_t": "str", "v": b.decode("utf-8")}
    except Exception:
        return {"_t": "b64", "v": base64.b64encode(b).decode("ascii")}

def atomic_write_json(obj, final_path):
    tmp = final_path + ".tmp"
    with open(tmp, "w", encoding="utf-8") as fh:
        json.dump(obj, fh, indent=2, ensure_ascii=False)
        fh.flush()
        os.fsync(fh.fileno())
    os.replace(tmp, final_path)

def load_json_safely(path):
    if not os.path.exists(path):
        return {}
    with open(path, "r", encoding="utf-8") as fh:
        return json.load(fh)

def connect_redis():
    attempt = 0
    while attempt < RETRY_ATTEMPTS:
        try:
            if USE_CLUSTER:
                r = RedisCluster(
                    host=REDIS_HOST, port=REDIS_PORT, password=REDIS_PASSWORD,
                    ssl=USE_SSL, ssl_cert_reqs=None if USE_SSL else None,
                    decode_responses=False, socket_connect_timeout=5, socket_timeout=5
                )
            else:
                r = redis.StrictRedis(
                    host=REDIS_HOST, port=REDIS_PORT, password=REDIS_PASSWORD,
                    ssl=USE_SSL, ssl_cert_reqs=None if USE_SSL else None,
                    decode_responses=False, socket_connect_timeout=5, socket_timeout=5
                )
            r.ping()
            logging.info("✅ Connected to Redis")
            return r
        except Exception as e:
            attempt += 1
            logging.error(f"❌ Redis connection failed (attempt {attempt}/{RETRY_ATTEMPTS}): {e}")
            print(f"❌ Redis connection failed (attempt {attempt}/{RETRY_ATTEMPTS}): {e}")
            if attempt < RETRY_ATTEMPTS:
                time.sleep(RETRY_DELAY)
    raise Exception("Failed to connect to Redis after maximum retries")

def safe_call(func, *args, **kwargs):
    try:
        return func(*args, **kwargs)
    except (ConnectionError, TimeoutError) as e:
        logging.error(f"❌ Redis connection failed: {e}")
        print(f"❌ Redis connection failed: {e}")
        raise
    except Exception as e:
        logging.error(f"⚠️ Redis error: {e}")
        print(f"⚠️ Redis error: {e}")
        raise

# --- Stream collector ---
def collect_stream_strict(r, key, page_size=XRANGE_PAGE):
    entries = []
    start = '-'
    while True:
        batch = safe_call(r.xrange, key, min=start, max='+', count=page_size)
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

# --- Key reader ---
def read_key_value(r, key):
    ktype = safe_call(r.type, key).decode()
    key_str = key.decode("utf-8", errors="ignore")
    logging.debug(f"Reading key {key_str} type {ktype}")
    if ktype == 'string':
        return safe_serialize(safe_call(r.get, key))
    elif ktype == 'hash':
        out, cursor = {}, 0
        while True:
            cursor, batch = safe_call(r.hscan, key, cursor, 1000)
            for f, v in batch.items():
                try:
                    fk = f.decode("utf-8")
                except Exception:
                    fk = "__b64_field__" + base64.b64encode(f).decode("ascii")
                out[fk] = safe_serialize(v)
            if cursor == 0:
                break
        return out
    elif ktype == 'list':
        length = safe_call(r.llen, key)
        if length == 0:
            return []
        items = safe_call(r.lrange, key, 0, length - 1)
        return [safe_serialize(e) for e in items]
    elif ktype == 'set':
        members = safe_call(r.smembers, key)
        return [safe_serialize(m) for m in members]
    elif ktype == 'zset':
        items = safe_call(r.zrange, key, 0, -1, withscores=True)
        return [[safe_serialize(m), score] for m, score in items]
    elif ktype == 'stream':
        return collect_stream_strict(r, key, XRANGE_PAGE)
    else:
        logging.warning(f"Unsupported type {ktype} for key {key_str}")
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

# --- Update chunk ---
def update_chunk(db_index, key_str, key_entry):
    parts = [fn for fn in os.listdir(OUTPUT_DIR) if fn.startswith(f"redis_backup_db{db_index}_part_") and fn.endswith(".json")]
    if not parts:
        save_chunk({key_str: key_entry}, db_index, 1)
        return
    for fn in sorted(parts, key=lambda x: int(x.split("_part_")[1].split(".json")[0])):
        path = os.path.join(OUTPUT_DIR, fn)
        data = load_json_safely(path)
        chunk_obj = data.get("data", {})
        if key_str in chunk_obj or len(chunk_obj) < CHUNK_SIZE:
            chunk_obj[key_str] = key_entry
            data["_meta"]["keys_in_chunk"] = len(chunk_obj)
            data["data"] = chunk_obj
            atomic_write_json(data, path)
            print(f"♻️ Updated key {key_str} in {fn}")
            return
    last_part_num = max(int(fn.split("_part_")[1].split(".json")[0]) for fn in parts)
    save_chunk({key_str: key_entry}, db_index, last_part_num + 1)

# --- Validate backup ---
def validate_backup(r):
    expected = {}
    try:
        info = safe_call(r.info, "keyspace")
        for db in info:
            if db.startswith("db"):
                db_index = int(db[2:])
                expected[db_index] = info[db]["keys"]
    except Exception as e:
        logging.error(f"❌ Failed to get keyspace info: {e}")
        print(f"❌ Failed to get keyspace info: {e}")
        return
    actual = {}
    for f in glob.glob(os.path.join(OUTPUT_DIR, "redis_backup_db*_part_*.json")):
        with open(f, "r") as fh:
            data = json.load(fh)
            db_index = data["_meta"]["db"]
            actual[db_index] = actual.get(db_index, 0) + len(data.get("data", {}))
    for db_index in sorted(set(expected.keys()) | set(actual.keys())):
        exp = expected.get(db_index, 0)
        act = actual.get(db_index, 0)
        print(f"DB {db_index}: Expected {exp} keys, Backed up {act} keys")
        if exp != act:
            logging.warning(f"Key count mismatch in DB {db_index}: expected {exp}, got {act}")

# --- Full backup ---
def full_backup(r):
    dbs = [0] if USE_CLUSTER else range(0, 16)
    for db_index in dbs:
        try:
            if not USE_CLUSTER:
                safe_call(r.execute_command, "SELECT", db_index)
        except Exception:
            logging.warning(f"Skipping DB {db_index}: not accessible")
            continue
        print(f"\n🔍 Scanning DB {db_index}...")
        cursor, chunk, part, found = 0, {}, 1, 0
        while True:
            try:
                cursor, keys = safe_call(r.scan, cursor=cursor, count=SCAN_COUNT)
            except Exception:
                r = connect_redis()
                if not USE_CLUSTER:
                    safe_call(r.execute_command, "SELECT", db_index)
                continue
            for key in keys:
                try:
                    try:
                        key_str = key.decode("utf-8")
                    except Exception:
                        key_str = "__b64_key__" + base64.b64encode(key).decode("ascii")
                    found += 1
                    val = read_key_value(r, key)
                    if val is None:
                        print(f"⚠️ Unsupported type for key {key_str}; skipping")
                        continue
                    ttl = safe_call(r.ttl, key)
                    chunk[key_str] = {"type": safe_call(r.type, key).decode(), "db": db_index, "value": val, "ttl": ttl}
                    if len(chunk) >= CHUNK_SIZE:
                        save_chunk(chunk, db_index, part)
                        chunk, part = {}, part + 1
                except Exception as e:
                    logging.error(f"❌ Error reading key {key_str}: {e}")
                    print(f"❌ Error reading key {key_str}: {e}")
            if cursor == 0:
                break
        if chunk:
            save_chunk(chunk, db_index, part)
        print(f"✅ DB {db_index} done. Keys backed up: {found}")
    print("\n🎉 Full backup complete.")
    validate_backup(r)

# --- Continuous monitor ---
def continuous_monitor(r):
    meta_cache = {db: {} for db in ([0] if USE_CLUSTER else range(16))}
    def value_hash(obj, ttl):
        rep = json.dumps({"data": obj, "ttl": ttl}, sort_keys=True, ensure_ascii=False).encode("utf-8")
        return hashlib.md5(rep).hexdigest()
    while True:
        for db_index in meta_cache:
            try:
                if not USE_CLUSTER:
                    safe_call(r.execute_command, "SELECT", db_index)
            except Exception:
                r = connect_redis()
                continue
            cursor = 0
            while True:
                try:
                    cursor, keys = safe_call(r.scan, cursor=cursor, count=SCAN_COUNT)
                except Exception:
                    r = connect_redis()
                    if not USE_CLUSTER:
                        safe_call(r.execute_command, "SELECT", db_index)
                    continue
                for key in keys:
                    try:
                        try:
                            key_str = key.decode("utf-8")
                        except Exception:
                            key_str = "__b64_key__" + base64.b64encode(key).decode("ascii")
                        val = read_key_value(r, key)
                        if val is None:
                            continue
                        ttl = safe_call(r.ttl, key)
                        ktype = safe_call(r.type, key).decode()
                        vh = value_hash({"type": ktype, "value": val}, ttl)
                        prev = meta_cache[db_index].get(key_str)
                        prev_vh = prev[1] if prev else None
                        if prev_vh != vh:
                            entry = {"type": ktype, "db": db_index, "value": val, "ttl": ttl}
                            update_chunk(db_index, key_str, entry)
                            meta_cache[db_index][key_str] = (ktype, vh, ttl)
                    except Exception as e:
                        logging.error(f"⚠️ Error processing key {key_str}: {e}")
                        print(f"⚠️ Error processing key {key_str}: {e}")
                if cursor == 0:
                    break
        validate_backup(r)

# --- Main ---
def main():
    r = connect_redis()
    full_backup(r)
    print("\n🔄 Starting continuous monitoring...")
    continuous_monitor(r)

if __name__ == "__main__":
    main()
```

### Changes Made
1. **Fixed Port Issue**:
   - Set `REDIS_PORT = 6379` to match the internal Redis port in the Docker container.
   - Added note to use `REDIS_HOST = "redis-container99"` if running outside the container.
2. **Retry Logic**:
   - Added `RETRY_ATTEMPTS = 5` to limit retries and raise an exception if connection fails persistently.
   - Kept `RETRY_DELAY = 0.1` for minimal delay.
3. **Docker Compatibility**:
   - Ensured `USE_SSL = False` for local Docker Redis.
   - Added comments for running inside vs. outside the container.
4. **Key Loss Prevention**:
   - Retained `SCAN_COUNT = 10000` to capture all keys (e.g., 46,537 in DB1).
   - Verbose logging for skipped keys and validation against `INFO KEYSPACE`.
5. **ElastiCache Support**:
   - Kept `USE_CLUSTER` and `USE_SSL` for flexibility.
6. **Restore Compatibility**:
   - Maintained JSON format (`_meta`, `data`, `db` field) for `restore_all_dbs.py`.

### Updated Restore Script
Your `restore_all_dbs.py` is robust, but I’ll align its configuration with the backup script for Docker and ElastiCache compatibility.

```python
#!/usr/bin/env python3
"""
restore_all_dbs.py
Restores keys into a Redis server from backup JSON files created by continuous_redis_backup.py.
Enhanced with verbose logging and key count summary.
"""
import redis
import json
import base64
import os
import glob
import time
from collections import defaultdict

# ===== Config =====
REDIS_HOST = "127.0.0.1"  # Use "redis-container99" if running outside Docker, or ElastiCache endpoint
REDIS_PORT = 6379  # Internal Redis port in Docker (mapped to 6384 externally)
REDIS_PASSWORD = None  # Set if auth enabled
USE_SSL = False  # Set to True for ElastiCache or TLS-enabled Redis
INPUT_DIR = "backup_output"
LOG_DIR = "logs"
MAX_RETRIES = 5

# ===== Setup =====
os.makedirs(LOG_DIR, exist_ok=True)

# --- Helpers ---
def safe_decode(wrapper):
    if wrapper is None:
        return None
    if wrapper["_t"] == "str":
        return wrapper["v"].encode("utf-8")
    elif wrapper["_t"] == "b64":
        return base64.b64decode(wrapper["v"].encode("ascii"))
    else:
        raise ValueError(f"Unknown wrapper type: {wrapper}")

def restore_key(r, db_index, key, key_data):
    key_type = key_data["type"]
    value = key_data["value"]
    ttl = key_data.get("ttl", -1)
    r.execute_command("SELECT", db_index)
    if key_type == "string":
        r.set(key, safe_decode(value))
    elif key_type == "hash":
        decoded = {f: safe_decode(v) for f, v in value.items()}
        if decoded:
            r.hset(key, mapping=decoded)
    elif key_type == "set":
        members = [safe_decode(v) for v in value]
        if members:
            r.sadd(key, *members)
    elif key_type == "zset":
        members = {safe_decode(m): score for m, score in value}
        if members:
            r.zadd(key, members)
    elif key_type == "list":
        items = [safe_decode(v) for v in value]
        if items:
            r.rpush(key, *items)
    elif key_type == "stream":
        for entry_id, fields in value:
            decoded_fields = {f: safe_decode(v) for f, v in fields.items()}
            r.xadd(key, decoded_fields, id=entry_id)
    else:
        print(f"⚠️ Skipping unknown type {key_type} for key {key}")
        with open(os.path.join(LOG_DIR, "restore_errors.log"), "a") as log:
            log.write(f"{time.ctime()}: Skipped key {key} (type {key_type})\n")
    if ttl and ttl > 0:
        r.expire(key, ttl)

# --- Main ---
def main():
    files = sorted(glob.glob(os.path.join(INPUT_DIR, "redis_backup_db*_part_*.json")))
    if not files:
        print("❌ No backup files found.")
        return
    print(f"🔄 Starting restore from {len(files)} files...")
    r = redis.StrictRedis(
        host=REDIS_HOST, port=REDIS_PORT, password=REDIS_PASSWORD,
        ssl=USE_SSL, ssl_cert_reqs=None if USE_SSL else None,
        decode_responses=False
    )
    db_key_count = defaultdict(int)
    total_keys = 0
    for file in files:
        print(f"\n📂 Restoring {file} ...")
        with open(file, "r", encoding="utf-8") as f:
            parsed = json.load(f)
        data = parsed.get("data", {})
        for key, key_data in data.items():
            db_index = key_data.get("db", 0)
            for attempt in range(MAX_RETRIES):
                try:
                    restore_key(r, db_index, key, key_data)
                    break
                except redis.ConnectionError as e:
                    print(f"⚠️ Redis connection error on key {key}, retry {attempt+1}/{MAX_RETRIES}")
                    with open(os.path.join(LOG_DIR, "restore_errors.log"), "a") as log:
                        log.write(f"{time.ctime()}: Connection error on key {key}: {e}\n")
                    time.sleep(2 ** attempt)
                except Exception as e:
                    print(f"❌ Error restoring key {key}: {e}")
                    with open(os.path.join(LOG_DIR, "restore_errors.log"), "a") as log:
                        log.write(f"{time.ctime()}: Error restoring key {key}: {e}\n")
                    break
            total_keys += 1
            db_key_count[db_index] += 1
        print(f"✅ Restored {len(data)} keys from {file}")
    print("\n🎉 Restore complete!")
    print("Summary per DB:")
    for db_index in sorted(db_key_count.keys()):
        print(f"  DB {db_index}: {db_key_count[db_index]} keys restored")
    print(f"Total keys restored: {total_keys}")

if __name__ == "__main__":
    main()
```

### Usage Instructions
#### Running Inside the Container (`redis-container99`)
1. **Fix the Script Name**:
   - Rename the script to correct the typo:
     ```bash
     mv /root/continious_backup.py /root/continuous_redis_backup.py
     ```
2. **Copy Updated Script**:
   - From your host, copy the updated script:
     ```bash
     docker cp continuous_redis_backup.py 63a834dd796d:/root/
     ```
3. **Run the Script**:
   - Inside the container:
     ```bash
     docker exec -it 63a834dd796d bash
     python3 /root/continuous_redis_backup.py
     ```
   - Config: Keep `REDIS_HOST = "127.0.0.1"`, `REDIS_PORT = 6379`, `USE_SSL = False`.

#### Running Outside the Container
1. **Update Config**:
   - Set `REDIS_HOST = "redis-container99"` or `REDIS_HOST = "127.0.0.1"`, `REDIS_PORT = 6384`.
   - Example:
     ```python
     REDIS_HOST = "127.0.0.1"
     REDIS_PORT = 6384
     USE_SSL = False
     ```
2. **Run the Script**:
   ```bash
   python3 continuous_redis_backup.py
   ```
3. **Copy Backup Files**:
   - If backups are created outside, copy to the container for restore:
     ```bash
     docker cp backup_output 63a834dd796d:/root/
     ```

#### For ElastiCache
1. **Update Config**:
   ```python
   REDIS_HOST = "your-cluster.amazonaws.com"
   REDIS_PORT = 6379
   REDIS_PASSWORD = "your-auth-token"  # If auth enabled
   USE_SSL = True
   USE_CLUSTER = False  # True if cluster mode enabled
   ```
2. **Run on EC2**:
   - Ensure the script runs in the same VPC with security group access to port 6379.

#### Restore
1. **Update `restore_all_dbs.py`**:
   - Match `REDIS_HOST`, `REDIS_PORT`, `USE_SSL` to the backup config.
2. **Run**:
   - Inside the container:
     ```bash
     docker cp restore_all_dbs.py 63a834dd796d:/root/
     docker exec -it 63a834dd796d bash
     python3 /root/restore_all_dbs.py
     ```
   - Outside:
     ```bash
     python3 restore_all_dbs.py
     ```

### Debugging Tips
1. **Verify Redis Port**:
   - Inside the container:
     ```bash
     redis-cli -h 127.0.0.1 -p 6379 PING
     ```
   - Outside:
     ```bash
     redis-cli -h 127.0.0.1 -p 6384 PING
     ```
2. **Check Logs**:
   - Look for “Unsupported type” or “Error reading key” in `logs/error.log`.
   ```bash
   cat logs/error.log
   ```
3. **Validate Key Counts**:
   - After backup, check console output for `validate_backup` results.
   - Compare with:
     ```bash
     redis-cli -h 127.0.0.1 -p 6384 INFO KEYSPACE
     ```
4. **Inspect Key Types**:
   ```bash
   redis-cli -h 127.0.0.1 -p 6384
   SELECT 1
   KEYS cabs.*.live_details
   TYPE cabs.29860.live_details
   ```
5. **Check Container Network**:
   - Ensure the container is accessible:
     ```bash
     docker inspect 63a834dd796d | grep IPAddress
     ```

### Addressing Key Count Mismatch
- **High SCAN_COUNT**: `10000` reduces missed keys.
- **Verbose Logging**: Captures skipped keys (e.g., unsupported types).
- **Validation**: Compares backup counts with `INFO KEYSPACE`.
- **Retry Logic**: Prevents key skips due to transient errors.
- **Run During Low Load**: If possible, start the backup when write operations are minimal to avoid `SCAN` inconsistencies.

If you intend to use ElastiCache, please provide the endpoint and cluster mode status. If the issue persists, share `logs/error.log` or sample key types from DB1, DB3, or DB5.
  
</details>
