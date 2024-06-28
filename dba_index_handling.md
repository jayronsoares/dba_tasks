Here are advanced SQL scripts for MySQL and PostgreSQL that you can use as a DBA to handle index-related issues promptly:

### For MySQL

1. **Identify Unused Indexes**
   ```sql
   SELECT * 
   FROM information_schema.statistics 
   WHERE table_schema = 'your_database_name' 
     AND INDEX_NAME NOT IN (
       SELECT DISTINCT index_name 
       FROM information_schema.query_cache 
       WHERE index_name IS NOT NULL 
         AND index_name != 'PRIMARY'
     ) 
   ORDER BY INDEX_SCHEMA, INDEX_NAME;
   ```
   This script identifies indexes that are not used by any queries, allowing you to consider dropping them to reduce storage overhead and improve insert/update performance.

2. **Find Duplicate Indexes**
   ```sql
   SELECT DISTINCT a.TABLE_NAME, 
          a.INDEX_NAME 
   FROM information_schema.statistics a 
   JOIN information_schema.statistics b 
     ON a.INDEX_NAME = b.INDEX_NAME 
        AND a.TABLE_NAME = b.TABLE_NAME 
        AND a.TABLE_SCHEMA = b.TABLE_SCHEMA 
        AND a.COLUMN_NAME != b.COLUMN_NAME 
   WHERE a.TABLE_SCHEMA = 'your_database_name';
   ```
   This script helps identify duplicate indexes on the same columns, which can waste storage and slow down write operations.

3. **Check Index Fragmentation**
   ```sql
   SELECT TABLE_NAME, 
          INDEX_NAME, 
          INDEX_TYPE, 
          ROUND((INDEX_LENGTH / DATA_LENGTH) * 100, 2) AS 'Fragmentation %' 
   FROM information_schema.tables 
   WHERE table_schema = 'your_database_name' 
     AND INDEX_TYPE = 'BTREE' 
     AND DATA_LENGTH > 0 
   ORDER BY `Fragmentation %` DESC;
   ```
   It identifies fragmented indexes, where the index size is significantly larger than the data size, potentially impacting query performance.

### For PostgreSQL

1. **Identify Unused Indexes**
   ```sql
   SELECT schemaname, 
          tablename, 
          indexname, 
          idx_scan, 
          idx_tup_read, 
          idx_tup_fetch 
   FROM pg_stat_user_indexes 
   WHERE idx_scan = 0 
   ORDER BY tablename, indexname;
   ```
   This script identifies indexes that have never been scanned, indicating potential candidates for removal to reduce overhead.

2. **Find Duplicate Indexes**
   ```sql
   SELECT conrelid::regclass AS table_name, 
          indexrelid::regclass AS index_name, 
          ARRAY_AGG(attname) AS column_names 
   FROM pg_index 
   JOIN pg_attribute ON pg_attribute.attrelid = pg_index.indrelid 
                     AND pg_attribute.attnum = ANY(pg_index.indkey) 
   GROUP BY conrelid, indexrelid 
   HAVING COUNT(*) > 1;
   ```
   This script identifies duplicate indexes on the same set of columns, which can be consolidated or dropped to improve performance.

3. **Check Index Bloat**
   ```sql
   SELECT schemaname, 
          tablename, 
          indexname, 
          pg_relation_size(indexrelid) AS index_size, 
          pg_size_pretty(pg_relation_size(indexrelid)) AS index_size_pretty, 
          idx_scan 
   FROM pg_stat_user_indexes 
   WHERE indexrelid IN (
       SELECT indexrelid 
       FROM pg_stat_user_indexes 
       WHERE idx_scan = 0 
   ) 
   ORDER BY pg_relation_size(indexrelid) DESC;
   ```
   It identifies indexes that have a large physical size relative to their usage, indicating potential bloat issues.
