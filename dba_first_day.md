To effectively manage your new position as a DBA for AWS RDS instances of MySQL, PostgreSQL, and MongoDB, you need essential SQL commands and AWS CLI commands to gather information and maintain the databases. Here are the key commands to use:

### Essential SQL Commands

#### MySQL

1. **List All Databases:**
   ```sql
   SHOW DATABASES;
   ```

2. **Show Database Size:**
   ```sql
   SELECT table_schema "Database", 
          ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) "Size (MB)" 
   FROM information_schema.TABLES 
   GROUP BY table_schema;
   ```

3. **Check Table Status:**
   ```sql
   SHOW TABLE STATUS;
   ```

4. **View Running Processes:**
   ```sql
   SHOW PROCESSLIST;
   ```

5. **Check Replication Status:**
   ```sql
   SHOW SLAVE STATUS\G;
   ```

6. **View User Privileges:**
   ```sql
   SHOW GRANTS FOR 'username'@'host';
   ```

#### PostgreSQL

1. **List All Databases:**
   ```sql
   \l
   ```

2. **Show Database Size:**
   ```sql
   SELECT pg_database.datname, pg_size_pretty(pg_database_size(pg_database.datname)) AS size 
   FROM pg_database;
   ```

3. **Check Table Status:**
   ```sql
   \dt+
   ```

4. **View Running Queries:**
   ```sql
   SELECT * FROM pg_stat_activity;
   ```

5. **Check Replication Status:**
   ```sql
   SELECT * FROM pg_stat_replication;
   ```

6. **View User Privileges:**
   ```sql
   \du
   ```

#### MongoDB

1. **List All Databases:**
   ```javascript
   show dbs;
   ```

2. **Show Collection Sizes:**
   ```javascript
   db.stats();
   ```

3. **Check Replica Set Status:**
   ```javascript
   rs.status();
   ```

4. **View Current Operations:**
   ```javascript
   db.currentOp();
   ```

5. **List Users:**
   ```javascript
   db.getUsers();
   ```

### AWS CLI Commands

1. **Configure AWS CLI:**
   ```sh
   aws configure
   ```

2. **List All RDS Instances:**
   ```sh
   aws rds describe-db-instances
   ```

3. **Get Detailed Information About a Specific RDS Instance:**
   ```sh
   aws rds describe-db-instances --db-instance-identifier your-instance-id
   ```

4. **Check RDS Instance Status:**
   ```sh
   aws rds describe-db-instances --query 'DBInstances[*].[DBInstanceIdentifier,DBInstanceStatus]'
   ```

5. **List All Snapshots:**
   ```sh
   aws rds describe-db-snapshots
   ```

6. **Describe Events for RDS Instances:**
   ```sh
   aws rds describe-events --duration 360
   ```

7. **List RDS Security Groups:**
   ```sh
   aws rds describe-db-security-groups
   ```

8. **Get CloudWatch Metrics for RDS Instances:**
   ```sh
   aws cloudwatch get-metric-statistics --namespace AWS/RDS --metric-name CPUUtilization --start-time 2023-06-01T00:00:00Z --end-time 2023-06-02T00:00:00Z --period 3600 --statistics Average --dimensions Name=DBInstanceIdentifier,Value=your-instance-id
   ```

### Setting Up Your Laptop for Productivity

1. **Install AWS CLI:**
   ```sh
   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
   unzip awscliv2.zip
   sudo ./aws/install
   aws configure
   ```

2. **Install Database Clients:**
   - **MySQL Client:**
     ```sh
     sudo apt-get update
     sudo apt-get install mysql-client
     ```
   - **PostgreSQL Client:**
     ```sh
     sudo apt-get update
     sudo apt-get install postgresql-client
     ```
   - **MongoDB Client:**
     ```sh
     wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
     echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
     sudo apt-get update
     sudo apt-get install -y mongodb-org-shell
     ```

3. **Install Monitoring Tools:**
   - **Prometheus:**
     ```sh
     sudo apt-get install prometheus
     ```
   - **Grafana:**
     ```sh
     sudo apt-get install grafana
     ```

4. **Install GUI Tools:**
   - **pgAdmin:**
     ```sh
     sudo apt-get install pgadmin4
     ```
   - **MySQL Workbench:**
     ```sh
     sudo apt-get install mysql-workbench
     ```
   - **Robo 3T for MongoDB:**
     ```sh
     wget https://download-test.robomongo.org/1.4.3/linux/robo3t-1.4.3-linux-x86_64-7419c406.tar.gz
     tar -xvzf robo3t-1.4.3-linux-x86_64-7419c406.tar.gz
     sudo mv robo3t-1.4.3-linux-x86_64-7419c406 /usr/local/bin/robo3t
     ```

By using these essential SQL commands and AWS CLI commands, you can gather critical information about your databases and ensure you have the necessary tools set up on your laptop to perform your duties effectively.
