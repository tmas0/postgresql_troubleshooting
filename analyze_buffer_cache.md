# Analyze the use of buffer cache

You need install the pg_buffercache extension. For install, you should execute the following query:

```sql
CREATE EXTENSION pg_buffercache;
```

Analyze your buffer cache utilization via:
```sql
select pg_size_pretty(count(*) * 8192) as buffers,
isdirty,
usagecount
from pg_class c
inner join pg_buffercache b on b.relfilenode = c.relfilenode
inner join pg_database d on (b.reldatabase = d.oid and d.datname =current_database())
group by usagecount,isdirty
order by usagecount,isdirty;
```

To get all relations buffered in the database buffer cache:
```sql
select c.relname,pg_size_pretty(count(*) * 8192) as buffered, 
        round(100.0 * count(*) / ( 
           select setting from pg_settings 
           where name='shared_buffers')::integer,1)
        as buffer_percent, 
        round(100.0*count(*)*8192 / pg_table_size(c.oid),1) as percent_of_relation
from pg_class c inner join pg_buffercache b on b.relfilenode = c.relfilenode inner 
join pg_database d on ( b.reldatabase =d.oid and d.datname =current_database()) 
group by c.oid,c.relname order by 3 desc limit 10;
```

To get top of big relation buffered in database buffer cache:
```sql
select c.relname,count(*) as buffers 
from pg_class c 
     inner join pg_buffercache b on b.relfilenode=c.relfilenode 
     inner join pg_database d on (b.reldatabase=d.oid and d.datname=current_database()) 
group by c.relname order by 2 desc limit 20;
```