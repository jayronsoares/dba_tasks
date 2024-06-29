### Creating a comprehensive report on your AWS RDS Aurora MySQL instances to identify cost-saving opportunities involves gathering data on various dimensions.

### 1. Storage Types and IOPS
- **AWS CLI:**
  ```bash
  aws rds describe-db-instances --query 'DBInstances[*].[DBInstanceIdentifier,StorageType,Iops]' --output table
  ```

- **SQL (DBeaver):**
  ```sql
  SELECT
      @@global.innodb_file_per_table,
      @@global.innodb_io_capacity
  FROM DUAL;
  ```

### 2. Data Storage Size
- **AWS CLI:**
  ```bash
  aws rds describe-db-instances --query 'DBInstances[*].[DBInstanceIdentifier,AllocatedStorage]' --output table
  ```

- **SQL (DBeaver):**
  ```sql
  SELECT table_schema, 
         ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS "Size (MB)" 
  FROM information_schema.tables 
  GROUP BY table_schema;
  ```

### 3. Backups Size and Retention
- **AWS CLI:**
  ```bash
  aws rds describe-db-snapshots --query 'DBSnapshots[*].[DBSnapshotIdentifier,AllocatedStorage,SnapshotCreateTime,RetentionPeriod]' --output table
  ```

- **SQL (DBeaver):**
  ```sql
  SELECT engine, backup_type, backup_time, backup_size 
  FROM mysql.backups
  ORDER BY backup_time DESC;
  ```

### 4. Logs Size and Retention
- **AWS CLI:**
  ```bash
  aws rds describe-db-log-files --db-instance-identifier <db-instance-id> --query 'DescribeDBLogFiles[*].[LogFileName,Size]' --output table
  ```

- **SQL (DBeaver):**
  ```sql
  SHOW VARIABLES LIKE 'expire_logs_days';
  ```

### 5. High Availability and Replicas
- **AWS CLI:**
  ```bash
  aws rds describe-db-instances --query 'DBInstances[*].[DBInstanceIdentifier,MultiAZ,ReadReplicaDBInstanceIdentifiers]' --output table
  ```

- **SQL (DBeaver):**
  ```sql
  SELECT master_id, slave_hosts 
  FROM information_schema.replication 
  WHERE slave_hosts IS NOT NULL;
  ```

### 6. Cross-Region Data Transfer Size
- **AWS CLI:**
  ```bash
  aws cloudwatch get-metric-statistics \
    --namespace AWS/RDS \
    --metric-name NetworkTransmitThroughput \
    --dimensions Name=DBInstanceIdentifier,Value=<db-instance-id> \
    --statistics Average \
    --period 3600 \
    --start-time 2023-06-01T00:00:00Z \
    --end-time 2023-06-30T23:59:59Z \
    --query 'Datapoints[*].[Timestamp,Average]' \
    --output table
  ```

- **SQL (DBeaver):**
  ```sql
  SELECT 
      SUM(data_length + index_length) AS "Cross-Region Data Transfer (Bytes)" 
  FROM information_schema.tables;
  ```

### 7. Encryption Key
- **AWS CLI:**
  ```bash
  aws rds describe-db-instances --query 'DBInstances[*].[DBInstanceIdentifier,KmsKeyId]' --output table
  ```

- **SQL (DBeaver):**
  ```sql
  SHOW VARIABLES LIKE 'innodb_encryption_key_id';
  ```

### 8. Secrets to Store Passwords
- **AWS CLI:**
  ```bash
  aws secretsmanager list-secrets --query 'SecretList[*].[Name,ARN]' --output table
  ```

- **SQL (DBeaver):**
  ```sql
  SELECT * FROM information_schema.key_column_usage WHERE table_schema = 'mysql' AND column_name = 'password';
  ```

### 9. General Instance Information
- **AWS CLI:**
  ```bash
  aws rds describe-db-instances --query 'DBInstances[*].[DBInstanceIdentifier,DBInstanceClass,Engine,DBInstanceStatus,MasterUsername,Endpoint.Address,Endpoint.Port,InstanceCreateTime]' --output table
  ```

- **SQL (DBeaver):**
  ```sql
  SELECT * FROM information_schema.processlist;
  ```

### 10. Performance Metrics
- **AWS CLI:**
  ```bash
  aws cloudwatch get-metric-statistics \
    --namespace AWS/RDS \
    --metric-name CPUUtilization \
    --dimensions Name=DBInstanceIdentifier,Value=<db-instance-id> \
    --statistics Average \
    --period 3600 \
    --start-time 2023-06-01T00:00:00Z \
    --end-time 2023-06-30T23:59:59Z \
    --query 'Datapoints[*].[Timestamp,Average]' \
    --output table
  ```

- **SQL (DBeaver):**
  ```sql
  SELECT * FROM information_schema.performance_schema;
  ```
---

### Top 8 Columns for the Report

1. **Instance Identifier**
   - **Description:** Unique identifier for each RDS instance.
   - **Purpose:** Identifies each database instance and allows for easy cross-referencing.
   - **Source:** AWS CLI - `describe-db-instances`

2. **Storage Details**
   - **Description:** Information about the storage type, IOPS, and data storage size.
   - **Purpose:** Provides insights into storage costs and performance.
   - **Source:** AWS CLI - `describe-db-instances`, SQL - `information_schema.tables`

3. **Backup Details**
   - **Description:** Backup size and retention period.
   - **Purpose:** Helps in assessing the backup strategy and potential cost savings.
   - **Source:** AWS CLI - `describe-db-snapshots`, SQL - `mysql.backups`

4. **Log Management**
   - **Description:** Log file sizes and retention period.
   - **Purpose:** Identifies log management practices and potential for optimization.
   - **Source:** AWS CLI - `describe-db-log-files`, SQL - `SHOW VARIABLES`

5. **High Availability & Replicas**
   - **Description:** Details on multi-AZ setup and read replicas.
   - **Purpose:** Evaluates high availability setup and redundancy, important for cost and performance.
   - **Source:** AWS CLI - `describe-db-instances`, SQL - `information_schema.replication`

6. **Cross-Region Data Transfer**
   - **Description:** Size of data transferred across regions.
   - **Purpose:** Helps in understanding data transfer costs and opportunities for optimization.
   - **Source:** AWS CLI - `get-metric-statistics`, SQL - `information_schema.tables`

7. **Encryption & Security**
   - **Description:** Encryption key details and password storage practices.
   - **Purpose:** Ensures compliance with security policies and identifies security risks.
   - **Source:** AWS CLI - `describe-db-instances`, `list-secrets`, SQL - `information_schema.key_column_usage`

8. **Performance Metrics**
   - **Description:** Key performance indicators like CPU utilization.
   - **Purpose:** Provides data for performance tuning and cost optimization.
   - **Source:** AWS CLI - `get-metric-statistics`, SQL - `information_schema.performance_schema`

### Example Table

| Instance Identifier | Storage Details        | Backup Details      | Log Management         | High Availability & Replicas | Cross-Region Data Transfer | Encryption & Security      | Performance Metrics       |
|---------------------|------------------------|---------------------|------------------------|-----------------------------|---------------------------|----------------------------|---------------------------|
| db-instance-1       | General Purpose SSD, 300 IOPS, 200 GB | 50 GB, 7 days | 10 GB, 7 days           | Multi-AZ, 2 replicas        | 500 MB                    | KMS Key: abc123, Secrets: Yes | CPU Avg: 30%, Mem: 60% |
| db-instance-2       | Provisioned IOPS, 1000 IOPS, 500 GB  | 120 GB, 14 days | 20 GB, 14 days         | No Multi-AZ, 1 replica     | 1 GB                      | KMS Key: def456, Secrets: No | CPU Avg: 25%, Mem: 55% |
| ...                 | ...                    | ...                 | ...                    | ...                         | ...                       | ...                        | ...                       |

