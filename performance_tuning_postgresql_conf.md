# PostgreSQL configuration parameters

The following list should be reviewed when configuring a PostgreSQL instance.

## max_connections

Number of maximum accepted connections.

When you don't known, a valid default value is:
```
max_connections = 100
```

## shared_buffers

What does this parameter mean?
* Amount of RAM that can be allocated to shared buffers.

How to set it correctly?
* Leave more RAM for caching.
* Sometimes 25% of Total RAM or an 8GB (magic number)
* Sometimes 90% of Total RAM on a dedicated servers.
```
shared_buffers = 25%/90% of Total RAM or an 8GB.
```

How to check that parameter set correctly?
* Using [pg_buffercache](https://www.postgresql.org/docs/current/pgbuffercache.html) extension.
* Check query blocks using pg_stat_statements
* Views: pg_statio_user_tables and pg_statio_user_indexs.
* EXPLAIN (ANALYZE,BUFFERS) query.

## effective_cache_size

* This parameter is only used for estimation purposes by the Planner.
* Assumption on available cache for each query.
* Lower value prefers sequence scans over index scans.

How to set it correctly?
* Generally 50% of RAM.
* When faster disks and indexes can always be cached: 70% of RAM.
* *Does not reserve any memory but used ONLY for estimates*.

When you don't known, a valid default value is:
```
effective_cache_size = 50% or 70% of RAM.
```

## maintenance_work_mem

* Allocated memory used by autovacuum worker process when autovacuum_work mem is left at -1 (default).
* Used on:
** CREATE INDEX
** ALTER TABLE ADD FOREIGN KEY
*** All parallel CREATE INDEX processes will only use up to maintenance_work_mem in total.

How to set it correctly?
* Number of concurrent maintenance operations during peak * maintenance_work_mem <<< Total of RAM - (shared_buffers + (work_mem * max_concurrent_connections))

When you don't known, a valid default value is:
```
maintenance_work_mem = 128MB
```

## checkpoint_completion_target

* Defines the checkpoint across amount of time.
* Default is 0.5.
* If the checkpoint_timeout is 30min, the checkpoint writes are spread across 15min (30*0.5)

How to set it correctly?
* 0.8/0.9 may be helpful.

When you don't known, a valid default value is:
```
checkpoint_completion_target = 0.9
```

## wal_buffers

* The amount of shared memory used for WAL data that has not yet been written to disk.
* Default value is -1, selects a size equal to 1/32nd (about 3%) of shared_buffers, but not less than 64kB.

When you don't known, a valid default value is:
```
wal_buffers = 16MB
```

## default_statistics_target

* Sets the default statistics target for table columns without a column-specific target.
* Larger values increase the time needed to do ANALYZE.
* If you want obtain better quality of the planner estimates, set a value larger than 100.

When you don't known, a valid default value is:
```
default_statistics_target = 100
```

## random_page_cost

* Set the cost of non-sequentially fetched disk page.
* Affects the query planner's decisions which is cost-based.
* Default is 4.

How to set it correctly?
* SSD disks use the default value.
* Others type of disks, set a 1.1

When you don't known, a valid default value is:
```
random_page_cost = 1.1
```

## effective_io_concurrency

* Sets the number of concurrent disk I/O operations that PostgreSQL expects can be executed simultaneously.
* Currently only affects to bitmap heap scans.
* Default value is 1.

How to set it correctly?
* Now, you can use the default value.
* Check if you use intensive bitmap heap scans, if true, set more than 100.

When you don't known, a valid default value is:
```
effective_io_concurrency = 1
```

## work_mem

* Memory allocated for each sort operation such as ORDER BY or DISTINCT, and merge joins
* Allocated for hash tables used in hash joins or hash-based processing of IN queries.
* Default to 4MB.
* Complex queries may require more parallel sorts.

How to set it correctly?
* Huge value globally, allocates each so much RAM.
* Use EXPLAIN(ANALYZE,BUFFERS) query.
* To avoid OOM killer, use the following formula:
** Number of active concurrent connections during peak * average sort operations per each connection * work_mem <<< Total of RAM - (shared_buffers + (maintenance_work_mem * autovacuum_max_workers))

When you don't known, a valid default value is:
```
work_mem = Number of active concurrent connections during peak * average sort operations per each connection * work_mem <<< Total of RAM - (shared_buffers + (maintenance_work_mem * autovacuum_max_workers))
```

## min_wal_size

* This can be used to ensure that enough WAL space is reserved to handle spikes in WAL usage.
* The default value is 80MB, lower value.

How to set it correctly?
* min_wal_size < max_wal_size.
* 256MB is a magic value.

When you don't known, a valid default value is:
```
min_wal_size = 256MB
```

## max_wal_size

* Issue checkpoint when checkpoint_timeout or max_wal_size is about to reach, whichever comes first.

How to set it correctly?
* Must have enough space on filesystem.
* Frequent checkpoint indicates less size. Then increase it.

When you don't known, a valid default value is:
```
max_wal_size = 2GB
```

## max_worker_processes

* Sets the maximum number of background processes that the system can support.
* The default is 8.
* When running a standby server, you must set this parameter to the same or higher value than on the master server. Otherwise, queries will not be allowed in the standby server.

How to set it correctly?
* On a dedicated server, set it as a same number of available CPUs.

When you don't known, a valid default value is:
```
max_worker_processes = 8
```

## max_parallel_workers_per_gather

* Sets the maximum number of workers that can be started by a single Gather or Gather Merge node.
* Parallel workers are taken from the pool of processes established by max_worker_processes, limited by max_parallel_workers.
* Note that the requested number of workers may not actually be available at run time. If this occurs, the plan will run with fewer workers than expected, which may be inefficient.
* The default value is 2.
* Setting this value to 0 disables parallel query execution.

How to set it correctly?
* If you only have a one CPU, disable it feature.

When you don't known, a valid default value is:
```
max_parallel_workers_per_gather = 2
```

## max_parallel_workers

* Sets the maximum number of workers that the system can support for parallel operations.
* The default value is 8. 

How to set it correctly?
* On a dedicated server, same as number of CPUs.
* If you only have a one CPU, disable parallel query execution.

When you don't known, a valid default value is:
```
max_parallel_workers = 8
```

## autovacuum

* Starts an autovacuum worker process on table then thresholds are reached.
* Defaults to ON. Not disable.

How to set it correctly?
* Make sure it is enabled. Never set this to OFF.

Always, the unique and valid value is:
```
autovacuum = on
```

## checkpoint_warning

* Write a message to the server log if checkpoints caused by the filling of WAL segment files happen closer together than this amount of time (which suggests that max_wal_size ought to be raised).
* The default is 30 seconds (30s).
* Zero disables the warning. No warnings will be generated if checkpoint_timeout is less than checkpoint_warning.

How to set it correctly?
* Enable and set to 30s.

When you don't known, a valid default value is:
```
checkpoint_warning = 30s
```

## checkpoint_timeout

* Force a checkpoint every checkpoint_timeout amount of time.
* Default to 5min.

How to set correctly?
* 15/30/60 minutes may be better to start with.
* Higher checkpoint_timeout may increase the crash recovery time.

When you don't known, a valid default value is:
```
checkpoint_timeout = 5min
```

## autovacuum_vacuum_threshold

* Minimum number of obsolete records or updates/deletes needed to trigger an autovacuum vacuum on a table.
* The default is 50 tuples.

When you don't known, a valid default value is:
```
autovacuum_vacuum_threshold = 50
```

## autovacuum_vacuum_scale_factor

* Fraction of the table records that autovacuum needs start a vacuum of table.
* 0.2 equals to 20% of the table records.

When you don't known, a valid default value is:
```
autovacuum_vacuum_scale_factor = 0.2
```

## autovacuum_analyze_threshold

* Minimum number of obsolete records or dml's needed to trigger an autovacuum analyze on a table.

How to set it correctly?
* Check number of delete queries, and check number of total records per table.

When you don't known, a valid default value is:
```
autovacuum_analyze_threshold = 50
```

## autovacuum_analyze_scale_factor

* Fraction of the table records that will be checked.

How to set it correctly?
* Check number of total tuples for each table.

When you don't known, a valid default value is:
```
autovacuum_analyze_scale_factor = 0.1
```

## autovacuum_max_workers

* Number of autovacuum workers that could run in a PostgreSQL instance concurrently.
* Defaults to 3.

How to set it correctly?
* Check for a number of databases.

When you don't known, a valid default value is:
```
autovacuum_max_workers = 3
```

## vacuum_cost_limit

* Total cost limit vacuum could reach.

When you don't known, a valid default value is:
```
vacuum_cost_limit = 200
```

## wal_compression

* Compress a full page image written to a WAL.
* Default to off.

How to set it correctly?
* Reduce IO and increase the CPU consumption.
* Reduce WAL files size, fit storage usage.

When you don't known, a valid default value is:
```
wal_compression = on
```

## log_line_prefix

* Set the parameters of log lines.

How to set it correctly?
* Must set this parameter to [PgBadger](https://pgbadger.darold.net/) requirements.

The unique format for this is:
```
log_line_prefix = '%t [%p]: [%l-l] %quser=%u,db=%d,app=%a,client=%h '
```

## log_checkpoints

* Needed to set on to be checked with pgBadger.

A valid value is:
```
log_checkpoints = on
```

## log_connections

When you don't known, a valid default value is:
log_connections = on

## log_disconnections

* Needed to set on to be checked with pgBadger.

A valid value is:
```
log_disconnections = on
```

## log_lock_waits

* Needed to set on to be checked with pgBadger.

A valid value is:
```
log_lock_waits = on
```

## log_temp_files

* Needed to set on to be checked with pgBadger.

A valid value is:
```
log_temp_files = 0
```

## log_autovacuum_min_duration

* Needed to set on to be checked with pgBadger.

A valid value is:
```
log_autovacuum_min_duration = 0
```

## log_min_duration_statement

* Causes the duration of each completed statement to be logged if the statement ran for at least the specified amount of time.
* If this value is specified without units, it is taken as milliseconds.

A valid value is:
```
log_min_duration_statement = 30
```
If you want log all queries, set it to 0.

## log_rotation_age

* postgresql.log rotation age.
* One day is a appropiated value.

A valid value is:
```
log_rotation_age = 1d
```

# Sources
* https://www.percona.com/live/e18/sites/default/files/slides/High%20Performance%20PostgreSQL,%20Tuning%20and%20Optimization%20Guide%20-%20FileId%20-%20160682.pdf
* https://www.percona.com/resources/videos/mostly-mistaken-and-ignored-parameters-while-optimizing-postgresql-database-percona
* https://postgresql.org