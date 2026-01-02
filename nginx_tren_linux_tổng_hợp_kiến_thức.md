# NGINX trên Linux – Tài liệu tổng hợp

Tài liệu này tổng hợp các kiến thức **thực tế – dễ tra cứu – dùng được ngay** về Nginx trên Linux (Ubuntu/Debian), phù hợp để làm **document nội bộ** hoặc học tập.

---

## 1. Tổng quan về Nginx

### 1.1 Nginx là gì?
- Nginx là **web server** hiệu năng cao
- Có thể dùng làm:
  - Web server (phục vụ HTML, CSS, JS)
  - Reverse proxy
  - Load balancer
  - HTTP cache

### 1.2 So sánh nhanh
| Thành phần | Nginx | Apache |
|---------|-------|--------|
| Kiến trúc | Event-driven | Process/Thread |
| Hiệu năng | Rất cao | Trung bình |
| Phù hợp | API, SPA, Microservice | PHP truyền thống |

---

## 2. Cài đặt Nginx

### 2.1 Cài đặt trên Ubuntu/Debian
```bash
sudo apt update
sudo apt install -y nginx
```

### 2.2 Kiểm tra trạng thái
```bash
systemctl status nginx
```

### 2.3 Các lệnh thường dùng
```bash
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl enable nginx
```

---

## 3. Cấu trúc thư mục quan trọng

```text
/etc/nginx/
├── nginx.conf          # File cấu hình chính
├── sites-available/    # Khai báo virtual host
├── sites-enabled/      # Các site đang bật (symlink)
├── conf.d/             # Config mở rộng
├── mime.types
└── snippets/
```

### 3.1 Nguyên tắc hoạt động
- `nginx.conf` → load các file con
- `sites-enabled/*` → **mới thực sự được dùng**

---

## 4. Virtual Host (Server Block)

### 4.1 Tạo website mới
```bash
sudo nano /etc/nginx/sites-available/example.conf
```

### 4.2 Cấu hình web tĩnh
```nginx
server {
    listen 80;
    server_name example.com;

    root /var/www/example;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### 4.3 Kích hoạt site
```bash
sudo ln -s /etc/nginx/sites-available/example.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## 5. Reverse Proxy (Node.js / API)

### 5.1 Mô hình
```
Client → Nginx (80/443) → Node.js (3000)
```

### 5.2 Cấu hình reverse proxy
```nginx
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

---

## 6. Phục vụ Frontend (React / Vue / Angular)

### 6.1 Sau khi build
```bash
npm run build
```

### 6.2 Cấu hình SPA (rất quan trọng)
```nginx
server {
    listen 80;
    server_name app.example.com;

    root /home/appuser/frontend/build;
    index index.html;

    location / {
        try_files $uri /index.html;
    }
}
```

👉 **Nếu thiếu `try_files` → refresh F5 sẽ 404**

---

## 7. SSL / HTTPS (Let's Encrypt)

### 7.1 Cài Certbot
```bash
sudo apt install -y certbot python3-certbot-nginx
```

### 7.2 Cấp SSL
```bash
sudo certbot --nginx -d example.com
```

### 7.3 Tự động gia hạn
```bash
sudo certbot renew --dry-run
```

---

## 8. Log & Debug

### 8.1 Log mặc định
```text
/var/log/nginx/access.log
/var/log/nginx/error.log
```

### 8.2 Xem realtime
```bash
tail -f /var/log/nginx/error.log
```

### 8.3 Test cấu hình
```bash
sudo nginx -t
```

---

## 9. Các lỗi thường gặp & cách fix

### 9.1 Bên trong server truy cập được, bên ngoài không
Checklist:
- ❌ Firewall chưa mở port 80/443
- ❌ Security List / Network rule chưa allow
- ❌ Nginx chỉ listen `127.0.0.1`

Fix:
```nginx
listen 0.0.0.0:80;
```

---

### 9.2 403 Forbidden
Nguyên nhân:
- Sai quyền thư mục
- User nginx không có quyền đọc

Fix:
```bash
sudo chown -R www-data:www-data /var/www/example
sudo chmod -R 755 /var/www/example
```

---

### 9.3 404 khi reload SPA
👉 Thiếu:
```nginx
try_files $uri /index.html;
```

---

## 10. Best Practices

- Mỗi website → **1 file config riêng**
- Không sửa trực tiếp `default`
- Luôn `nginx -t` trước reload
- Dùng reverse proxy thay vì expose port app
- Log rõ ràng để debug

---

## 11. Ghi chú thực tế (Production)

- Node/PM2 chạy bằng user riêng
- Nginx chạy port 80/443
- Không chạy app bằng root
- Backup config trước khi sửa

---

## 12. Cheat Sheet nhanh

```bash
nginx -t
systemctl reload nginx
systemctl restart nginx
tail -f /var/log/nginx/error.log
```

---

**Tài liệu dùng cho Linux / Ubuntu / Server thực tế**

