# Postgres / MySQL Cheat Sheet

## Connecting
```bash
psql -U user -d dbname -h localhost -p 5432
mysql -u user -p -h localhost -D dbname
psql "postgresql://user:pass@host:5432/dbname"       # connection string
```

## Postgres (psql) meta-commands
```
\l              list databases
\c dbname       connect to a database
\dt             list tables
\dt+            list tables with sizes
\d tablename    describe table (columns, types, indexes)
\du             list roles/users
\df             list functions
\di             list indexes
\x              toggle expanded (vertical) output — great for wide rows
\timing         toggle query timing
\q              quit
\i script.sql   run SQL from a file
```

## MySQL meta-commands
```
SHOW DATABASES;
USE dbname;
SHOW TABLES;
DESCRIBE tablename;          -- or: SHOW COLUMNS FROM tablename;
SHOW INDEX FROM tablename;
SHOW GRANTS FOR 'user'@'%';
SHOW PROCESSLIST;
```

## Common SQL
```sql
SELECT * FROM users WHERE active = true ORDER BY created_at DESC LIMIT 10;
SELECT COUNT(*) FROM users;
INSERT INTO users (name, email) VALUES ('Chris', 'chris@example.com');
UPDATE users SET active = false WHERE id = 5;
DELETE FROM users WHERE id = 5;
SELECT column, COUNT(*) FROM table GROUP BY column ORDER BY COUNT(*) DESC;
```

## Schema changes
```sql
-- Postgres
ALTER TABLE users ADD COLUMN age INT;
ALTER TABLE users ALTER COLUMN age SET DEFAULT 0;
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);   -- no table lock

-- MySQL
ALTER TABLE users ADD COLUMN age INT;
ALTER TABLE users MODIFY COLUMN age INT DEFAULT 0;
CREATE INDEX idx_users_email ON users(email);
```

## Backup & restore
```bash
# Postgres
pg_dump -U user -d dbname -F c -f backup.dump     # custom format (compressed)
pg_dump -U user -d dbname > backup.sql              # plain SQL
pg_restore -U user -d dbname backup.dump
psql -U user -d dbname < backup.sql

# MySQL
mysqldump -u user -p dbname > backup.sql
mysqldump -u user -p --all-databases > all.sql
mysql -u user -p dbname < backup.sql
```

## Users & permissions
```sql
-- Postgres
CREATE USER app_user WITH PASSWORD 'secret';
GRANT ALL PRIVILEGES ON DATABASE dbname TO app_user;
GRANT SELECT, INSERT ON users TO app_user;
ALTER USER app_user WITH PASSWORD 'newpass';

-- MySQL
CREATE USER 'app_user'@'%' IDENTIFIED BY 'secret';
GRANT ALL PRIVILEGES ON dbname.* TO 'app_user'@'%';
FLUSH PRIVILEGES;
```

## Performance / diagnostics
```sql
-- Postgres
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'x@example.com';
SELECT * FROM pg_stat_activity;                        -- current connections/queries
SELECT pg_size_pretty(pg_database_size('dbname'));       -- db size
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE pid = 12345;  -- kill query

-- MySQL
EXPLAIN SELECT * FROM users WHERE email = 'x@example.com';
SHOW PROCESSLIST;
KILL 12345;
SELECT table_schema, ROUND(SUM(data_length+index_length)/1024/1024,1) AS mb
  FROM information_schema.tables GROUP BY table_schema;
```

## Useful one-liners
```bash
psql -U user -d dbname -c "SELECT COUNT(*) FROM users;"     # run inline query, no prompt
psql -U user -d dbname -c "\copy users TO 'users.csv' CSV HEADER"   # export to CSV
mysql -u user -p -e "SELECT * FROM users LIMIT 5" dbname       # inline query, MySQL
watch -n2 "psql -U user -d dbname -c 'SELECT count(*) FROM pg_stat_activity;'"  # monitor conns
```
