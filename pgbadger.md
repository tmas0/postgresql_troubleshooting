# PgBadger

## Prerequisites

```shell
sudo apt-get install libjson-xs-perl
or
sudo yum install perl-JSON-XS
```

## Installation

### Manual

* Download from [PgBadger](https://github.com/darold/pgbadger/releases)
* Install pgBadger
```shell
tar xzf pgbadger-11.x.tar.gz
cd pgbadger-11.x/
perl Makefile.PL
make && sudo make install
```

### Linux distro package.

If you are using Linux distributions same as Ubuntu, Debion, CentOS, RedHat, etc. The best way to install pgBadger is:
```shell
sudo apt-get install pgbadger
```

## Configuration

Make sure that the following parameter are set correctly.

* [log_line_prefix](performance_tuning_postgresql_conf.md#log_line_prefix)
* [log_checkpoints](performance_tuning_postgresql_conf.md#log_checkpoints)
* [log_connections](performance_tuning_postgresql_conf.md#log_connections)
* [log_disconnections](performance_tuning_postgresql_conf.md#log_disconnections)
* [log_lock_waits](performance_tuning_postgresql_conf.md#log_lock_waits)
* [log_temp_files](performance_tuning_postgresql_conf.md#log_temp_files)
* [log_autovacuum_min_duration](performance_tuning_postgresql_conf.md#log_autovacuum_min_duration)
* [log_min_duration_statement](performance_tuning_postgresql_conf.md#log_min_duration_statement)

## Run analyzer

You can run same as:
```shell
pgbadger /var/log/postgresql.log
pgbadger /var/log/postgres.log.2.gz /var/log/postgres.log.1.gz /var/log/postgres.log
pgbadger /var/log/postgresql/postgresql-2012-05-*
```


## The report explanation


### Global stats

* Queries: The number of queries found, duration, number of normalized queries, etc.
* Events: Total number of events, normalized events, etc.
* Vacuums: The total number of auto vacuums and auto analyzes found.
* Temporary Files: The total number of temporary files found, max size, and average size.
* Session: Total number of sessions, peak sessions time, total duration of sessions, average duration of sessions, average queries found per session, average queries duration per session.
* Connections: Total number of connections, peak connections, and total number of databases connected to.

### Connections

* Established connections: Showing maximum, minimum, and average number of connections over time.
* Connections per database: A pie chart and table view showing the number of connections for each database found.
* Connections per user: A pie chart and table view showing the number of connections for each user found.
* Connections per host: A pie chart and table view showing the number of connections for each source host found.

### Sessions

* Simultaneous Sessions: A line chart showing the number of sessions over time.
* Histogram of Session Times: A bar chart and table showing session times.
* Sessions per database: A pie chart and table view showing the number of sessions for each database found.
* Sessions per user: A pie chart and table view showing the number of sessions for each user found.
* Sessions per host: A pie chart and table view showing the number of sessions for each source host found.
* Sessions per Application: The number of sessions connected per application.

### Checkpoints

* Checkpoint Buffers: A line chart showing the amount of buffers written by the checkpoint process over time.
* Checkpoints WAL Files: A line chart showing the number of WAL files added, removed, or recycled by the checkpointer over time.
* Checkpoint Distance: A line chart showing the distance and estimate for checkpoints.
* Checkpoints Activity: A table showing the previous four data points in table form.

### Temporally files

* Size of temporary files: A line chart showing the space used by temporary files over time.
* Number of temporary files: A line chart showing the number of temporary files used over time.
* Temporary Files Activity: A table showing the information provided in the previous charts but in table form.

### Vacuums

* Vacuums / Analyzes Distribution: A line chart showing VACUUMs and ANALYZEs over time, as well as information on the table that consumed the most CPU processing power.
* Analyzes per table: A pie chart and table showing the tables with the most analyzes, suggesting these tables are in a high state of change.
* Vacuums per table: A pie chart and table showing the tables with the most vacuums, suggesting these tables are in a high state of change.
* Tuples removed per table: A pie chart and table showing the number of tuples and pages removed in vacuum processes for tables.
* Pages removed per table: A pie chart and table showing the number of pages and tuples removed in vacuum processes for tables.
* Autovacuum activity: A table showing the VACUUMs and ANALYZEs over time per hour.

### Locks

* Locks by types
* Most frequent waiting queries: A list of queries found to be waiting, ranked most frequent to least.
* Queries that waited the most: A list of queries and how long they waited, ordered from longest to shortest.

### Queries

* Queries by type: A pie chart and table showing the number of different types of queries, such as INSERT, UPDATE, DELETE, SELECT, etc.
* Queries by database: A pie chart and table showing the number of queries found per database.
* Queries by application: A pie chart and table showing the number of queries found per application.
* Number of cancelled queries: Information on any queries that were canceled.

### Top

The most interesting tab.

* Histogram of query times: A histogram representing how many queries fall into each grouping of timings.
* Slowest individual queries: A list of the slowest queries found, ordered by longest to shortest.
* Time consuming queries: A list of normalized queries and their total duration, ordered by greatest time consumed to least.
* Most frequent queries: A list of normalized queries and how many times they were executed, ordered from most to least.
* Normalized slowest queries: A list of normalized queries and their average duration, ordered by longest to shortest.

### Events

* Log levels: The different levels of logs appeared by line, such as CONTEXT, LOG, STATEMENT, HINTs, WARNINGs, and others.
* Events distribution: A line graph of events over time for PANIC, FATAL, ERROR, and WARNING events.
* Most Frequent Errors/Events: A list of EVENTS and their frequency, ordered from most common to least.

# Sources

* https://github.com/darold/pgbadger
* https://severalnines.com/database-blog/postgresql-log-analysis-pgbadger