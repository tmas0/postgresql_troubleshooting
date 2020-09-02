# Get all stats of vacuum or autovacuum process

```sql
select relname,last_vacuum, last_autovacuum, last_analyze, last_autoanalyze
from pg_stat_user_tables;
```