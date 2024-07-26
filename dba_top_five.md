1. **High CPU Usage**: Often caused by inefficient queries or resource-intensive operations.
2. **Blocking Transactions**: When one transaction holds locks that block others, causing contention.
3. **Slow Queries**: Queries that take longer than expected, impacting overall performance.
4. **Deadlocks**: When two or more transactions are waiting for each other to release locks.
5. **Connection Issues**: Problems with too many open connections or long-running connections.

### Advanced Scripts for Troubleshooting and Resolving Issues

#### 1. Kill High CPU Queries
This script identifies and kills queries consuming high CPU.

```sql
-- Identify high CPU consuming queries
SELECT ID, USER, HOST, DB, COMMAND, TIME, STATE, INFO 
FROM INFORMATION_SCHEMA.PROCESSLIST 
WHERE COMMAND != 'Sleep' AND TIME > 30 
ORDER BY TIME DESC;

-- Kill high CPU queries
CALL mysql.rds_kill(<PROCESS_ID>);
```

#### 2. Kill Blocking Transactions
This script helps identify and kill blocking transactions.

```sql
-- Identify blocking transactions
SELECT r.trx_id waiting_trx_id,
       r.trx_mysql_thread_id waiting_thread,
       r.trx_query waiting_query,
       b.trx_id blocking_trx_id,
       b.trx_mysql_thread_id blocking_thread,
       b.trx_query blocking_query
FROM information_schema.innodb_lock_waits w
JOIN information_schema.innodb_trx b ON b.trx_id = w.blocking_trx_id
JOIN information_schema.innodb_trx r ON r.trx_id = w.requesting_trx_id;

-- Kill blocking transactions
CALL mysql.rds_kill(<BLOCKING_THREAD_ID>);
```

#### 3. Kill Slow Queries
This script identifies and kills slow queries.

```sql
-- Identify slow queries
SELECT ID, USER, HOST, DB, COMMAND, TIME, STATE, INFO 
FROM INFORMATION_SCHEMA.PROCESSLIST 
WHERE COMMAND != 'Sleep' AND TIME > 60 
ORDER BY TIME DESC;

-- Kill slow queries
CALL mysql.rds_kill(<PROCESS_ID>);
```

#### 4. Get Processlist IDs
Use this script to map `THREAD_ID` to `PROCESSLIST_ID`.

```sql
-- Map THREAD_ID to PROCESSLIST_ID
SELECT NAME, THREAD_ID, PROCESSLIST_ID, THREAD_OS_ID
FROM performance_schema.threads;
```

#### 5. Best Practices for Using `mysql.rds_kill`
Use the `mysql.rds_kill` procedure with caution to avoid unintentional disruption of critical transactions.

```sql
-- Best practices example for safely killing a process
-- Step 1: Identify critical queries/processes
SELECT ID, USER, HOST, DB, COMMAND, TIME, STATE, INFO 
FROM INFORMATION_SCHEMA.PROCESSLIST 
WHERE TIME > 30 AND COMMAND != 'Sleep' AND USER NOT IN ('rdsadmin', 'system_user') 
ORDER BY TIME DESC;

-- Step 2: Kill non-critical long-running queries
CALL mysql.rds_kill(<PROCESS_ID>);
```

In a production environment using MySQL on AWS RDS, you may encounter several common issues. Here are some top issues along with advanced scripts to troubleshoot and resolve them:

### Common Issues in MySQL on AWS RDS

1. **High CPU Usage**:
   - This can be due to poorly optimized queries, lack of proper indexing, or resource-intensive operations.
2. **Blocking Transactions**:
   - Long-running transactions or deadlocks can block other queries and degrade performance.
3. **Slow Queries**:
   - Inefficient queries, missing indexes, or suboptimal query plans can cause slow query performance.
4. **Storage Issues**:
   - Running out of storage, improper storage scaling, or high IOPS usage can lead to performance bottlenecks.
5. **Replication Delays**:
   - Replication lag or issues with read replicas can cause data consistency problems and delays.

### Advanced Troubleshooting Scripts

#### 1. Kill High CPU Queries

This script identifies and kills queries consuming high CPU resources.

```sql
-- Identify high CPU queries
SELECT 
    THREAD_ID, 
    PROCESSLIST_ID, 
    PROCESSLIST_USER, 
    PROCESSLIST_HOST, 
    PROCESSLIST_DB, 
    PROCESSLIST_COMMAND, 
    PROCESSLIST_TIME, 
    PROCESSLIST_STATE, 
    PROCESSLIST_INFO 
FROM 
    performance_schema.threads 
WHERE 
    PROCESSLIST_COMMAND = 'Query' 
    AND PROCESSLIST_TIME > 10  -- Customize this threshold
ORDER BY 
    PROCESSLIST_TIME DESC;

-- Kill the high CPU query
CALL mysql.rds_kill(<PROCESSLIST_ID>);
```

Replace `<PROCESSLIST_ID>` with the ID of the process you want to terminate.

#### 2. Kill Blocking Transactions

This script identifies and terminates transactions that are blocking others.

```sql
-- Find blocking transactions
SELECT 
    r.trx_id AS blocking_trx_id,
    r.trx_mysql_thread_id AS blocking_thread,
    r.trx_query AS blocking_query,
    b.trx_id AS blocked_trx_id,
    b.trx_mysql_thread_id AS blocked_thread,
    b.trx_query AS blocked_query
FROM 
    information_schema.innodb_lock_waits w
JOIN 
    information_schema.innodb_trx b ON b.trx_id = w.blocking_trx_id
JOIN 
    information_schema.innodb_trx r ON r.trx_id = w.requesting_trx_id;

-- Kill the blocking transaction
CALL mysql.rds_kill(<BLOCKING_THREAD>);
```

Replace `<BLOCKING_THREAD>` with the thread ID of the blocking transaction.

#### 3. Kill Slow Queries

This script finds and kills slow-running queries.

```sql
-- Identify slow queries
SELECT 
    THREAD_ID, 
    PROCESSLIST_ID, 
    PROCESSLIST_USER, 
    PROCESSLIST_HOST, 
    PROCESSLIST_DB, 
    PROCESSLIST_COMMAND, 
    PROCESSLIST_TIME, 
    PROCESSLIST_STATE, 
    PROCESSLIST_INFO 
FROM 
    performance_schema.threads 
WHERE 
    PROCESSLIST_COMMAND = 'Query' 
    AND PROCESSLIST_TIME > 60  -- Customize this threshold
ORDER BY 
    PROCESSLIST_TIME DESC;

-- Kill the slow query
CALL mysql.rds_kill(<PROCESSLIST_ID>);
```

Replace `<PROCESSLIST_ID>` with the ID of the process you want to terminate.

#### 4. Check Performance Schema for Thread Details

This script retrieves details of active threads from the performance schema.

```sql
SELECT 
    NAME, 
    THREAD_ID, 
    PROCESSLIST_ID, 
    THREAD_OS_ID 
FROM 
    performance_schema.threads 
WHERE 
    PROCESSLIST_COMMAND = 'Query';
```

This query helps you gather necessary details for identifying problematic queries.

#### 5. Kill Deadlocks or Specific Problematic Threads

This script is used for killing specific problematic threads based on `THREAD_ID` and `PROCESSLIST_ID`.

```sql
-- Find problematic threads
SELECT 
    THREAD_ID, 
    PROCESSLIST_ID, 
    THREAD_OS_ID, 
    PROCESSLIST_INFO 
FROM 
    performance_schema.threads 
WHERE 
    PROCESSLIST_STATE = 'Locked';

-- Kill the problematic thread
CALL mysql.rds_kill(<PROCESSLIST_ID>);
```

Replace `<PROCESSLIST_ID>` with the ID of the thread causing the issue.
