### Tutorial: Restoring a MySQL Database from Dump Files Using MySQL Shell

#### Overview
This tutorial provides a step-by-step guide for restoring a MySQL database from a set of dump files using MySQL Shell. This approach includes handling foreign key constraints to ensure a smooth import process.

#### Prerequisites
- **MySQL Shell**: Ensure MySQL Shell is installed on your system. If not, download and install it from the [official MySQL website](https://dev.mysql.com/downloads/shell/).
- **Database Access**: Confirm you have access to the MySQL instance where you want to restore the database.

---

### Step 1: Prepare Your Environment

1. **Extract Files**
   - Download the `airport-db.zip` file.
   - Extract its contents to a local directory (e.g., `C:/Users/ageu/Documents/DBA SQL Optimizer/airport-db/airport-db`).

2. **Verify Directory Structure**
   - Ensure the directory contains all necessary files, such as `airportdb.sql`, `airportdb@booking@0.tsv`, and other related files.

---

### Step 2: Open MySQL Shell

1. **Launch MySQL Shell**
   - Open your command prompt or terminal.
   - Type `mysqlsh` and press Enter to launch MySQL Shell.

---

### Step 3: Connect to the MySQL Instance

1. **Connect Using MySQL Shell**
   - In MySQL Shell, connect to your MySQL instance using the following command:
     ```bash
     \connect root@localhost
     ```
     Replace `root` with your MySQL username if different and adjust `localhost` if needed.

---

### Step 4: Disable Foreign Key Checks

1. **Run Command**
   - To prevent issues with table dependencies during the import, execute the following SQL command:
     ```sql
     \sql
     SET foreign_key_checks = 0;
     ```

---

### Step 5: Load the Dump Files

1. **Execute the `util.loadDump` Command**
   - Use MySQL Shell's `util.loadDump` function to import the database dump files:
     ```javascript
     util.loadDump("C:/Users/ageu/Documents/DBA SQL Optimizer/airport-db/airport-db", {
         threads: 16,
         deferTableIndexes: "all",
         ignoreVersion: true
     });
     ```
   - This command loads all files from the specified directory into the MySQL instance using 16 threads, defers index creation, and ignores version checks.

---

### Step 6: Re-enable Foreign Key Checks

1. **Run Command**
   - After the data import is complete, re-enable foreign key checks with:
     ```sql
     SET foreign_key_checks = 1;
     ```

---

### Step 7: Verify the Restore

1. **Check Database**
   - Confirm the database restoration by listing tables and checking their contents:
     ```sql
     SHOW TABLES;
     ```

2. **Run Sample Queries**
   - Execute a few sample SQL queries to ensure data is accessible and correctly restored.

---

### Step 8: Handle Errors (If Any)

1. **Review Error Logs**
   - If errors occur, check MySQL Shell and server error logs for details.

2. **Verify File Paths**
   - Ensure all file paths in commands are correct and accessible.

3. **Check File Permissions**
   - Verify that MySQL Shell has the necessary permissions to access and read the files.

---

### Step 9: Optimize and Review

1. **Rebuild Indexes**
   - If indexes were deferred during import, consider rebuilding them.

2. **Update Statistics**
   - Update database statistics for optimal performance:
     ```sql
     ANALYZE TABLE table_name;
     ```
     Replace `table_name` with the name of the tables you want to analyze.
