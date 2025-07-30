
# Errors
### 1. App cannot connect to the Amazon ElastiCache Redis endpoint
```
testing.ERROR: Connection timed out [tcp://-------.bp8cjs.ng.0001.aps1.cache.amazonaws.com:6379]
{     "exception": "[object] (Predis\\Connection\\ConnectionException(code: 110): Connection timed out [tcp://------.bp8cjs.ng.0001.aps1.cache.amazonaws.com:6379] at
```
<details>
  <summary>Click to View the Solution</summary>
  
# Redis Connection Timeout – Resolution Steps

**Error Observed**

```
[testing.ERROR] Connection timed out [tcp://mallick-devortest.bp8cjs.ng.0001.aps1.cache.amazonaws.com:6379]
exception: Predis\Connection\ConnectionException(code: 110)
```

---

## Root Cause

The ECS task's security group was **not authorized** to access the Redis port (`6379`) on the ElastiCache (Redis OSS) cluster. As a result, Redis was timing out.

---

## Step-by-Step Resolution

### **1. Identify Redis ElastiCache VPC and Security Group**

* Navigate to: **AWS Console → ElastiCache → Redis → Your Cluster**
* Confirm:

  * **VPC** where Redis is deployed
  * **Security Group** attached to Redis

### **2. Identify ECS Task's Security Group**

* Navigate to: **ECS → Clusters → Tasks**
* Click on the task → Check **Network** section:

  * **VPC** (should match Redis)
  * **Security Group** attached to task ENI

### **3. Modify Redis Security Group Inbound Rules**

* Go to: **EC2 → Security Groups → Redis Security Group**
* Edit **Inbound Rules**:

  * **Type**: Custom TCP
  * **Protocol**: TCP
  * **Port Range**: 6379
  * **Source**: Select **ECS Task's Security Group**
* Save changes

### Outcome:

After adding the ECS task's security group as an allowed source on port **6379**, Laravel was able to connect to Redis successfully.

---

## Notes

* Both ECS and Redis **must be in the same VPC**, or VPC peering must be properly configured.
* Security Groups must allow **bidirectional traffic** (ECS → Redis on 6379).
* No VPC NACLs or route table issues were present.
* This fix applies to Redis **without public access** — common in VPC-isolated environments.

---

</details>

---
