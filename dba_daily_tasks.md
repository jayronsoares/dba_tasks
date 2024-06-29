
### 1. **Unused Indexes Detection and Drop**
This script finds and generates the SQL to drop unused indexes.

```sql
-- Find and generate drop statements for unused indexes
SET @schema := '%';  -- Set to a specific schema if needed

SELECT
    CONCAT(
        'ALTER TABLE ', t1.TABLE_SCHEMA, '.', t1.TABLE_NAME, ' DROP INDEX ', t1.INDEX_NAME, ';'
    ) AS Drop_Index_SQL
FROM information_schema.STATISTICS t1
INNER JOIN sys.schema_unused_indexes t2 
    ON t1.TABLE_NAME = t2.object_name AND t1.INDEX_NAME = t2.index_name
WHERE t1.TABLE_SCHEMA LIKE @schema
GROUP BY t1.TABLE_NAME, t1.INDEX_NAME
ORDER BY t1.TABLE_SCHEMA, t1.TABLE_NAME, t1.INDEX_NAME;
```

### 2. **Database Size in GB**
This script calculates the size of each database.

```sql
-- Calculate the size of each database in GB
SELECT 
    TABLE_SCHEMA, 
    ROUND(SUM(DATA_LENGTH + INDEX_LENGTH) / 1024 / 1024 / 1024, 2) AS Size_GB 
FROM information_schema.tables 
GROUP BY TABLE_SCHEMA;
```

### 3. **Total Server Size in GB**
This script calculates the total size of the server's databases.

```sql
-- Calculate the total size of all databases in GB
SELECT 
    ROUND(SUM(DATA_LENGTH + INDEX_LENGTH) / 1024 / 1024 / 1024, 2) AS Total_Size_GB 
FROM information_schema.tables;
```

### 4. **Database Uptime**
This script shows the uptime of the database server.

```sql
-- Show the uptime of the database server
SELECT TIME_FORMAT(SEC_TO_TIME(VARIABLE_VALUE), '%Hh %im %ss') AS Uptime 
FROM information_schema.GLOBAL_STATUS 
WHERE VARIABLE_NAME = 'Uptime';
```

### 5. **Queries Not Using Good Indexes**
This script identifies queries that do not use a good index.

```sql
-- Identify queries not using a good index
SELECT 
    THREAD_ID, 
    SQL_TEXT, 
    ROWS_SENT, 
    ROWS_EXAMINED, 
    CREATED_TMP_TABLES, 
    NO_INDEX_USED, 
    NO_GOOD_INDEX_USED
FROM performance_schema.events_statements_history_long
WHERE NO_INDEX_USED > 0 OR NO_GOOD_INDEX_USED > 0;
```

### 6. **Queries Creating Temporary Tables**
This script finds queries that created temporary tables.

```sql
-- Find queries that created temporary tables
SELECT 
    THREAD_ID, 
    SQL_TEXT, 
    ROWS_SENT, 
    ROWS_EXAMINED, 
    CREATED_TMP_TABLES, 
    CREATED_TMP_DISK_TABLES
FROM performance_schema.events_statements_history_long
WHERE CREATED_TMP_TABLES > 0 OR CREATED_TMP_DISK_TABLES > 0;
```

### 7. **Expensive Queries**
This script identifies queries that examine more rows than they affect or return.

```sql
-- Identify expensive queries
SELECT 
    THREAD_ID, 
    SQL_TEXT, 
    ROWS_SENT, 
    ROWS_EXAMINED, 
    ROWS_AFFECTED, 
    ERRORS, 
    CREATED_TMP_DISK_TABLES, 
    CREATED_TMP_TABLES, 
    SELECT_FULL_JOIN, 
    SELECT_FULL_RANGE_JOIN, 
    SELECT_RANGE, 
    SELECT_RANGE_CHECK, 
    SELECT_SCAN, 
    SORT_MERGE_PASSES, 
    SORT_RANGE
FROM performance_schema.events_statements_history_long
WHERE 
    ROWS_EXAMINED > ROWS_SENT 
    OR ROWS_EXAMINED > ROWS_AFFECTED
    OR ERRORS > 0
    OR CREATED_TMP_DISK_TABLES > 0
    OR CREATED_TMP_TABLES > 0
    OR SELECT_FULL_JOIN > 0
    OR SELECT_FULL_RANGE_JOIN > 0
    OR SELECT_RANGE > 0
    OR SELECT_RANGE_CHECK > 0
    OR SELECT_SCAN > 0
    OR SORT_MERGE_PASSES > 0
    OR SORT_RANGE > 0;
```

### 8. **Long-Running Queries**
This script identifies long-running queries that might impact performance.

```sql
-- Identify long-running queries
SELECT 
    THREAD_ID, 
    SQL_TEXT, 
    TIMER_WAIT / 1000000000 AS Duration_ms
FROM performance_schema.events_statements_history_long
WHERE TIMER_WAIT / 1000000000 > 1000  -- Threshold in milliseconds
ORDER BY Duration_ms DESC;
```

### 9. **High CPU Usage Queries**
This script finds queries that consume a lot of CPU time.

```sql
-- Identify queries with high CPU usage
SELECT 
    THREAD_ID, 
    SQL_TEXT, 
    CPU_TIME / 1000000000 AS CPU_Time_s
FROM performance_schema.events_statements_history_long
WHERE CPU_TIME / 1000000000 > 1  -- Threshold in seconds
ORDER BY CPU_Time_s DESC;
```

### 10. **High Memory Usage Queries**
This script identifies queries that use a significant amount of memory.

```sql
-- Identify queries with high memory usage
SELECT 
    THREAD_ID, 
    SQL_TEXT, 
    MEMORY_USED
FROM performance_schema.events_statements_history_long
WHERE MEMORY_USED > 1000000  -- Threshold in bytes
ORDER BY MEMORY_USED DESC;
```
---
### 1. **Check Replication Lag**
This query helps monitor replication lag for MySQL instances configured with read replicas.

```sql
-- Check replication lag
SELECT 
    r.ReplicaServerID, 
    r.ReplicaHost, 
    UNIX_TIMESTAMP() - UNIX_TIMESTAMP(r.LastIoErrorTimestamp) AS ReplicationLag_seconds
FROM performance_schema.replication_connection_status r
WHERE r.ReplicaIoRunning = 'Yes' AND r.ReplicaSqlRunning = 'Yes';
```

### 2. **Check Long Transactions**
Identify transactions that have been open for an extended period, which can indicate potential issues.

```sql
-- Check long-running transactions
SELECT 
    trx_id, 
    trx_started, 
    TIME_TO_SEC(TIMEDIFF(NOW(), trx_started)) AS duration_seconds,
    trx_mysql_thread_id,
    trx_query
FROM information_schema.innodb_trx
WHERE TIME_TO_SEC(TIMEDIFF(NOW(), trx_started)) > 60;  -- Adjust threshold as needed
```

### 3. **Check InnoDB Buffer Pool Usage**
Monitor the InnoDB buffer pool usage to ensure efficient memory utilization.

```sql
-- Check InnoDB buffer pool usage
SHOW ENGINE INNODB STATUS;
```

### 4. **Check Table Fragmentation**
Identify fragmented tables that might impact performance and storage efficiency.

```sql
-- Check table fragmentation
SELECT 
    TABLE_SCHEMA, 
    TABLE_NAME, 
    DATA_FREE / 1024 / 1024 AS FreeSpace_MB
FROM information_schema.tables
WHERE DATA_FREE > 0
ORDER BY DATA_FREE DESC;
```

### 5. **Check for Table Locks**
Identify tables that are locked to prevent other transactions from accessing them.

```sql
-- Check for table locks
SELECT 
    blocking_pid AS Blocking_Process_ID,
    blocking_query AS Blocking_Query,
    blocked_pid AS Blocked_Process_ID,
    blocked_query AS Blocked_Query
FROM performance_schema.data_locks
WHERE lock_status = 'GRANTED'
    AND lock_type = 'TABLE';
```
