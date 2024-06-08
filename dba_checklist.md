To perform an inventory of the databases you are managing, including their sizes, usage patterns, and workloads, you can follow a structured approach. Here’s a step-by-step guide to help you gather this information:

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
