# How to determine client connected through unix socket?

## Prerequisites

1. Determine the status of connection/s.
   * Current overall state of this backend. Possible values are:
      - **active**: The backend is executing a query.
      - **idle**: The backend is waiting for a new client command.
      - **idle in transaction**: The backend is in a transaction, but is not currently executing a query.
      - **idle in transaction (aborted)**: This state is similar to idle in transaction, except one of the statements in the transaction caused an error.
      - **fastpath function call**: The backend is executing a fast-path function.
      - **disabled**: This state is reported if track_activities is disabled in this backend.

## Steps to find the client connected

1. Login into psql client:
```shell
psql -Upostgres -p
```
2. Search your target session. For example, if you want to find a session that is not idle, you will execute the following query:
```
postgres=# SELECT * from pg_stat_activity where state <> 'idle';
-[ RECORD 1 ]----+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
datid            | 986587
datname          | your_database
pid              | 248243
usesysid         | 16384
usename          | your_username
application_name |
client_addr      |
client_hostname  |
client_port      | -1
backend_start    | 2020-08-18 12:25:11.613585+02
xact_start       | 2020-08-18 20:25:00.897228+02
query_start      | 2020-08-24 15:35:51.000282+02
state_change     | 2020-08-24 15:35:51.000284+02
wait_event_type  |
wait_event       |
state            | active
backend_xid      | 5790454
backend_xmin     | 5790454
query            | INSERT INTO "your_table" ("id", "create_date", "write_date", "age") VALUES (nextval('your_table_sequence'), (now() at time zone 'UTC'), (now() at time zone 'UTC')) RETURNING id
backend_type     | client backend
```
   * **client_addr**: IP address of the client connected to this backend. If this field is null, it indicates either that the client is connected via a Unix socket on the server machine or that this is an internal process such as autovacuum.
      * Is empty, it indicates that the client is connected via a Unix socket.
   * **client_port**: CP port number that the client is using for communication with this backend, or -1 if a Unix socket is used
      * -1, it indicates that a Unix socket is used.
3. Determine the inode that used to connect to the database. Replace the 248432 with the pid found above. Execute the following command:
```shell
user@host# sudo lsof |grep PGSQL |grep 2482432
postgres   248243                              postgres    8u     unix 0xffff92fcfdd9dc00         0t0    4828584 /var/run/postgresql/.s.PGSQL.5432 type=STREAM
```
4. Determine the client using the following command, replace 4828584 with the inode number found above:
```shell
user@host# sudo ss -xp |grep 4828584
u_str              ESTAB               0                    0                                                                                    * 4837080                                               * 4828584                               users:(("./your_client-",pid=247966,fd=4))
u_str              ESTAB               0                    0                                                    /var/run/postgresql/.s.PGSQL.5432 4828584                                               * 4837080                               users:(("postgres",pid=248243,fd=8))
```
