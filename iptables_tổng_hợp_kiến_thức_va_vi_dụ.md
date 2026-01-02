# Linux – iptables và câu lệnh `sudo iptables -I INPUT 1 -p tcp --dport 3000 -j ACCEPT`

Tài liệu này tổng hợp kiến thức về **iptables trên Linux**, đặc biệt tập trung vào câu lệnh để mở port hoặc cho phép traffic TCP đến server.

---

## 1. Tổng quan về iptables

- `iptables` là **firewall mặc định trên Linux**.
- Dùng để **lọc gói tin**, **mở port**, **chặn IP**, **quản lý network traffic**.
- Hoạt động dựa trên **chains** và **tables**:
  - **Tables**:
    - `filter` (mặc định, kiểm soát packet)
    - `nat` (Network Address Translation)
    - `mangle` (thay đổi packet header)
  - **Chains**:
    - `INPUT` – gói tin vào server
    - `OUTPUT` – gói tin đi ra
    - `FORWARD` – gói tin đi qua (router)

---

## 2. Cấu trúc câu lệnh cơ bản

```bash
sudo iptables [chain] [action] [options]
```

### 2.1 Các thành phần quan trọng
- `-I INPUT 1` → Insert rule vào **đầu chain INPUT**, vị trí thứ 1
- `-p tcp` → Chỉ áp dụng cho **giao thức TCP**
- `--dport 3000` → Chỉ áp dụng cho **cổng đích 3000**
- `-j ACCEPT` → Hành động **chấp nhận packet này**

### 2.2 Câu lệnh ví dụ
```bash
sudo iptables -I INPUT 1 -p tcp --dport 3000 -j ACCEPT
```
- Mở port **3000** cho tất cả kết nối TCP đến server
- Rule được thêm **vào đầu chain INPUT**, ưu tiên cao

---

## 3. Xem và quản lý rules

### 3.1 Xem tất cả rules
```bash
sudo iptables -L -v -n --line-numbers
```
- `-L` → Liệt kê rules
- `-v` → Hiển thị chi tiết (bytes, packets)
- `-n` → Không resolve IP/port (nhanh hơn)
- `--line-numbers` → Số thứ tự rule

### 3.2 Xóa rule
```bash
sudo iptables -D INPUT 1   # Xóa rule thứ 1 trong chain INPUT
```

### 3.3 Lưu rules để reboot không mất
```bash
sudo apt install iptables-persistent
sudo netfilter-persistent save
```
- Trên CentOS/RHEL: `service iptables save`

---

## 4. Các ví dụ phổ biến

### 4.1 Mở port 80 và 443
```bash
sudo iptables -I INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -I INPUT -p tcp --dport 443 -j ACCEPT
```

### 4.2 Chặn IP cụ thể
```bash
sudo iptables -A INPUT -s 1.2.3.4 -j DROP
```

### 4.3 Cho phép traffic từ subnet
```bash
sudo iptables -A INPUT -s 192.168.1.0/24 -j ACCEPT
```

### 4.4 Chặn tất cả còn lại (default policy)
```bash
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT
```

---

## 5. Lưu ý quan trọng

- **Thứ tự rules quan trọng** – rule đầu tiên khớp sẽ dừng kiểm tra
- **Kiểm tra trước khi save** – tránh khóa mình ra ngoài server
- **Dùng `iptables -I` để ưu tiên, `-A` để append cuối**
- **Sử dụng firewall khác (UFW, firewalld)** có thể ảnh hưởng đến iptables
- **Test port trước khi deploy**: `nc -zv <server> 3000`

---

## 6. Tips debug nhanh
- `sudo iptables -L -v -n --line-numbers` → xem rules hiện tại
- `sudo iptables -F` → xóa tất cả rules (cẩn thận!)
- `sudo iptables -D INPUT <num>` → xóa rule theo số
- `sudo iptables -P INPUT ACCEPT` → restore tạm thời tránh bị khóa

---

**Tài liệu này dùng làm reference nhanh cho việc quản lý port TCP, firewall và iptables trên Linux**.

