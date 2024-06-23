### Here are the top 10 DBA scripts every DBA should have:

1. **Backup and Restore Script**:
   - Automates the process of taking backups and restoring them.
   ```sql
   -- Backup script
   mysqldump -h your-db-hostname -u your-username -p your-database-name > backup.sql

   -- Restore script
   mysql -h your-db-hostname -u your-username -p your-database-name < backup.sql
   ```

2. **Performance Monitoring Script**:
   - Helps monitor the performance of the MySQL instance.
   ```sql
   SELECT * FROM performance_schema.events_statements_summary_by_digest ORDER BY SUM_TIMER_WAIT DESC LIMIT 10;
   ```

3. **Slow Query Log Analysis Script**:
   - Identifies slow queries and provides insights for optimization.
   ```sql
   SELECT query_time, lock_time, rows_sent, rows_examined, db, last_insert_id, insert_id, server_id, sql_text
   FROM mysql.slow_log
   ORDER BY query_time DESC LIMIT 10;
   ```

4. **User and Permissions Management Script**:
   - Manages users and their permissions.
   ```sql
   -- Create a new user
   CREATE USER 'newuser'@'%' IDENTIFIED BY 'password';

   -- Grant permissions
   GRANT ALL PRIVILEGES ON your-database-name.* TO 'newuser'@'%';

   -- Apply changes
   FLUSH PRIVILEGES;
   ```

5. **Database Health Check Script**:
   - Checks the health of the database instance.
   ```sql
   -- Check database status
   SHOW STATUS LIKE 'uptime';

   -- Check database errors
   SHOW ENGINE INNODB STATUS;
   ```

6. **Schema Comparison Script**:
   - Compares schema between different instances or databases.
   ```sql
   -- Use pt-table-sync from Percona Toolkit
   pt-table-sync --execute h=source-host,D=source-database,t=table-name h=destination-host,D=destination-database,t=table-name
   ```

7. **Index Usage Analysis Script**:
   - Analyzes and identifies unused indexes.
   ```sql
   SELECT object_schema, object_name, index_name, rows_selected, rows_inserted, rows_updated, rows_deleted
   FROM performance_schema.table_io_waits_summary_by_index_usage
   WHERE index_name IS NOT NULL AND rows_selected = 0;
   ```

8. **Disk Space Monitoring Script**:
   - Monitors disk space usage.
   ```sql
   SELECT table_schema AS 'Database',
          ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)'
   FROM information_schema.tables
   GROUP BY table_schema;
   ```

9. **Replication Status Check Script**:
   - Checks the status of replication.
   ```sql
   SHOW SLAVE STATUS \G;
   ```

10. **Table Maintenance Script**:
    - Performs maintenance tasks like optimizing tables.
    ```sql
    -- Analyze tables
    ANALYZE TABLE your_table_name;

    -- Optimize tables
    OPTIMIZE TABLE your_table_name;
    ```
###
### Having these scripts at your disposal can significantly improve your efficiency as a DBA managing AWS RDS MySQL instances.
###

### 1. **Database Health Check Script**
   This script provides an overview of the database's health by checking key metrics like uptime, connections, buffer pool, etc.
   ```sql
   SELECT 
       VARIABLE_NAME, VARIABLE_VALUE 
   FROM 
       performance_schema.global_status 
   WHERE 
       VARIABLE_NAME IN ('Uptime', 'Threads_connected', 'Innodb_buffer_pool_pages_free', 'Innodb_buffer_pool_pages_total');
   ```

### 2. **Slow Query Log Analysis**
   This script helps identify slow-running queries by analyzing the slow query log.
   ```sql
   SELECT 
       start_time, 
       user_host, 
       query_time, 
       sql_text 
   FROM 
       mysql.slow_log 
   ORDER BY 
       query_time DESC 
   LIMIT 10;
   ```

### 3. **Index Usage Statistics**
   This script checks the usage statistics of indexes to help optimize index strategies.
   ```sql
   SELECT 
       table_name, 
       index_name, 
       stat_name, 
       stat_value 
   FROM 
       mysql.innodb_index_stats 
   WHERE 
       stat_name IN ('n_reads', 'n_inserts', 'n_updates', 'n_deletes');
   ```

### 4. **Table Size Information**
   This script provides the size of each table in the database.
   ```sql
   SELECT 
       table_schema AS 'Database', 
       table_name AS 'Table', 
       ROUND((data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)' 
   FROM 
       information_schema.tables 
   WHERE 
       table_schema = 'your_database_name' 
   ORDER BY 
       (data_length + index_length) DESC;
   ```

### 5. **Database Performance Metrics**
   This script retrieves important performance metrics from the database.
   ```sql
   SELECT 
       variable_name, 
       variable_value 
   FROM 
       performance_schema.global_status 
   WHERE 
       variable_name IN ('Queries', 'Questions', 'Connections', 'Created_tmp_tables', 'Select_full_join');
   ```

### 6. **User Privileges Audit**
   This script lists all user privileges to ensure proper access control.
   ```sql
   SELECT 
       user, 
       host, 
       Select_priv, 
       Insert_priv, 
       Update_priv, 
       Delete_priv 
   FROM 
       mysql.user 
   ORDER BY 
       user;
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
       * 
   FROM 
       performance_schema.events_stages_history_long 
   WHERE 
       NAME = 'stage/sql/backup';
   ```

### 9. **Replication Status Check**
   This script checks the status of replication to ensure it is running smoothly.
   ```sql
   SHOW SLAVE STATUS\G;
   ```

### 10. **Disk Space Usage**
   This script provides information on the disk space usage of the database.
   ```sql
   SELECT 
       table_schema AS 'Database', 
       SUM(data_length + index_length) / 1024 / 1024 AS 'Disk Space (MB)' 
   FROM 
       information_schema.tables 
   GROUP BY 
       table_schema;
   ```

### Bonus: **AWS RDS Specific Checks**
These scripts are tailored for AWS RDS MySQL environments:
1. **Parameter Group Settings**
   ```sql
   SELECT 
       * 
   FROM 
       performance_schema.session_variables;
   ```

2. **Snapshot and Automated Backup Information**
   ```bash
   aws rds describe-db-snapshots --db-instance-identifier your-db-instance-identifier
   aws rds describe-db-instances --db-instance-identifier your-db-instance-identifier --query 'DBInstances[*].{DBInstanceIdentifier:DBInstanceIdentifier,BackupRetentionPeriod:BackupRetentionPeriod}'
   ```
