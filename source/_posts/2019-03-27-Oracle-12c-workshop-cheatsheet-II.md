---
title: Oracle 12c workshop cheatsheet II
date: 2019-03-27 13:39:35
tags:
---

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

Enable Local Undo Mode

```sql
STARTUP UPGRADE
ALTER DATABASE LOCAL UNDO ON;
SHUTDOWN IMMEDIATE
STARTUP
```

Enable Temp Undo

```sql
ALTER SESSION SET temp_undo_enabled = true;
```