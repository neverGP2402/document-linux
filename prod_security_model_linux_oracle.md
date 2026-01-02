# 🛡️ MÔ HÌNH BẢO MẬT PROD CHUẨN CÔNG TY (LINUX + ORACLE CLOUD)

> Áp dụng cho: **Oracle Cloud / AWS / GCP / VPS Linux**  
> Đối tượng: **Dev, Backend, DevOps, Sysadmin**  
> Mục tiêu: **Không mất quyền server – Không phá nhầm PROD – Dễ audit – Dễ mở rộng**

---

## 1️⃣ Nguyên tắc cốt lõi (BẮT BUỘC TUÂN THỦ)

- ❌ **KHÔNG login root qua SSH**
- ❌ **KHÔNG làm việc hằng ngày bằng user có full sudo**
- ✅ **Phân tầng user rõ ràng**
- ✅ **Quyền cao nhất nằm ở Cloud Console, không nằm trong OS**
- ✅ **Mọi hành động nguy hiểm phải có log**

---

## 2️⃣ Phân tầng quyền chuẩn (QUAN TRỌNG NHẤT)

### 🟦 TẦNG 0 – CLOUD PROVIDER (QUYỀN CAO NHẤT)

- Oracle / AWS / GCP Console
- Có quyền:
  - Stop / Start VM
  - Detach / Attach disk
  - Snapshot / Backup
  - Offline recovery OS

> ⚠️ **Account cloud phải bật MFA**

---

### 🟥 TẦNG 1 – USER CỨU HỘ (EMERGENCY ONLY)

```
ubuntu
```

- Có **sudo full**
- SSH key riêng
- ❌ Không deploy
- ❌ Không code
- ❌ Không dùng hằng ngày

> Chỉ dùng khi **sự cố nghiêm trọng**

---

### 🟨 TẦNG 2 – USER DEPLOY (VẬN HÀNH CHÍNH)

```
deploy
```

- Có **sudo GIỚI HẠN**
- Deploy code
- Restart service
- Reload nginx

❌ KHÔNG được:
- Xoá user
- Sửa sudoers
- Thao tác disk

---

### 🟩 TẦNG 3 – USER CHẠY ỨNG DỤNG

```
app
```

- ❌ Không sudo
- ❌ Không SSH
- Chỉ chạy service

> 👉 App bị hack **KHÔNG chiếm được server**

---

## 3️⃣ Sơ đồ tổng thể

```
Cloud Console (MFA)
        ↓
     ubuntu (sudo full)   ← cứu hộ
        ↓
     deploy (sudo limited)
        ↓
     app (no sudo, no ssh)
```

---

## 4️⃣ Quy trình triển khai CHUẨN

### 4.1 Tạo user

```bash
sudo adduser deploy
sudo adduser app
```

```bash
sudo usermod -aG sudo deploy
```

---

### 4.2 Giới hạn sudo cho `deploy`

```bash
sudo visudo
```

Thêm cuối file:

```ini
deploy ALL=(ALL) NOPASSWD: \
/bin/systemctl restart myapp, \
/bin/systemctl status myapp, \
/bin/systemctl reload nginx, \
/bin/systemctl restart nginx
```

---

## 5️⃣ Chạy backend bằng user `app`

### Ví dụ systemd service

```ini
[Unit]
Description=My Backend Service
After=network.target

[Service]
User=app
Group=app
ExecStart=/usr/bin/java -jar /opt/myapp/app.jar
Restart=always

[Install]
WantedBy=multi-user.target
```

---

## 6️⃣ SSH HARDENING (BẮT BUỘC)

### `/etc/ssh/sshd_config`

```ini
PermitRootLogin no
PasswordAuthentication no

AllowUsers ubuntu deploy
```

```bash
sudo systemctl reload sshd
```

---

## 7️⃣ Firewall (UFW)

```bash
sudo ufw allow OpenSSH
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```

❌ Không mở port DB  
❌ Không expose port backend

---

## 8️⃣ Logging & Audit

- Sudo log: `/var/log/auth.log`
- Service log: `journalctl -u myapp`
- Nginx log: `/var/log/nginx/`

---

## 9️⃣ Backup & Recovery

### Cloud
- Boot Volume Backup (auto)
- Snapshot định kỳ

### OS
- `/etc`
- `/home/deploy`
- `/opt/myapp`

---

## 🔟 Kịch bản rủi ro & cách xử lý

### ❓ Deploy bị chiếm sudo?

✅ Cloud Console → Stop VM → Detach disk → Gắn VM khác → Sửa `/etc/sudoers`

---

### ❓ App bị hack?

✅ Chỉ user `app` bị ảnh hưởng  
❌ Không chiếm được server

---

## 1️⃣1️⃣ Checklist PROD an toàn

✔ Root SSH disabled  
✔ Password login disabled  
✔ Có user cứu hộ  
✔ Deploy sudo giới hạn  
✔ App không sudo  
✔ MFA Cloud Console  

---

## 1️⃣2️⃣ KẾT LUẬN (QUY TẮC VÀNG)

> 🔥 **KHÔNG BAO GIỜ giao full sudo cho user làm việc hằng ngày**  
> 🔥 **QUYỀN CAO NHẤT LUÔN NẰM Ở CLOUD CONSOLE**

---

📌 *Tài liệu này được dùng làm chuẩn triển khai PROD cho toàn công ty.*

