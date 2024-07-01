
## **Handling Deadlocks in AWS RDS MySQL**

### **Step-by-Step Process**

1. **Enable Deadlock Logging**: Ensure that deadlocks are logged so you can quickly diagnose the problem.
    - **AWS Console**: Go to **RDS** > **Databases** > select your DB instance > **Logs & events** > enable **General Log** and **Slow Query Log**.
      
2. **Query Performance Schema to Identify Deadlocks**:
    Use the `performance_schema` to find threads and processes involved in deadlocks.

    **Example Script**:
    ```sql
    SELECT NAME, THREAD_ID, PROCESSLIST_ID, THREAD_OS_ID
    FROM performance_schema.threads
    WHERE PROCESSLIST_STATE = 'locked';
    ```

3. **Identify the Thread Holding the Lock**:
    Find the offending threads causing the deadlock.
    ```sql
    SELECT t.THREAD_ID, t.PROCESSLIST_ID, t.NAME, l.LOCK_TYPE, l.OBJECT_SCHEMA, l.OBJECT_NAME
    FROM performance_schema.data_locks l
    JOIN performance_schema.threads t ON l.THREAD_ID = t.THREAD_ID
    WHERE l.LOCK_STATUS = 'GRANTED';
    ```

4. **Kill the Problematic Sessions**:
    Use the AWS RDS specific stored procedure to kill the sessions identified in the previous step.

    **Example Script**:
    ```sql
    -- Replace <PROCESSLIST_ID> with the actual process ID from the above query.
    CALL mysql.rds_kill(<PROCESSLIST_ID>);
    ```

### **Automated Script to Handle Deadlocks**

You can create a stored procedure to automate this process.

**Example Stored Procedure**:
```sql
DELIMITER //

CREATE PROCEDURE handle_deadlocks()
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE lock_thread_id BIGINT;
    DECLARE lock_processlist_id BIGINT;
    DECLARE cur CURSOR FOR 
        SELECT THREAD_ID, PROCESSLIST_ID 
        FROM performance_schema.threads 
        WHERE PROCESSLIST_STATE = 'locked';
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

    OPEN cur;

    read_loop: LOOP
        FETCH cur INTO lock_thread_id, lock_processlist_id;
        IF done THEN
            LEAVE read_loop;
        END IF;

        -- Kill the session holding the lock
        CALL mysql.rds_kill(lock_processlist_id);
    END LOOP;

    CLOSE cur;
END //

DELIMITER ;
```

**To execute the procedure**:
```sql
CALL handle_deadlocks();
```

---

## **Handling Deadlocks in AWS RDS PostgreSQL**

### **Step-by-Step Process**

1. **Enable Deadlock Logging**:
    Modify the parameter group for your PostgreSQL instance to log deadlocks.
    - **AWS Console**: Go to **RDS** > **Parameter groups** > select your parameter group > set `log_lock_waits` to **1** and `log_min_messages` to **WARNING**.

2. **Identify Blocking and Blocked Sessions**:
    Use the `pg_stat_activity` view to find the blocking sessions.

    **Example Script**:
    ```sql
    SELECT
        bl.pid AS blocked_pid,
        a.usename AS blocked_user,
        ka.query AS blocking_statement,
        ka.pid AS blocking_pid
    FROM
        pg_catalog.pg_locks bl
        JOIN pg_catalog.pg_stat_activity a
            ON a.pid = bl.pid
        JOIN pg_catalog.pg_locks kl
            ON kl.locktype = bl.locktype AND kl.database = bl.database AND kl.relation = bl.relation
        JOIN pg_catalog.pg_stat_activity ka
            ON ka.pid = kl.pid
    WHERE
        NOT bl.granted AND bl.pid != kl.pid;
    ```

3. **Kill the Blocking Sessions**:
    Use the process ID to terminate the blocking session.

    **Example Script**:
    ```sql
    -- Replace <blocking_pid> with the actual blocking process ID from the above query.
    SELECT pg_terminate_backend(<blocking_pid>);
    ```

### **Automated Script to Handle Deadlocks**

You can create a function to handle deadlocks automatically.

**Example Function**:
```sql
CREATE OR REPLACE FUNCTION handle_deadlocks() RETURNS VOID AS $$
DECLARE
    rec RECORD;
BEGIN
    FOR rec IN
        SELECT
            ka.pid AS blocking_pid
        FROM
            pg_catalog.pg_locks bl
            JOIN pg_catalog.pg_stat_activity a
                ON a.pid = bl.pid
            JOIN pg_catalog.pg_locks kl
                ON kl.locktype = bl.locktype AND kl.database = bl.database AND kl.relation = bl.relation
            JOIN pg_catalog.pg_stat_activity ka
                ON ka.pid = kl.pid
        WHERE
            NOT bl.granted AND bl.pid != kl.pid
    LOOP
        -- Terminate the blocking session
        PERFORM pg_terminate_backend(rec.blocking_pid);
    END LOOP;
END;
$$ LANGUAGE plpgsql;
```

**To execute the function**:
```sql
SELECT handle_deadlocks();
```

---

### **General Recommendations for AWS RDS**

1. **Monitoring and Alerts**:
    - Use AWS CloudWatch to monitor RDS metrics for deadlocks and set up alerts.
    - Monitor `Deadlocks` and `DBLoad` metrics for early detection of potential issues.

2. **Parameter Group Adjustments**:
    - Ensure `innodb_print_all_deadlocks` is set to `ON` for MySQL to log all deadlocks.
    - For PostgreSQL, ensure `log_lock_waits` and `deadlock_timeout` are set to appropriate values in the parameter group.

3. **Optimizing Queries**:
    - Ensure queries are optimized to reduce the time they hold locks.
    - Use indexes and review execution plans to minimize lock contention.

4. **Consistent Locking Order**:
    - Ensure that transactions acquire locks in a consistent order across your application to prevent deadlocks.
