# Linux – Quản lý Process

Tài liệu này tổng hợp các kiến thức về **process trong Linux**, bao gồm cách xem, quản lý, kill, và debug process.

---

## 1. Tổng quan về Process

- Process là **một chương trình đang chạy** trên hệ thống.
- Mỗi process có:
  - PID (Process ID)
  - PPID (Parent PID)
  - UID/GID (User/Group sở hữu)
  - Trạng thái (R: running, S: sleeping, Z: zombie...)

- Process có thể là:
  - Foreground (chạy trong terminal)
  - Background (chạy tách terminal)

---

## 2. Xem process

### 2.1 ps
```bash
ps aux
```
- `a` – hiện tất cả user
- `u` – show user, CPU, MEM
- `x` – show process không có terminal

### 2.2 top / htop
```bash
top
htop   # cần cài thêm
```
- Hiển thị realtime CPU, memory, process
- `htop` có giao diện đẹp, dễ kill, filter process

### 2.3 pgrep / pidof
```bash
pgrep node        # tìm PID của node
pidof nginx       # PID của nginx
```

### 2.4 watch ps / top
```bash
watch -n 1 'ps aux | grep node'
```
- Theo dõi process liên tục

---

## 3. Quản lý process

### 3.1 Kill process
```bash
kill <PID>          # gửi SIGTERM (mềm)
kill -9 <PID>       # gửi SIGKILL (force)
pkill node           # kill theo tên process
killall node         # kill tất cả process node
```

### 3.2 Start process trong background
```bash
node server.js &
```
- `&` chạy background
- `jobs` xem job đang chạy
- `fg %1` đưa job về foreground
- `bg %1` chạy job background

### 3.3 nohup / disown
```bash
nohup node server.js > output.log 2>&1 &
disown
```
- Giúp process chạy **không bị kill khi logout**

### 3.4 systemd service
- Tạo file `/etc/systemd/system/myapp.service`
```ini
[Unit]
Description=My Node App
After=network.target

[Service]
User=appuser
WorkingDirectory=/home/appuser/app
ExecStart=/usr/bin/node server.js
Restart=always

[Install]
WantedBy=multi-user.target
```
- Enable & start
```bash
sudo systemctl daemon-reload
sudo systemctl enable myapp
sudo systemctl start myapp
sudo systemctl status myapp
```

---

## 4. Debug process

### 4.1 strace
```bash
strace -p <PID>
```
- Theo dõi system call, open/read/write

### 4.2 lsof
```bash
lsof -p <PID>
```
- Liệt kê files, network mà process mở

### 4.3 netstat / ss
```bash
sudo ss -tulnp | grep <PID>
```
- Kiểm tra port mà process đang listen

---

## 5. Trạng thái process

- R – Running / runnable
- S – Sleeping (idle)
- D – Uninterruptible sleep (disk I/O)
- Z – Zombie (parent chưa thu hồi)
- T – Stopped (suspended)

Xem trạng thái:
```bash
ps -o pid,stat,cmd -p <PID>
```

---

## 6. Priorities & nice

- `nice` – set priority khi chạy process (mặc định 0, -20 cao nhất, 19 thấp nhất)
- `renice` – thay đổi priority process đang chạy

```bash
nice -n 10 node server.js
sudo renice -n 5 -p <PID>
```

---

## 7. Monitoring & performance

- `top` / `htop` – CPU / Memory usage
- `vmstat 1` – xem memory & swap
- `iostat` – xem I/O disk
- `free -h` – xem RAM
- `uptime` – load average

---

**Tài liệu này dùng làm reference nhanh cho việc quản lý process trên Linux, phù hợp cho admin, dev ops, hoặc developer quản lý app server**.

