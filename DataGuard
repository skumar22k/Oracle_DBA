# Why We Need to Enable FORCE LOGGING Option in the Data Guard Environment

``` sql
ALTER DATABASE FORCE LOGGING;
```

The **FORCE LOGGING** option is the safest method to ensure that all
changes made in the database will be captured and available for recovery
in the redo logs.

FORCE LOGGING is a feature added to the family of logging attributes.

## Tablespace-Level FORCE LOGGING

``` sql
ALTER TABLESPACE <tablespace_name> FORCE LOGGING;

ALTER TABLESPACE <tablespace_name> NO FORCE LOGGING;
```

Oracle 9i Release 2 introduced the **FORCE LOGGING** option.

The FORCE LOGGING option can be set at the **database level** or
**tablespace level**.

The precedence is from **database to tablespace**.

If a tablespace is created or altered to have FORCE LOGGING enabled, any
change in that tablespace will go into the redo log and be available for
recovery.

Similarly, if a database is created or altered to have FORCE LOGGING
enabled, any change across the database, with the exception of
**temporary segments and temporary tablespaces**, will be available in
the redo logs for recovery.

The FORCE LOGGING option can be set at database creation time or later
using the `ALTER DATABASE` command.

## Check FORCE LOGGING Status

``` sql
SELECT force_logging
FROM v$database;
```

To check the FORCE LOGGING status of a tablespace:

``` sql
SELECT force_logging
FROM dba_tablespaces;
```

## Performance Impact

Putting a database in **FORCE LOGGING** mode will have some
**performance impact**.
