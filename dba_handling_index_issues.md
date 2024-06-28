### Handle index-related issues promptly:

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
   ---

1. **Identify Unused Indexes (based on Query Cache)**
   ```sql
   SELECT DISTINCT t.TABLE_NAME, 
          t.INDEX_NAME, 
          t.INDEX_TYPE 
   FROM information_schema.statistics AS t 
   LEFT JOIN information_schema.query_cache AS q 
          ON t.INDEX_NAME = q.index_name 
   WHERE t.table_schema = 'your_database_name' 
         AND q.index_name IS NULL 
         AND t.INDEX_NAME != 'PRIMARY';
   ```
   This script identifies indexes that are not used by the query cache, indicating potential candidates for removal to optimize performance.

2. **Find Indexes Not Used in a Long Time**
   ```sql
   SELECT s.* 
   FROM information_schema.statistics AS s 
   LEFT JOIN information_schema.tables AS t 
          ON s.table_schema = t.table_schema 
             AND s.table_name = t.table_name 
   WHERE t.update_time < DATE_SUB(NOW(), INTERVAL 3 MONTH) 
         AND s.index_name != 'PRIMARY';
   ```
   This script identifies indexes that have not been used for a specified period, helping to determine if they are obsolete and can be dropped.

3. **Identify Duplicate Indexes**
   ```sql
   SELECT s1.TABLE_NAME, 
          s1.INDEX_NAME, 
          s1.COLUMN_NAME 
   FROM information_schema.statistics AS s1 
   INNER JOIN information_schema.statistics AS s2 
          ON s1.TABLE_SCHEMA = s2.TABLE_SCHEMA 
             AND s1.TABLE_NAME = s2.TABLE_NAME 
             AND s1.INDEX_NAME != s2.INDEX_NAME 
             AND s1.COLUMN_NAME = s2.COLUMN_NAME 
   WHERE s1.TABLE_SCHEMA = 'your_database_name';
   ```
   This script identifies duplicate indexes on the same columns, which can be consolidated or removed to reduce redundancy.

4. **Check Index Fragmentation**
   ```sql
   SELECT table_name, 
          index_name, 
          ROUND(avg_fragmentation_in_percent,2) 
   FROM information_schema.innodb_index_stats 
   WHERE avg_fragmentation_in_percent >= 10 
   ORDER BY avg_fragmentation_in_percent DESC;
   ```
   This script identifies fragmented indexes in InnoDB tables, where fragmentation exceeds a specified threshold, helping to optimize storage and performance.

5. **Review Index Statistics**
   ```sql
   SHOW INDEX STATISTICS;
   ```
   This command provides statistics about indexes, including cardinality and histogram data, helping to analyze index usage patterns and optimize query performance.

6. **Check InnoDB Buffer Pool Efficiency**
   ```sql
   SHOW ENGINE INNODB STATUS;
   ```
---
### Handling index-related issues specifically in an AWS RDS PostgreSQL environment:

1. **Identify Unused Indexes**
   ```sql
   SELECT schemaname, 
          tablename, 
          indexrelname AS index_name, 
          idx_scan 
   FROM pg_stat_user_indexes 
   WHERE idx_scan = 0 
         AND schemaname NOT IN ('pg_catalog', 'information_schema') 
   ORDER BY schemaname, tablename, indexrelname;
   ```
   This script identifies indexes that have never been scanned, indicating potential candidates for removal to optimize performance.

2. **Find Indexes Not Used Recently**
   ```sql
   SELECT schemaname, 
          tablename, 
          indexrelname AS index_name, 
          last_scanned 
   FROM (
       SELECT schemaname, 
              tablename, 
              indexrelname, 
              last_scanned, 
              row_number() OVER (PARTITION BY schemaname, tablename ORDER BY last_scanned DESC) AS rn 
       FROM (
           SELECT schemaname, 
                  tablename, 
                  indexrelname, 
                  last_scanned 
           FROM pg_stat_user_indexes 
           LEFT JOIN pg_index 
                  ON pg_stat_user_indexes.indexrelid = pg_index.indexrelid 
                  AND pg_stat_user_indexes.schemaname = pg_index.schemaname 
                  AND pg_stat_user_indexes.tablename = pg_index.tablename 
       ) AS subquery 
       WHERE idx_scan = 0 
             AND schemaname NOT IN ('pg_catalog', 'information_schema') 
   ) AS ranked 
   WHERE rn = 1 
   ORDER BY schemaname, tablename, indexrelname;
   ```
   This script identifies indexes that have not been used recently, helping to determine if they are obsolete and can be dropped.

3. **Identify Duplicate Indexes**
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
   This script identifies duplicate indexes on the same set of columns, which can be consolidated or removed to reduce redundancy.

4. **Check Index Bloat**
   ```sql
   SELECT schemaname, 
          tablename, 
          indexrelname AS index_name, 
          pg_size_pretty(pg_relation_size(indexrelid)) AS index_size 
   FROM pg_stat_user_indexes 
   WHERE indexrelid IN (
       SELECT indexrelid 
       FROM pg_stat_user_indexes 
       WHERE idx_scan = 0 
   ) 
         AND schemaname NOT IN ('pg_catalog', 'information_schema') 
   ORDER BY pg_relation_size(indexrelid) DESC;
   ```
   This script identifies indexes that have a large physical size relative to their usage, indicating potential bloat issues.

5. **Check Index Fragmentation**
   ```sql
   SELECT schemaname, 
          tablename, 
          indexrelname AS index_name, 
          pg_size_pretty(pg_relation_size(indexrelid)) AS index_size 
   FROM pg_stat_user_indexes 
   WHERE indexrelid IN (
       SELECT indexrelid 
       FROM pg_stat_user_indexes 
       WHERE idx_scan = 0 
   ) 
         AND schemaname NOT IN ('pg_catalog', 'information_schema') 
   ORDER BY pg_relation_size(indexrelid) DESC;
   ```
   This script helps identify fragmented indexes where the index size is significantly larger than the data size, potentially impacting query performance.

6. **Review Index Usage**
   ```sql
   SELECT schemaname, 
          tablename, 
          indexrelname AS index_name, 
          idx_scan, 
          idx_tup_read, 
          idx_tup_fetch 
   FROM pg_stat_user_indexes 
   ORDER BY idx_scan DESC, idx_tup_read DESC, idx_tup_fetch DESC;
   ```
