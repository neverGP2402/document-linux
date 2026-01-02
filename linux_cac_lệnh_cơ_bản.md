# Linux – Các lệnh cơ bản

Tài liệu này tổng hợp các lệnh cơ bản trên Linux, phù hợp làm **reference nhanh** cho dev, sysadmin hoặc học tập.

---

## 1. `ls` – Liệt kê file / thư mục
```bash
ls               # liệt kê file trong thư mục hiện tại
ls -l            # liệt kê chi tiết (permissions, owner, size, date)
ls -a            # hiển thị file ẩn
ls -lh           # hiển thị chi tiết, size dễ đọc
```

## 2. `cd` – Thay đổi thư mục
```bash
cd /path/to/folder   # chuyển đến thư mục
cd ..                # quay về thư mục cha
cd ~                 # về home directory
```

## 3. `pwd` – Hiển thị thư mục hiện tại
```bash
pwd
```

## 4. `cat` – Xem hoặc nối file
```bash
cat file.txt             # xem nội dung file
cat file1.txt file2.txt > all.txt   # nối file
```

## 5. `tail` – Xem dòng cuối file
```bash
tail file.log            # xem 10 dòng cuối
tail -n 50 file.log      # xem 50 dòng cuối
tail -f file.log         # theo dõi realtime
```

## 6. `echo` – In ra text hoặc biến
```bash
echo "Hello World"           # in text ra màn hình
echo $HOME                  # in giá trị biến môi trường
echo "text" >> file.txt     # ghi vào file (append)
```

## 7. `grep` – Tìm kiếm trong file
```bash
grep "keyword" file.txt        # tìm keyword
grep -i "keyword" file.txt     # không phân biệt chữ hoa thường
grep -r "keyword" /path       # tìm trong thư mục
```

## 8. `ps` / `top` – Quản lý process
```bash
ps aux                      # xem tất cả process
ps -ef                      # xem chi tiết
top                         # realtime CPU / memory
htop                        # giao diện đẹp, cần cài thêm
```

## 9. `kill` / `pkill` / `killall` – Dừng process
```bash
kill <PID>                 # gửi SIGTERM
kill -9 <PID>              # gửi SIGKILL (force)
pkill node                  # kill theo tên process
killall node                # kill tất cả process node
```

## 10. `chmod` / `chown` – Thay đổi quyền & owner
```bash
chmod 755 file.txt          # quyền rwxr-xr-x
chmod +x script.sh          # thêm quyền thực thi
chown user:group file.txt    # đổi owner & group
```

## 11. `mkdir` / `rmdir` / `rm` – Thư mục & file
```bash
mkdir newfolder             # tạo thư mục
rm file.txt                  # xóa file
rm -rf folder                # xóa thư mục và tất cả file bên trong
```

## 12. `find` – Tìm file / folder
```bash
find /path -name "file.txt"
find /path -type d -name "folder"
```

## 13. `df` / `du` – Disk usage
```bash
df -h                        # dung lượng disk
du -sh /path                  # dung lượng thư mục
```

## 14. `scp` / `rsync` – Copy file qua mạng
```bash
scp file.txt user@host:/path  # copy file
rsync -avz /local/ user@host:/remote/  # đồng bộ thư mục
```

## 15. `ssh` – Kết nối remote server
```bash
ssh user@host
ssh -p 2222 user@host        # dùng port khác
```

## 16. `tar` – Nén / giải nén file
```bash
tar -czvf archive.tar.gz folder/   # nén
tar -xzvf archive.tar.gz           # giải nén
```

---

**Tài liệu này dùng làm reference nhanh các lệnh cơ bản Linux, dễ nhớ và áp dụng trực tiếp trên terminal**.

