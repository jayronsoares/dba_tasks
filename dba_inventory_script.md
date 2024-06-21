To create a comprehensive inventory of your databases, including their sizes, usage patterns, and workloads, follow these steps:

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
