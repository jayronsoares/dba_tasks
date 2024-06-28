Managing AWS RDS for MySQL and PostgreSQL requires regular monitoring and maintenance to ensure the health and performance of the databases. Below are 10 crucial scripts that you should consider using daily for both MySQL and PostgreSQL environments to help maintain their health and ensure they are running optimally.

### For MySQL on AWS RDS

1. **Check for Slow Queries**
   ```sql
   SELECT * FROM mysql.slow_log ORDER BY start_time DESC LIMIT 10;
   ```
   This script fetches the latest slow queries, helping you identify and optimize slow-running queries.

2. **Monitor Database Size**
   ```sql
   SELECT table_schema AS 'Database', 
          ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)'
   FROM information_schema.tables
   GROUP BY table_schema;
   ```
   This script provides the size of each database, useful for monitoring growth and planning storage.

3. **Check for Open Connections**
   ```sql
   SHOW STATUS WHERE `variable_name` = 'Threads_connected';
   ```
   It shows the number of open connections, helping you monitor database load and connectivity issues.

4. **Review Active Queries**
   ```sql
   SHOW FULL PROCESSLIST;
   ```
   This script lists all active queries, which is crucial for diagnosing performance issues and locking problems.

5. **Check Database Uptime**
   ```sql
   SHOW STATUS LIKE 'Uptime';
   ```
   This gives you the uptime of your MySQL instance, useful for ensuring that the database is stable and has not restarted unexpectedly.

6. **Verify Table Status**
   ```sql
   SELECT TABLE_NAME, ENGINE, TABLE_ROWS, DATA_LENGTH, INDEX_LENGTH 
   FROM information_schema.tables
   WHERE table_schema = 'your_database_name';
   ```
   This script checks the status of tables in your database, including the number of rows and the amount of data stored.

7. **Monitor Database Connections**
   ```sql
   SELECT user, host, db, command, time 
   FROM information_schema.processlist;
   ```
   It helps in tracking connection activity and identifying long-running queries or locked tables.

8. **Check Index Usage**
   ```sql
   SELECT TABLE_NAME, INDEX_NAME, INDEX_TYPE, SEQ_IN_INDEX, COLUMN_NAME, CARDINALITY 
   FROM information_schema.statistics
   WHERE table_schema = 'your_database_name';
   ```
   This script provides details about index usage, which can be crucial for performance tuning.

9. **Review Database Errors**
   ```sql
   SHOW ENGINE INNODB STATUS;
   ```
   It gives you a detailed status of the InnoDB engine, useful for diagnosing problems and errors within your MySQL database.

10. **Analyze Performance Metrics**
    ```sql
    SHOW STATUS LIKE 'Queries';
    SHOW STATUS LIKE 'Connections';
    SHOW STATUS LIKE 'Aborted_clients';
    ```
    These commands provide important performance metrics like the number of queries, connections, and aborted clients.

### For PostgreSQL on AWS RDS

1. **Monitor Long-Running Queries**
   ```sql
   SELECT pid, age(clock_timestamp(), query_start), usename, query 
   FROM pg_stat_activity 
   WHERE state != 'idle' 
     AND query_start < now() - interval '5 minutes' 
   ORDER BY query_start;
   ```
   This script helps in identifying long-running queries that could be causing performance issues.

2. **Check Database Size**
   ```sql
   SELECT pg_database.datname, 
          pg_size_pretty(pg_database_size(pg_database.datname)) AS size 
   FROM pg_database;
   ```
   It provides the size of each database, useful for monitoring storage and planning for growth.

3. **Monitor Active Connections**
   ```sql
   SELECT datname, numbackends 
   FROM pg_stat_database;
   ```
   This script shows the number of active connections to each database.

4. **Review Table Bloat**
   ```sql
   SELECT schemaname, 
          tablename, 
          reltuples::numeric AS num_rows, 
          pg_size_pretty(pg_total_relation_size(relid)) AS total_size, 
          pg_size_pretty(pg_relation_size(relid)) AS table_size, 
          pg_size_pretty(pg_total_relation_size(relid) - pg_relation_size(relid)) AS index_size 
   FROM pg_catalog.pg_statio_user_tables
   ORDER BY pg_total_relation_size(relid) DESC;
   ```
   It helps in identifying table bloat, which can degrade performance over time.

5. **Analyze Index Usage**
   ```sql
   SELECT indexrelname AS index, 
          relname AS table, 
          idx_scan AS scans, 
          idx_tup_read AS tuples_read, 
          idx_tup_fetch AS tuples_fetched 
   FROM pg_stat_user_indexes 
   JOIN pg_index 
     ON pg_stat_user_indexes.indexrelid = pg_index.indexrelid;
   ```
   This script provides insights into index usage, aiding in performance optimization.

6. **Check for Lock Conflicts**
   ```sql
   SELECT pid, locktype, relation::regclass AS table, transactionid, 
          mode, granted 
   FROM pg_locks 
   JOIN pg_stat_activity 
     ON pg_locks.pid = pg_stat_activity.pid;
   ```
   It shows current lock information, useful for diagnosing contention issues.

7. **Monitor Database Uptime**
   ```sql
   SELECT date_trunc('second', now() - pg_postmaster_start_time()) AS uptime;
   ```
   <p>This provides the uptime of the PostgreSQL instance, useful for ensuring stability.</p>

8. **Review Vacuum and Analyze Status**
   ```sql
   SELECT relname AS table_name, 
          last_vacuum, 
          last_autovacuum, 
          last_analyze, 
          last_autoanalyze 
   FROM pg_stat_all_tables;
   ```
   <p>It helps in monitoring the vacuum and analyze status for maintaining table performance.</p>

9. **Check Replication Status**
   ```sql
   SELECT * FROM pg_stat_replication;
   ```
   This script checks the status of replication, crucial for high availability setups.

10. **Inspect Database Errors**
    ```sql
    SELECT log_time, 
           user_name, 
           database_name, 
           session_id, 
           message 
    FROM pg_logs 
    WHERE log_time > current_date - interval '1 day';
    ```
