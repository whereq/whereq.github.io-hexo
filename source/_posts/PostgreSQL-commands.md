---
title: PostgreSQL commands
date: 2025-03-23 10:12:56
categories:
- Postgres
tags:
- Postgres
---

## Database Management

### List All Databases
```sql
\l
```

### Connect to a Database
```sql
\c database_name
```

### Create a New Database
```sql
CREATE DATABASE database_name;
```

### Drop a Database
```sql
DROP DATABASE database_name;
```

### Check Database Size
```sql
SELECT pg_size_pretty(pg_database_size('database_name'));
```

### List All Users/Roles
```sql
\du
```

### Check Current Database
```sql
SELECT current_database();
```

---

## Table Management

### List All Tables in the Current Database
```sql
\dt
```

### List All Tables with Additional Details
```sql
\dt+
```

### Describe a Table (Columns, Types, etc.)
```sql
\d table_name
```

### Show Table Schema
```sql
\d+ table_name
```

### Check Table Size (Including Indexes)
```sql
SELECT pg_size_pretty(pg_total_relation_size('table_name'));
```

### Check Table Row Count
```sql
SELECT count(*) FROM table_name;
```

### Show Indexes of a Table
```sql
\di table_name
```

### Show Table Constraints
```sql
\d table_name
```

---

## Query Execution and Statistics

### Explain a Query (Execution Plan)
```sql
EXPLAIN SELECT * FROM table_name;
```

### Explain with Detailed Statistics
```sql
EXPLAIN ANALYZE SELECT * FROM table_name;
```

### Show Query Execution Time
```sql
\timing
```

### Show Active Queries
```sql
SELECT * FROM pg_stat_activity;
```

### Cancel a Running Query
```sql
SELECT pg_cancel_backend(pid);
```

### Terminate a Running Query
```sql
SELECT pg_terminate_backend(pid);
```

---

## User and Role Management

### Create a New User/Role
```sql
CREATE ROLE role_name WITH LOGIN PASSWORD 'password';
```

### Grant Privileges to a User
```sql
GRANT ALL PRIVILEGES ON DATABASE database_name TO role_name;
```

### Revoke Privileges from a User
```sql
REVOKE ALL PRIVILEGES ON DATABASE database_name FROM role_name;
```

### Alter User Password
```sql
ALTER USER role_name WITH PASSWORD 'new_password';
```

### Delete a User/Role
```sql
DROP ROLE role_name;
```

---

## Backup and Restore

### Backup a Database
```bash
pg_dump -U username -d database_name -f backup_file.sql
```

### Restore a Database
```bash
psql -U username -d database_name -f backup_file.sql
```

### Backup All Databases
```bash
pg_dumpall -U username -f backup_file.sql
```

### Restore All Databases
```bash
psql -U username -f backup_file.sql
```

---

## Miscellaneous

### Show PostgreSQL Version
```sql
SELECT version();
```

### Show Current User
```sql
SELECT current_user;
```

### Show All Settings
```sql
SHOW ALL;
```

### Quit psql
```sql
\q
```

### Help for SQL Commands
```sql
\h
```

### Help for psql Commands
```sql
\?
```