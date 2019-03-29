---
title: Oracle 12c workshop cheatsheet II
date: 2019-03-27 13:39:35
tags:
---

This is a cheat sheet for Oracle 12C R2 workshop Part II

Will conver those topics:

- Storage space
- Undo
- Moving Data
- Backup and Recovery

<!--more-->

# Manage storage space

```sql
-- check disk space
SELECT df.tablespace_name tablespace, fs.bytes free, df.bytes, fs.bytes *100/ df.bytes PCT_FREE
FROM dba_data_files df ,dba_free_space fs
WHERE df.tablespace_name = fs.tablespace_name 4 AND df.tablespace_name = 'TBSALERT';

```

```sql
-- checkout metric
SELECT warning_value, critical_value
FROM dba_thresholds
WHERE metrics_name='Tablespace Space Usage' 4 AND object_name IS NULL;
```

# Undo

[Transaction Management](https://docs.oracle.com/cd/B19306_01/server.102/b14220/transact.htm)

Two UNDO modes: SHARED versus LOCAL

- There is only one shared UNDO tablespace (in CDB root).
- There can be a - local UNDO tablespace in each PDB.

```sql
-- check local undo flag

SELECT property_name, property_value
FROM database_properties
WHERE property_name = 'LOCAL_UNDO_ENABLED'

```

## Enable Local Undo Mode

```sql
STARTUP UPGRADE
ALTER DATABASE LOCAL UNDO ON;
SHUTDOWN IMMEDIATE
STARTUP
```

## Enable Temp Undo

```sql
ALTER SESSION SET temp_undo_enabled = true;
```

# Moving Data

## Directory

```sql
-- Create
CREATE DIRECTORY dp_for_oe AS '/u01/app/oracle/admin/ORCL/dpdump';
-- Grant permission
GRANT read, write ON DIRECTORY dp_for_oe TO oe;
```

## expdp/impdp

```bash
# Export
expdp oe/<password>@PDB1 SCHEMAS=oe DIRECTORY=dp_for_oe DUMPFILE=expoe.dmp

# dummy import, convert to sql script
impdp oe/<password>@PDB1 SCHEMAS=oe DIRECTORY=dp_for_oe DUMPFILE=expoe.dmp SQLFILE=oe_SQL

# Import By File
impdp SYSTEM/<password>@PDB2 REMAP_SCHEMA=oe:oetest DIRECTORY=dp_for_oe DUMPFILE=expoe.dmp

# Import by DB Link
impdp SYSTEM/<password>@PDB2 SCHEMAS=oe REMAP_SCHEMA=oe:oetest NETWORK_LINK=link_pdb1
```

## sqlldr

[SQL Loder Doc](https://docs.oracle.com/cd/B19306_01/server.102/b14215/ldr_params.htm)

```sql
sqlldr userid=sh/<password>@PDB2 control=DP_inventories.ctl log=inventories.log data=DP_inventories.dat

# Direct Mode.  The direct load loads records into the blocks
sqlldr userid=sh/<password>@PDB2 control=DP_inventories.ctl log=inventories.log data=DP_inventories.dat ROWS=10 DIRECT=TRUE

```

# External Table

```sql
-- Verify that the locations are correctly set for the partitions
SELECT table_name, partition_name, location, directory_name FROM DBA_XTERNAL_LOC_PARTITIONS;
```

Create external table

```sql
CREATE TABLE sh.sales_ext_range
(
    time_id DATE NOT NULL,
    prod_id INTEGER NOT NULL,
    cust_id INTEGER NOT NULL,
    channel_id INTEGER NOT NULL,
    promo_id INTEGER NOT NULL,
    quantity_sold NUMBER(10,2),
    amount_sole NUMBER(10,2)
)
ORGANIZATION EXTERNAL
(
TYPE ORACLE_LOADER
   DEFAULT DIRECTORY ext_dir
   ACCESS PARAMETERS
   (
      RECORDS DELIMITED BY NEWLINE
      BADFILE 'sh%a_%p.bad'
      LOGFILE 'sh%a_%p.log'
      FIELDS TERMINATED BY ','
      MISSING FIELD VALUES ARE NULL
    )
)
PARALLEL
REJECT LIMIT UNLIMITED
PARTITION by range (time_id)
(
    PARTITION year1998 VALUES LESS THAN (TO_DATE('31-12-1998', 'DD-MM-YYYY')) LOCATION ('DP_sales_1998.dat'),
    PARTITION year1999 VALUES LESS THAN (TO_DATE('31-12-1999', 'DD-MM-YYYY')) LOCATION (ext_dir2:'DP2_sales_1999.dat')
);

```

# Backup & Recovery

## Checkpoint

Responsible

- Updating data file headers with checkpoint info
- Updating control files with checkpoint info
- Signaling DBWn at full checkpoints

## Redo log and Log Writer (LGWR)

Trigger

- At commit
- When on-third full
- Every 3 sec
- Before DBWn write
- Before clean shutdowns

## Automatic Instance or Crash Recovery

Is caused by attempts to open a database whose files are not synchronized on shutdown
Uses information stored in redo log groups to synchronize files

Involves two distinct operations:

- Rolling forward: Redo log changes (both committed and uncommitted) are applied to data files.
- Rolling back: Changes that are made but not committed are returned to their original state.

## Backup Type

- Level 0. FULL
- Level 1. Incremental
  - Cumulative
  - Differential

## Complete Recovery

Brings the database or tablespace up to the present, including all committed data changes made to the point in time when the recovery was requested

1. Restored data files
2. Changes applied (redo logs)
3. Data files containning committed and uncommitted trans
4. Dndo applied
5. Recovered data files

## Pint-in-Time Recovery Process

1. Restored data files from as far back as necessary
2. Changes applied to point in time (PIT). (Redo logs)
3. Data files containing committed and un committed trans up to PIT
4. Database opened
5. Undo applied
6. PIT-recovered data files

## List control files

```sql
SELECT name FROM v$controlfile;
SQL> SHOW PARAMETER control_files
```

## Create a Control File

```sql
SQL> CREATE PFILE FROM SPFILE;
SQL> SHUTDOWN IMMEDIATE
```

```bash
# Upcate pfile
$ vi $ORACLE_HOME/dbs/initORCL.ora

*.control_files='/u01/app/oracle/oradata/ORCL/control01.ctl', '/u01/app/oracle/fast_recovery_area/ORCL/control02.ctl','/u01/app/oracle/ controlfiles_dir/ORCL/control03.ctl'

# copy file
$ cp /u01/app/oracle/oradata/ORCL/control01.ctl /u01/app/oracle/controlfiles_dir /ORCL/control03.ctl
```

```sql
-- refresh from pfile to spfile
CREATE SPFILE FROM PFILE;
STARTUP

-- check control file
SHOW PARAMETER control_files
```

## Fast Recovery Area (FRA)

```sql
SQL> SHOW PARAMETER db_recovery_file_dest

-- Change the size of the fast recovery area
SQL> ALTER SYSTEM SET db_recovery_file_dest_size = 18G SCOPE=both;
SQL> SHOW PARAMETER db_recovery_file_dest_size
```

## Redo Log

```sql
SELECT group#, status, member FROM v$logfile;
SELECT group#, members, archived, status FROM v$log;

-- Switch log
ALTER SYSTEM SWITCH LOGFILE;

-- Add log
ALTER DATABASE ADD LOGFILE MEMBER '/u01/app/oracle/fast_recovery_area/ORCL/redo01b.log' TO GROUP 1;
```

## ARCHIVELOG Mode

```sql
-- List archived logs
SQL> SELECT name FROM v$archived_log ORDER BY stamp;
SQL> ARCHIVE LOG LIST;

-- Enable archive mode
SQL> SHUTDOWN IMMEDIATE
SQL> STARTUP MOUNT
SQL> ALTER DATABASE ARCHIVELOG;
SQL> ALTER DATABASE OPEN;
```

## Backing up the Control File

The control file and server parameter file (SPFILE) are backup automatically by default

Manually backup it as below

```sql
-- Back up the control file to a trace file
-- Location: /u01/app/oracle/diag/rdbms/orcl/ORCL/trace
ALTER DATABASE BACKUP controlfile TO trace;
```

## Whole DB backup

```bash
$ rman target /
RMAN> REPORT schema;
RMAN> BACKUP DATABASE PLUS ARCHIVELOG;

# partial backup
RMAN> BACKUP PLUGGABLE DATABASE PDB1 PLUS ARCHIVELOG;

# backup ts
RMAN> BACKUP TABLESPACE tbs_app;


# List backup
RMAN> LIST BACKUP;
```

## Recovery

```sql
RMAN> RECOVER DATAFILE 1;


--- Recover failure
RMAN> LIST FAILURE;
RMAN> ADVISE FAILURE;
RMAN> REPAIR FAILURE PREVIEW;
```

```sql
rman target SYS/<password>@PDB1

RMAN> SHUTDOWN IMMEDIATE;
RMAN> RESTORE DATABASE;


-- restores and recovers the data file in one step
RMAN> REPAIR PLUGGABLE DATABASE pdb1;
```

# Monitor DB

```sql
-- kill locked session
ALTER SYSTEM KILL SESSION '278,21828';
```