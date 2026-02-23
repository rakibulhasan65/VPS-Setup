# PostgreSQL VPS Management Guide

## 11. Upload Files from Windows to VPS

### Method 1: Using Windows CMD / PowerShell (Recommended)
Run from your Windows PC (NOT from VPS):
```
scp C:\temp\truepresence-frontend.zip root@SERVER_IP:/root/
```

### Method 2: Using Git Bash (Windows)
```
scp /c/temp/truepresence-frontend.zip root@SERVER_IP:/root/
```

---

## 12. PostgreSQL Service Management

Check PostgreSQL status:
```
sudo systemctl status postgresql
```

If using PostgreSQL 15:
```
sudo systemctl status postgresql-15
```

If stopped:
```
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

Restart service:
```
sudo systemctl restart postgresql
```

---

## 13. PostgreSQL Configuration (VPS Production)

### Main Config File Location
Ubuntu/Debian:
```
/etc/postgresql/15/main/postgresql.conf
```
CentOS/RHEL:
```
/var/lib/pgsql/data/postgresql.conf
```

### Important Settings
```
listen_addresses = '*'
port = 5432
```

### Edit pg_hba.conf

Ubuntu/Debian:
```
/etc/postgresql/15/main/pg_hba.conf
```

CentOS/RHEL:
```
/var/lib/pgsql/data/pg_hba.conf
```

Allow localhost access:
```
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```

After changes:
```
sudo systemctl restart postgresql
```

---

## 14. PostgreSQL Database Access (Example: Truepresence)

Login to database:
```
psql -h localhost -U attendance -d attendance
```

List databases:
```
\l
```

List tables:
```
\dt
```

Exit psql:
```
\q
```

---

## 15. PostgreSQL Backup Commands

Backup single database:
```
pg_dump -U truepresence -h localhost resence | gzip > resence_backup.sql.gz
```

Backup without gzip:
```
pg_dump -U attendance -h localhost attendance > attendance_backup.sql
```

Backup all databases:
```
pg_dumpall -U postgres > all_databases_backup.sql
```

---

## 16. PostgreSQL Import Commands

Import from SQL file:
```
psql -U attendance -d attendance -f /root/truepresence_backup.sql
```

Import from inside psql shell:
```
\i /root/truepresence_backup.sql
```

Import from gzip backup:
```
gunzip -c resence_backup.sql.gz | psql -U truepresence -d resence
```

---

## 17. PostgreSQL Important VPS Commands (Quick Reference)

Switch to postgres user:
```
sudo -i -u postgres
```

Create database:
```sql
CREATE DATABASE attendance;
```

Create user:
```sql
CREATE USER attendance WITH PASSWORD 'strongpassword';
```

Grant privileges:
```sql
GRANT ALL PRIVILEGES ON DATABASE attendance TO attendance;
```

Check active connections:
```sql
SELECT * FROM pg_stat_activity;
```

Check PostgreSQL version:
```
psql --version
```

Check listening port:
```
ss -tulpn | grep 5432
```

---

PostgreSQL VPS Guide Completed and Ready for GitHub.

