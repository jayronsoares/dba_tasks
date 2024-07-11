### Advanced Troubleshooting Examples for DBA in AWS RDS Aurora MySQL

#### 1. **CPU Utilization**

**Query**: Identify the queries consuming the most CPU time.

```sql
SELECT 
    sql_text,
    current_schema,
    exec_count,
    total_cpu_time/1000 AS total_cpu_time_ms,
    avg_cpu_time/1000 AS avg_cpu_time_ms
FROM 
    performance_schema.events_statements_summary_by_digest
ORDER BY 
    total_cpu_time DESC
LIMIT 10;
```

#### 2. **Freeable Memory**

**Query**: Identify the largest memory-consuming queries.

```sql
SELECT 
    sql_text,
    current_schema,
    exec_count,
    sum_created_tmp_disk_tables AS tmp_disk_tables,
    sum_created_tmp_tables AS tmp_tables
FROM 
    performance_schema.events_statements_summary_by_digest
ORDER BY 
    (sum_created_tmp_disk_tables + sum_created_tmp_tables) DESC
LIMIT 10;
```

#### 3. **Read IOPS**

**Query**: Identify the tables with the highest read operations.

```sql
SELECT 
    OBJECT_SCHEMA,
    OBJECT_NAME,
    COUNT_READ/1000 AS count_read_k
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
    OBJECT_SCHEMA,
    OBJECT_NAME,
    COUNT_WRITE/1000 AS count_write_k
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
    sql_text,
    current_schema,
    exec_count,
    sum_select_full_join AS full_joins,
    sum_select_full_range_join AS full_range_joins,
    sum_select_range AS range_selects,
    sum_select_range_check AS range_checks
FROM 
    performance_schema.events_statements_summary_by_digest
ORDER BY 
    (sum_select_full_join + sum_select_full_range_join + sum_select_range + sum_select_range_check) DESC
LIMIT 10;
```

#### 6. **Write Throughput**

**Query**: Identify the queries with the highest write throughput.

```sql
SELECT 
    sql_text,
    current_schema,
    exec_count,
    sum_insert_select AS insert_selects,
    sum_insert_select_rows AS insert_select_rows,
    sum_update AS updates,
    sum_update_rows AS update_rows
FROM 
    performance_schema.events_statements_summary_by_digest
ORDER BY 
    (sum_insert_select + sum_update) DESC
LIMIT 10;
```

#### 7. **Disk Queue**

**Query**: Identify the queries contributing to disk I/O waits.

```sql
SELECT 
    sql_text,
    current_schema,
    exec_count,
    sum_io_wait/1000 AS sum_io_wait_ms,
    avg_io_wait/1000 AS avg_io_wait_ms
FROM 
    performance_schema.events_statements_summary_by_digest
ORDER BY 
    sum_io_wait DESC
LIMIT 10;
```

#### 8. **Query Performance**

**Query**: Identify slow queries using performance schema.

```sql
SELECT 
    sql_text,
    current_schema,
    exec_count,
    total_latency/1000000 AS total_latency_ms,
    avg_latency/1000000 AS avg_latency_ms
FROM 
    performance_schema.events_statements_summary_by_digest
ORDER BY 
    total_latency DESC
LIMIT 10;
```

#### 9. **Index Usage**

**Query**: Check for unused indexes to improve write performance.

```sql
SELECT 
    table_schema,
    table_name,
    index_name,
    user_seeks,
    user_scans,
    user_lookups,
    user_updates
FROM 
    sys.schema_index_statistics
WHERE 
    user_seeks = 0 AND user_scans = 0 AND user_lookups = 0
ORDER BY 
    user_updates DESC
LIMIT 10;
```

#### 10. **Table Lock Contention**

**Query**: Identify tables with the highest lock contention.

```sql
SELECT 
    OBJECT_SCHEMA,
    OBJECT_NAME,
    COUNT_STAR/1000 AS count_star_k,
    SUM_TIMER_WAIT/1000000000 AS wait_time_sec
FROM 
    performance_schema.table_lock_waits_summary_by_table
ORDER BY 
    SUM_TIMER_WAIT DESC
LIMIT 10;
```
