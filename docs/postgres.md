# PostgreSQL CLI Cheatsheet

## Enter PostgreSQL CLI
```bash
docker exec -it postgres psql -U postgres
```

## Create Database
```sql
CREATE DATABASE mydb;
```

## Connect to Database
```sql
\c mydb
```

## Reset Password
```sql
ALTER USER postgres WITH PASSWORD 'newpassword';
```

## Run SQL File
```bash
docker exec -i postgres psql -U postgres -d mydb < file.sql
```

## Useful Meta Commands
```
\l       -- list databases
\dt      -- list tables
\d table -- describe table
\q       -- quit
```