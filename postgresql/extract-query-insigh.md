# Detect slow queries in Postgres

`pg_stat_statement` is a Postgres extension that tracks statistics on the queries executed by a server. It is an unvaluable tool for detecting slow queries.

## Enable the extension

Add the extension to `postgres.conf`

```
shared_preload_libraries = 'pg_stat_statements'
pg_stat_statements.max = 10000
pg_stat_statements.track = all
```

Note the `pg_stat_statements` value: determine the right size of the tracking table based on your needs.

The extension has to be enabled for each database. In `psql`, select the target database and type:

```
CREATE EXTENSION pg_stat_statements;
```

Restart the server.

## Extract metrics

Metrics can be extracted by quering the `pg_stat_statements` table. This are some useful queries:

Fetch all the tracked queries, ordered by `mean_time` in descending order.

```

select query,calls,total_time,min_time,max_time,mean_time,stddev_time,rows from pg_stat_statements order by mean_time desc;
``` 

Fetch the first 100 queries, ordered by total execution time.

```
SELECT 
  (total_time / 1000 / 60) as total, 
  (total_time/calls) as avg, 
  query 
FROM pg_stat_statements 
ORDER BY 1 DESC 
LIMIT 100;
```

Fetch the first 20 queries, ordered by total_time. Also includes the CPU time

```
SELECT substring(query, 1, 50) AS short_query,
              round(total_time::numeric, 2) AS total_time,
              calls,
              round(mean_time::numeric, 2) AS mean,
              round((100 * total_time / sum(total_time::numeric) OVER ())::numeric, 2) AS percentage_cpu
FROM  pg_stat_statements
ORDER BY total_time DESC
LIMIT 20;


                    short_query                     | total_time |  calls   |  mean  | percentage_cpu
----------------------------------------------------+------------+----------+--------+----------------
 select ax."dx",ax."co",ax."monthly", sum(value) as |  318759.35 |     2110 | 151.07 |          24.75
 update users set uid=$1, code=$2, created=$3, last |  307108.80 |    58565 |   5.24 |          23.85
 select ax."dx",ax."ao",ax."monthly", sum(value) as |  130027.48 |     5275 |  24.65 |          10.10
 select ax."dx",ax."co",ax."ao",ax."monthly", sum(v |  111492.40 |     3165 |  35.23 |           8.66
 select count(table_name) from information_schema.t |  101477.88 |    46385 |   2.19 |           7.88
 select dataelemen0_.dataelementid as dataelem1_49_ |   60082.05 |  1289272 |   0.05 |           4.67
 select ax."dx",ax."monthly", sum(value) as value f |   54975.80 |     1053 |  52.21 |           4.27
 select ax."dx",ax."monthly", sum(value) as value f |   48703.15 |     1053 |  46.25 |           3.78
 SET SESSION CHARACTERISTICS AS TRANSACTION READ WR |   45862.29 | 12628987 |   0.00 |           3.56
 SET SESSION CHARACTERISTICS AS TRANSACTION READ ON |   45812.48 | 12628961 |   0.00 |           3.56
 select dataelemen0_.dataelementid as dataelem1_49_ |   20298.39 |   351611 |   0.06 |           1.58
 select dataelemen0_.dataelementid as dataelem1_49_ |   19096.82 |   703161 |   0.03 |           1.48
 select dataelemen0_.dataelementid as dataelem1_49_ |    5872.07 |    58578 |   0.10 |           0.46
 select ax."dx",ax."yearly", sum(daysxvalue) / $1 a |    4368.62 |     4216 |   1.04 |           0.34
 select ax."dx",ax."co",ax."sixmonthly", sum(daysxv |    2272.33 |     2110 |   1.08 |           0.18
 select ax."dx",ax."co",ax."sixmonthly", sum(daysxv |    2254.51 |     2110 |   1.07 |           0.18
 select usercreden0_.userid as userid1_338_, usercr |    1981.65 |    64830 |   0.03 |           0.15
 COMMIT                                             |    1284.75 |  2535393 |   0.00 |           0.10
 BEGIN                                              |    1231.66 |  2535390 |   0.00 |           0.10
 select ax."dx",ax."yearly", sum(daysxvalue) / $1 a |    1109.74 |     2107 |   0.53 |           0.09

```

## Reset metrics

In order to reset the metrics, run the following SQL:

```
select pg_stat_statements_reset();
```

## Export metrics

`pg_stat_statement`  data can be exported to a CSV file using the following statement:


```
COPY (...) TO '/tmp/pg_stat_statement.csv' (format CSV, HEADER);
``` 

The three dots correspond to the query for which we want to export the data.


## More queries

```
-- Most CPU intensive queries (PGSQL v9.4)
SELECT substring(query, 1, 50) AS short_query, round(total_time::numeric, 2) AS total_time, calls, rows, round(total_time::numeric / calls, 2) AS avg_time, round((100 * total_time / sum(total_time::numeric) OVER ())::numeric, 2) AS percentage_cpu FROM pg_stat_statements ORDER BY total_time DESC LIMIT 20;

-- Most time consuming queries (PGSQL v9.4)
SELECT substring(query, 1, 100) AS short_query, round(total_time::numeric, 2) AS total_time, calls, rows, round(total_time::numeric / calls, 2) AS avg_time, round((100 * total_time / sum(total_time::numeric) OVER ())::numeric, 2) AS percentage_cpu FROM pg_stat_statements ORDER BY avg_time DESC LIMIT 20;

-- Maximum transaction age
select client_addr, usename, datname, clock_timestamp() - xact_start as xact_age, clock_timestamp() - query_start as query_age, query from pg_stat_activity order by xact_start, query_start;
-- Long-running transactions are bad because they prevent Postgres from vacuuming old data. This causes database bloat and, in extreme circumstances, shutdown due to transaction ID (xid) wraparound. Transactions should be kept as short as possible, ideally less than a minute.

-- Bad xacts
select * from pg_stat_activity where state in ('idle in transaction', 'idle in transaction (aborted)');

-- Waiting Connections for a lock
SELECT count(distinct pid) FROM pg_locks WHERE granted = false;

-- Connections
select client_addr, usename, datname, count(*) from pg_stat_activity group by 1,2,3 order by 4 desc;

-- User Connections Ratio
select count(*)*100/(select current_setting('max_connections')::int) from pg_stat_activity;

-- Average Statement Exec Time
select (sum(total_time) / sum(calls))::numeric(6,3) from pg_stat_statements;

-- Most writing (to shared_buffers) queries
select query, shared_blks_dirtied from pg_stat_statements where shared_blks_dirtied > 0 order by 2 desc;

-- Block Read Time
select * from pg_stat_statements where blk_read_time <> 0 order by blk_read_time desc;
```

