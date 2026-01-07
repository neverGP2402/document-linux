# 📦 HƯỚNG DẪN CHI TIẾT CÀI ĐẶT & VẬN HÀNH POSTGRESQL TRÊN VPS UBUNTU 22.04 / 24.04

> **Tài liệu nội bộ dùng để đào tạo & triển khai thống nhất PostgreSQL trên nhiều server**  
> Phù hợp cho DEV / STAGING / PROD  
> Có thể chỉnh sửa trực tiếp tài liệu này trước khi áp dụng thực tế

---

## 0. Mục tiêu tài liệu
- Chuẩn hóa quy trình cài PostgreSQL cho toàn bộ server
- Tránh lỗi sai phổ biến khi vận hành PostgreSQL trên VPS
- Đảm bảo an toàn, dễ backup, dễ bàn giao cho nhân sự mới

---

## 1. Thông tin chuẩn hệ thống
| Thành phần | Giá trị |
|---------|--------|
| OS | Ubuntu 22.04 / 24.04 LTS |
| PostgreSQL | 16.x (repo mặc định Ubuntu) |
| Port | 5432 |
| DB Engine | PostgreSQL |
| User hệ thống | postgres |

> ⚠️ Quy ước: **Tuyệt đối không dùng user `postgres` cho ứng dụng**

---

## 2. Chuẩn bị trước khi cài đặt
### 2.1 Kiểm tra OS
```bash
lsb_release -a
```

### 2.2 Update hệ thống
```bash
sudo apt update && sudo apt upgrade -y
```

---

## 3. Cài đặt PostgreSQL
```bash
sudo apt install postgresql postgresql-contrib -y
```

Kiểm tra version:
```bash
psql --version
```

Ví dụ:
```
psql (PostgreSQL) 16.11
```

---

## 4. Hiểu đúng về service PostgreSQL (RẤT QUAN TRỌNG)
### 4.1 Kiểm tra service
```bash
systemctl status postgresql
```

Nếu thấy:
```
Active: active (exited)
```
👉 **HOÀN TOÀN BÌNH THƯỜNG**

> PostgreSQL trên Ubuntu chạy theo **cluster**, service `postgresql.service` chỉ đóng vai trò khởi động

---

## 5. Kiểm tra PostgreSQL Cluster
```bash
pg_lsclusters
```

Output chuẩn:
```
Ver Cluster Port Status Owner    Data directory
16  main    5432 online postgres /var/lib/postgresql/16/main
```

| Trạng thái | Ý nghĩa |
|----------|--------|
| online | PostgreSQL đang chạy |
| down | PostgreSQL chưa chạy |

Khởi động cluster nếu cần:
```bash
sudo pg_ctlcluster 16 main start
```

---

## 6. Đăng nhập PostgreSQL bằng user hệ thống
```bash
sudo -i -u postgres
psql
```

Thoát:
```sql
\q
```

---

## 7. Thiết lập mật khẩu cho user postgres
> Bắt buộc để tăng bảo mật

```bash
sudo -u postgres psql
```

```sql
ALTER USER postgres PASSWORD 'StrongPasswordHere';
```

---

## 8. Tạo Database & User cho ứng dụng (CHUẨN PROD)
> Mỗi ứng dụng = 1 database + 1 user riêng

```sql
CREATE DATABASE mydb;
CREATE USER myuser WITH ENCRYPTED PASSWORD 'MyStrongPass123';
GRANT ALL PRIVILEGES ON DATABASE mydb TO myuser;
```

Kiểm tra:
```sql
\l
\du
```

---

## 9. Cấu hình cho phép kết nối từ xa
### 9.1 File cấu hình chính
```bash
sudo nano /etc/postgresql/16/main/postgresql.conf
```

Sửa:
```conf
listen_addresses = '*'
```

---

### 9.2 Phân quyền kết nối (pg_hba.conf)
```bash
sudo nano /etc/postgresql/16/main/pg_hba.conf
```

❌ Không khuyến nghị:
```conf
host    all     all     0.0.0.0/0     md5
```

✅ PROD chuẩn:
```conf
host    all     all     YOUR_IP/32    md5
```

---

## 10. Restart PostgreSQL đúng cách
```bash
sudo pg_ctlcluster 16 main restart
```

Hoặc:
```bash
sudo systemctl restart postgresql@16-main
```

---

## 11. Kiểm tra PostgreSQL lắng nghe port
```bash
ss -lntp | grep 5432
```

Output chuẩn:
```
LISTEN 0 128 0.0.0.0:5432
```

---

## 12. Firewall (UFW)
```bash
sudo ufw allow 5432/tcp
sudo ufw reload
sudo ufw status
```

> ⚠️ PROD nên whitelist IP cụ thể

---

## 13. Test kết nối PostgreSQL
### 13.1 Local
```bash
psql -h localhost -U myuser -d mydb
```

### 13.2 Remote
```bash
psql -h <IP_VPS> -U myuser -d mydb -p 5432
```

---

## 14. Thư mục quan trọng
| Thành phần | Đường dẫn |
|---------|----------|
| Config | /etc/postgresql/16/main |
| Data | /var/lib/postgresql/16/main |
| Log | /var/log/postgresql |

---

## 15. Lệnh PostgreSQL thường dùng
```sql
\l      -- Danh sách database
\du     -- Danh sách user
\c db   -- Kết nối database
\dt     -- Danh sách table
```

---

## 16. Checklist triển khai PROD (BẮT BUỘC)

### 16.1 Bảo mật hệ thống
- [ ] Không dùng user `postgres` cho bất kỳ ứng dụng nào
- [ ] Đã đặt mật khẩu mạnh cho user `postgres`
- [ ] Mỗi ứng dụng sử dụng **1 database + 1 user riêng**
- [ ] Không chia sẻ user DB giữa các hệ thống

### 16.2 Network & truy cập
- [ ] Port 5432 **không public 0.0.0.0/0** nếu không cần thiết
- [ ] `pg_hba.conf` chỉ whitelist IP cụ thể (`/32` hoặc subnet cần thiết)
- [ ] Firewall (UFW / Security Group) chỉ mở IP được phép
- [ ] Không expose PostgreSQL trực tiếp ra Internet nếu có thể (ưu tiên kết nối nội bộ)

### 16.3 Cấu hình PostgreSQL
- [ ] `listen_addresses` được cấu hình đúng theo nhu cầu
- [ ] Đã restart **cluster** sau khi chỉnh config
- [ ] Không chỉnh sửa file config khi PostgreSQL đang ghi dữ liệu quan trọng

### 16.4 Vận hành & giám sát
- [ ] Kiểm tra log PostgreSQL định kỳ (`/var/log/postgresql`)
- [ ] Có người chịu trách nhiệm giám sát DB
- [ ] Có quy trình xử lý khi DB down

### 16.5 Backup & khôi phục (RẤT QUAN TRỌNG)
- [ ] Backup database **hằng ngày**
- [ ] Backup lưu **ngoài server** (Google Drive / Object Storage)
- [ ] Backup có **version theo ngày**
- [ ] Đã test **restore backup** ít nhất 1 lần

### 16.6 Chuẩn hoá nội bộ
- [ ] Version PostgreSQL đồng nhất giữa DEV / STAGING / PROD
- [ ] Có tài liệu bàn giao cho nhân sự mới
- [ ] Mọi thay đổi DB phải có log / ghi chú

---

## 17. Ghi chú vận hành nội bộ
- Mọi thay đổi config phải backup trước
- Sau khi chỉnh config → restart cluster
- Log PostgreSQL cần được theo dõi định kỳ

---

**Owner tài liệu:**  
**Ngày cập nhật:**  
**Áp dụng cho server:** DEV / PROD

