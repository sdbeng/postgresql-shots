##PSQL
starting up:

psql -U postgres
Some interesting flags (to see all, use -h):

-E: will describe the underlaying queries of the \ commands (cool for learning!)
-l: psql will list all databases and then exit (useful if the user you connect with doesn't has a default database, like at AWS RDS)
Most \d commands support additional param of **schema**.name\_\_ and accept wildcards like _._

\q: Quit/Exit
\c **database**: Connect to a database
\d **table**: Show table definition including triggers
\l: List databases
\dy: List events
\df: List functions
\di: List indexes
\dn: List schemas
\dt _._: List tables from all schemas (if _._ is omitted will only show SEARCH_PATH ones)
\dv: List views
\df+ **function** : Show function SQL code.
\x: Pretty-format query results instead of the not-so-useful ASCII tables
\copy (SELECT \* FROM **table_name**) TO 'file_path_and_name.csv' WITH CSV: Export a table as CSV
User Related:

\du: List users
\du **username**: List a username if present.
create role **test1**: Create a role with an existing username.
create role **test2** noinherit login password **passsword**;: Create a role with username and password.
set role **test**;: Change role for current session to **test**.
grant **test2** to **test1**;: Allow **test1** to set its role as **test2**.
Configuration
Service management commands:
sudo service postgresql stop
sudo service postgresql start
sudo service postgresql restart
Changing verbosity & querying Postgres log:

1.  First edit the config file, set a decent verbosity, save and restart postgres:
    sudo vim /etc/postgresql/9.3/main/postgresql.conf

# Uncomment/Change inside:

log_min_messages = debug5
log_min_error_statement = debug5
log_min_duration_statement = -1

sudo service postgresql restart
Now you will get tons of details of every statement, error, and even background tasks like VACUUMs
tail -f /var/log/postgresql/postgresql-9.3-main.log
How to add user who executed a PG statement to log (editing postgresql.conf):
log_line_prefix = '%t %u %d %a '
Create command
There are many CREATE choices, like CREATE DATABASE **database_name**, CREATE TABLE **table_name** ... Parameters differ but can be checked at the official documentation.

Handy queries
SELECT _ FROM pg_proc WHERE proname='**procedurename**': List procedure/function
SELECT _ FROM pg*views WHERE viewname='**viewname**';: List view (including the definition)
SELECT pg_size_pretty(pg_total_relation_size('**table_name**'));: Show DB table space in use
SELECT pg_size_pretty(pg_database_size('**database_name**'));: Show DB space in use
show statement_timeout;: Show current user's statement timeout
SELECT * FROM pg*indexes WHERE tablename='**table_name**' AND schemaname='**schema_name**';: Show table indexes
Get all indexes from all tables of a schema:
SELECT
t.relname AS table_name,
i.relname AS index_name,
a.attname AS column_name
FROM
pg_class t,
pg_class i,
pg_index ix,
pg_attribute a,
pg_namespace n
WHERE
t.oid = ix.indrelid
AND i.oid = ix.indexrelid
AND a.attrelid = t.oid
AND a.attnum = ANY(ix.indkey)
AND t.relnamespace = n.oid
AND n.nspname = 'kartones'
ORDER BY
t.relname,
i.relname
Execution data:
Queries being executed at a certain DB:
SELECT datname, application_name, pid, backend_start, query_start, state_change, state, query
FROM pg_stat_activity
WHERE datname='**database_name**';
Get all queries from all dbs waiting for data (might be hung):
SELECT * FROM pg_stat_activity WHERE waiting='t'
Currently running queries with process pid:
SELECT pg_stat_get_backend_pid(s.backendid) AS procpid,
pg_stat_get_backend_activity(s.backendid) AS current_query
FROM (SELECT pg_stat_get_backend_idset() AS backendid) AS s;
Casting:

CAST (column AS type) or column::type
'**table_name**'::regclass::oid: Get oid having a table name
Query analysis:

EXPLAIN **query**: see the query plan for the given query
EXPLAIN ANALYZE **query**: see and execute the query plan for the given query
ANALYZE [__table__]: collect statistics
Tools
pg-top: top for PG. sudo apt-get install ptop + pg_top
Unix-like reverse search in psql:
$ echo "bind "^R" em-inc-search-prev" > $HOME/.editrc
$ source $HOME/.editrc
PostgreSQL Exercises: An awesome resource to learn to learn SQL, teaching you with simple examples in a great visual way. Highly recommended.
A Performance Cheat Sheet for PostgreSQL: Great explanations of EXPLAIN, EXPLAIN ANALYZE, VACUUM, configuration parameters and more. Quite interesting if you need to tune-up a postgres setup.

## commands ii

```
CREATE TABLE flights (
 id SERIAL PRIMARY KEY,
 origin VARCHAR NOT NULL,
 destination VARCHAR NOT NULL,
 duration INTEGER NOT NULL
);

INSERT INTO flights
 (origin, destination, duration)
 VALUES ('New York', 'London', 415);

 INSERT INTO flights
 (origin, destination, duration)
 VALUES ('New York', 'London', 415);

SELECT * FROM flights;

SELECT origin, destination FROM flights;

SELECT * FROM flights WHERE id = 3;

SELECT * FROM flights WHERE origin = 'New York';
SELECT * FROM flights WHERE duration > 500;

SELECT * FROM flights
WHERE destination = 'Paris' AND duration > 500;

SELECT * FROM flights
WHERE destination = 'Paris' OR duration > 500;

SELECT * FROM flights
WHERE origin IN ('New York', 'Lima');

-now when i need to build a partial search
SELECT * FROM flights
WHERE origin LIKE '%a%';
```

### update and delete data

```
UPDATE flights
 SET duration = 430
 WHERE origin = 'New York'
 AND destination = 'London';

 DELETE FROM countries
 WHERE destination = 'Tokyo';
```

## Other Clauses

• LIMIT
• ORDER BY
• GROUP BY
• HAVING
