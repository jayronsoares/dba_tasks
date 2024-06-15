Here are ten essential scripts for daily support, maintenance, and troubleshooting of AWS RDS MySQL and PostgreSQL databases:

### 1. Monitor Database Health
#### MySQL and PostgreSQL
```bash
#!/bin/bash

DB_INSTANCE_IDENTIFIER="your-rds-instance-id"

aws rds describe-db-instances --db-instance-identifier $DB_INSTANCE_IDENTIFIER --query 'DBInstances[*].{DBInstanceIdentifier:DBInstanceIdentifier,DBInstanceStatus:DBInstanceStatus,DBInstanceClass:DBInstanceClass,AllocatedStorage:AllocatedStorage,StorageType:StorageType,Engine:Engine,EngineVersion:EngineVersion,Endpoint:Endpoint.Address,Port:Endpoint.Port,MasterUsername:MasterUsername,DBName:DBName}' --output table
```

### 2. Check CPU Utilization
#### MySQL and PostgreSQL
```bash
#!/bin/bash

DB_INSTANCE_IDENTIFIER="your-rds-instance-id"

aws cloudwatch get-metric-statistics --metric-name CPUUtilization --start-time $(date -u -d '1 hour ago' +"%Y-%m-%dT%H:%M:%SZ") --end-time $(date -u +"%Y-%m-%dT%H:%M:%SZ") --period 60 --namespace AWS/RDS --statistics Average --dimensions Name=DBInstanceIdentifier,Value=$DB_INSTANCE_IDENTIFIER --output table
```

### 3. Check Database Storage Usage
#### MySQL and PostgreSQL
```bash
#!/bin/bash

DB_INSTANCE_IDENTIFIER="your-rds-instance-id"

aws rds describe-db-instances --db-instance-identifier $DB_INSTANCE_IDENTIFIER --query 'DBInstances[*].{DBInstanceIdentifier:DBInstanceIdentifier,AllocatedStorage:AllocatedStorage,FreeStorage:FreeStorage}' --output table
```

### 4. List Active Connections
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

### 5. Identify Long-Running Queries
#### MySQL
```sql
SELECT 
    * 
FROM 
    information_schema.processlist 
WHERE 
    time > 60 
    AND command != 'Sleep';
```

#### PostgreSQL
```sql
SELECT 
    pid, 
    now() - pg_stat_activity.query_start AS duration, 
    query 
FROM 
    pg_stat_activity 
WHERE 
    state != 'idle' 
    AND now() - pg_stat_activity.query_start > interval '1 minute';
```

### 6. Rebuild Indexes
#### MySQL
```sql
ALTER TABLE your_table ENGINE=InnoDB;
```

#### PostgreSQL
```sql
REINDEX TABLE your_table;
```

### 7. Update Statistics
#### MySQL
```sql
ANALYZE TABLE your_table;
```

#### PostgreSQL
```sql
VACUUM ANALYZE;
```

### 8. Backup Database
#### MySQL
```bash
#!/bin/bash

DB_INSTANCE_IDENTIFIER="your-rds-instance-id"
BUCKET_NAME="your-s3-bucket"
DATE=$(date +%Y-%m-%d)

mysqldump -h $DB_INSTANCE_IDENTIFIER.cwovfnrvhjuj.us-west-2.rds.amazonaws.com -u username -p'password' --all-databases > backup-$DATE.sql
aws s3 cp backup-$DATE.sql s3://$BUCKET_NAME/backup-$DATE.sql --region your-region
```

#### PostgreSQL
```bash
#!/bin/bash

DB_INSTANCE_IDENTIFIER="your-rds-instance-id"
BUCKET_NAME="your-s3-bucket"
DATE=$(date +%Y-%m-%d)

PGPASSWORD=password pg_dump -h $DB_INSTANCE_IDENTIFIER.cwovfnrvhjuj.us-west-2.rds.amazonaws.com -U username -F c -b -v -f backup-$DATE.backup
aws s3 cp backup-$DATE.backup s3://$BUCKET_NAME/backup-$DATE.backup --region your-region
```

### 9. Restore Database
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

### 10. Monitor Disk Space Usage by Table
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

These scripts are designed to help with the daily monitoring, maintenance, and troubleshooting tasks required to keep your AWS RDS MySQL and PostgreSQL databases running smoothly. Adjust the scripts to fit your specific environment and requirements.


Here are ten crucial scripts for daily support, maintenance, and troubleshooting of AWS RDS MySQL and PostgreSQL databases:

### 1. Monitor Database Health
#### MySQL
```sql
SHOW STATUS LIKE 'Uptime';
SHOW STATUS LIKE 'Threads_connected';
SHOW STATUS LIKE 'Questions';
SHOW STATUS LIKE 'Slow_queries';
SHOW STATUS LIKE 'Aborted_connects';
```

#### PostgreSQL
```sql
SELECT
    current_database(),
    pg_size_pretty(pg_database_size(current_database())) AS size,
    numbackends,
    xact_commit,
    xact_rollback,
    blks_read,
    blks_hit,
    tup_returned,
    tup_fetched,
    tup_inserted,
    tup_updated,
    tup_deleted
FROM pg_stat_database
WHERE datname = current_database();
```

### 2. Check Disk Space Usage
#### MySQL
```sql
SELECT table_schema AS 'Database', 
       ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)'
FROM information_schema.TABLES
GROUP BY table_schema;
```

#### PostgreSQL
```sql
SELECT 
    pg_database.datname, 
    pg_size_pretty(pg_database_size(pg_database.datname)) AS size 
FROM 
    pg_database;
```

### 3. Identify Long-Running Queries
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
    command != 'Sleep' 
    AND time > 10 
ORDER BY time DESC;
```

#### PostgreSQL
```sql
SELECT 
    pid, 
    now() - pg_stat_activity.query_start AS duration, 
    query 
FROM 
    pg_stat_activity 
WHERE 
    state != 'idle' 
    AND now() - pg_stat_activity.query_start > interval '10 minutes';
```

### 4. Backup Database
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

### 5. Check Index Usage
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

### 6. Analyze Query Performance
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
ORDER BY 
    total_exec_time DESC 
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
ORDER BY 
    total_time DESC 
LIMIT 10;
```

### 7. Rebuild Indexes
#### MySQL
```sql
ALTER TABLE table_name ENGINE=InnoDB;
```

#### PostgreSQL
```sql
REINDEX DATABASE database_name;
```

### 8. Update Statistics
#### MySQL
```sql
ANALYZE TABLE table_name;
```

#### PostgreSQL
```sql
VACUUM ANALYZE;
```

### 9. Check for Deadlocks
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

### 10. Monitor Active Connections
#### MySQL
```sql
SHOW PROCESSLIST;
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

These scripts cover a range of daily tasks necessary for maintaining the health, performance, and reliability of your AWS RDS MySQL and PostgreSQL databases. Make sure to customize and test them in your specific environment before regular use.
