# DBA Troubleshooting

This guide covers essential SQL queries for performance troubleshooting in MySQL. It focuses on identifying slow queries, table scans, and index usage inefficiencies. 

### 1. Identify High Latency Queries

This query lists the top 10 queries with the highest average row examinations, providing insights into inefficient queries:

```sql
SELECT 
    substring(query, 1, 50) AS query_sample, 
    avg_latency, 
    rows_examined_avg
FROM 
    sys.statements_with_runtimes_in_95th_percentile
ORDER BY 
    rows_examined_avg DESC
LIMIT 10;
```

- **query_sample**: A truncated version of the query for quick identification.
- **avg_latency**: The average latency time for the query.
- **rows_examined_avg**: The average number of rows examined by the query.

### 2. Check Index Usage

This query identifies how often indexes are being used for a specific table in a specific schema, helping determine if any indexes are underutilized:

```sql
SELECT 
    OBJECT_SCHEMA, 
    OBJECT_NAME, 
    INDEX_NAME, 
    COUNT_STAR AS usage_count
FROM 
    performance_schema.table_io_waits_summary_by_index_usage
WHERE 
    object_schema = 'madeiramadeira' 
    AND object_name = 'produtcs';
```

- **OBJECT_SCHEMA**: The schema of the object being queried.
- **OBJECT_NAME**: The name of the table being analyzed.
- **INDEX_NAME**: The name of the index being evaluated.
- **usage_count**: The number of times the index was used.

### 3. Detect Full Table Scans

This query helps find statements that perform full table scans, which are usually a sign of missing indexes:

```sql
USE sys;
SELECT 
    query, 
    db, 
    exec_count, 
    total_latency
FROM 
    sys.statements_with_full_table_scans
ORDER BY 
    exec_count DESC
LIMIT 5;
```

- **query**: The full SQL query that performed a table scan.
- **db**: The database in which the query was executed.
- **exec_count**: The number of times the query was executed.
- **total_latency**: The total latency time for the query executions.

### 4. Analyze Resource-Intensive Queries

This query identifies the top 10 queries consuming the most resources based on the wait time, helping prioritize performance optimization efforts:

```sql
SELECT 
    (100 * SUM_TIMER_WAIT / SUM(SUM_TIMER_WAIT) OVER()) AS percent,
    SUM_TIMER_WAIT AS total_wait_time,
    COUNT_STAR AS calls,
    AVG_TIMER_WAIT AS average_wait_time,
    substring(DIGEST_TEXT, 1, 75) AS query_snippet,
    DIGEST_TEXT AS full_query
FROM 
    performance_schema.events_statements_summary_by_digest
ORDER BY 
    SUM_TIMER_WAIT DESC
LIMIT 10;
```

- **percent**: The percentage of total wait time attributed to each query.
- **total_wait_time**: The total wait time for the query.
- **calls**: The number of times the query was called.
- **average_wait_time**: The average wait time for each call.
- **query_snippet**: A short version of the query for quick reference.
- **full_query**: The complete text of the query.


### 1. **Analyze InnoDB Buffer Pool Usage**

The InnoDB buffer pool is crucial for MySQL performance, especially in AWS RDS, where you might have limited control over the hardware. This query provides insights into how effectively the buffer pool is being used, which is key to optimizing memory allocation:

```sql
SELECT 
    CONCAT(FORMAT(100 * innodb_buffer_pool_pages_data / innodb_buffer_pool_pages_total, 2), '%') AS buffer_pool_usage,
    FORMAT(innodb_buffer_pool_pages_free * @@innodb_page_size / 1024 / 1024, 2) AS free_memory_MB,
    FORMAT(innodb_buffer_pool_pages_data * @@innodb_page_size / 1024 / 1024, 2) AS used_memory_MB,
    FORMAT(innodb_buffer_pool_read_requests / innodb_buffer_pool_reads, 2) AS read_efficiency
FROM 
    information_schema.INNODB_METRICS
WHERE 
    name IN ('buffer_pool_pages_data', 'buffer_pool_pages_total', 'buffer_pool_pages_free', 
             'buffer_pool_read_requests', 'buffer_pool_reads');
```

- **buffer_pool_usage**: Percentage of the buffer pool that is currently being used.
- **free_memory_MB**: Amount of free memory in the buffer pool in MB.
- **used_memory_MB**: Amount of used memory in the buffer pool in MB.
- **read_efficiency**: Efficiency ratio of buffer pool reads, indicating how often requests are served from the buffer versus being read from disk. A higher value indicates better performance.

### 2. **Identify Slow Queries Using Performance Schema**

In AWS RDS, monitoring slow queries is critical for maintaining optimal performance. This query examines the top slow queries based on execution time and execution count:

```sql
SELECT 
    DIGEST_TEXT AS query,
    COUNT_STAR AS exec_count,
    ROUND(SUM_TIMER_WAIT / 1000000000000, 2) AS total_time_sec,
    ROUND(AVG_TIMER_WAIT / 1000000000000, 2) AS avg_time_sec,
    ROUND(SUM_LOCK_TIME / 1000000000000, 2) AS total_lock_time_sec,
    ROUND(SUM_ROWS_SENT / exec_count, 2) AS avg_rows_sent
FROM 
    performance_schema.events_statements_summary_by_digest
WHERE 
    SCHEMA_NAME NOT IN ('mysql', 'information_schema', 'performance_schema', 'sys')
ORDER BY 
    total_time_sec DESC
LIMIT 10;
```

- **query**: The SQL statement executed.
- **exec_count**: Number of times the query was executed.
- **total_time_sec**: Total execution time for the query in seconds.
- **avg_time_sec**: Average execution time per execution in seconds.
- **total_lock_time_sec**: Total lock time spent waiting for resources in seconds.
- **avg_rows_sent**: Average number of rows sent by the query.

### 3. **Monitor AWS RDS Instance Resource Utilization**

This query helps monitor CPU and memory usage to ensure the AWS RDS instance operates efficiently. It's essential for capacity planning and identifying potential performance bottlenecks:

```sql
SELECT 
    CASE 
        WHEN variable_name = 'innodb_buffer_pool_size' THEN 'InnoDB Buffer Pool Size'
        WHEN variable_name = 'innodb_buffer_pool_bytes_data' THEN 'InnoDB Buffer Pool Data'
        WHEN variable_name = 'innodb_buffer_pool_bytes_dirty' THEN 'InnoDB Buffer Pool Dirty Data'
        WHEN variable_name = 'Threads_running' THEN 'Active Threads'
        WHEN variable_name = 'Threads_connected' THEN 'Connected Threads'
        WHEN variable_name = 'max_connections' THEN 'Max Connections'
        WHEN variable_name = 'innodb_data_reads' THEN 'Data Reads'
        WHEN variable_name = 'innodb_data_writes' THEN 'Data Writes'
    END AS metric_name,
    CASE 
        WHEN variable_name LIKE 'innodb_buffer_pool_%' THEN FORMAT(variable_value / 1024 / 1024, 2)
        ELSE FORMAT(variable_value, 0)
    END AS metric_value
FROM 
    performance_schema.global_status
WHERE 
    variable_name IN ('innodb_buffer_pool_size', 'innodb_buffer_pool_bytes_data', 
                      'innodb_buffer_pool_bytes_dirty', 'Threads_running', 
                      'Threads_connected', 'max_connections', 
                      'innodb_data_reads', 'innodb_data_writes')
UNION ALL
SELECT 
    'Free Memory (MB)' AS metric_name, 
    FORMAT((@@innodb_buffer_pool_size - 
           (innodb_buffer_pool_bytes_data + innodb_buffer_pool_bytes_dirty)) / 1024 / 1024, 2) AS metric_value
FROM 
    information_schema.global_variables, 
    information_schema.global_status
WHERE 
    VARIABLE_NAME = 'innodb_buffer_pool_size'
LIMIT 10;
```

- **metric_name**: Name of the monitored metric.
- **metric_value**: Current value of the metric, formatted for readability.
