### Here’s a step-by-step guide to perform the databases inventory:

### 1. **Access Your Databases**
   - Ensure you have the necessary permissions to access all the databases you need to inventory.
   - Connect to each database using appropriate tools (e.g., MySQL Workbench, pgAdmin, MongoDB Compass, or command-line tools).

### 2. **List Databases**
   - **MySQL**:
     ```sql
     SHOW DATABASES;
     ```
   - **PostgreSQL**:
     ```sql
     \l
     ```
   - **MongoDB**:
     ```shell
     show dbs;
     ```

### 3. **Gather Database Sizes**
   - **MySQL**: 
     ```sql
     SELECT table_schema AS "Database",
            ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS "Size (MB)"
     FROM information_schema.tables
     GROUP BY table_schema;
     ```
   - **PostgreSQL**:
     ```sql
     SELECT pg_database.datname as "Database",
            pg_size_pretty(pg_database_size(pg_database.datname)) as "Size"
     FROM pg_database;
     ```
   - **MongoDB**:
     ```shell
     db.stats();  // Run this command for each database
     ```

### 4. **Identify Usage Patterns**
   - Review the number of connections, read/write operations, and other relevant metrics.
   - **MySQL**: 
     ```sql
     SHOW PROCESSLIST;
     SHOW STATUS LIKE 'Connections';
     SHOW STATUS LIKE 'Com%';
     ```
   - **PostgreSQL**:
     ```sql
     SELECT datname, numbackends as "Connections",
            xact_commit as "Commits",
            xact_rollback as "Rollbacks",
            blks_read as "Blocks Read",
            blks_hit as "Blocks Hit"
     FROM pg_stat_database;
     ```
   - **MongoDB**:
     ```shell
     db.serverStatus();  // Provides extensive metrics
     ```

### 5. **Analyze Workloads**
   - Look at query logs, slow queries, and execution plans.
   - **MySQL**: Enable and review the slow query log.
     ```sql
     SET GLOBAL slow_query_log = 'ON';
     SET GLOBAL long_query_time = 1;  // Queries longer than 1 second
     ```
   - **PostgreSQL**: Enable and review the slow query log.
     ```sql
     ALTER SYSTEM SET log_min_duration_statement = '1s';  // Log queries longer than 1 second
     ```
   - **MongoDB**: Use the profiler to identify slow operations.
     ```shell
     db.setProfilingLevel(1, { slowms: 100 });  // Log queries slower than 100ms
     ```

### 6. **Document the Findings**
   - Create a structured document or spreadsheet to record the inventory.
   - Include columns for:
     - **Database Name**
     - **Type (MySQL, PostgreSQL, MongoDB)**
     - **Size (MB/GB)**
     - **Number of Connections**
     - **Read/Write Operations**
     - **Slow Queries/Operations**
     - **Comments** (e.g., notable patterns, issues)

### 7. **Use Automation Tools (Optional)**
   - **AWS CLI**: For databases hosted on AWS, you can use the AWS CLI to list and describe your RDS and DocumentDB instances.
     ```shell
     aws rds describe-db-instances
     aws docdb describe-db-clusters
     ```
   - **Scripts**: Write scripts in Python or other languages to automate the collection of this information.

### Example of Documenting Inventory
Here’s an example structure for your documentation:

| Database Name | Type       | Size (MB/GB) | Connections | Read Ops | Write Ops | Slow Queries | Comments                |
|---------------|------------|--------------|-------------|----------|-----------|--------------|-------------------------|
| db1           | MySQL      | 500 MB       | 10          | 1000/sec | 200/sec   | 5/sec        | High read workload      |
| db2           | PostgreSQL | 2 GB         | 5           | 300/sec  | 100/sec   | 2/sec        | Moderate usage          |
| db3           | MongoDB    | 1.5 GB       | 15          | 500/sec  | 150/sec   | 10/sec       | Needs index optimization|

### 1. **Identify and List Databases**

#### **MySQL and PostgreSQL (RDS)**

1. **AWS Management Console**:
   - Go to the **RDS Dashboard**.
   - List all instances and their respective details.

2. **AWS CLI**:
   - Install and configure the AWS CLI if not already done.
   - Use the following command to list your RDS instances:
     ```sh
     aws rds describe-db-instances
     ```
   - This command will output details of all your RDS instances, including MySQL and PostgreSQL.

#### **MongoDB (Amazon DocumentDB)**

1. **AWS Management Console**:
   - Go to the **DocumentDB Dashboard**.
   - List all clusters and their respective details.

2. **AWS CLI**:
   - Use the following command to list your DocumentDB clusters:
     ```sh
     aws docdb describe-db-clusters
     ```
   - This command will provide details of all your DocumentDB clusters.

### 2. **Gather Database Sizes**

#### **MySQL and PostgreSQL (RDS)**

1. **AWS Management Console**:
   - In the **RDS Dashboard**, click on each instance to view details.
   - Note down the allocated storage and used storage.

2. **AWS CLI**:
   - Use the following command to get storage information:
     ```sh
     aws rds describe-db-instances --query 'DBInstances[*].{DBInstanceIdentifier:DBInstanceIdentifier,AllocatedStorage:AllocatedStorage,FreeStorageSpace:FreeStorageSpace}'
     ```

3. **SQL Queries** (if more granular details are needed):
   - Connect to each database and run the following SQL queries:
     - For MySQL:
       ```sql
       SELECT table_schema AS 'Database', SUM(data_length + index_length) / 1024 / 1024 AS 'Size (MB)'
       FROM information_schema.TABLES
       GROUP BY table_schema;
       ```
     - For PostgreSQL:
       ```sql
       SELECT pg_database.datname, pg_size_pretty(pg_database_size(pg_database.datname)) AS size
       FROM pg_database;
       ```

#### **MongoDB (Amazon DocumentDB)**

1. **MongoDB Shell**:
   - Connect to each cluster and run:
     ```javascript
     db.stats()
     ```
   - This command provides an overview of the database size.

2. **AWS CLI**:
   - Use the following command to describe each cluster and get size details:
     ```sh
     aws docdb describe-db-clusters --query 'DBClusters[*].{DBClusterIdentifier:DBClusterIdentifier,AllocatedStorage:AllocatedStorage}'
     ```

### 3. **Analyze Usage Patterns**

#### **MySQL and PostgreSQL (RDS)**

1. **CloudWatch Metrics**:
   - Go to **CloudWatch** in the AWS Management Console.
   - Set up dashboards to monitor CPU usage, memory usage, read/write IOPS, and database connections.

2. **Performance Insights**:
   - Enable **Performance Insights** in the RDS Dashboard.
   - Analyze query performance, wait statistics, and resource usage.

#### **MongoDB (Amazon DocumentDB)**

1. **CloudWatch Metrics**:
   - Similar to RDS, set up CloudWatch dashboards for monitoring metrics like CPU usage, memory usage, read/write IOPS, and database connections.

2. **Database Profiling**:
   - Enable profiling in MongoDB to collect detailed information about query performance:
     ```javascript
     db.setProfilingLevel(2)
     ```
   - Review the profiling data using:
     ```javascript
     db.system.profile.find().pretty()
     ```

### 4. **Determine Workloads**

#### **MySQL and PostgreSQL (RDS)**

1. **SQL Query Logs**:
   - Enable and analyze slow query logs to identify heavy queries and workloads.
   - Use the `mysql.slow_log` table in MySQL or the `pg_stat_activity` view in PostgreSQL to gather information.

2. **Application Logs**:
   - Review application logs to understand usage patterns and high-load periods.

#### **MongoDB (Amazon DocumentDB)**

1. **Query Logs**:
   - Enable and review slow query logs to identify heavy operations:
     ```javascript
     db.setProfilingLevel(1)
     db.system.profile.find({ millis: { $gt: 100 } }).pretty()
     ```

2. **Application Logs**:
   - Similar to RDS, review application logs to understand how MongoDB is being used and identify peak usage times.

### 5. **Compile the Inventory**

Create a document or spreadsheet with the following columns:
- **Database Type (MySQL/PostgreSQL/MongoDB)**
- **Database Name**
- **Instance/Cluster Identifier**
- **Size (GB)**
- **Allocated Storage (GB)**
- **Used Storage (GB)**
- **CPU Usage (average and peak)**
- **Memory Usage (average and peak)**
- **IOPS (average and peak)**
- **Connections (average and peak)**
- **Primary Workloads (OLTP, OLAP, mixed, etc.)**
- **Heavy Query Patterns or Times**
