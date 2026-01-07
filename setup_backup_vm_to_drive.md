TẠO THƯ MỤC BACKUP TRÊN VPS

```bash
sudo mkdir -p /backup/{pgsql,nginx,projects,logs}
sudo chown -R $USER:$USER /backup
```

4️⃣ SCRIPT BACKUP HOÀN CHỈNH (CỰC QUAN TRỌNG)

Tạo file:

```bash
nano ~/backup.sh
```

Dán nguyên khối này (đã chuẩn PROD):

```bash
#!/bin/bash
set -Eeuo pipefail
# --------- CLEAN LOCAL ----------
rm -f "$BACKUP_DIR"/*.tar.gz || true

# ================= CONFIG =================
DATE=$(date +"%Y-%m-%d_%H-%M")
HOSTNAME=$(hostname)
BACKUP_NAME="backup_${HOSTNAME}_${DATE}"

BACKUP_DIR="/backup/data"
TMP_DIR="/backup/tmp"
LOG_DIR="/backup/logs"
LOG_FILE="$LOG_DIR/backup_${DATE}.log"

DRIVE_REMOTE="gdrive:oracle-backup-${HOSTNAME}"
PYTHON="/backup/venv/bin/python"
REPORT_SCRIPT="/backup/backup_report.py"

# ================= STATE ==================
STATUS="success"
ERROR_LINE=""

# ================ INIT ====================
mkdir -p "$BACKUP_DIR" "$TMP_DIR" "$LOG_DIR"
exec > >(tee -a "$LOG_FILE") 2>&1

echo "================================================"
echo " BACKUP START : $(date)"
echo " HOST         : $HOSTNAME"
echo "================================================"

# ================ TRAPS ===================

on_error() {
    STATUS="fail"
    ERROR_LINE=$LINENO
    echo "[ERROR] Failure at line $ERROR_LINE"
}

on_exit() {
    echo "------------------------------------------------"
    echo " BACKUP END   : $(date)"
    echo " STATUS       : $STATUS"
    [ -n "$ERROR_LINE" ] && echo " ERROR LINE   : $ERROR_LINE"
    echo "------------------------------------------------"

    # gửi mail LUÔN LUÔN
    "$PYTHON" "$REPORT_SCRIPT" "$STATUS" "$LOG_FILE" "$BACKUP_NAME.tar.gz" || true
}

trap on_error ERR
trap on_exit EXIT

# ================ HELPERS ===================
safe_copy() {
    local SRC="$1"
    local DEST="$2"
    local DESC="$3"

    if [ -e "$SRC" ]; then
        echo "[+] Backup $DESC"
        cp -a "$SRC" "$DEST/" || echo "[WARN] Copy failed: $SRC"
    else
        echo "[SKIP] $DESC not found ($SRC)"
    fi
}

# ================ WORK ======================
WORK_DIR="$TMP_DIR/$BACKUP_NAME"
mkdir -p "$WORK_DIR"

# --------- DATA ----------
safe_copy /home/files "$WORK_DIR" "home files"
safe_copy /etc/nginx "$WORK_DIR" "nginx"
safe_copy /etc/letsencrypt "$WORK_DIR" "SSL certs"
safe_copy /etc/environment "$WORK_DIR" "environment"

# --------- USERS ----------
echo "[+] Backup users"
cp /etc/{passwd,group,shadow} "$WORK_DIR/" || echo "[WARN] User files backup failed"
echo "[+] Backup users | OK"

# --------- ENV FILES ----------
echo "[+] Backup .env files  | OK"
if find /home -type f -name ".env" 2>/dev/null | grep -q .; then
    find /home -type f -name ".env" -exec cp --parents {} "$WORK_DIR/" \;
    echo "[+] Backup .env files completed | OK"
else
    echo "[SKIP] No .env files found"
fi

# --------- CRON ----------
safe_copy /etc/crontab "$WORK_DIR" "system crontab | OK"
safe_copy /var/spool/cron "$WORK_DIR" "user crons | OK"

# --------- SYSTEM ----------
safe_copy /etc/hosts "$WORK_DIR" "hosts | OK"
safe_copy /etc/fstab "$WORK_DIR" "fstab | OK"
safe_copy /etc/sysctl.conf "$WORK_DIR" "sysctl | OK"

# --------- DATABASE ----------
echo "[+] Backup PostgreSQL"

PG_STATUS="success"

if command -v pg_dumpall >/dev/null 2>&1; then
    if sudo -u postgres pg_isready >/dev/null 2>&1; then
        if sudo -u postgres pg_dumpall > "$WORK_DIR/postgresql.sql" 2>>"$WORK_DIR/postgresql_error.log"; then
            echo "[+] PostgreSQL backup completed | OK"
        else
            echo "[WARN] PostgreSQL dump failed (check postgresql_error.log)"
            PG_STATUS="warning"
        fi
    else
        echo "[WARN] PostgreSQL not ready"
        PG_STATUS="warning"
    fi
else
    echo "[SKIP] pg_dumpall not found"
    PG_STATUS="warning"
fi
# --------- COMPRESS ----------
echo "[+] Compress backup"
tar -czf "$BACKUP_DIR/$BACKUP_NAME.tar.gz" -C "$TMP_DIR" "$BACKUP_NAME"
echo "[+] Compress backup completed | OK"

# --------- CLEAN TMP ----------
rm -rf "$WORK_DIR"

# --------- UPLOAD ----------
echo "[+] Upload to Google Drive"
if rclone lsd "$DRIVE_REMOTE" >/dev/null 2>&1; then
    rclone delete "$DRIVE_REMOTE" --rmdirs || true
    rclone mkdir "$DRIVE_REMOTE"
    rclone copy "$BACKUP_DIR/$BACKUP_NAME.tar.gz" "$DRIVE_REMOTE"
    echo "[+] Upload to Google Drive completed | OK"
else
    echo "[ERROR] Google Drive not reachable"
    false   # ép ERR để đánh dấu fail
fi

```

Tạo file:

```bash
nano ~/backup_report.py
```

Dán nguyên khối này (đã chuẩn PROD):

```python
import smtplib
import socket
import datetime
import sys
import subprocess
from email.mime.text import MIMEText
import logging

# ===== LOGGING =====
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)s | %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S"
)


# ===== CONFIG =====
SMTP_SERVER = "smtp.gmail.com"
SMTP_PORT = 587
SMTP_USER = "pmduc1438436@gmail.com"
SMTP_PASS = "vvrh iiik xygd lqdj"
MAIL_TO = ["minhduc20012402@gmail.com", "lanhlinh07@gmail.com"]

STATUS = sys.argv[1]
LOG_FILE = sys.argv[2]
BACKUP_FILE = sys.argv[3]

HOST = socket.gethostname()
TIME = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
IP = subprocess.getoutput("curl -s ifconfig.me")


SUBJECT = f"[{STATUS.upper()}] Oracle VM Backup - {HOST} - {TIME}"
LOG = ""
with open(LOG_FILE) as f:
    LOG = f.read()
logging.info("🚀 Bắt đầu test gửi mail SMTP")

STATUS_COLOR = "#16a34a" if STATUS.lower() == "success" else "#dc2626"
STATUS_BG = "#dcfce7" if STATUS.lower() == "success" else "#fee2e2"

HTML_BODY = f"""
    <!DOCTYPE html>
    <html lang="en">
    <head>
    <meta charset="UTF-8">
    <title>Oracle VM Backup Notification</title>
    </head>

    <body style="margin:0;padding:0;background:#eef2f7;font-family:Inter,Segoe UI,Roboto,Arial,sans-serif;">
    <table width="100%" cellpadding="0" cellspacing="0">
        <tr>
        <td align="center" style="padding:40px 16px;">

            <!-- MAIN CARD -->
            <table width="720" cellpadding="0" cellspacing="0"
            style="background:#ffffff;border-radius:14px;overflow:hidden;
                    box-shadow:0 20px 40px rgba(15,23,42,0.12);">

            <!-- HEADER -->
            <tr>
                <td style="
                padding:28px 32px;
                background:linear-gradient(135deg,#0f172a,#020617);
                color:#ffffff;
                ">
                <h1 style="margin:0;font-size:22px;font-weight:600;">
                    Oracle VM Backup Report
                </h1>
                <p style="margin:6px 0 0;font-size:13px;color:#cbd5f5;">
                    Automated system notification
                </p>
                </td>
            </tr>

            <!-- STATUS -->
            <tr>
                <td style="padding:24px 32px 8px;">
                <span style="
                    display:inline-block;
                    padding:6px 14px;
                    font-size:12px;
                    font-weight:600;
                    color:{STATUS_COLOR};
                    background:{STATUS_BG};
                    border-radius:999px;
                ">
                    {STATUS.upper()}
                </span>
                </td>
            </tr>

            <!-- CONTENT -->
            <tr>
                <td style="padding:16px 32px 32px;color:#0f172a;">
                <p style="font-size:14px;line-height:1.6;color:#334155;">
                    This is an automated email sent from the server
                    <strong>{HOST}</strong>. Please do not reply to this message.
                </p>

                <!-- INFO GRID -->
                <table width="100%" cellpadding="8" cellspacing="0"
                    style="margin-top:16px;font-size:14px;border-collapse:collapse;">
                    <tr>
                    <td style="color:#64748b;width:180px;">Time</td>
                    <td style="font-weight:500;">{TIME}</td>
                    </tr>
                    <tr>
                    <td style="color:#64748b;">Host</td>
                    <td style="font-weight:500;">{HOST}</td>
                    </tr>
                    <tr>
                    <td style="color:#64748b;">Public IP</td>
                    <td style="font-weight:500;">{IP}</td>
                    </tr>
                    <tr>
                    <td style="color:#64748b;">Backup File</td>
                    <td style="font-weight:500;">{BACKUP_FILE}</td>
                    </tr>
                    <tr>
                    <td style="color:#64748b;">Drive Path</td>
                    <td style="font-weight:500;">gdrive:/oracle-backup/</td>
                    </tr>
                </table>

                <!-- LOG -->
                <h3 style="margin:32px 0 12px;font-size:16px;">
                    Backup Execution Log
                </h3>

                <pre style="
                    background:#020617;
                    color:#e5e7eb;
                    padding:18px;
                    border-radius:10px;
                    font-size:12px;
                    line-height:1.55;
                    overflow:auto;
                    max-height:320px;
                ">{LOG}</pre>
                </td>
            </tr>

            <!-- FOOTER -->
            <tr>
                <td style="
                padding:18px 32px;
                background:#f8fafc;
                font-size:12px;
                color:#64748b;
                ">
                © {datetime.datetime.now().year} Oracle VM · Automated Monitoring System
                </td>
            </tr>

            </table>

        </td>
        </tr>
    </table>
    </body>
    </html>
"""

logging.info("✅ Tạo nội dung email thành công")

# msg = MIMEText(HTML_BODY)
msg = MIMEText(HTML_BODY, "html")

msg["From"] = SMTP_USER
msg["To"] = ", ".join(MAIL_TO)
msg["Subject"] = SUBJECT

try:
    logging.info("🔌 Kết nối tới SMTP server %s:%s", SMTP_SERVER, SMTP_PORT)
    server = smtplib.SMTP(SMTP_SERVER, SMTP_PORT)
    server.starttls()
    logging.info("🔑 Đăng nhập SMTP với user %s", SMTP_USER)
    server.login(SMTP_USER, SMTP_PASS)
    logging.info("📨 Đang gửi mail tới %s", MAIL_TO)
    server.sendmail(SMTP_USER, MAIL_TO, msg.as_string())
    server.quit()
    logging.info("🎉 GỬI MAIL THÀNH CÔNG")
except Exception as e:
    logging.error("MAIL ERROR: %s", str(e))
    sys.exit(1)


```

Lưu & cấp quyền:

```bash
chmod +x ~/backup.sh
chmod +x ~/backup_report.py
```

5️⃣ TEST THỦ CÔNG (BẮT BUỘC)

```bash
./backup.sh
```

Kiểm tra:

```bash
rclone ls gdrive:/oracle_backup
```

👉 Nếu thấy file .tar.gz là THÀNH CÔNG 100%

6️⃣ CHO NÓ TỰ CHẠY MỖI NGÀY (CRON)

```bash
crontab -e
```

Thêm dòng:

```bash
# ┌─ phút (0–59) → 59
# │ ┌─ giờ (0–23) → 23
# │ │ ┌─ ngày trong tháng → *
# │ │ │ ┌─ tháng → *
# │ │ │ │ ┌─ thứ trong tuần → *
# │ │ │ │ │
# 59 23 * * *

#  Chạy vào 23 giờ 59 phút theo UTC +7
CRON_TZ=Asia/Ho_Chi_Minh
59 23 * * * /backup/backup.sh
```

⏰ 02:00 sáng mỗi ngày

7️⃣ KHI ORACLE THU HỒI VM – BẠN LÀM GÌ?

Tạo VPS mới

Cài rclone

Login Drive

Download file:

```bash
rclone copy gdrive:/oracle_backup /backup
```

Restore PostgreSQL:

```bash
gunzip pgsql_*.sql.gz
psql -f pgsql_*.sql
```

#### 💥 Toàn bộ dữ liệu sống lại

## 🛡️ MẸO SINH TỒN QUAN TRỌNG

Không chạy backup bằng root

Test restore 1 lần/tháng

Không tin Oracle Free Tier

Nếu bạn muốn, bước tiếp theo mình có thể:

🔐 Mã hóa backup

📩 Gửi email khi backup FAIL

🧹 Retention thông minh (giữ 30 bản)

🧠 Tối ưu backup PostgreSQL lớn

# 📦 Hướng Dẫn Backup VPS Lên Google Drive với Rclone

Tài liệu hoàn chỉnh từ A → Z để cài đặt và tự động backup VPS lên Google Drive.

## 🎯 Mục đích

- ✅ Backup dữ liệu VPS an toàn
- ✅ Đồng bộ dữ liệu giữa các máy
- ✅ Tự động backup hàng ngày
- ✅ An toàn khi VPS bị thu hồi (Oracle Cloud Free, AWS Free, v.v.)

---

## 🧩 Giới thiệu Rclone

**rclone** là công cụ dòng lệnh mạnh mẽ cho phép:

- Kết nối Google Drive, OneDrive, S3, Dropbox, v.v.
- Copy / sync / backup dữ liệu
- Chạy tốt trên VPS không có GUI (headless)

� **rclone KHÔNG lưu mật khẩu Google, chỉ lưu token truy cập an toàn.**

---

## 🖥️ Điều kiện cần

| Yêu cầu            | Mô tả                                |
| ------------------ | ------------------------------------ |
| **VPS**            | Ubuntu 20.04 / 22.04                 |
| **Quyền**          | User có quyền sudo                   |
| **Máy local**      | Có trình duyệt web (Windows / macOS) |
| **Google Account** | Có Google Drive                      |

---

## 🧱 PHẦN 1 - CÀI ĐẶT RCLONE

### 1️⃣ Kiểm tra đã có rclone chưa

```bash
rclone version
```

Nếu chưa có, cài đặt:

```bash
sudo apt update
sudo apt install -y rclone
```

Kiểm tra lại:

```bash
rclone version
```

---

## 🔑 PHẦN 2 - CẤU HÌNH GOOGLE DRIVE

⚠️ **QUAN TRỌNG: LÀM BẰNG ROOT**

```bash
sudo -i
```

Bạn sẽ thấy prompt đổi thành:

```
root@dev-server:~#
```

### 2️⃣ Chạy cấu hình rclone

```bash
rclone config
```

Bạn sẽ thấy:

```
No remotes found - make a new one
n) New remote
q) Quit
```

👉 Gõ: `n`

### 3️⃣ Đặt tên remote

```
name> gdrive
```

⚠️ **PHẢI là `gdrive` (trùng với script backup)**

### 4️⃣ Chọn loại storage

Danh sách sẽ hiện rất dài, bạn KHÔNG CẦN ĐỌC HẾT
👉 Tìm dòng: `Google Drive`

Ví dụ:

```
13 / Google Drive
```

👉 Gõ số tương ứng (thường là `13`)

### 5️⃣ Client ID & Secret

```
client_id>
```

👉 **BẤM ENTER**

```
client_secret>
```

👉 **BẤM ENTER**

📌 **Mặc định của rclone là OK cho 99% trường hợp**

### 6️⃣ Scope (QUAN TRỌNG)

```
scope>
```

Bạn sẽ thấy:

```
1 / Full access all files
```

👉 Gõ: `1`

### 7️⃣ Root folder ID

```
root_folder_id>
```

👉 **BẤM ENTER**

### 8️⃣ Service Account

```
service_account_file>
```

👉 **BẤM ENTER**

### 9️⃣ Advanced config?

```
Edit advanced config? (y/n)
```

👉 Gõ: `n`

---

## 🔐 PHẦN 3 - XÁC THỰC GOOGLE

Bạn sẽ thấy:

```
Use auto config?
```

👉 **VPS thường KHÔNG có browser, nên:** `n`

Rclone sẽ in ra 1 link rất dài, ví dụ:

```
https://accounts.google.com/o/oauth2/auth?....
```

👉 **COPY LINK NÀY**

### 🔓 Trên máy cá nhân (PC / Laptop):

1. Mở Chrome
2. Dán link vào
3. Đăng nhập Google
4. Cho phép rclone truy cập Drive
5. Google sẽ hiện 1 mã code

📌 **Copy CODE đó, quay lại VPS, dán vào:**

```
Enter verification code>
```

👉 **Enter**

### 10️⃣ Configure as Shared Drive?

```
Configure this as a Shared Drive?
```

👉 Gõ: `n`

### 11️⃣ Hoàn tất

```
y) Yes this is OK
```

👉 Gõ: `y`

---

## ✅ PHẦN 4 - KIỂM TRA KẾT NỐI

### 1️⃣ List Drive

```bash
rclone lsd gdrive:
```

👉 Nếu thấy danh sách thư mục → **OK**

### 2️⃣ Tạo thư mục backup

```bash
rclone mkdir gdrive:oracle_backup
```

### 3️⃣ Test upload thử

```bash
echo "test backup" > /tmp/test.txt
rclone copy /tmp/test.txt gdrive:oracle_backup
```

👉 **Vào Google Drive kiểm tra:** Có file test.txt → 🎉 **THÀNH CÔNG 100%**

---

## � PHẦN 5 - SCRIPT BACKUP TỰ ĐỘNG

### 1️⃣ Tạo thư mục backup trên VPS

```bash
sudo mkdir -p /backup/{pgsql,nginx,projects,logs}
sudo chown -R $USER:$USER /backup
```

### 2️⃣ Tạo script backup hoàn chỉnh

```bash
nano ~/backup_to_gdrive.sh
```

Dán nguyên khối script sau:

```bash
#!/bin/bash

DATE=$(date +%F)
BACKUP_DIR="/backup"
LOG="$BACKUP_DIR/logs/backup_$DATE.log"

echo "=== Backup start: $(date) ===" >> $LOG

# 1. PostgreSQL
echo "[+] Backup PostgreSQL" >> $LOG
sudo -u postgres pg_dumpall | gzip > $BACKUP_DIR/pgsql/pgsql_$DATE.sql.gz

# 2. Nginx config
echo "[+] Backup nginx config" >> $LOG
tar -czf $BACKUP_DIR/nginx/nginx_$DATE.tar.gz /etc/nginx

# 3. Project source (CHỈNH LẠI ĐƯỜNG DẪN)
echo "[+] Backup projects" >> $LOG
tar -czf $BACKUP_DIR/projects/projects_$DATE.tar.gz /var/www

# 4. Gom toàn bộ
echo "[+] Create full archive" >> $LOG
tar -czf $BACKUP_DIR/full_backup_$DATE.tar.gz \
    $BACKUP_DIR/pgsql \
    $BACKUP_DIR/nginx \
    $BACKUP_DIR/projects

# 5. Upload lên Google Drive
echo "[+] Upload to Google Drive" >> $LOG
rclone copy $BACKUP_DIR/full_backup_$DATE.tar.gz gdrive:/oracle_backup --progress >> $LOG 2>&1

# 6. Xóa file cũ hơn 7 ngày (local)
find $BACKUP_DIR -type f -mtime +7 -delete

echo "=== Backup end: $(date) ===" >> $LOG
```

### 3️⃣ Cấp quyền thực thi

```bash
chmod +x ~/backup_to_gdrive.sh
```

### 4️⃣ Test thủ công (BẮT BUỘC)

```bash
./backup_to_gdrive.sh
```

Kiểm tra kết quả:

```bash
rclone ls gdrive:/oracle_backup
```

👉 Nếu thấy file `.tar.gz` là **THÀNH CÔNG 100%**

---

## ⏰ PHẦN 6 - TỰ ĐỘNG HẰNG NGÀY (CRON)

### Thiết lập cron

```bash
crontab -e
```

Thêm dòng sau để backup lúc 02:00 sáng mỗi ngày:

```bash
0 2 * * * /home/ubuntu/backup_to_gdrive.sh
```

---

## � PHẦN 7 - KHÔI PHỤC DỮ LIỆU

Khi VPS bị thu hồi hoặc cần khôi phục:

### 1️⃣ Tạo VPS mới và cài rclone

Làm lại các bước trong **PHẦN 1-4**

### 2️⃣ Download file backup

```bash
rclone copy gdrive:/oracle_backup /backup
```

### 3️⃣ Restore PostgreSQL

```bash
gunzip /backup/pgsql/pgsql_*.sql.gz
sudo -u postgres psql -f /backup/pgsql/pgsql_*.sql
```

### 4️⃣ Restore Nginx config

```bash
tar -xzf /backup/nginx/nginx_*.tar.gz -C /
```

### 5️⃣ Restore Projects

```bash
tar -xzf /backup/projects/projects_*.tar.gz -C /
```

💥 **Toàn bộ dữ liệu sống lại!**

---

## 🛡️ MẸO SINH TỒN QUAN TRỌNG

- ✅ Không chạy backup bằng root
- ✅ Test restore 1 lần/tháng
- ✅ Không tin Oracle Free Tier
- ✅ Giữ nhiều bản backup (retention policy)

---

## 🔧 CÁC TÍNH NĂNG NÂNG CAO (TÙY CHỌN)

Nếu bạn muốn, có thể thêm:

### 🔐 Mã hóa backup

```bash
# Thêm vào script
gpg --cipher-algo AES256 --compress-algo 1 --symmetric --output $BACKUP_DIR/full_backup_$DATE.tar.gz.gpg $BACKUP_DIR/full_backup_$DATE.tar.gz
```

### 📩 Gửi email khi backup FAIL

```bash
# Thêm vào cuối script
if [ $? -ne 0 ]; then
    echo "Backup failed!" | mail -s "Backup Alert" your@email.com
fi
```

### 🧹 Retention thông minh

```bash
# Giữ 30 bản backup mới nhất trên Google Drive
rclone delete gdrive:/oracle_backup --min-age 30d
```

---

## 📝 Lưu ý quan trọng

- Luôn thực hiện cấu hình với quyền `root`
- Tên remote phải là `gdrive` để tương thích với script backup
- Đảm bảo có kết nối internet ổn định trong quá trình xác thực
- Lưu lại mã xác thực để sử dụng cho các lần sau

---

## 🔧 Troubleshooting

Nếu gặp lỗi trong quá trình cài đặt:

1. Kiểm tra lại phiên bản rclone: `rclone version`
2. Đảm bảo đang chạy với quyền root
3. Kiểm tra kết nối mạng
4. Thử lại quá trình xác thực Google

---

_Hướng dẫn này được thiết kế cho môi trường Ubuntu/Debian trên VPS_
