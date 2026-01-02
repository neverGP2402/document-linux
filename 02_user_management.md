# 02. Quản lý user trên VM

## Mục tiêu

- Mỗi user độc lập
- Mỗi user tự build & start web của họ
- Tránh đụng file & quyền

---

## 1. Tạo user mới

```bash
sudo adduser duc
```

Thêm quyền sudo (hạn chế cấp quyền sudo cho user thường, rất nguy hiểm):

```bash
sudo usermod -aG sudo duc
```

---

## 2. Chuyển sang user

```bash
su - duc
```

---

## 3. Cấu trúc thư mục cho user (thường khi tạo user sẽ có, nên bình thường sẽ không cần tạo)

```bash
mkdir -p ~/my-web
cd ~/my-web
```

---

## 4. Quyền truy cập cho Nginx (RẤT QUAN TRỌNG)

### Vấn đề thường gặp

- Nginx chạy bằng user: `www-data`
- `/home/<user>` mặc định **không cho user khác truy cập**

### Cách 1: Nhanh (dễ test)

```bash
sudo chmod o+x /home/duc
sudo chmod -R o+r /home/duc/my-web/build
```

### Cách 2: Chuẩn hơn (khuyến nghị)

```bash
sudo chown -R duc:www-data /home/duc/my-web/build
sudo chmod -R 750 /home/duc/my-web/build
```

---

## 5. Nhiều user thì sao?

### Mỗi user có:

- Thư mục riêng trong `/home`
- Build riêng
- Nginx server block riêng

Ví dụ:

```
/home/duc/my-web/build
/home/user1/app/dist
```

---

## 6. Những lỗi hay gặp

❌ 403 Forbidden  
👉 Thiếu quyền execute (`x`) trên `/home/<user>`

❌ Nginx đọc được thư mục nhưng không đọc được file  
👉 Thiếu quyền read (`r`) trong build
