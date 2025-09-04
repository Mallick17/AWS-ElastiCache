# AWS ElastiCache Redis Upgrade – Testing Environment

## 1. Objective
Upgrade the existing **AWS ElastiCache Redis (Testing Environment)** from version **5.0.6** to **7.1**, ensuring compatibility, stability, and no data loss.  

This document outlines the **upgrade steps**, **configuration comparison**, and **validation plan**.

---

## 2. Upgrade Steps

### **Step 1: Take Backup**
- Create a **manual snapshot** of the current Redis cluster before upgrade.
- Verify snapshot is available and restorable.

> **AWS Console**:  
ElastiCache → Redis → Snapshots → Create Snapshot → Name: `pre-upgrade-testing-redis-YYYYMMDD`

---

### **Step 2: Prepare for Upgrade**
- Confirm target version `7.1` is supported.
- Check application/library compatibility with Redis 7.
- Review AWS documentation on [Supported Redis versions](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/supported-engine-versions.html).

---

### **Step 3: Modify Redis Cluster**
- Navigate to **ElastiCache → Redis → Modify Cluster**.
- Change **Engine Version** from `5.0.6` → `7.1`.
- A **new parameter group** (`default.redis7`) will be created automatically.
- Keep all other settings the same (see comparison table below).
- Select **Apply Immediately** (since this is test environment).

---

### **Step 4: Post-Upgrade Validation**
- Verify cluster is in `available` state.
- Confirm **parameter group** and **engine version** have updated.
- Run application smoke tests:
  - `PING`, `SET`, `GET` from Redis client.
  - Application connectivity check.
- Monitor CloudWatch metrics:
  - CPU, Memory, Replica Lag, CurrConnections.
- Review CloudWatch alarms for anomalies.

---

### **Step 5: Rollback Plan (If Needed)**
- Restore from the pre-upgrade snapshot:
  - Create a new cluster from the snapshot with Redis 5.0.6.
  - Update application endpoints to point to restored cluster.
- Note: In-place downgrade is **not supported**, rollback requires snapshot restore.

---

## 3. Configuration Comparison

| **Component**            | **Current (Before Upgrade)**                        | **Target (After Upgrade)**            |
|---------------------------|----------------------------------------------------|---------------------------------------|
| **Engine (Redis/Valkey)**| 5.0.6                                              | 7.1                                   |
| **Cluster Mode**          | Disabled                                           | Same                                  |
| **Instance Type**         | cache.t3.micro                                    | Same                                  |
| **Node Count**            | 1 Primary                                          | Same                                  |
| **Parameter Group**       | default.redis5.0                                   | default.redis7                        |
| **Subnet Group**          | redtaxi-redis                                      | Same                                  |
| **Security Group**        | redtaxi-testing-elasticcache, rt-trsting-config-sg | Same                                  |
| **Encryption**            | At-rest: Enabled<br>In-transit: Enabled            | Same                                  |
| **Maintenance Window**    | Sunday 18:30                                       | Same                                  |
| **Maintenance Duration**  | 1 hour                                             | Same                                  |
| **Encryption at Rest**    | Disabled                                           | Same                                  |
| **Encryption in Transit** | Disabled                                           | Same                                  |
| **Monitoring / Alarms**   | CloudWatch (CPU, Memory, ReplicaLag)               | Same                                  |
| **Auto Upgrade Minor Versions** | Enabled                                    | Same                                  |
| **Amazon SNS Notifications** | Disabled                                       | Same                                  |

---

## 4. Validation Checklist ✅

- [ ] Manual snapshot created and available  
- [ ] Engine version updated to `7.1`  
- [ ] New parameter group applied (`default.redis7`)  
- [ ] Application smoke tests successful  
- [ ] CloudWatch metrics within normal range  
- [ ] No errors/warnings in application logs  


---

# To check all the Elasticache DB Indexes weather running or not in Cron.

<details>
  <summary>View the steps to check all the Elasticache DB Indexes weather running or not in Cron.</summary>

## 🔧 Steps

### 1. Install Redis CLI

Make sure your EC2/bastion/wherever the cron runs has `redis-cli` installed:

```bash
sudo yum install -y redis    # Amazon Linux / RHEL
# or
sudo apt-get install -y redis-tools  # Ubuntu/Debian
```

---

### 2. Create a Health Check Script

Create a script, e.g. `/usr/local/bin/check_redis_dbs.sh`:

```bash
#!/bin/bash
# Check all Redis DB indexes and log result
# Usage: ./check_redis_dbs.sh

REDIS_HOST="your-redis-endpoint.amazonaws.com"
REDIS_PORT=6379
MAX_DB=15   # change if you have more DBs
LOG_FILE="/var/log/redis_db_health.log"

TIMESTAMP=$(date "+%Y-%m-%d %H:%M:%S")

echo "===== Redis DB Health Check at $TIMESTAMP =====" >> $LOG_FILE

for DB in $(seq 0 $MAX_DB); do
    # Try a PING on each DB
    RESPONSE=$(redis-cli -h $REDIS_HOST -p $REDIS_PORT -n $DB PING 2>&1)

    if [ "$RESPONSE" == "PONG" ]; then
        echo "DB $DB: RUNNING" >> $LOG_FILE
    else
        echo "DB $DB: NOT RUNNING ($RESPONSE)" >> $LOG_FILE
    fi
done

echo "" >> $LOG_FILE
```

Make it executable:

```bash
chmod +x /usr/local/bin/check_redis_dbs.sh
```

---

### 3. Add Cron Job

Edit cron with:

```bash
crontab -e
```

Add this line to run every 5 minutes during upgrade (adjust as needed):

```bash
*/5 * * * * /usr/local/bin/check_redis_dbs.sh
```

---

### 4. View Logs

All results will go into:

```
/var/log/redis_db_health.log
```

Sample log:

```
===== Redis DB Health Check at 2025-08-26 11:00:00 =====
DB 0: RUNNING
DB 1: RUNNING
DB 2: NOT RUNNING (Error: Connection reset by peer)
...
```

---

### 5. Stop the running cron

<details>
  <summary>Click to view the steps to stop the cron</summary>

## 5.1. Stop the cron schedule (prevent future runs)

Edit the cron table:

```bash
crontab -e
```

* Find the line:

  ```bash
  */1 * * * * /usr/local/bin/check_redis_dbs.sh
  ```
* Delete it (or comment it out with `#`).
* Save and exit.

This stops **new cron runs** from being scheduled.

---

## 5.2. Kill the currently running cron process (if the script is still running)

First, find the process:

```bash
ps -ef | grep check_redis_dbs.sh
```

Example output:

```
root   12345  1  0 10:02 ?  00:00:00 /bin/bash /usr/local/bin/check_redis_dbs.sh
```

Kill it:

```bash
kill -9 12345
```

---

## 5.3. Stop all cron jobs (not usually recommended)

If you want to **stop the cron daemon completely**:

```bash
sudo systemctl stop crond   # Amazon Linux / RHEL
sudo systemctl stop cron    # Ubuntu/Debian
```

To disable auto-start at boot:

```bash
sudo systemctl disable crond
```

---

For the case (just stopping the Redis health check job):

* Remove the line from `crontab -e`
* And kill the process if one is currently running.

</details>

</details>

---

# Redis Health Check Script – Documentation

<details>
  <summary>View the steps to check the Redis Health</summary>

## 1. Purpose

This script continuously checks whether a Redis instance is alive and reachable.
It runs for a configurable duration, at a given interval, takes breaks between cycles, and logs success or failure with timestamps.

---

## 2. Script Location

```bash
/root/redis-health/redis_health.sh
```

---

## 3. Script Content

```bash
#!/bin/bash
# redis_health_flexible.sh
# Simple & customizable Redis health check script

# --- Configuration (YOU can modify these) ---
REDIS_HOST="<elasticahce-end-point>.bp8cjs.ng.0001.aps1.cache.amazonaws.com" # Redis hostname
REDIS_PORT=6379                  # Redis port
RUN_DURATION=300                 # Run time in seconds (e.g., 300s = 5 minutes)
CHECK_INTERVAL=60                # Interval between checks (seconds)
BREAK_DURATION=300               # Break time between runs (seconds)
CYCLES=4                         # How many times to repeat

# --- Log files ---
SUCCESS_LOG="/root/redis-health/redis_success.log"
ERROR_LOG="/root/redis-health/redis_error.log"

# --- Counter for Success logs ---
COUNT=1

for cycle in $(seq 1 $CYCLES); do
    echo "=== Cycle $cycle started at $(date '+%Y-%m-%d %H:%M:%S') ===" >> "$SUCCESS_LOG"
    START_TIME=$(date +%s)
    END_TIME=$((START_TIME + RUN_DURATION))

    while [ $(date +%s) -lt $END_TIME ]; do
        TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
        RESPONSE=$(/usr/local/bin/redis-cli -h "$REDIS_HOST" -p "$REDIS_PORT" PING 2>&1)

        if [ "$RESPONSE" == "PONG" ]; then
            echo "$TIMESTAMP SUCCESS $COUNT" >> "$SUCCESS_LOG"
        else
            echo "$TIMESTAMP FAILURE: $RESPONSE" >> "$ERROR_LOG"
        fi

        COUNT=$((COUNT+1))
        sleep $CHECK_INTERVAL
    done

    if [ $cycle -lt $CYCLES ]; then
        echo "=== Cycle $cycle completed, sleeping for $BREAK_DURATION seconds ===" >> "$SUCCESS_LOG"
        sleep $BREAK_DURATION
    fi
done

echo "=== All cycles finished at $(date '+%Y-%m-%d %H:%M:%S') ===" >> "$SUCCESS_LOG"
```

---

## 4. Configuration Parameters

| Variable         | Description                               |
| ---------------- | ----------------------------------------- |
| `REDIS_HOST`     | Redis cluster/instance hostname           |
| `REDIS_PORT`     | Redis port (default: 6379)                |
| `RUN_DURATION`   | How long to check in each cycle (seconds) |
| `CHECK_INTERVAL` | Interval between checks (seconds)         |
| `BREAK_DURATION` | Pause time after each cycle (seconds)     |
| `CYCLES`         | Number of cycles to repeat                |

---

## 5. Log Files

* **Success Log:**
  `/root/redis-health/redis_success.log`
  Format:

  ```
  2025-08-27 07:35:01 SUCCESS 1
  2025-08-27 07:36:01 SUCCESS 2
  === Cycle 1 completed, sleeping for 300 seconds ===
  ```

* **Error Log:**
  `/root/redis-health/redis_error.log`
  Format:

  ```
  2025-08-27 07:37:10 FAILURE: Could not connect to Redis
  ```

---

## 6. Run Manually

```bash
cd /root/redis-health
nohup sh redis_health.sh > redis_health.out 2>&1 &
```

* Runs in background
* Output stored in `redis_health.out`
* Logs saved in `redis_success.log` and `redis_error.log`

---

## 7. Run with Cron
To run manually
- Want the script `redis_health.sh` (located in `/root/redis-health/`) to keep running **even after closing the SSH session**, It should start it with `nohup` like this:

```bash
cd /root/redis-health
nohup sh redis_health.sh > redis_health.out 2>&1 &
```

* `nohup` → prevents the process from being killed when SSH closes.
* `> redis_health.out 2>&1` → redirects both standard output and errors to a file (`redis_health.out`).
* `&` → puts it in the background.

---

To run automatically every hour:

```bash
crontab -e
```

Add:

```cron
0 * * * * /root/redis-health/redis_health.sh
```

---

## 8. Example Run Cycle (with default settings)

* Check every **60 seconds**
* Run continuously for **5 minutes**
* Take a **5 minute break**
* Repeat **4 times**

Total run time = (5 min check + 5 min break) × 4 = **40 minutes**

---

✅ With this setup:

* You have **continuous monitoring with controlled cycles**
* Logs are **timestamped** for easy troubleshooting
* Can be run **manually** or via **cron automation**

---

</details>

---

# Redis Health Check Script – Read/Write Test (No Leftover Keys) --> No Replica or No Multi-AZ, Just Single Node

<details>
  <summary>Click to view the steps</summary>

## Script Overview

**Filename:** `redis_health_rw_check.sh`
**Purpose:** Monitors Redis availability by performing temporary read/write operations. Tracks successes and failures without leaving any keys behind.

**Key Features:**

* Performs **temporary SET and GET** operations to confirm full read/write functionality.
* **Deletes temporary keys immediately** after each check.
* Logs **successes** and **errors** separately.
* Customizable **check interval**, **total run duration**, and **number of cycles**.

---

## Script Content

```bash
#!/bin/bash
# redis_health_rw_check.sh
# Redis health check with temporary read/write tests, no leftover keys

# --- Configuration ---
REDIS_HOST="<redis-elasticache-endpoint>.bp8cjs.ng.0001.aps1.cache.amazonaws.com"
REDIS_PORT=6379
RUN_DURATION=1200        # Total run time in seconds
CHECK_INTERVAL=10        # Interval between checks in seconds
CYCLES=1                 # Number of cycles

# --- Log files ---
SUCCESS_LOG="/root/redis-health/redis_success.log"
ERROR_LOG="/root/redis-health/redis_error.log"

# --- Counter for Success logs ---
COUNT=1

for cycle in $(seq 1 $CYCLES); do
    echo "=== Cycle $cycle started at $(date '+%Y-%m-%d %H:%M:%S') ===" >> "$SUCCESS_LOG"
    START_TIME=$(date +%s)
    END_TIME=$((START_TIME + RUN_DURATION))

    while [ $(date +%s) -lt $END_TIME ]; do
        TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
        TEMP_KEY="healthcheck_$RANDOM"

        # Try SET
        SET_RESPONSE=$(/usr/local/bin/redis-cli -h "$REDIS_HOST" -p "$REDIS_PORT" SET "$TEMP_KEY" "ok" 2>&1)
        if [[ "$SET_RESPONSE" == "OK" ]]; then
            # Try GET
            GET_RESPONSE=$(/usr/local/bin/redis-cli -h "$REDIS_HOST" -p "$REDIS_PORT" GET "$TEMP_KEY" 2>&1)
            # Delete the temporary key
            /usr/local/bin/redis-cli -h "$REDIS_HOST" -p "$REDIS_PORT" DEL "$TEMP_KEY" >/dev/null 2>&1

            if [[ "$GET_RESPONSE" == "ok" ]]; then
                echo "$TIMESTAMP SUCCESS $COUNT: Redis read/write OK" >> "$SUCCESS_LOG"
            else
                echo "$TIMESTAMP FAILURE $COUNT: GET failed ($GET_RESPONSE)" >> "$ERROR_LOG"
            fi
        else
            echo "$TIMESTAMP FAILURE $COUNT: SET failed ($SET_RESPONSE)" >> "$ERROR_LOG"
        fi

        COUNT=$((COUNT+1))
        sleep $CHECK_INTERVAL
    done
done

echo "=== All cycles finished at $(date '+%Y-%m-%d %H:%M:%S') ===" >> "$SUCCESS_LOG"
```

---

## Configuration Options

| Parameter        | Description                                       | Example Value                           |
| ---------------- | ------------------------------------------------- | --------------------------------------- |
| `REDIS_HOST`     | Redis primary endpoint                            | `redis-primary.xyz.cache.amazonaws.com` |
| `REDIS_PORT`     | Redis port                                        | `6379`                                  |
| `RUN_DURATION`   | Total duration of each cycle in seconds           | `1200` (20 min)                         |
| `CHECK_INTERVAL` | Interval between each read/write check in seconds | `10`                                    |
| `CYCLES`         | Number of cycles to repeat the checks             | `1`                                     |
| `SUCCESS_LOG`    | File path to store successful operations          | `/root/redis-health/redis_success.log`  |
| `ERROR_LOG`      | File path to store failures                       | `/root/redis-health/redis_error.log`    |

---

## Execution

### 1. Ensure script is executable

```bash
chmod +x /root/redis-health/redis_health_rw_check.sh
```

### 2. Run the script in background using `nohup`

```bash
nohup bash /root/redis-health/redis_health_rw_check.sh > /root/redis-health/redis_health_rw.out 2>&1 &
```

* `nohup` ensures the script keeps running even if the terminal is closed.
* Output and errors will be captured in `/root/redis-health/redis_health_rw.out`.

### 3. Monitor logs

* **Success log:** `/root/redis-health/redis_success.log`

  * Shows timestamped successes with counter: `SUCCESS 1, SUCCESS 2 ...`
* **Error log:** `/root/redis-health/redis_error.log`

  * Shows timestamped failures if Redis is unresponsive or read/write fails.

### 4. Optional: Run via cron

To run the script **once every hour**:

```bash
crontab -e
```

Add the line:

```bash
0 * * * * /root/redis-health/redis_health_rw_check.sh
```

---

## How It Works

1. Each loop generates a **temporary key** (`healthcheck_<random>`).
2. Performs a **SET** and then a **GET**.
3. Immediately deletes the temporary key to avoid polluting Redis.
4. Logs **successes** and **failures** with timestamps.
5. Repeats for the specified **RUN\_DURATION** and **CYCLES**.

✅ This ensures **full read/write validation** without leaving any keys behind.

---

## Notes / Best Practices

* Adjust `CHECK_INTERVAL` to detect shorter downtime periods.
* Use **primary Redis endpoint** for read/write validation.
* Review logs periodically to detect any transient failures.
* For long-running monitoring, combine with `logrotate` to prevent log files from growing too large.

Perfect! That’s exactly how you can measure downtime in a single-node ElastiCache setup with auto-failover. Here’s the sequence and what to expect:

---

### 1️⃣ Before upgrade

* Start your **read/write monitoring script** (the one with temporary keys).
* It continuously logs `SUCCESS` in normal operation.

### 2️⃣ During upgrade

* AWS will **replace the node** with the new Redis version.
* There will likely be a **few seconds of downtime**:

  * SET/GET may fail temporarily → these failures get logged in `redis_error.log`.
  * This is exactly the period your script captures as “downtime.”

### 3️⃣ After upgrade

* Once the node is back and healthy:

  * SET/GET will succeed again → logged as `SUCCESS`.
  * Your script will show the exact duration of downtime in your logs (number of failed checks × `CHECK_INTERVAL`).

### 4️⃣ Notes

* With **auto-failover enabled**, if the node had a replica, AWS would promote it temporarily to reduce downtime. But since you have **single-node**, the downtime is for the **whole duration of the upgrade**.
* The **temporary keys** in your script ensure no leftover keys in Redis.

---

✅ **Tip:**

* Keep `CHECK_INTERVAL` small (5–10 seconds) for precise downtime measurement.
* Monitor both `redis_success.log` and `redis_error.log` during upgrade.

</details>

---

# Continous health check for the redis including milliseconds no sleep interval between checks

<details>
  <summary>Click to view the script</summary>

```python
#!/bin/bash
# redis_downtime_probe.sh
# Continuous Redis read/write health check (no leftover keys)

REDIS_HOST="<your-end-point>-dev-upgrade-test.bp8cjs.ng.0001.aps1.cache.amazonaws.com"
REDIS_PORT=6379
RUN_DURATION=1200        # total runtime in seconds (30 min)
CYCLES=1                 # number of cycles

SUCCESS_LOG="/root/redis-downtime-continious/redis_success.log"
ERROR_LOG="/root/redis-downtime-continious/redis_error.log"

COUNT=1

for cycle in $(seq 1 $CYCLES); do
    echo "=== Cycle $cycle started at $(date '+%Y-%m-%d %H:%M:%S.%3N') ===" >> "$SUCCESS_LOG"
    echo "=== Cycle $cycle started at $(date '+%Y-%m-%d %H:%M:%S.%3N') ===" >> "$ERROR_LOG"

    START_TIME=$(date +%s)
    END_TIME=$((START_TIME + RUN_DURATION))

    while [ $(date +%s) -lt $END_TIME ]; do
        TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S.%3N')  # include milliseconds
        TEMP_KEY="healthcheck_$RANDOM"

        # Try SET
        SET_RESPONSE=$(/usr/local/bin/redis-cli -h "$REDIS_HOST" -p "$REDIS_PORT" SET "$TEMP_KEY" "ok" 2>&1)
        if [[ "$SET_RESPONSE" == "OK" ]]; then
            # Try GET
            GET_RESPONSE=$(/usr/local/bin/redis-cli -h "$REDIS_HOST" -p "$REDIS_PORT" GET "$TEMP_KEY" 2>&1)
            /usr/local/bin/redis-cli -h "$REDIS_HOST" -p "$REDIS_PORT" DEL "$TEMP_KEY" >/dev/null 2>&1

            if [[ "$GET_RESPONSE" == "ok" ]]; then
                echo "$TIMESTAMP SUCCESS $COUNT: Redis read/write OK" >> "$SUCCESS_LOG"
            else
                echo "$TIMESTAMP FAILURE $COUNT: GET failed → Response: $GET_RESPONSE" >> "$ERROR_LOG"
            fi
        else
            echo "$TIMESTAMP FAILURE $COUNT: SET failed → Response: $SET_RESPONSE" >> "$ERROR_LOG"
        fi

        COUNT=$((COUNT+1))
        # No sleep → full continuous probing
    done
done

echo "=== All cycles finished at $(date '+%Y-%m-%d %H:%M:%S.%3N') ===" >> "$SUCCESS_LOG"
echo "=== All cycles finished at $(date '+%Y-%m-%d %H:%M:%S.%3N') ===" >> "$ERROR_LOG"
```
  
</details>
