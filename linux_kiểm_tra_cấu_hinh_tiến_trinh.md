# Linux – Các lệnh kiểm tra cấu hình tiến trình và hệ thống

Tài liệu này tổng hợp các **lệnh kiểm tra tiến trình, resource, network và thông tin hệ thống** trên Linux, giúp quản trị viên hoặc developer dễ tham khảo.

---

## 1. Xem tiến trình (Process)

### 1.1 `ps`
```bash
ps aux                 # Xem tất cả process chi tiết
ps -ef                 # Xem process theo chuẩn hệ thống
ps -u username         # Xem process của user cụ thể
```

### 1.2 `top` / `htop`
```bash
top                    # Xem realtime CPU, RAM, process
htop                   # Giao diện đẹp, hỗ trợ chọn kill, sort, cần cài thêm
```
- `M` trong top: sắp xếp theo Memory
- `P` trong top: sắp xếp theo CPU

### 1.3 `pgrep` / `pidof`
```bash
pgrep nginx            # tìm PID của nginx
pidof node             # PID của Node.js
```

### 1.4 `jobs`, `fg`, `bg`
- Quản lý tiến trình background trong shell
```bash
jobs                   # xem job đang chạy
fg %1                  # đưa job 1 lên foreground
bg %1                  # chạy job 1 background
```

---

## 2. Kiểm tra resource / hiệu năng

### 2.1 CPU / RAM
```bash
uptime                  # load average
free -h                 # RAM
vmstat 1                # memory, swap, CPU, I/O
top                     # realtime CPU/memory usage
htop                    # giao diện đẹp
```

### 2.2 Disk
```bash
df -h                   # dung lượng disk, filesystem
du -sh /path             # dung lượng thư mục
iostat                  # I/O disk
```

### 2.3 Network
```bash
ss -tulnp               # các port đang listen, PID
netstat -tulnp          # tương tự, cũ hơn
lsof -i :3000           # tiến trình đang dùng port 3000
```

---

## 3. Kiểm tra cấu hình tiến trình

### 3.1 `ulimit` – Giới hạn resource
```bash
ulimit -a               # xem tất cả giới hạn
ulimit -n               # số file descriptor tối đa
ulimit -u               # số process tối đa của user
```

### 3.2 `/proc` filesystem
- Thông tin tiến trình:
```bash
cat /proc/<PID>/status          # chi tiết tiến trình
cat /proc/<PID>/cmdline         # lệnh chạy tiến trình
cat /proc/<PID>/limits          # resource limit của tiến trình
cat /proc/cpuinfo               # thông tin CPU
cat /proc/meminfo               # thông tin RAM
```

### 3.3 `systemctl` – kiểm tra service
```bash
systemctl status nginx           # trạng thái service
systemctl list-units --type=service  # tất cả service
```

### 3.4 `journalctl` – log của service
```bash
journalctl -u nginx              # log của nginx
journalctl -xe                   # log lỗi system
journalctl -f                    # theo dõi realtime
```

---

## 4. Debug tiến trình

### 4.1 `strace`
```bash
strace -p <PID>                  # theo dõi system call
strace -c -p <PID>               # thống kê số lượng call
```

### 4.2 `lsof`
```bash
lsof -p <PID>                    # các file, socket mà process mở
lsof -i :3000                     # process dùng port 3000
```

### 4.3 `perf` (nâng cao)
- Dùng để **profiling CPU, memory, I/O**
```bash
sudo perf top
sudo perf record -p <PID>
sudo perf report
```

---

## 5. Các lệnh hỗ trợ khác

| Lệnh                | Chức năng                                           |
|--------------------|----------------------------------------------------|
| `watch`             | Theo dõi lệnh chạy định kỳ                        |
| `uptime`            | Load average, số user, thời gian hệ thống         |
| `vmstat 1`          | CPU, memory, I/O                                   |
| `free -h`           | Memory usage                                       |
| `iostat`            | Disk I/O                                           |
| `ss -tulnp`         | Port đang listen & PID                             |
| `netstat -tulnp`    | Port đang listen (cũ)                              |
| `lsof -i`           | Liệt kê các socket / network của tiến trình       |

---

**Tài liệu này dùng làm reference nhanh để kiểm tra cấu hình tiến trình, tài nguyên và debug tiến trình trên Linux, từ cơ bản đến nâng cao cho admin và developer**.

