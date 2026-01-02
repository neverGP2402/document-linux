# TỔNG HỢP KIẾN THỨC VỀ FILE & FOLDER TRONG LINUX

Tài liệu này dùng làm **document tham khảo nhanh** về cách Linux quản lý file và thư mục, phù hợp cho dev, sysadmin và người mới học Linux.

---

## 1. Khái niệm cơ bản

- **Linux coi mọi thứ là file**: file thường, thư mục, thiết bị, socket, process info...
- File và folder được tổ chức theo **cây thư mục (directory tree)**, bắt đầu từ root `/`.

Ví dụ:
```
/
├── bin
├── etc
├── home
│   └── user
├── var
└── tmp
```

---

## 2. Các thư mục quan trọng trong Linux

| Thư mục | Ý nghĩa |
|------|------|
| `/` | Root – thư mục gốc |
| `/bin` | Lệnh cơ bản (ls, cp, mv…) |
| `/sbin` | Lệnh hệ thống (root dùng) |
| `/etc` | File cấu hình hệ thống |
| `/home` | Thư mục người dùng |
| `/root` | Home của user root |
| `/var` | Log, cache, dữ liệu thay đổi |
| `/tmp` | File tạm (có thể bị xoá khi reboot) |
| `/usr` | App và thư viện người dùng |
| `/opt` | Phần mềm cài thêm |

---

## 3. Đường dẫn (Path)

### 3.1 Absolute path
- Bắt đầu bằng `/`
- Ví dụ: `/home/user/project/app.js`

### 3.2 Relative path
- Tính từ thư mục hiện tại
- Ví dụ:
  - `./file.txt`
  - `../config.yml`

Ký hiệu đặc biệt:
- `.` : thư mục hiện tại
- `..` : thư mục cha
- `~` : home user hiện tại

---

## 4. Các loại file trong Linux

| Ký hiệu | Loại |
|------|------|
| `-` | File thường |
| `d` | Directory |
| `l` | Symbolic link |
| `c` | Character device |
| `b` | Block device |
| `s` | Socket |
| `p` | Pipe |

Kiểm tra bằng:
```bash
ls -l
```

---

## 5. Quyền file (Permissions)

### 5.1 Cấu trúc quyền

Ví dụ:
```
-rwxr-xr--
```

| Phần | Ý nghĩa |
|---|---|
| `r` | read |
| `w` | write |
| `x` | execute |

Áp dụng cho:
- User (owner)
- Group
- Other

### 5.2 Thay đổi quyền

```bash
chmod 755 file.sh
chmod +x file.sh
```

### 5.3 Thay đổi owner

```bash
chown user:group file.txt
```

---

## 6. Lệnh thao tác file & folder cơ bản

### 6.1 Xem danh sách
```bash
ls
ls -la
```

### 6.2 Tạo
```bash
touch file.txt
mkdir folder
mkdir -p a/b/c
```

### 6.3 Sao chép
```bash
cp file1 file2
cp -r folder1 folder2
```

### 6.4 Di chuyển / đổi tên
```bash
mv old.txt new.txt
mv file /path/to/dir
```

### 6.5 Xoá
```bash
rm file.txt
rm -r folder
rm -rf folder   # cẩn thận
```

---

## 7. Xem nội dung file

```bash
cat file.txt
less file.txt
more file.txt
head file.txt
tail file.txt
tail -f log.txt
```

---

## 8. Tìm kiếm file & folder

### 8.1 find
```bash
find /path -name "*.log"
find . -type d
```

### 8.2 grep
```bash
grep "keyword" file.txt
grep -r "keyword" /folder
```

---

## 9. Symbolic Link & Hard Link

### 9.1 Symbolic link (shortcut)
```bash
ln -s source target
```

### 9.2 Hard link
```bash
ln source target
```

Khác biệt:
- Symlink có thể link folder
- Hard link không vượt filesystem

---

## 10. File ẩn

- File bắt đầu bằng `.`
- Ví dụ: `.env`, `.bashrc`

Hiển thị:
```bash
ls -a
```

---

## 11. Phân quyền đặc biệt

| Quyền | Ý nghĩa |
|---|---|
| SUID | Chạy bằng quyền owner |
| SGID | Chạy bằng quyền group |
| Sticky bit | Chỉ owner được xoá file |

Ví dụ:
```bash
chmod +s file
chmod +t folder
```

---

## 12. Thực hành nên nhớ

- Không dùng `rm -rf /` 😄
- Luôn kiểm tra `pwd` trước khi xoá
- Dùng `ls -la` để debug permission
- Với server, cẩn thận thư mục `/etc`, `/var`

---

## 13. Ghi chú cho Dev & VPS

- Code thường đặt tại `/var/www` hoặc `/home/user/app`
- Log nằm trong `/var/log`
- App chạy service thường có user riêng
- Không chạy app bằng root nếu không cần thiết

---

**End of document**

