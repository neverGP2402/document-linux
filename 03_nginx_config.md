# 03. Cấu hình Nginx & fix lỗi không truy cập được từ bên ngoài

## Mục tiêu
- Public frontend build từ home user
- Fix lỗi: test OK trong VM nhưng bên ngoài không vào được

---

## 1. Cài Nginx
```bash
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

Kiểm tra:
```bash
systemctl status nginx
```

---

## 2. Tạo server block cho user

Ví dụ user `duc`:

```bash
sudo nano /etc/nginx/sites-available/duc
```

Nội dung:
```nginx
server {
    listen 80;
    server_name _;

    root /home/duc/my-web/build;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

---

## 3. Enable site
```bash
sudo ln -s /etc/nginx/sites-available/duc /etc/nginx/sites-enabled/
```

Nếu không dùng default:
```bash
sudo rm /etc/nginx/sites-enabled/default
```

---

## 4. Test & reload Nginx
```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## 5. Fix lỗi TEST OK nhưng BÊN NGOÀI KHÔNG VÀO ĐƯỢC

### 5.1. Chưa mở port 80 trên VM
```bash
sudo ss -tulpn | grep :80
```

Nếu không thấy nginx listen → config sai

---

### 5.2. Firewall (UFW)
```bash
sudo ufw status
sudo ufw allow 80
sudo ufw reload
```

---

### 5.3. OCI / Cloud Security List (RẤT HAY GẶP)

Cần Ingress Rule:
- Protocol: TCP
- Port: 80
- Source: 0.0.0.0/0

👉 Nếu không mở, chỉ curl được trong máy

---

### 5.4. SELinux (nếu có)
```bash
getenforce
```

Nếu `Enforcing`:
```bash
sudo setenforce 0
```

---

## 6. Các lỗi phổ biến khác

❌ 403 Forbidden  
👉 Sai quyền thư mục `/home/<user>`

❌ 404 khi refresh React/Vue  
👉 Thiếu `try_files`

❌ Truy cập bằng IP nhưng server_name sai  
👉 Dùng `server_name _;`

---

## 7. Mở rộng
- Mỗi user 1 domain / subdomain
- HTTPS với Let's Encrypt
- Reverse proxy backend

