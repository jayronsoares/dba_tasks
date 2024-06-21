## MySQL Backup Verification
## Script to Verify Backup in MySQL:

#!/bin/bash

# Variables
RDS_INSTANCE="your-rds-instance-endpoint"
MYSQL_USER="your-username"
MYSQL_PASSWORD="your-password"
BACKUP_DIR="/path/to/backup"

## # Create a backup
mysqldump -h $RDS_INSTANCE -u $MYSQL_USER -p$MYSQL_PASSWORD --all-databases > $BACKUP_DIR/all_databases_backup.sql

# Verify the backup
if [ $? -eq 0 ]; then
  echo "MySQL backup completed successfully."
else
  echo "MySQL backup failed." >&2
  exit 1
fi

# Test restore (to a test database)
mysql -h $RDS_INSTANCE -u $MYSQL_USER -p$MYSQL_PASSWORD -e "CREATE DATABASE test_restore;"
mysql -h $RDS_INSTANCE -u $MYSQL_USER -p$MYSQL_PASSWORD test_restore < $BACKUP_DIR/all_databases_backup.sql

if [ $? -eq 0 ]; then
  echo "MySQL restore test completed successfully."
else
  echo "MySQL restore test failed." >&2
  exit 1
fi

# Clean up
mysql -h $RDS_INSTANCE -u $MYSQL_USER -p$MYSQL_PASSWORD -e "DROP DATABASE test_restore;"


## PostgreSQL Backup Verification

#!/bin/bash

# Variables
RDS_INSTANCE="your-rds-instance-endpoint"
PG_USER="your-username"
PG_PASSWORD="your-password"
BACKUP_DIR="/path/to/backup"

export PGPASSWORD=$PG_PASSWORD

# Create a backup
pg_dumpall -h $RDS_INSTANCE -U $PG_USER > $BACKUP_DIR/all_databases_backup.sql

# Verify the backup
if [ $? -eq 0 ]; then
  echo "PostgreSQL backup completed successfully."
else
  echo "PostgreSQL backup failed." >&2
  exit 1
fi

## Test restore (to a test database)
psql -h $RDS_INSTANCE -U $PG_USER -c "CREATE DATABASE test_restore;"
psql -h $RDS_INSTANCE -U $PG_USER test_restore < $BACKUP_DIR/all_databases_backup.sql

if [ $? -eq 0 ]; then
  echo "PostgreSQL restore test completed successfully."
else
  echo "PostgreSQL restore test failed." >&2
  exit 1
fi

# Clean up
psql -h $RDS_INSTANCE -U $PG_USER -c "DROP DATABASE test_restore;"



# MySQL Security Review
# Script to Check User Permissions in MySQL:

-- SQL Script to Review User Permissions in MySQL

SELECT user, host, authentication_string FROM mysql.user;

-- Check for users with excessive privileges
SELECT user, host, Select_priv, Insert_priv, Update_priv, Delete_priv, Create_priv, Drop_priv, Reload_priv, Shutdown_priv, Process_priv, File_priv, Grant_priv, References_priv, Index_priv, Alter_priv, Show_db_priv, Super_priv, Create_tmp_table_priv, Lock_tables_priv, Execute_priv, Repl_slave_priv, Repl_client_priv, Create_view_priv, Show_view_priv, Create_routine_priv, Alter_routine_priv, Create_user_priv, Event_priv, Trigger_priv 
FROM mysql.user WHERE Super_priv = 'Y';


## PostgreSQL Security Review
# Script to Check User Permissions in PostgreSQL:

-- SQL Script to Review User Permissions in PostgreSQL

SELECT usename, usecreatedb, usesuper, usecatupd FROM pg_user;

-- Check roles and their permissions
SELECT rolname, rolsuper, rolcreaterole, rolcreatedb, rolcanlogin FROM pg_roles;

-- Ensure there are no roles with unnecessary superuser privileges
SELECT rolname FROM pg_roles WHERE rolsuper IS TRUE;

## 1. Inventory of RDS Instances
## Using the AWS CLI, you can list all your RDS instances along with their details.

## AWS CLI Script to List RDS Instances:

#!/bin/bash

# List all RDS instances
aws rds describe-db-instances --query "DBInstances[*].{DBInstanceIdentifier:DBInstanceIdentifier,Engine:Engine,DBInstanceStatus:DBInstanceStatus,AllocatedStorage:AllocatedStorage,DBInstanceClass:DBInstanceClass,Endpoint:Endpoint.Address,MultiAZ:MultiAZ}" --output table


#!/bin/bash

# AWS CLI: List RDS Instances
echo "Listing RDS Instances:"
aws rds describe-db-instances --query "DBInstances[*].{DBInstanceIdentifier:DBInstanceIdentifier,Engine:Engine,DBInstanceStatus:DBInstanceStatus,AllocatedStorage:AllocatedStorage,DBInstanceClass:DBInstanceClass,Endpoint:Endpoint.Address,MultiAZ:MultiAZ}" --output table

# MySQL Health Check
MYSQL_HOST="your-mysql-host"
MYSQL_USER="your-username"
MYSQL_PASSWORD="your-password"
MYSQL_CMD="mysql -h $MYSQL_HOST -u $MYSQL_USER -p$MYSQL_PASSWORD"

echo "MySQL Server Status:"
$MYSQL_CMD -e "SHOW GLOBAL STATUS;"

echo "MySQL Slow Queries:"
$MYSQL_CMD -e "SHOW VARIABLES LIKE 'slow_query_log';"
$MYSQL_CMD -e "SHOW VARIABLES LIKE 'long_query_time';"
$MYSQL_CMD -e "SHOW GLOBAL STATUS LIKE 'Slow_queries';"

echo "MySQL Replication Status:"
$MYSQL_CMD -e "SHOW SLAVE STATUS\G"

# PostgreSQL Health Check
PG_HOST="your-pg-host"
PG_USER="your-username"
PG_PASSWORD="your-password"
PG_CMD="psql -h $PG_HOST -U $PG_USER"

export PGPASSWORD=$PG_PASSWORD

echo "PostgreSQL Server Status:"
$PG_CMD -c "SELECT * FROM pg_stat_activity;"

echo "PostgreSQL Long-Running Queries:"
$PG_CMD -c "SELECT pid, age(clock_timestamp(), query_start), usename, query FROM pg_stat_activity WHERE state != 'idle' AND query_start < now() - interval '5 minutes' ORDER BY query_start;"

echo "PostgreSQL Replication Status:"
$PG_CMD -c "SELECT * FROM pg_stat_replication;"

# MongoDB Health Check
MONGO_HOST="your-mongo-host"
MONGO_PORT="your-mongo-port"
MONGO_USER="your-username"
MONGO_PASSWORD="your-password"
MONGO_CMD="mongo --host $MONGO_HOST --port $MONGO_PORT -u $MONGO_USER -p $MONGO_PASSWORD --eval"

echo "MongoDB Server Status:"
$MONGO_CMD "db.serverStatus();"

echo "MongoDB Replication Status:"
$MONGO_CMD "rs.status();"

echo "MongoDB Slow Queries:"
$MONGO_CMD "db.system.profile.find({ millis: { \$gt: 1000 } }).sort({ ts: -1 });"
