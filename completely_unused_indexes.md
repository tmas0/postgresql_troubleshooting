# Get all unused big indexes

```sql
SELECT relid::regclass as table, indexrelid::regclass as index
     , pg_size_pretty(pg_relation_size(indexrelid))
  FROM pg_stat_user_indexes
  JOIN pg_index
 USING (indexrelid)
 WHERE idx_scan = 0
   AND indisunique IS FALSE
 ORDER BY pg_relation_size(indexrelid);
```