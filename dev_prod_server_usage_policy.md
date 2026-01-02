# 📘 QUY ĐỊNH SỬ DỤNG SERVER DEV & PROD

> Áp dụng cho toàn bộ hệ thống server Linux của công ty  
> Mục tiêu: **An toàn – Rõ trách nhiệm – Không phá nhầm – Dễ vận hành**

---

## 1️⃣ Mục đích phân tách DEV & PROD

| Server | Mục đích | Mức độ an toàn |
|------|--------|---------------|
| **DEV** | Phát triển, test, demo | Linh hoạt, cho phép thử nghiệm |
| **PROD** | Chạy hệ thống thật | An toàn tuyệt đối, hạn chế thao tác |

> ❗ **CẤM deploy trực tiếp code thử nghiệm lên PROD**

---

## 2️⃣ Quy định chung (BẮT BUỘC)

- ❌ KHÔNG login root qua SSH
- ❌ KHÔNG làm việc thường ngày bằng user cứu hộ
- ❌ KHÔNG build project trên PROD
- ✅ Mỗi user có mục đích rõ ràng
- ✅ Mọi thao tác hệ thống phải có log

---

## 3️⃣ Quy định USER THEO SERVER

---

## 🔵 SERVER DEV

### 3.1 Danh sách user DEV

| User | Quyền | Mục đích |
|----|------|---------|
| `ubuntu` | sudo full | Cứu hộ, setup ban đầu |
| `dev` | sudo full | Build, test, debug |
| `app` | không sudo | Chạy service |

---

### 3.2 Quy định sử dụng user DEV

#### ✅ User `dev`

ĐƯỢC PHÉP:
- Login SSH
- Pull code
- Build project
- Restart service
- Sửa cấu hình DEV

KHÔNG NÊN:
- Thao tác trên data thật
- Can thiệp hệ thống PROD

---

#### ⚠️ User `ubuntu`

- Chỉ dùng khi `dev` lỗi
- Không dùng cho công việc hằng ngày

---

#### 🧱 User `app`

- Chỉ chạy backend
- Không SSH
- Không sudo

---

### 3.3 Build & Start project trên DEV

#### 🔧 Build

```bash
su - dev
cd /home/dev/project
./build.sh
```

Hoặc:
```bash
mvn clean package
npm run build
```

---

#### ▶️ Start

- Start qua systemd
- Service chạy bằng user `app`

```bash
sudo systemctl restart myapp-dev
```

---

## 🔴 SERVER PROD

### 4.1 Danh sách user PROD

| User | Quyền | Mục đích |
|----|------|---------|
| `ubuntu` | sudo full | CỨU HỘ – KHÔNG DÙNG HẰNG NGÀY |
| `deploy` | sudo giới hạn | Deploy & restart |
| `app` | không sudo | Chạy backend |

---

### 4.2 Quy định sử dụng user PROD

#### 🚀 User `deploy`

ĐƯỢC PHÉP:
- Login SSH
- Pull artifact đã build
- Restart service
- Reload nginx

❌ CẤM:
- Build project
- Sửa sudoers
- Xoá user
- Format disk

---

#### ⚠️ User `ubuntu`

- Chỉ dùng khi sự cố nghiêm trọng
- Phải có lý do rõ ràng

---

#### 🧱 User `app`

- Chỉ chạy service
- Không SSH
- Không sudo

---

## 5️⃣ Quy định BUILD & DEPLOY

### ❌ TUYỆT ĐỐI CẤM trên PROD

- `mvn package`
- `npm run build`
- Build Docker image
- Sửa code trực tiếp

---

### ✅ Quy trình chuẩn

1. Build tại **DEV hoặc CI/CD**
2. Tạo artifact (`.jar`, `.zip`, image)
3. Upload sang PROD
4. Login user `deploy`
5. Restart service

```bash
sudo systemctl restart myapp
```

---

## 6️⃣ Start / Stop project (CẢ DEV & PROD)

- BẮT BUỘC qua `systemd`
- CẤM chạy tay bằng `java -jar`

```bash
sudo systemctl status myapp
sudo systemctl restart myapp
```

---

## 7️⃣ Trách nhiệm & kiểm soát

| Hành động | User chịu trách nhiệm |
|-------|--------------------|
| Build | dev / CI |
| Deploy | deploy |
| Cấu hình hệ thống | ubuntu |
| Runtime app | app |

---

## 8️⃣ Checklist trước khi deploy PROD

✔ Build từ DEV / CI  
✔ Artifact đúng version  
✔ Backup đã sẵn sàng  
✔ Deploy bằng user `deploy`  
✔ Không SSH user `app`  

---

## 9️⃣ Quy tắc vàng

> 🔥 **PROD = CHỈ DEPLOY – KHÔNG BUILD**  
> 🔥 **APP KHÔNG BAO GIỜ CÓ SUDO**  
> 🔥 **USER CỨU HỘ KHÔNG DÙNG HẰNG NGÀY**

---

📌 *Tài liệu này là quy định chính thức khi sử dụng server DEV & PROD trong công ty.*

