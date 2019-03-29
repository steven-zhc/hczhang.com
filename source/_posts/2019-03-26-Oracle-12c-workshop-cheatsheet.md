---
title: Oracle 12c workshop cheatsheet
date: 2019-03-26 11:00:30
tags:
---

This is a cheat sheet for Oracle 12C R2 workshop Part I

Will conver those topics:

- EM
- CBD
- PDB
- Network
- User Security
- Tablespaces

<!--more-->

# EM Express

```sql
-- enable em and listen at https://localhost:5500/em.
sqlplus / as sysdba
SELECT dbms_xdb_config.gethttpsport() FROM dual;
exec dbms_xdb_config.SetGlobalPortEnabled(TRUE);
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

## Create

```sql
-- Prepare folder
$ mkdir $ORACLE_BASE/oradata/ORCL/PDB2
$ sqlplus / as sysdba

CREATE PLUGGABLE DATABASE PDB2 ADMIN USER PDB2ADMIN IDENTIFIED BY <password>
ROLES=(dba)
DEFAULT TABLESPACE USERS
DATAFILE '/u01/app/oracle/oradata/ORCL/PDB2/users01.dbf'
SIZE 250M AUTOEXTEND ON
FILE_NAME_CONVERT=('/u01/app/oracle/oradata/ORCL/pdbseed/', '/u01/app/oracle/oradata/ORCL/PDB2/');

-- Open it
ALTER PLUGGABLE DATABASE PDB2 OPEN;

-- Select service
SQL> SELECT name FROM v$services;
```

## Clone

```sql
mkdir $ORACLE_BASE/oradata/ORCL/PDB3
sqlplus / as sysdba

CREATE PLUGGABLE DATABASE PDB3 FROM PDB1 CREATE_FILE_DEST= '/u01/app/oracle/oradata/ORCL/PDB3';
ALTER PLUGGABLE DATABASE PDB3 OPEN;
```

## Unplugging and Plugging a PDB

### Unplugging

```sql
ALTER PLUGGABLE DATABASE PDB3 CLOSE IMMEDIATE;

ALTER PLUGGABLE DATABASE PDB3 UNPLUG INTO '/u01/app/oracle/oradata/PDB3.xml';

DROP PLUGGABLE DATABASE PDB3 KEEP DATAFILES;
```

### check compatiable

```sql
SQL> set serveroutput on

DECLARE
compatible BOOLEAN := FALSE;
BEGIN
compatible := DBMS_PDB.CHECK_PLUG_COMPATIBILITY(
           pdb_descr_file => '/u01/app/oracle/oradata/PDB3.xml');
if compatible then
DBMS_OUTPUT.PUT_LINE('PDB3 is compatible');
else DBMS_OUTPUT.PUT_LINE('PDB3 is not compatible');
end if;
END;
 /
```

### Plugging

```sql
CREATE PLUGGABLE DATABASE HRPDB USING '/u01/app/oracle/oradata/PDB3.xml' NOCOPY TEMPFILE REUSE;
```

## Drop

```sql
ALTER PLUGGABLE DATABASE HRPDB CLOSE;
DROP PLUGGABLE DATABASE HRPDB INCLUDING DATAFILES;
```

# Database Instance

## Lifecycle

```sql
SHUTDOWN
SHUTDOWN ABORT
SHUTDOWN IMMEDIATE

STARTUP NOMOUNT;
ALTER DATABASE MOUNT;
ALTER DATABASE OPEN;
ALTER PLUGGABLE DATABASE pdb1 OPEN;
```

## spfile/pfile

Load order:

1. spfile
2. pfile

```sql
-- Locate the default spfile for your database instance
SHOW PARAMETER spfile

-- create pfile from spfile
CREATE PFILE = 'initORCL.ora' FROM SPFILE;

-- check spfile
-- If the value is null, which means the database instance was started with a pfile.
SHOW PARAMETER spfile
```

View parameters:

- V$PARAMETER
- V$SPPARAMETER
- V$PARAMETER2
- V$SYSTEM_PARAMETER

```sql
SHOW PARAMETER db_name
SHOW PARAMETER db_domain
SHOW PARAMETER db_recovery_file_dest
SHOW PARAMETER sga
SHOW PARAMETER undo_tablespace
SHOW PARAMETER compatible
SHOW PARAMETER control_files
SHOW PARAMETER shared_pool_size
SHOW PARAMETER db_block_size
SHOW PARAMETER db_cache_size
SHOW PARAMETER undo_management
SHOW PARAMETER memory_target
SHOW PARAMETER memory_max_target
SHOW PARAMETER pga_aggregate_target
```

## Update

```sql
-- setup parameter
ALTER SESSION SET nls_date_format = 'mon dd yyyy';

-- update in both the database instance memory and in the spfile
ALTER SYSTEM SET job_queue_processes=15 SCOPE=BOTH;
```

## Diagnostic

```sql
SELECT name, value FROM v$diag_info;
```

## adrci

```bash
$ adrci
adrci> SHOW ALERT

# Find the retention policy values
adrci> SET HOMEPATH diag/rdbms/orcl/ORCL
adrci> SELECT sizep_policy FROM adr_control_aux;

# Limit the target size for ADR ORCL diagnostics files to 200MB.
adrci> SET CONTROL (SIZEP_POLICY = 200000000)
adrci> SELECT sizep_policy FROM adr_control_aux;

# purge the ADR down to 5MB
# $ORACLE_BASE/diag/rdbms/orcl/ORCL
adrci> PURGE -size 5000000
```

## Log DDL

Target file: */u01/app/oracle/diag/rdbms/orcl/ORCL/log/ddl_ORCL.log*

```sql
-- check enable/disable
SQL> SHOW PARAMETER enable_ddl_logging

-- enable it
SQL> ALTER SESSION SET enable_ddl_logging = TRUE;
```

# Networking

Two files:

- listeners.ora: lnsrctl used
- tnsnames.ora: Network configuration file. 
  - /u01/app/oracle/product/12.2.0/dbhome_1/network/admin/tnsnames.ora

```sql
SQL> SHOW PARAMETER INSTANCE_NAME
SQL> SHOW PARAMETER SERVICE_NAMES
SQL> SHOW PARAMETER LOCAL_LISTENER
SQL> SHOW PARAMETER REMOTE_LISTENER
```

## lsnrctl

```bash
lsnrctl

LSNRCTL> show current_listener
LSNRCTL> status
LSNRCTL> services


```

## Dynamic Listener

create a listener, named LISTENER2, that listens on the non-default port 1561 for all database
services

```bash
# tnsnames.ora Network Configuration File
LISTENER2 =
  (ADDRESS = (PROTOCOL = TCP)(HOST = 12cr2db.example.com)(PORT = 1561))

LISTENER_ORCL =
  (ADDRESS = (PROTOCOL = TCP)(HOST = 12cr2db.example.com )(PORT = 1521))

```

```bash
# listener.ora. used for tnsrctl
LISTENER2 =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 12cr2db.example.com)(PORT = 1561))
  )
ADR_BASE_LISTENER2 = /u01/app/oracle
```

```sql
SQL> SHOW PARAMETER local_listener
SQL> SELECT isses_modifiable, issys_modifiable FROM v$parameter WHERE name='local_listener';
SQL> ALTER SYSTEM SET local_listener="LISTENER_ORCL,LISTENER2" SCOPE=BOTH;

-- Start LISTENER2
LSNRCTL> start LISTENER2
```

## Static Listener for a PDB

create a listener named LISTENER_PDB1 that listens on the non-default port 1562 for the
PDB1.example.com service

```bash
# listener.ora
LISTENER_PDB1 =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 12cr2db.example.com)(PORT = 1562))
      )
SID_LIST_LISTENER_PDB1 =
 (SID_LIST =
    (SID_DESC =
        (GLOBAL_DBNAME = PDB1.example.com)
        (SID_NAME = ORCL)
        (ORACLE_HOME = /u01/app/oracle/product/12.2.0/dbhome_1)
    )
)

lsnrctl
LSNRCTL> start LISTENER_PDB1
```

## TNS name

$ORACLE_HOME/network/admin/tnsnames.ora

```bash
MYPDB1 =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 12cr2db.example.com)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVICE_NAME = PDB1.example.com )
    )
)

tnsping MyPDB1
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

## Dictionary

Tablespace information:

- DBA_TABLESPACES
- V$TABLESPACE

Data file information:

- DBA_DATA_FILES
- V$DATAFILE

Temp file information:

- DBA_TEMP_FILES
- V$TEMPFILE

Tables in a tablespace:

- ALL_TABLES

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

## Maintain

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