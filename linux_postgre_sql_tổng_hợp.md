# Linux – PostgreSQL (PGSQL) Tổng Hợp

Tài liệu này tổng hợp kiến thức về **PostgreSQL trên Linux**, bao gồm cài đặt, quản lý database, user, backup/restore, bảo mật và lệnh thường dùng. Tài liệu càng chi tiết càng tốt, phù hợp làm **document reference**.

---

## 1. Giới thiệu PostgreSQL
- PostgreSQL là **hệ quản trị cơ sở dữ liệu quan hệ mã nguồn mở** (RDBMS).
- Hỗ trợ: SQL chuẩn, JSON, PostGIS, replication, backup, stored procedure.
- Trên Linux, PostgreSQL thường cài từ package (`apt` trên Debian/Ubuntu, `yum/dnf` trên CentOS).

---

## 2. Cài đặt PostgreSQL trên Linux

### 2.1 Ubuntu / Debian
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib -y
sudo systemctl enable postgresql
sudo systemctl start postgresql
sudo systemctl status postgresql
```

### 2.2 CentOS / RHEL
```bash
sudo dnf install postgresql-server postgresql-contrib -y
sudo postgresql-setup --initdb
sudo systemctl enable postgresql
sudo systemctl start postgresql
sudo systemctl status postgresql
```

### 2.3 Kiểm tra version
```bash
psql --version
```

---

## 3. Kiến trúc PostgreSQL
- **Postmaster / Server process**: quản lý database và client connection
- **Database cluster**: tập hợp các database
- **Databases**: chứa schema, table, index, sequence
- **Roles / User**: quản lý quyền
- **Tablespaces**: nơi lưu trữ dữ liệu

---

## 4. Quản lý User / Role

### 4.1 Login PostgreSQL
```bash
sudo -i -u postgres
psql
```

### 4.2 Tạo user
```sql
CREATE ROLE username WITH LOGIN PASSWORD 'password';
```

### 4.3 Gán quyền
```sql
GRANT ALL PRIVILEGES ON DATABASE dbname TO username;
```

### 4.4 List users
```sql
\du
```

### 4.5 Thay đổi mật khẩu user
```sql
ALTER ROLE username WITH PASSWORD 'newpassword';
```

---

## 5. Quản lý Database

### 5.1 Tạo database
```sql
CREATE DATABASE dbname OWNER username;
```

### 5.2 Xem danh sách database
```sql
\l
```

### 5.3 Kết nối vào database
```sql
\c dbname
```

### 5.4 Xóa database
```sql
DROP DATABASE dbname;
```

---

## 6. Quản lý Table & Schema

### 6.1 Tạo table
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE,
    created_at TIMESTAMP DEFAULT NOW()
);
```

### 6.2 Xem danh sách table
```sql
\dt
```

### 6.3 Schema management
```sql
CREATE SCHEMA myschema AUTHORIZATION username;
SET search_path TO myschema;
```

### 6.4 Thêm dữ liệu
```sql
INSERT INTO users (name, email) VALUES ('John Doe', 'john@example.com');
```

### 6.5 Query dữ liệu
```sql
SELECT * FROM users;
SELECT id, name FROM users WHERE email LIKE '%@example.com';
```

### 6.6 Update / Delete
```sql
UPDATE users SET name='Jane' WHERE id=1;
DELETE FROM users WHERE id=1;
```

---

## 7. Backup & Restore

### 7.1 Backup database
```bash
pg_dump dbname > dbname_backup.sql
pg_dump -U username dbname > dbname_backup.sql
```

### 7.2 Restore database
```bash
psql dbname < dbname_backup.sql
pg_restore -U username -d dbname dbname_backup.dump   # nếu backup bằng pg_dump -Fc
```

### 7.3 Backup tất cả cluster
```bash
pg_dumpall > all_databases.sql
```

---

## 8. Cấu hình PostgreSQL

### 8.1 File quan trọng
- `/etc/postgresql/<version>/main/postgresql.conf` – cấu hình server
- `/etc/postgresql/<version>/main/pg_hba.conf` – cấu hình authentication
- `/var/lib/postgresql/` – nơi lưu trữ dữ liệu

### 8.2 Thay đổi port hoặc listen address
```conf
# postgresql.conf
port = 5432
listen_addresses = '*'
```

### 8.3 Authentication
- `pg_hba.conf` format:
```
# TYPE  DATABASE  USER  ADDRESS  METHOD
host    all       all   0.0.0.0/0  md5
```

### 8.4 Reload config
```bash
sudo systemctl reload postgresql
```

---

## 9. Performance & Monitoring

### 9.1 Kiểm tra connections
```sql
SELECT * FROM pg_stat_activity;
```

### 9.2 Kiểm tra table / index size
```sql
SELECT pg_size_pretty(pg_total_relation_size('users'));
```

### 9.3 Kiểm tra server status
```bash
sudo systemctl status postgresql
```

### 9.4 Logs
- File log thường nằm ở `/var/log/postgresql/` hoặc theo cấu hình `postgresql.conf`
- Theo dõi log realtime:
```bash
tail -f /var/log/postgresql/postgresql-<version>-main.log
```

---

## 10. Tính năng nâng cao

### 10.1 Replication (Streaming replication)
- Cấu hình **primary / standby** để replica realtime

### 10.2 Roles & permission nâng cao
- SUPERUSER, CREATEDB, CREATEROLE, REPLICATION

### 10.3 JSON / JSONB
```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    data JSONB
);
INSERT INTO orders(data) VALUES('{"product":"book

