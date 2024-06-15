Here are ten crucial scripts to perform routine support and maintenance tasks on AWS RDS MySQL and PostgreSQL databases:

### 1. Backup Database
#### MySQL
```bash
#!/bin/bash

DB_INSTANCE_IDENTIFIER="your-rds-instance-id"
BUCKET_NAME="your-s3-bucket"
REGION="your-region"
DATE=$(date +%Y-%m-%d)

mysqldump -h $DB_INSTANCE_IDENTIFIER.cwovfnrvhjuj.us-west-2.rds.amazonaws.com -u username -p'password' --all-databases > backup-$DATE.sql
aws s3 cp backup-$DATE.sql s3://$BUCKET_NAME/backup-$DATE.sql --region $REGION
```

#### PostgreSQL
```bash
#!/bin/bash

DB_INSTANCE_IDENTIFIER="your-rds-instance-id"
BUCKET_NAME="your-s3-bucket"
REGION="your-region"
DATE=$(date +%Y-%m-%d)

PGPASSWORD=password pg_dump -h $DB_INSTANCE_IDENTIFIER.cwovfnrvhjuj.us-west-2.rds.amazonaws.com -U username -F c -b -v -f backup-$DATE.backup
aws s3 cp backup-$DATE.backup s3://$BUCKET_NAME/backup-$DATE.backup --region $REGION
```

### 2. Restore Database
#### MySQL
```bash
#!/bin/bash

DB_INSTANCE_IDENTIFIER="your-rds-instance-id"
BUCKET_NAME="your-s3-bucket"
DATE="backup-date"

aws s3 cp s3://$BUCKET_NAME/backup-$DATE.sql backup-$DATE.sql --region your-region
mysql -h $DB_INSTANCE_IDENTIFIER.cwovfnrvhjuj.us-west-2.rds.amazonaws.com -u username -p'password' < backup-$DATE.sql
```

#### PostgreSQL
```bash
#!/bin/bash

DB_INSTANCE_IDENTIFIER="your-rds-instance-id"
BUCKET_NAME="your-s3-bucket"
DATE="backup-date"

aws s3 cp s3://$BUCKET_NAME/backup-$DATE.backup backup-$DATE.backup --region your-region
PGPASSWORD=password pg_restore -h $DB_INSTANCE_IDENTIFIER.cwovfnrvhjuj.us-west-2.rds.amazonaws.com -U username -d database_name -v backup-$DATE.backup
```

### 3. Monitor Database Performance
#### MySQL
```bash
#!/bin/bash

DB_INSTANCE_IDENTIFIER="your-rds-instance-id"
aws rds describe-db-instances --db-instance-identifier $DB_INSTANCE_IDENTIFIER --query 'DBInstances[*].{DBInstanceIdentifier:DBInstanceIdentifier,CPUUtilization:CPUUtilization,DBInstanceStatus:DBInstanceStatus}' --output table
```

#### PostgreSQL
```bash
#!/bin/bash

DB_INSTANCE_IDENTIFIER="your-rds-instance-id"
aws rds describe-db-instances --db-instance-identifier $DB_INSTANCE_IDENTIFIER --query 'DBInstances[*].{DBInstanceIdentifier:DBInstanceIdentifier,CPUUtilization:CPUUtilization,DBInstanceStatus:DBInstanceStatus}' --output table
```

### 4. Check Database Size
#### MySQL
```sql
SELECT table_schema "Database Name", 
       SUM(data_length + index_length) / 1024 / 1024 "Database Size (MB)" 
FROM information_schema.tables 
GROUP BY table_schema;
```

#### PostgreSQL
```sql
SELECT pg_database.datname, 
       pg_size_pretty(pg_database_size(pg_database.datname)) AS size 
FROM pg_database;
```

### 5. Check Active Connections
#### MySQL
```sql
SHOW PROCESSLIST;
```

#### PostgreSQL
```sql
SELECT * FROM pg_stat_activity;
```

### 6. Rebuild Indexes
#### MySQL
```sql
ALTER TABLE table_name ENGINE=InnoDB;
```

#### PostgreSQL
```sql
REINDEX DATABASE database_name;
```

### 7. Update Statistics
#### MySQL
```sql
ANALYZE TABLE table_name;
```

#### PostgreSQL
```sql
VACUUM ANALYZE;
```

### 8. Check Slow Queries
#### MySQL
```sql
SHOW GLOBAL STATUS LIKE 'Slow_queries';
```

#### PostgreSQL
```sql
SELECT * FROM pg_stat_statements WHERE total_time > 10000;  -- Adjust time threshold as needed
```

### 9. Manage Users and Permissions
#### MySQL
```sql
CREATE USER 'new_user'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'new_user'@'%';
FLUSH PRIVILEGES;
```

#### PostgreSQL
```sql
CREATE USER new_user WITH PASSWORD 'password';
GRANT ALL PRIVILEGES ON DATABASE database_name TO new_user;
```

### 10. Cleanup Old Backups (Local Script)
#### MySQL and PostgreSQL
```bash
#!/bin/bash

BACKUP_DIR="/path/to/backup"
RETENTION_DAYS=7

find $BACKUP_DIR -type f -name "*.sql" -mtime +$RETENTION_DAYS -exec rm {} \;
```

-----------------------------------------------
### 26. List Slow Queries
#### MySQL
```sql
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1; -- Set threshold to 1 second
SELECT * FROM mysql.slow_log ORDER BY query_time DESC LIMIT 10;
```

#### PostgreSQL
```sql
SELECT * FROM pg_stat_statements WHERE total_time > 10000 ORDER BY total_time DESC LIMIT 10;  -- Adjust time threshold as needed
```

### 27. Check Query Execution Time
#### MySQL
```sql
SELECT 
    query,
    exec_count,
    total_exec_time / exec_count AS avg_exec_time,
    total_exec_time,
    lock_time,
    rows_sent,
    rows_examined
FROM 
    performance_schema.events_statements_summary_by_digest
ORDER BY total_exec_time DESC
LIMIT 10;
```

#### PostgreSQL
```sql
SELECT 
    query,
    calls,
    total_time,
    mean_time,
    rows
FROM 
    pg_stat_statements
ORDER BY total_time DESC
LIMIT 10;
```

### 28. Monitor Active Sessions
#### MySQL
```sql
SELECT 
    user,
    host,
    db,
    command,
    time,
    state,
    info
FROM 
    information_schema.processlist
WHERE 
    command != 'Sleep';
```

#### PostgreSQL
```sql
SELECT 
    pid, 
    usename, 
    application_name, 
    client_addr, 
    state, 
    query 
FROM 
    pg_stat_activity 
WHERE 
    state != 'idle';
```

### 29. Analyze Query Locking Issues
#### MySQL
```sql
SHOW ENGINE INNODB STATUS;
```

#### PostgreSQL
```sql
SELECT 
    pg_stat_activity.pid,
    pg_stat_activity.usename,
    pg_stat_activity.query,
    pg_locks.locktype,
    pg_locks.mode,
    pg_locks.granted
FROM 
    pg_stat_activity
JOIN 
    pg_locks
ON 
    pg_stat_activity.pid = pg_locks.pid
WHERE 
    pg_locks.granted IS FALSE;
```

### 30. Check Disk Usage by Table
#### MySQL
```sql
SELECT 
    table_schema AS `Database`, 
    table_name AS `Table`, 
    ROUND((data_length + index_length) / 1024 / 1024) AS `Size (MB)`
FROM 
    information_schema.TABLES
ORDER BY 
    (data_length + index_length) DESC
LIMIT 10;
```

#### PostgreSQL
```sql
SELECT 
    schemaname, 
    relname, 
    pg_size_pretty(pg_total_relation_size(relid)) AS size 
FROM 
    pg_catalog.pg_statio_user_tables 
ORDER BY 
    pg_total_relation_size(relid) DESC
LIMIT 10;
```

### 31. Identify Unused Indexes
#### MySQL
```sql
SELECT 
    s.table_name,
    s.index_name,
    s.non_unique,
    s.seq_in_index,
    s.column_name,
    s.collation,
    s.cardinality,
    s.sub_part,
    s.packed,
    s.nullable,
    s.index_type,
    s.comment,
    s.index_comment
FROM 
    information_schema.statistics AS s
LEFT JOIN 
    (SELECT 
         table_name, 
         index_name
     FROM 
         sys.schema_index_statistics
     WHERE 
         rows_selected = 0) AS t 
ON 
    s.table_name = t.table_name 
    AND s.index_name = t.index_name
WHERE 
    t.table_name IS NOT NULL
ORDER BY 
    s.table_name, 
    s.index_name;
```

#### PostgreSQL
```sql
SELECT 
    indexrelid::regclass AS index,
    relid::regclass AS table,
    idx_scan AS scans
FROM 
    pg_stat_user_indexes 
JOIN 
    pg_index 
USING (indexrelid)
WHERE 
    idx_scan = 0
ORDER BY 
    pg_relation_size(indexrelid) DESC
LIMIT 10;
```

### 32. Identify Duplicate Indexes
#### MySQL
```sql
SELECT 
    s1.table_name,
    s1.index_name AS index1,
    s2.index_name AS index2,
    s1.column_name AS column1,
    s2.column_name AS column2
FROM 
    information_schema.statistics s1
JOIN 
    information_schema.statistics s2 
ON 
    s1.table_schema = s2.table_schema 
    AND s1.table_name = s2.table_name 
    AND s1.index_name != s2.index_name 
    AND s1.seq_in_index = s2.seq_in_index 
    AND s1.column_name = s2.column_name
WHERE 
    s1.non_unique = s2.non_unique
ORDER BY 
    s1.table_name, 
    s1.index_name;
```

#### PostgreSQL
```sql
SELECT 
    ind1.indexrelid::regclass AS indname1,
    ind2.indexrelid::regclass AS indname2,
    ind1.indrelid::regclass AS tablename
FROM 
    pg_index ind1
JOIN 
    pg_index ind2 
ON 
    ind1.indrelid = ind2.indrelid 
    AND ind1.indexrelid <> ind2.indexrelid
WHERE 
    (ind1.indkey = ind2.indkey OR ind1.indkey @> ind2.indkey)
    AND pg_relation_size(ind1.indexrelid) > pg_relation_size(ind2.indexrelid)
ORDER BY 
    pg_relation_size(ind1.indexrelid) DESC
LIMIT 10;
```

### 33. Check Index Usage
#### MySQL
```sql
SELECT 
    table_name, 
    index_name, 
    COUNT(*) AS usage_count 
FROM 
    performance_schema.events_statements_history_long 
WHERE 
    index_name IS NOT NULL 
GROUP BY 
    table_name, 
    index_name 
ORDER BY 
    usage_count DESC
LIMIT 10;
```

#### PostgreSQL
```sql
SELECT 
    indexrelid::regclass AS index,
    relid::regclass AS table,
    idx_scan AS scans,
    idx_tup_read AS tuples_read,
    idx_tup_fetch AS tuples_fetched
FROM 
    pg_stat_user_indexes 
ORDER BY 
    idx_scan DESC
LIMIT 10;
```

### 34. Track Deadlocks
#### MySQL
```sql
SHOW ENGINE INNODB STATUS;
-- Look for the LATEST DETECTED DEADLOCK section in the output.
```

#### PostgreSQL
```sql
SELECT 
    deadlock_id,
    deadlock_start,
    deadlock_end,
    victim,
    blocking_query,
    blocked_query
FROM 
    pg_catalog.pg_deadlock_log
ORDER BY 
    deadlock_end DESC
LIMIT 10;
```

### 35. Analyze Query Plans
#### MySQL
```sql
EXPLAIN SELECT * FROM your_table WHERE your_conditions;
```

#### PostgreSQL
```sql
EXPLAIN ANALYZE SELECT * FROM your_table WHERE your_conditions;
```

### 36. Monitor Query Cache Usage
#### MySQL
```sql
SHOW STATUS LIKE 'Qcache%';
```

#### PostgreSQL
```sql
-- PostgreSQL does not have a query cache, but you can monitor the shared buffer usage:
SELECT 
    pg_database.datname, 
    pg_size_pretty(pg_database_size(pg_database.datname)) AS size, 
    pg_stat_database.blks_hit, 
    pg_stat_database.blks_read
FROM 
    pg_stat_database;
```

<p>These scripts will help you keep a close eye on the queries running on your databases, identify performance bottlenecks, and optimize query execution to ensure efficient database operations.</p>





### 17. Rotate Database Passwords
#### MySQL
```sql
SET PASSWORD FOR 'username'@'%' = PASSWORD('newpassword');
```

#### PostgreSQL
```sql
ALTER USER username WITH PASSWORD 'newpassword';
```

### 19. Monitor Disk Space Usage
#### MySQL
```sql
SELECT table_schema AS 'Database Name',
       ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)'
FROM information_schema.tables
GROUP BY table_schema;
```

#### PostgreSQL
```sql
SELECT pg_database.datname,
       pg_size_pretty(pg_database_size(pg_database.datname)) AS size
FROM pg_database;
```

### 21. Adjust Storage Auto Scaling
#### MySQL and PostgreSQL
```bash
aws rds modify-db-instance --db-instance-identifier your-rds-instance-id --allocated-storage 100 --max-allocated-storage 200 --apply-immediately
```

### 22. Enabling Encryption at Rest
#### MySQL and PostgreSQL
```bash
aws rds modify-db-instance --db-instance-identifier your-rds-instance-id --storage-encrypted --kms-key-id your-kms-key-id
```

### 23. Perform Table Maintenance
#### MySQL
```sql
OPTIMIZE TABLE table_name;
```

#### PostgreSQL
```sql
VACUUM FULL table_name;
```

### 24. Set Up Database Alarms with CloudWatch
#### MySQL and PostgreSQL
```bash
aws cloudwatch put-metric-alarm --alarm-name "HighCPUUtilization" --metric-name CPUUtilization --namespace "AWS/RDS" --statistic Average --period 300 --threshold 80 --comparison-operator GreaterThanOrEqualToThreshold --dimensions Name=DBInstanceIdentifier,Value=your-rds-instance-id --evaluation-periods 1 --alarm-actions arn:aws:sns:your-region:your-account-id:your-topic
```

### 25. Checking Long-Running Queries
#### MySQL
```sql
SELECT * FROM information_schema.processlist WHERE command != 'Sleep' AND time > 10;
```

#### PostgreSQL
```sql
SELECT pid, now() - pg_stat_activity.query_start AS duration, query
FROM pg_stat_activity
WHERE state != 'idle' AND now() - pg_stat_activity.query_start > interval '10 minutes';
```
