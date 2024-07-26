# DBA Troubleshooting Mini Tutorial

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
