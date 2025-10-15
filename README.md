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
