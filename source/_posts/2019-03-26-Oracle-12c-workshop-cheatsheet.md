---
title: Oracle 12c workshop cheatsheet
date: 2019-03-26 11:00:30
tags:
---

This is a cheat sheet for Oracle 12C R2 workshop Part I

<!--more-->

# show

```sql
-- the current container name
SQL> SHOW con_name

-- current container id
SQL> SHOW con_id

-- current user
SQL> SHOW user
```

## show parameter

```sql
SQL> show parameter instance_name
SQL> show parameter service_names
```

# V$

```sql

-- List services
SQL > select con_id, name from v$services;
```

# CDB

## Basic Information

```sql
-- Checkout wheather the db is a non-CDB or CDB
SELECT name, cdb, con_id FROM v$database;

-- List containers in CDB
SELECT name, con_id FROM v$containers ORDER BY con_id;

-- List PDB in CDB
SQL> SHOW pdbs;
SELECT pdb_name, status FROM cdb_pdbs ORDER BY 1;

-- List users
SELECT DISTINCT username FROM cdb_users WHERE common ='YES'
SELECT con_id, username FROM cdb_users;

```

## Disk Informaiton

```sql
-- List all tablespace in CDB
SELECT d.file#, ts.name, ts.ts#, ts.con_id FROM v$datafile d, v$tablespace ts WHERE d.ts#=ts.ts# AND d.con_id=ts.con_id;

-- List all the data files in the CDB
SELECT file_name, tablespace_name FROM cdb_data_files;

-- List all tmp tablespace in CDB
SELECT file_name, tablespace_name FROM cdb_temp_files;

-- List all redo log in CDB
SELECT group#, member, con_id FROM v$logfile;

-- List all control file
SELECT name, con_id FROM v$controlfile;
```

## Instance Information

```sql
-- View the database instance name, its status
SELECT instance_name, status, con_id FROM v$instance;

-- List the services for all the containers in the CDB
SELECT con_id, name FROM v$services ORDER BY 1;
```

# PDB

```sql
-- switch CDB/PDB
SQL > ALTER SESSION SET CONTAINER = PDB1;
SQL > ALTER SESSION SET CONTAINER = CDB$ROOT;

-- open pdb
ALTER PLUGGABLE DATABASE PDB1 OPEN;
```

# lsnrctl

```bash

```

# Privilege

## Query

```sql
-- List common users
SELECT DISTINCT username FROM dba_users WHERE common='YES'

-- List PDB users
SELECT DISTINCT username FROM dba_users WHERE common='NO';

-- list privileges
select * from session_privs;
-- list roles
SELECT * FROM session_roles;

-- List system privilege granted
SELECT * FROM dba_sys_privs WHERE grantee='PDBADMIN';

-- list role granted for a specific user
SELECT granted_role, admin_option, default_role FROM cdb_role_privs WHERE grantee='PDBADMIN';

-- profile details
SELECT resource_type, resource_name, limit FROM dba_profiles WHERE profile='HRPROFILE';
```

## User

```sql
-- create a common user
CREATE USER c##CDB_ADMIN1 IDENTIFIED BY <password> CONTAINER=ALL DEFAULT TABLESPACE users TEMPORARY TABLESPACE temp ACCOUNT UNLOCK;

-- Create an application user
CREATE USER INVENTORY IDENTIFIED BY <password> DEFAULT TABLESPACE users QUOTA UNLIMITED ON users;

-- Grant privilege
GRANT CREATE SESSION, dba, sysdba TO c##CDB_ADMIN1 CONTAINER=ALL;
GRANT CREATE SESSION TO INVENTORY;
```

## Role

```sql
-- grant a role to user
GRANT DBA TO PDBADMIN;

-- create a role
create role "HRCLERK" NOT IDENTIFIED;
grant UPDATE on 'HR'.'EMPLOYEES' to 'HRCLERK';
grant SELECT on 'HR'.'EMPLOYEES' to 'HRCLERK';

-- alter default role
ALTER USER JGOODMAN DEFAULT ROLE HRCLERK;

```

## Profile

```sql
create profile 'HRPROFILE' limit
        cpu_per_session UNLIMITED
        cpu_per_call UNLIMITED
        connect_time UNLIMITED
        idle_time 60
        sessions_per_user UNLIMITED
        password_life_time UNLIMITED
        ...

-- update a profile
ALTER PROFILE hrprofile LIMIT INACTIVE_ACCOUNT_TIME 10;
```

## Audit Policy

```sql
-- Verify that unified auditing is now enabled
SELECT * FROM v$option WHERE parameter = 'Unified Auditing';

-- create policy
CREATE AUDIT POLICY drop_pol PRIVILEGES DROP ANY TABLE;

-- Verify that the audit policy is enabled for users granted the DBA role.
SELECT entity_name, entity_type, enabled_option FROM audit_unified_enabled_policies WHERE policy_name = 'DROP_POL';

-- View audit log
SELECT dbusername, action_name, object_name FROM unified_audit_trail WHERE dbusername = 'PDBADMIN' AND action_name = 'DROP TABLE'

-- diable policy
NOAUDIT POLICY drop_pol BY USERS WITH GRANTED ROLES dba;

-- drop policy
DROP AUDIT POLICY drop_pol;
```

# Tablespace

## dictionary

Tablespace information:

* DBA_TABLESPACES
* V$TABLESPACE

Data file information:

* DBA_DATA_FILES
* V$DATAFILE

Temp file information:

* DBA_TEMP_FILES
* V$TEMPFILE

Tables in a tablespace:

* ALL_TABLES

```sql
-- list tablespaces
SELECT DISTINCT tablespace_name FROM dba_tablespaces;

-- find out tablespace contains HR schema
SELECT DISTINCT tablespace_name FROM all_tables WHERE owner='HR';

-- select tables name
SELECT table_name FROM all_tables WHERE tablespace_name='INVENTORY';

-- grab tablespace details
SELECT status, contents, logging, plugged_in, bigfile, extent_management, allocation_type FROM dba_tablespaces where tablespace_name='SYSAUX';

SELECT * FROM v$tablespace WHERE name='SYSAUX';

-- data file
SELECT file_name, autoextensible, bytes, maxbytes, user_bytes FROM dba_data_files WHERE tablespace_name='SYSAUX';

-- check out index
SELECT index_name FROM all_indexes WHERE tablespace_name='SYSAUX' AND owner='HR';

```

## maintain

```sql
-- Create tablespace
CREATE SMALLFILE TABLESPACE INVENTORY
DATAFILE '/u01/app/oracle/oradata/ORCL/PDB1/INVENTORY01.DBF' SIZE 5242880 DEFAULT NOCOMPRESS
ONLINE
SEGMENT SPACE MANAGEMENT AUTO
EXTENT MANAGEMENT LOCAL AUTOALLOCATE;

-- Resize file
ALTER DATABASE DATAFILE '/u01/app/oracle/oradata/ORCL/PDB1/INVENTORY01.DBF' RESIZE 40M;

-- Add data file
ALTER TABLESPACE "INVENTORY" ADD DATAFILE '/u01/app/oracle/oradata/ORCL/PDB1/INVENTORY02.DBF' SIZE 30M AUTOEXTEND OFF;

-- Relocating an online data file
ALTER DATABASE MOVE DATAFILE '/disk1/myexample01.dbf' TO '/disk2/myexample01.dbf';

-- Rename
ALTER DATABASE MOVE DATAFILE '/disk1/myexample01.dbf' TO '/disk1/myexample02.dbf';

-- Delete
DROP TABLESPACE inventory INCLUDING CONTENTS AND DATAFILES;
```