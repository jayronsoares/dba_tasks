###
### Here are the top 10 DBA scripts that every DBA should have:
###
### 1. **Database Health Check Script**
   This script provides an overview of the database's health by checking key metrics like uptime, connections, and load.
   ```sql
   SELECT 
       current_database(), 
       current_user, 
       version(), 
       pg_postmaster_start_time(), 
       now() - pg_postmaster_start_time() AS uptime;
   ```

### 2. **Slow Query Log Analysis**
   This script helps identify slow-running queries by analyzing the pg_stat_statements view.
   ```sql
   SELECT 
       query, 
       calls, 
       total_time, 
       mean_time, 
       rows 
   FROM 
       pg_stat_statements 
   ORDER BY 
       total_time DESC 
   LIMIT 10;
   ```

### 3. **Index Usage Statistics**
   This script checks the usage statistics of indexes to help optimize index strategies.
   ```sql
   SELECT 
       relname AS table_name, 
       indexrelname AS index_name, 
       idx_scan AS index_scans 
   FROM 
       pg_stat_user_indexes 
   JOIN 
       pg_stat_user_tables 
   ON 
       pg_stat_user_indexes.relid = pg_stat_user_tables.relid 
   ORDER BY 
       idx_scan DESC 
   LIMIT 10;
   ```

### 4. **Table Size Information**
   This script provides the size of each table in the database.
   ```sql
   SELECT 
       table_schema || '.' || table_name AS table_full_name, 
       pg_size_pretty(pg_total_relation_size(table_schema || '.' || table_name)) AS size 
   FROM 
       information_schema.tables 
   WHERE 
       table_type = 'BASE TABLE' 
       AND table_schema NOT IN ('pg_catalog', 'information_schema') 
   ORDER BY 
       pg_total_relation_size(table_schema || '.' || table_name) DESC 
   LIMIT 10;
   ```

### 5. **Database Performance Metrics**
   This script retrieves important performance metrics from the database.
   ```sql
   SELECT 
       name, 
       setting 
   FROM 
       pg_settings 
   WHERE 
       name IN ('max_connections', 'shared_buffers', 'work_mem', 'maintenance_work_mem', 'effective_cache_size');
   ```

### 6. **User Privileges Audit**
   This script lists all user privileges to ensure proper access control.
   ```sql
   SELECT 
       usename, 
       usecreatedb, 
       usesuper, 
       userepl 
   FROM 
       pg_user 
   ORDER BY 
       usename;
   ```

### 7. **Query Execution Plan Analysis**
   This script helps analyze the execution plan of a specific query to optimize performance.
   ```sql
   EXPLAIN 
   SELECT 
       your_columns 
   FROM 
       your_table 
   WHERE 
       your_conditions;
   ```

### 8. **Backup Status Check**
   This script checks the status of recent backups to ensure they are completed successfully.
   ```sql
   SELECT 
       pg_last_xlog_replay_location(), 
       pg_last_xact_replay_timestamp();
   ```

### 9. **Replication Status Check**
   This script checks the status of replication to ensure it is running smoothly.
   ```sql
   SELECT 
       client_addr, 
       state, 
       sync_priority, 
       sync_state 
   FROM 
       pg_stat_replication;
   ```

### 10. **Disk Space Usage**
   This script provides information on the disk space usage of the database.
   ```sql
   SELECT 
       pg_database.datname, 
       pg_size_pretty(pg_database_size(pg_database.datname)) AS size 
   FROM 
       pg_database 
   ORDER BY 
       pg_database_size(pg_database.datname) DESC;
   ```

### Bonus: **AWS RDS Specific Checks**
These scripts are tailored for AWS RDS PostgreSQL environments:
1. **Parameter Group Settings**
   ```sql
   SELECT 
       name, 
       setting 
   FROM 
       pg_settings 
   WHERE 
       context = 'user';
   ```

2. **Snapshot and Automated Backup Information**
   ```bash
   aws rds describe-db-snapshots --db-instance-identifier your-db-instance-identifier
   aws rds describe-db-instances --db-instance-identifier your-db-instance-identifier --query 'DBInstances[*].{DBInstanceIdentifier:DBInstanceIdentifier,BackupRetentionPeriod:BackupRetentionPeriod}'
   ```
###
### Here are the top 10 DBA scripts every DBA should have:
###
### 1. **Database Health Check Script**
   This script provides an overview of the database's health by checking key metrics like uptime, connections, buffer cache, etc.
   ```sql
   SELECT 
       now() - pg_postmaster_start_time() AS uptime,
       (SELECT count(*) FROM pg_stat_activity) AS connections,
       (SELECT setting FROM pg_settings WHERE name = 'max_connections') AS max_connections,
       pg_size_pretty(pg_database_size(current_database())) AS db_size;
   ```

### 2. **Slow Query Log Analysis**
   This script helps identify slow-running queries by analyzing the slow query log.
   ```sql
   SELECT 
       query, 
       calls, 
       total_time, 
       mean_time, 
       max_time, 
       stddev_time 
   FROM 
       pg_stat_statements 
   ORDER BY 
       total_time DESC 
   LIMIT 10;
   ```

### 3. **Index Usage Statistics**
   This script checks the usage statistics of indexes to help optimize index strategies.
   ```sql
   SELECT 
       schemaname, 
       relname, 
       indexrelname, 
       idx_scan, 
       idx_tup_read, 
       idx_tup_fetch 
   FROM 
       pg_stat_user_indexes 
   JOIN 
       pg_index 
   ON 
       pg_stat_user_indexes.indexrelid = pg_index.indexrelid 
   WHERE 
       idx_scan < 50 
       AND pg_index.indisunique IS FALSE;
   ```

### 4. **Table Size Information**
   This script provides the size of each table in the database.
   ```sql
   SELECT 
       schemaname AS schema, 
       relname AS table, 
       pg_size_pretty(pg_total_relation_size(relid)) AS total_size, 
       pg_size_pretty(pg_relation_size(relid)) AS table_size, 
       pg_size_pretty(pg_total_relation_size(relid) - pg_relation_size(relid)) AS index_size 
   FROM 
       pg_catalog.pg_statio_user_tables 
   ORDER BY 
       pg_total_relation_size(relid) DESC;
   ```

### 5. **Database Performance Metrics**
   This script retrieves important performance metrics from the database.
   ```sql
   SELECT 
       sum(numbackends) AS total_connections,
       sum(xact_commit) AS total_commits,
       sum(xact_rollback) AS total_rollbacks,
       sum(blks_read) AS total_disk_reads,
       sum(blks_hit) AS total_buffer_hits 
   FROM 
       pg_stat_database;
   ```

### 6. **User Privileges Audit**
   This script lists all user privileges to ensure proper access control.
   ```sql
   SELECT 
       grantee, 
       table_catalog, 
       table_schema, 
       table_name, 
       privilege_type 
   FROM 
       information_schema.role_table_grants 
   ORDER BY 
       grantee, 
       table_name;
   ```

### 7. **Query Execution Plan Analysis**
   This script helps analyze the execution plan of a specific query to optimize performance.
   ```sql
   EXPLAIN ANALYZE 
   SELECT 
       your_columns 
   FROM 
       your_table 
   WHERE 
       your_conditions;
   ```

### 8. **Backup Status Check**
   This script checks the status of recent backups to ensure they are completed successfully. In an AWS RDS environment, this is often monitored through the AWS Management Console or CLI.
   ```bash
   aws rds describe-db-snapshots --db-instance-identifier your-db-instance-identifier
   ```

### 9. **Replication Status Check**
   This script checks the status of replication to ensure it is running smoothly.
   ```sql
   SELECT 
       client_addr, 
       state, 
       sent_lsn, 
       write_lsn, 
       flush_lsn, 
       replay_lsn, 
       sync_priority, 
       sync_state 
   FROM 
       pg_stat_replication;
   ```

### 10. **Disk Space Usage**
   This script provides information on the disk space usage of the database.
   ```sql
   SELECT 
       pg_size_pretty(pg_database_size(current_database())) AS db_size, 
       pg_size_pretty(pg_total_relation_size('your_table')) AS table_size 
   FROM 
       pg_database 
   WHERE 
       datname = current_database();
   ```

### Bonus: **AWS RDS Specific Checks**
These scripts are tailored for AWS RDS PostgreSQL environments:
1. **Parameter Group Settings**
   ```sql
   SELECT 
       * 
   FROM 
       pg_settings 
   WHERE 
       name LIKE '%rds%';
   ```

2. **Snapshot and Automated Backup Information**
   ```bash
   aws rds describe-db-snapshots --db-instance-identifier your-db-instance-identifier
   aws rds describe-db-instances --db-instance-identifier your-db-instance-identifier --query 'DBInstances[*].{DBInstanceIdentifier:DBInstanceIdentifier,BackupRetentionPeriod:BackupRetentionPeriod}'
   ```
