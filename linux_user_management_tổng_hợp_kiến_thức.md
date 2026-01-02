# Tổng hợp kiến thức về User trong Linux

Tài liệu này dùng để làm **document nội bộ**, tổng hợp các kiến thức quan trọng liên quan đến **user, group, quyền hạn và thực hành chuẩn** khi làm việc với Linux (đặc biệt hữu ích cho server / VM).

---

## 1. Khái niệm cơ bản

### 1.1 User là gì?
- **User** là tài khoản dùng để đăng nhập và thực thi lệnh trên Linux
- Mỗi user có:
  - Username
  - UID (User ID)
  - Group chính (Primary Group)
  - Home directory
  - Shell mặc định

### 1.2 Phân loại user
| Loại user | UID | Mô tả |
|---------|-----|------|
| root | 0 | Toàn quyền hệ thống |
| system user | 1–999 | User cho service (nginx, mysql, www-data…) |
| normal user | >=1000 | User đăng nhập thông thường |

---

## 2. Các file hệ thống liên quan đến user

### 2.1 /etc/passwd
Chứa thông tin user (KHÔNG chứa mật khẩu)
```bash
cat /etc/passwd
```
Format:
```
username:x:UID:GID:comment:home:shell
```

### 2.2 /etc/shadow
Chứa mật khẩu đã hash (chỉ root đọc được)
```bash
sudo cat /etc/shadow
```

### 2.3 /etc/group
Chứa thông tin group
```bash
cat /etc/group
```

---

## 3. Tạo, xóa, chỉnh sửa user

### 3.1 Tạo user
```bash
sudo adduser username
```
Hoặc:
```bash
sudo useradd username
```
> Khuyến nghị dùng **adduser** vì thân thiện hơn

### 3.2 Xóa user
```bash
sudo deluser username
```
Xóa cả home:
```bash
sudo deluser --remove-home username
```

### 3.3 Chỉnh sửa user
```bash
sudo usermod [option] username
```
Ví dụ:
```bash
sudo usermod -s /bin/bash username
sudo usermod -d /new/home username
```

---

## 4. Group và phân quyền

### 4.1 Group là gì?
- Group dùng để gom user và phân quyền truy cập file/thư mục

### 4.2 Tạo group
```bash
sudo groupadd groupname
```

### 4.3 Thêm user vào group
```bash
sudo usermod -aG groupname username
```
Kiểm tra:
```bash
groups username
```

### 4.4 Xóa user khỏi group
```bash
sudo gpasswd -d username groupname
```

---

## 5. Quyền file & thư mục (Permission)

### 5.1 Các loại quyền
| Ký hiệu | Ý nghĩa |
|------|--------|
| r | read |
| w | write |
| x | execute |

### 5.2 Cấu trúc permission
```bash
ls -l
```
Ví dụ:
```
drwxr-xr--
```
| Đối tượng | Quyền |
|--------|-------|
| Owner | rwx |
| Group | r-x |
| Other | r-- |

### 5.3 Thay đổi quyền
```bash
chmod 755 file
chmod -R 755 folder
```

### 5.4 Thay đổi owner / group
```bash
sudo chown user:group file
sudo chown -R user:group folder
```

---

## 6. Sudo và quyền admin

### 6.1 Sudo là gì?
- Cho phép user thường chạy lệnh với quyền root

### 6.2 Thêm user vào sudo
```bash
sudo usermod -aG sudo username
```

### 6.3 Kiểm tra quyền sudo
```bash
sudo -l
```

---

## 7. Chuyển đổi user

### 7.1 Chuyển sang user khác
```bash
su - username
```

### 7.2 Chuyển sang root
```bash
sudo -i
```
Hoặc:
```bash
su -
```

---

## 8. Quản lý session & đăng nhập

### 8.1 Xem user đang đăng nhập
```bash
who
w
```

### 8.2 Xem lịch sử đăng nhập
```bash
last
```

---

## 9. Thực hành chuẩn khi làm việc trên server

✅ **KHÔNG làm việc trực tiếp bằng root**

✅ Mỗi người / mỗi service dùng **user riêng**

✅ Source code nên thuộc về user build/run
```bash
/home/appuser/project
```

✅ Nginx chỉ làm reverse proxy (www-data)

✅ Node / Java / App chạy bằng user riêng

---

## 10. Ví dụ mô hình user chuẩn cho server

| User | Mục đích |
|----|--------|
| root | Quản trị hệ thống |
| ubuntu | User mặc định VM |
| appuser | Build & run app |
| www-data | Nginx |

Cấu trúc thư mục:
```text
/home/appuser
 ├── backend
 ├── frontend
 └── logs
```

---

## 11. Lệnh kiểm tra nhanh (cheat sheet)

```bash
id username
groups username
whoami
ls -l
stat file
```

---

## 12. Gợi ý mở rộng
- ACL (`setfacl`, `getfacl`)
- SELinux / AppArmor
- SSH key theo user
- Systemd service chạy theo user

---

**Tài liệu dùng cho học tập & triển khai thực tế trên Linux Server**

