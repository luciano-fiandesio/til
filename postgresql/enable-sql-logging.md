# Enable Postgresql SQL logging

## Find out where postgresql.conf is located

```
psql

SHOW config_file;
```

## Apply the logging configuration

Paste the following settings at the end of `postgresql.conf`

```
log_statement = 'all'
log_directory = 'pg_log'
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
logging_collector = on
```

## Restart postgres

```
sudo service postgresql restart
```

## Locate the logs

```
psql


SHOW data_directory;
```

The log files are located in the `pg_log` directory within the directory displayed by `SHOW data_directory;`.