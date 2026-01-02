# Linux – Tổng hợp lỗi phổ biến và cách fix

Tài liệu này tổng hợp các **lỗi thường gặp trên Linux (Ubuntu/Debian/CentOS)** và các cách xử lý thực tế, phù hợp làm **document tham khảo nhanh**.

---

## 1. Lỗi liên quan đến quyền (Permission)

### 1.1 Permission denied khi chạy lệnh
Nguyên nhân:
- User hiện tại không có quyền thực thi file hoặc truy cập thư mục

Cách fix:
```bash
sudo chmod +x filename     # Thêm quyền thực thi
sudo chown user:group file  # Thay đổi owner
```

### 1.2 Không ghi được vào thư mục
Nguyên nhân:
- User không có quyền write

Fix:
```bash
sudo chown -R user:group /path/to/dir
sudo chmod -R 755 /path/to/dir
```

---

## 2. Lỗi liên quan đến apt / package manager (Ubuntu/Debian)

### 2.1 Could not get lock /var/lib/dpkg/lock
Nguyên nhân:
- Một tiến trình apt khác đang chạy

Fix:
```bash
sudo killall apt apt-get
sudo rm /var/lib/dpkg/lock-frontend
sudo rm /var/cache/apt/archives/lock
sudo dpkg --configure -a
```

### 2.2 Package not found / Unable to locate package
- Chưa update cache apt
```bash
sudo apt update
```

---

## 3. Lỗi liên quan đến firewall / network

### 3.1 Không thể truy cập server từ bên ngoài
- Kiểm tra firewall
```bash
sudo ufw status
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```
- Kiểm tra service đang lắng nghe
```bash
sudo ss -tulnp
```
- Kiểm tra Security Group / Network ACL nếu là cloud (AWS, OCI, GCP…)

### 3.2 Port đã được sử dụng
```bash
sudo lsof -i :80
sudo kill -9 <PID>
```

---

## 4. Lỗi liên quan đến service

### 4.1 Service không start được
```bash
sudo systemctl status nginx
sudo journalctl -xe
```
- Nguyên nhân: cấu hình sai, port trùng, thiếu quyền
- Fix: chỉnh lại config và test
```bash
sudo nginx -t
sudo systemctl restart nginx
```

### 4.2 Service báo lỗi failed
- Kiểm tra log service và fix theo error log
- Restart service sau khi sửa

---

## 5. Lỗi liên quan đến Node.js / Python / App

### 5.1 Cannot find module (Node.js)
- Nguyên nhân: chưa cài dependencies hoặc path sai
```bash
npm install
node src/server/index.js
```

### 5.2 Module not found (Python)
```bash
pip install <module_name>
```
- Kiểm tra môi trường virtualenv

### 5.3 502 Bad Gateway khi dùng Nginx reverse proxy
- Nguyên nhân: App không chạy, port sai, firewall chặn
- Fix:
  1. Kiểm tra app có chạy không
  2. Kiểm tra port trong proxy_pass
  3. Reload Nginx

---

## 6. Lỗi liên quan đến disk / filesystem

### 6.1 Full disk / no space left
```bash
df -h
du -sh /var/log/*
sudo rm -rf /var/log/*.old
```
- Dọn log cũ, mở rộng volume nếu cloud

### 6.2 Read-only filesystem
- Nguyên nhân: lỗi filesystem, mất điện, hỏng disk
- Fix:
```bash
sudo fsck /dev/sdX
sudo mount -o remount,rw /
```

---

## 7. Lỗi liên quan đến SSH / remote access

### 7.1 Permission denied (publickey)
- Nguyên nhân: key sai hoặc không có trong `~/.ssh/authorized_keys`
- Fix:
```bash
ssh-copy-id user@server
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
```

### 7.2 SSH timeout / connection refused
- Kiểm tra firewall, ssh service
```bash
sudo systemctl status ssh
sudo ufw allow 22/tcp
```

---

## 8. Lỗi liên quan đến systemd / boot

### 8.1 Failed to start service
```bash
sudo systemctl status <service>
sudo journalctl -u <service>
```

### 8.2 Server không boot được
- Kiểm tra initramfs, disk, grub
```bash
sudo update-grub
sudo update-initramfs -u
```

---

## 9. Lỗi phổ biến khác

### 9.1 Command not found
- Nguyên nhân: package chưa cài hoặc path không có
```bash
sudo apt install <package>
echo $PATH
```

### 9.2 Syntax error khi chỉnh file config
- Luôn test trước khi reload
```bash
nginx -t
sudo systemctl reload nginx
```

### 9.3 Lỗi locale / encoding
```bash
sudo locale-gen en_US.UTF-8
sudo update-locale LANG=en_US.UTF-8
```

---

## 10. Tips debug nhanh
- `tail -f /var/log/syslog` hoặc log app
- `systemctl status <service>` + `journalctl -xe`
- `strace <command>` debug process
- `top` / `htop` xem CPU/memory
- `ss -tulnp` kiểm tra port đang listen

---

**Tài liệu này tổng hợp các lỗi phổ biến trên Linux, dùng được cho Ubuntu/Debian/CentOS, làm reference cho admin / dev ops**

