## Top 10 Troubleshooting

### 1. Identifying and Resolving Slow Queries

**Problem:** Slow queries affecting application performance.

**Solution:**
1. **Enable Slow Query Log:**
   ```sql
   CALL mysql.rds_enable_slow_query_log();
   ```
2. **Check Slow Queries:**
   ```sql
   SELECT * FROM mysql.slow_log ORDER BY query_time DESC LIMIT 10;
   ```
3. **Analyze Execution Plan:**
   ```sql
   EXPLAIN FORMAT=JSON <your-slow-query>;
   ```
4. **Optimize the Query:** Add necessary indexes, rewrite the query, or break it into smaller queries.

### 2. Managing High CPU Utilization

**Problem:** High CPU usage causing performance degradation.

**Solution:**
1. **Identify High CPU Queries:**
   ```sql
   SELECT * FROM performance_schema.events_statements_summary_by_digest
   WHERE SUM_TIMER_WAIT > 100000000000
   ORDER BY SUM_TIMER_WAIT DESC;
   ```
2. **Analyze and Optimize:**
   ```sql
   EXPLAIN FORMAT=JSON <high-cpu-query>;
   ```
3. **Tune Instance Parameters:** Adjust `innodb_buffer_pool_size`, `max_connections`, etc.

### 3. Fixing Lock Wait Timeout Issues

**Problem:** Frequent lock wait timeouts leading to application errors.

**Solution:**
1. **Identify Blocking Queries:**
   ```sql
   SELECT * FROM information_schema.INNODB_LOCK_WAITS;
   ```
2. **Analyze the Blocking Query:**
   ```sql
   SELECT * FROM information_schema.INNODB_LOCKS WHERE LOCK_TRX_ID=<blocking_trx_id>;
   ```
3. **Kill the Blocking Transaction if Necessary:**
   ```sql
   CALL mysql.rds_kill(<blocking_thread_id>);
   ```

### 4. Resolving Disk Space Issues

**Problem:** Running out of disk space affecting database operations.

**Solution:**
1. **Check Disk Usage:**
   ```sql
   SELECT table_schema AS "Database",
   ROUND(SUM(data_length + index_length) / 1024 / 1024, 1) AS "Size (MB)"
   FROM information_schema.tables
   GROUP BY table_schema;
   ```
2. **Clean Up Unused Data:**
   ```sql
   DELETE FROM <table> WHERE <condition>;
   ```
3. **Enable Binary Log Compression:**
   ```sql
   CALL mysql.rds_enable_log_bin_compression();
   ```

### 5. Handling Replication Lag

**Problem:** Replication lag causing read replica inconsistencies.

**Solution:**
1. **Check Replication Status:**
   ```sql
   SHOW SLAVE STATUS\G;
   ```
2. **Identify Delays:**
   ```sql
   SELECT * FROM mysql.rds_replica_status WHERE Time_Lag > 0;
   ```
3. **Optimize Write Operations:** Reduce the load on the master instance by optimizing write-heavy queries.

### 6. Resolving Memory Issues

**Problem:** Memory bottlenecks leading to database performance degradation.

**Solution:**
1. **Identify Memory Usage:**
   ```sql
   SHOW ENGINE INNODB STATUS\G;
   ```
2. **Optimize Memory Parameters:**
   ```sql
   SET GLOBAL innodb_buffer_pool_size=<new_size>;
   ```
3. **Analyze and Optimize Queries:** Rewrite queries to reduce memory consumption.

### 7. Addressing Frequent Failovers

**Problem:** Unstable environment due to frequent failovers.

**Solution:**
1. **Check Recent Failovers:**
   ```sql
   SELECT * FROM mysql.rds_history WHERE source = 'FAILOVER';
   ```
2. **Analyze the Cause:**
   - Network issues
   - Instance-level issues (e.g., CPU, memory)
3. **Mitigate the Issue:** Improve instance type, optimize queries, or enhance network stability.

### 8. Troubleshooting Connection Issues

**Problem:** Frequent connection drops affecting application connectivity.

**Solution:**
1. **Check Connection Logs:**
   ```sql
   SHOW PROCESSLIST;
   ```
2. **Adjust Connection Parameters:**
   ```sql
   SET GLOBAL max_connections=<new_value>;
   SET GLOBAL wait_timeout=<new_value>;
   SET GLOBAL interactive_timeout=<new_value>;
   ```

### 9. Handling Deadlocks

**Problem:** Deadlocks causing transaction rollbacks.

**Solution:**
1. **Identify Deadlocks:**
   ```sql
   SHOW ENGINE INNODB STATUS\G;
   ```
2. **Analyze and Resolve:**
   ```sql
   SELECT * FROM information_schema.INNODB_DEADLOCKS;
   ```
3. **Optimize Transaction Logic:** Review and optimize the transaction sequence in application code.

### 10. Monitoring and Optimizing Read Performance

**Problem:** Slow read operations affecting application performance.

**Solution:**
1. **Analyze Query Performance:**
   ```sql
   SELECT * FROM performance_schema.events_statements_summary_by_digest
   WHERE DIGEST_TEXT LIKE 'SELECT%'
   ORDER BY COUNT_STAR DESC LIMIT 10;
   ```
2. **Add Indexes:**
   ```sql
   CREATE INDEX idx_column ON table(column);
   ```
3. **Partition Large Tables:**
   ```sql
   ALTER TABLE table PARTITION BY RANGE(column) (
       PARTITION p0 VALUES LESS THAN (1991),
       PARTITION p1 VALUES LESS THAN (1995),
       PARTITION p2 VALUES LESS THAN (2000)
   );
   ```

### Advanced Troubleshooting Examples for DBA in AWS RDS Aurora MySQL

When managing an AWS RDS Aurora MySQL instance, monitoring and troubleshooting various performance metrics such as CPU utilization, freeable memory, IOPS, and throughput is crucial.

#### 1. **CPU Utilization**

**Query**: Identify the queries consuming the most CPU.

```sql
SELECT 
    ps.id, 
    ps.user, 
    ps.host, 
    ps.db, 
    ps.command, 
    ps.time, 
    ps.state, 
    ps.info, 
    ps.cpu_time, 
    ps.cpu_usage
FROM 
    performance_schema.threads ps
JOIN 
    performance_schema.events_statements_summary_by_thread_by_event_name es 
ON 
    ps.thread_id = es.thread_id
WHERE 
    ps.cpu_time > 0
ORDER BY 
    ps.cpu_usage DESC 
LIMIT 10;
```

#### 2. **Freeable Memory**

**Query**: Identify the queries with high memory usage.

```sql
SELECT 
    ps.id, 
    ps.user, 
    ps.host, 
    ps.db, 
    ps.command, 
    ps.time, 
    ps.state, 
    ps.info, 
    ps.memory_used, 
    ps.memory_allocated
FROM 
    performance_schema.threads ps
JOIN 
    performance_schema.events_statements_summary_by_thread_by_event_name es 
ON 
    ps.thread_id = es.thread_id
WHERE 
    ps.memory_used > 0
ORDER BY 
    ps.memory_used DESC 
LIMIT 10;
```

#### 3. **Read IOPS**

**Query**: Identify the tables with the highest read operations.

```sql
SELECT 
    table_schema, 
    table_name, 
    COUNT_READ, 
    SUM_TIMER_READ
FROM 
    performance_schema.table_io_waits_summary_by_table
ORDER BY 
    COUNT_READ DESC 
LIMIT 10;
```

#### 4. **Write IOPS**

**Query**: Identify the tables with the highest write operations.

```sql
SELECT 
    table_schema, 
    table_name, 
    COUNT_WRITE, 
    SUM_TIMER_WRITE
FROM 
    performance_schema.table_io_waits_summary_by_table
ORDER BY 
    COUNT_WRITE DESC 
LIMIT 10;
```

#### 5. **Read Throughput**

**Query**: Identify the queries with the highest read throughput.

```sql
SELECT 
    ps.id, 
    ps.user, 
    ps.host, 
    ps.db, 
    ps.command, 
    ps.time, 
    ps.state, 
    ps.info, 
    ps.bytes_received
FROM 
    performance_schema.threads ps
JOIN 
    performance_schema.events_statements_summary_by_thread_by_event_name es 
ON 
    ps.thread_id = es.thread_id
WHERE 
    ps.bytes_received > 0
ORDER BY 
    ps.bytes_received DESC 
LIMIT 10;
```

#### 6. **Write Throughput**

**Query**: Identify the queries with the highest write throughput.

```sql
SELECT 
    ps.id, 
    ps.user, 
    ps.host, 
    ps.db, 
    ps.command, 
    ps.time, 
    ps.state, 
    ps.info, 
    ps.bytes_sent
FROM 
    performance_schema.threads ps
JOIN 
    performance_schema.events_statements_summary_by_thread_by_event_name es 
ON 
    ps.thread_id = es.thread_id
WHERE 
    ps.bytes_sent > 0
ORDER BY 
    ps.bytes_sent DESC 
LIMIT 10;
```

#### 7. **Disk Queue**

**Query**: Identify the queries contributing to disk I/O waits.

```sql
SELECT 
    ps.id, 
    ps.user, 
    ps.host, 
    ps.db, 
    ps.command, 
    ps.time, 
    ps.state, 
    ps.info, 
    es.FILE_SUM_TIMER_READ, 
    es.FILE_SUM_TIMER_WRITE
FROM 
    performance_schema.threads ps
JOIN 
    performance_schema.file_summary_by_instance es 
ON 
    ps.thread_id = es.thread_id
WHERE 
    es.FILE_SUM_TIMER_READ > 0 OR es.FILE_SUM_TIMER_WRITE > 0
ORDER BY 
    (es.FILE_SUM_TIMER_READ + es.FILE_SUM_TIMER_WRITE) DESC 
LIMIT 10;
```

#### 8. **Query Performance**

**Query**: Identify slow queries using performance schema.

```sql
SELECT 
    query, 
    avg_timer_wait / 1000000000 AS avg_latency_ms, 
    exec_count, 
    errors
FROM 
    performance_schema.events_statements_summary_by_digest
ORDER BY 
    avg_timer_wait DESC 
LIMIT 10;
```

#### 9. **Index Usage**

**Query**: Check for unused indexes to improve write performance.

```sql
SELECT 
    object_schema, 
    object_name, 
    index_name, 
    rows_selected, 
    rows_inserted, 
    rows_updated, 
    rows_deleted
FROM 
    performance_schema.table_io_waits_summary_by_index_usage
WHERE 
    index_name IS NOT NULL
AND 
    rows_selected = 0
ORDER BY 
    rows_inserted + rows_updated + rows_deleted DESC 
LIMIT 10;
```

#### 10. **Table Lock Contention**

**Query**: Identify tables with the highest lock contention.

```sql
SELECT 
    table_schema, 
    table_name, 
    count_star, 
    sum_timer_wait / 1000000000 AS wait_time_sec
FROM 
    performance_schema.table_lock_waits_summary_by_table
ORDER BY 
    sum_timer_wait DESC 
LIMIT 10;
```
