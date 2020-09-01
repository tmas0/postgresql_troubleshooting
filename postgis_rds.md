# PostGIS on RDS

PostGIS is an extension to PostgreSQL for storing and managing spatial information.

## Connect to the DB instance using the master user name used to create the DB instance

Connect through proxy server or direct connect.
```shell
psql -U<master_user_name> -h instance_uri
```

## Load the PostGIS extensions

```sql
create extension postgis;
create extension fuzzystrmatch;
create extension postgis_tiger_geocoder;
create extension postgis_topology;
```

## Move ownership of the extensions to the rds_superuser role

```sql
alter schema tiger owner to rds_superuser;
alter schema tiger_data owner to rds_superuser;
alter schema topology owner to rds_superuser;
```

## Move ownership of the objects to the rds_superuser role

First, create a auxiliary function to help to move all objects of tiger and topology schemas.

```sql
CREATE FUNCTION exec(text) returns text language plpgsql volatile AS $f$ BEGIN EXECUTE $1; RETURN $1; END; $f$;
```

Next, run the following query to move all objects of tiger and topology schemas.
```sql
SELECT exec('ALTER TABLE ' || quote_ident(s.nspname) || '.' || quote_ident(s.relname) || ' OWNER TO rds_superuser;')
  FROM (
    SELECT nspname, relname
    FROM pg_class c JOIN pg_namespace n ON (c.relnamespace = n.oid) 
    WHERE nspname in ('tiger','topology') AND
    relkind IN ('r','S','v') ORDER BY relkind = 'S')
s;
```


## Test the extensions

Add tiger to your search path using the following command.
```sql
SET search_path=public,tiger;
```

Test tiger by using the following SELECT statement.
```sql
select na.address, na.streetname, na.streettypeabbrev, na.zip
from normalize_address('1 Devonshire Place, Boston, MA 02109') as na;
```

Finally, create and delete new custom topology:
```sql
select topology.createtopology('my_new_topo',26986,0.5);

select topology.droptopology('my_new_topo');
```


## Extras

Query to extract all database schemas:

```sql
select
schema_name,
schema_owner
from information_schema.schemata
where schema_name not like 'pg_%' and schema_name != 'information_schema';
```

## Sources

* https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.PostgreSQL.CommonDBATasks.html#Appendix.PostgreSQL.CommonDBATasks.PostGIS