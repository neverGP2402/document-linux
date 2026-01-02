# 01. Setup VM & cài thư viện cần thiết

## Mục tiêu

- Chuẩn bị VM Linux (Ubuntu)
- Cài các thư viện cơ bản để chạy web
- Cài Node.js, npm và các package phổ biến (express, pm2)

---

## 1. Update hệ thống

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 2. Cài các package nền tảng

```bash
sudo apt install -y \
  curl \
  git \
  unzip \
  vim \
  build-essential \
  ca-certificates
```

---

## 3. Cài Node.js (khuyến nghị dùng NodeSource)

### Cài Node 18 LTS (ổn định)

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
```

Kiểm tra:

```bash
node -v
npm -v
```

---

## 4. Cài các thư viện Node dùng chung (global)

### Express (dùng cho backend đơn giản)

```bash
npm install -g express-generator
```

### PM2 (quản lý process Node)

```bash
npm install -g pm2
```

Kiểm tra:

```bash
pm2 -v
```

---

## 5. Cấu trúc thư mục gợi ý cho mỗi user

### 5.1 Đối với 1 user chứa cả App từ Front-end -> Back-end -> Gateway

```
/home/<user>/
  ├── app-backend/
  ├── app-frontend/
  │     └── build/ (hoặc dist)
  └── logs/
```

### 5.2 Đối với 1 user chỉ chứa và chịu 1 trách nhiệm trong 1 App, như Back-end hoặc Front-end hoặc Gateway,...

```
/home/<user>_<module>/
  ├── src/
  │     └── ... (chứa source chính)
  ├── start.sh (dùng để start module)
  ├── stop.sh (dùng để stop module)
  ├── restart.sh (dùng để restart module)
  └── logs/
```

---

## 6. Ghi chú quan trọng

- KHÔNG build bằng root
- Mỗi user tự quản lý project của mình
- Nginx chỉ đọc file, không chạy Node
