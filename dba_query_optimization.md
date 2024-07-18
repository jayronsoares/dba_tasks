### Checking Cardinality in MySQL

To check the cardinality of columns in a table, you can analyze the uniqueness and distribution of values within those columns. High cardinality indicates a large number of unique values, while low cardinality indicates a small number of unique values. 

#### 1. High Cardinality Check

To identify columns with high cardinality, you can compare the number of distinct values in each column to the total number of rows in the table. Columns with a high ratio of distinct values to total rows have high cardinality.

```sql
SELECT 
    COLUMN_NAME,
    COUNT(DISTINCT COLUMN_NAME) AS distinct_values,
    COUNT(*) AS total_rows,
    ROUND(COUNT(DISTINCT COLUMN_NAME) / COUNT(*) * 100, 2) AS cardinality_percentage
FROM 
    your_table_name
GROUP BY 
    COLUMN_NAME
ORDER BY 
    cardinality_percentage DESC;
```

#### 2. Low Cardinality Check

To identify columns with low cardinality, you can use a similar query but look for columns with a low ratio of distinct values to total rows.

```sql
SELECT 
    COLUMN_NAME,
    COUNT(DISTINCT COLUMN_NAME) AS distinct_values,
    COUNT(*) AS total_rows,
    ROUND(COUNT(DISTINCT COLUMN_NAME) / COUNT(*) * 100, 2) AS cardinality_percentage
FROM 
    your_table_name
GROUP BY 
    COLUMN_NAME
ORDER BY 
    cardinality_percentage ASC;
```

### Comprehensive Cardinality Check Script

Here’s a script that you can run to check both high and low cardinality for all columns in a table:

```sql
-- Create a temporary table to store the results
CREATE TEMPORARY TABLE column_cardinality (
    column_name VARCHAR(255),
    distinct_values BIGINT,
    total_rows BIGINT,
    cardinality_percentage DECIMAL(10, 2)
);

-- Loop through each column in the table and calculate cardinality
SET @table_name = 'your_table_name';
SET @schema_name = 'your_schema_name';

-- Generate dynamic SQL to calculate cardinality for each column
SELECT 
    CONCAT(
        'INSERT INTO column_cardinality (column_name, distinct_values, total_rows, cardinality_percentage) ',
        'SELECT ''', COLUMN_NAME, ''' AS column_name, ',
        'COUNT(DISTINCT ', COLUMN_NAME, ') AS distinct_values, ',
        'COUNT(*) AS total_rows, ',
        'ROUND(COUNT(DISTINCT ', COLUMN_NAME, ') / COUNT(*) * 100, 2) AS cardinality_percentage ',
        'FROM ', @schema_name, '.', @table_name, ';'
    )
INTO @sql
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = @schema_name
AND TABLE_NAME = @table_name;

-- Execute the generated SQL
PREPARE stmt FROM @sql;
EXECUTE stmt;
DEALLOCATE PREPARE stmt;

-- Query the results
SELECT * FROM column_cardinality ORDER BY cardinality_percentage DESC;

-- Drop the temporary table
DROP TEMPORARY TABLE column_cardinality;
```

### Explanation:

1. **Temporary Table Creation**: A temporary table `column_cardinality` is created to store the cardinality results.
2. **Dynamic SQL Generation**: A loop goes through each column in the specified table and generates dynamic SQL to calculate the cardinality.
3. **Execution**: The generated SQL is executed to insert the cardinality results into the temporary table.
4. **Query Results**: The results are queried from the temporary table, sorted by `cardinality_percentage`.
5. **Cleanup**: The temporary table is dropped after use.

### Usage:

1. Replace `'your_table_name'` and `'your_schema_name'` with the actual table and schema names.
2. Execute the script in your MySQL client to analyze the cardinality of all columns in the specified table.

-----------
### Query to Check Table Size, Number of Rows, and I/O Block Size
SELECT 
    TABLE_SCHEMA,
    TABLE_NAME,
    TABLE_ROWS,
    DATA_LENGTH + INDEX_LENGTH AS total_size,
    DATA_LENGTH AS data_size,
    INDEX_LENGTH AS index_size,
    AVG_ROW_LENGTH,
    ROW_FORMAT,
    ENGINE,
    TABLE_COLLATION,
    ifnull(ROW_FORMAT, 'DEFAULT') AS row_format,
    case when ENGINE = 'InnoDB' then @@innodb_page_size
         else null end as io_block_size
FROM 
    INFORMATION_SCHEMA.TABLES
WHERE 
    TABLE_SCHEMA = 'your_database_name'
    AND TABLE_TYPE = 'BASE TABLE'
ORDER BY 
    total_size DESC;


-----------------

### 1. The best join combination for joining the tables is found by trying all possibilities. If all columns in ORDER BY and GROUP BY clauses come from the same table, that table is preferred first when joining.

#### Example:
If you have two tables, `orders` and `customers`, and you want to join them with an `ORDER BY` clause on a column from `orders`, the optimizer will prefer to join `orders` first.

```sql
SELECT o.order_id, o.order_date, c.customer_name
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
ORDER BY o.order_date;
```

Here, since the `ORDER BY` clause refers to a column from the `orders` table, `orders` is preferred first in the join.

### 2. If there is an ORDER BY clause and a different GROUP BY clause, or if the ORDER BY or GROUP BY contains columns from tables other than the first table in the join queue, a temporary table is created.

#### Example:
If you want to group by a column from `customers` but order by a column from `orders`, MySQL may create a temporary table.

```sql
SELECT c.customer_name, COUNT(o.order_id) AS total_orders
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_name
ORDER BY o.order_date;
```

In this case, the `GROUP BY` is on `customer_name` from `customers`, and `ORDER BY` is on `order_date` from `orders`, which can lead to the creation of a temporary table.

### 3. If you use the SQL_SMALL_RESULT modifier, MySQL uses an in-memory temporary table.

#### Example:
Using the `SQL_SMALL_RESULT` modifier in a query with a `GROUP BY` clause to hint that the result set will be small.

```sql
SELECT SQL_SMALL_RESULT c.customer_name, COUNT(o.order_id) AS total_orders
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_name;
```

Here, `SQL_SMALL_RESULT` tells MySQL that the result set is expected to be small, prompting it to use an in-memory temporary table.

### 4. Each table index is queried, and the best index is used unless the optimizer believes that it is more efficient to use a table scan.

#### Example:
Querying a table with an indexed column versus a table scan.

```sql
-- Assume `order_date` is indexed in the `orders` table
SELECT order_id, order_date
FROM orders
WHERE order_date = '2024-01-01';
```

In this case, MySQL will likely use the index on `order_date`. However, if the condition matched a significant portion of the table, the optimizer might decide to do a full table scan.

### 5. In some cases, MySQL can read rows from the index without even consulting the data file.

#### Example:
Selecting only indexed numeric columns.

```sql
-- Assume `order_id` and `order_date` are indexed numeric columns
SELECT order_id, order_date
FROM orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31';
```

If both `order_id` and `order_date` are indexed, and the query only involves these columns, MySQL can resolve the query using just the index tree.

### 6. Before each row is output, those that do not match the HAVING clause are skipped.

#### Example:
Using a `HAVING` clause to filter aggregated results.

```sql
SELECT c.customer_name, COUNT(o.order_id) AS total_orders
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_name
HAVING total_orders > 5;
```

Here, the `HAVING` clause ensures that only customers with more than 5 orders are included in the final result set, skipping those that don't match this condition before output.
