# Linux – Hướng dẫn VIM

Tài liệu này tổng hợp các kiến thức cơ bản và nâng cao về **VIM** trên Linux, phù hợp làm **document tham khảo nhanh** cho developer và admin.

---

## 1. Giới thiệu VIM

- VIM là trình soạn thảo **text editor** mạnh mẽ trên Linux.
- Là **phiên bản nâng cao của vi**, hỗ trợ:
  - Syntax highlighting
  - Multiple buffers / windows
  - Search & replace
  - Macro / plugin

- 3 chế độ cơ bản:
  1. **Normal mode** – điều hướng, xóa, copy, paste
  2. **Insert mode** – gõ văn bản (`i`, `a`, `o`)
  3. **Command mode** – chạy lệnh `:` (save, quit, search, replace)

---

## 2. Mở & thoát file

### 2.1 Mở file
```bash
vim filename.txt
```

### 2.2 Thoát file
- `:q` → thoát nếu không chỉnh sửa
- `:q!` → thoát bỏ thay đổi
- `:wq` hoặc `:x` → lưu + thoát
- `ZZ` → lưu + thoát (Normal mode)

---

## 3. Chế độ Insert

- `i` → chèn trước cursor
- `I` → chèn đầu dòng
- `a` → chèn sau cursor
- `A` → chèn cuối dòng
- `o` → mở dòng mới bên dưới
- `O` → mở dòng mới bên trên

### 3.1 Thoát insert mode
- `Esc` → trở về Normal mode

---

## 4. Điều hướng

| Lệnh | Chức năng |
|------|-----------|
| h | sang trái |
| j | xuống |
| k | lên |
| l | sang phải |
| w | sang đầu từ tiếp theo |
| e | đến cuối từ |
| 0 | đầu dòng |
| ^ | đầu dòng, bỏ space |
| $ | cuối dòng |
| G | xuống cuối file |
| gg | lên đầu file |
| Ctrl + f | next page |
| Ctrl + b | previous page |

---

## 5. Chỉnh sửa cơ bản

### 5.1 Xóa & thay thế
- `x` → xóa ký tự dưới cursor
- `dd` → xóa dòng hiện tại
- `D` → xóa từ cursor đến cuối dòng
- `r<char>` → thay thế ký tự
- `R` → thay thế nhiều ký tự (overwrite)

### 5.2 Copy & Paste
- `yy` → copy dòng
- `p` → paste sau cursor
- `P` → paste trước cursor
- `y<number>y` → copy nhiều dòng
- `d` + movement → cut

---

## 6. Undo / Redo

- `u` → undo
- `Ctrl + r` → redo

---

## 7. Search & Replace

### 7.1 Tìm kiếm
- `/text` → tìm kiếm xuống
- `?text` → tìm kiếm lên
- `n` → next match
- `N` → previous match

### 7.2 Thay thế
- `:s/old/new/` → thay thế trong dòng hiện tại
- `:s/old/new/g` → thay thế tất cả trong dòng
- `:%s/old/new/g` → thay thế tất cả file
- `:%s/old/new/gc` → confirm từng lần thay thế

---

## 8. Lệnh mở rộng

### 8.1 Split window
- `:split filename` → split ngang
- `:vsplit filename` → split dọc
- `Ctrl + w + h/j/k/l` → di chuyển giữa các cửa sổ

### 8.2 Buffers
- `:e filename` → mở file khác
- `:bnext` / `:bn` → next buffer
- `:bprev` / `:bp` → previous buffer
- `:bd` → đóng buffer

### 8.3 Tab
- `:tabnew filename` → mở file trong tab mới
- `:tabnext` / `:tabn` → next tab
- `:tabprev` / `:tabp` → previous tab
- `:tabclose` → đóng tab

---

## 9. File & Settings

- `:w filename` → lưu file với tên khác
- `:set number` → hiển thị số dòng
- `:set relativenumber` → số dòng tương đối
- `:set tabstop=4` → tab = 4 space
- `:set expandtab` → tab = space

---

## 10. Tips & Tricks

- `.` → lặp lại lệnh vừa làm
- `>>` / `<<` → thụt đầu dòng / lùi đầu dòng
- `*` / `#` → tìm từ dưới cursor
- Sử dụng **vimrc** để lưu config cá nhân
- Có thể dùng **plugin manager**: Vundle, vim-plug, Pathogen

---

**Tài liệu này dùng làm reference nhanh cho việc sử dụng VIM trên Linux, từ cơ bản đến nâng cao, phù hợp cho dev và sysadmin**.

