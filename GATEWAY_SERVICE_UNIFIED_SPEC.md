# GATEWAY – SERVICE – CLIENT

# UNIFIED COMMUNICATION & AUTH SPECIFICATION

Version: 1.0.1 (REVIEW)

---

## 1. Mục tiêu

- Toàn bộ **AUTH / API / SOCKET** dùng **CHUNG 1 FORMAT**
- Gateway **KHÔNG phá format**
- Service **chịu trách nhiệm sinh token**
- Gateway **verify token & quản lý session**
- Hỗ trợ **reconnect / mất kết nối / F5 / resume app**

---

## 2. Vai trò hệ thống

### Client

- Web / Mobile / Desktop
- Gửi request đúng format
- Lưu token sau khi login
- Khi reconnect **bắt buộc gửi lại token**

### Gateway

- Entry point duy nhất
- Verify token
- Map `socket_id ↔ token ↔ user`
- Forward request qua RabbitMQ
- Không sinh token

### Service

- Xử lý nghiệp vụ
- Service AUTH sinh token
- Trả token về Client thông qua Gateway

---

## 3. FORMAT CHUNG – Client → Gateway

(Ap dụng cho HTTP & Socket)

```json
{
  "service": {
    "biz": "AUTH",
    "object": "USER",
    "function": "login",
    "action": "GET",
    "data": {
      "username": "admin",
      "password": "123456"
    }
  },
  "type": "REQUEST",
  "screen": "LOGIN",
  "url": "/auth/login",
  "client_type": "DESKTOP_WEB",
  "request_id": "req-1001",
  "language": "vi",
  "token": null,
  "user": null,
  "signature": "HMAC_SHA256",
  "meta": {
    "source": "client",
    "version": "1.0.0",
    "ts": 1704179200
  }
}
```

> Chưa login → `token = null`

---

## 4. FORMAT – Gateway → Service (RabbitMQ)

```json
{
  "service": {
    "biz": "AUTH",
    "object": "USER",
    "function": "login",
    "action": "GET",
    "data": {}
  },
  "type": "REQUEST",
  "screen": "LOGIN",
  "url": "/auth/login",
  "client_type": "DESKTOP_WEB",
  "request_id": "req-1001",
  "language": "vi",
  "token": null,
  "user": null,
  "meta": {
    "source": "gateway",
    "version": "1.0.0",
    "ts": 1704179200,
    "ip_address": "192.168.1.10"
  }
}
```

---

## 5. FORMAT – Service → Gateway → Client (RESPONSE)

```json
{
  "request_id": "req-1001",
  "status": "SUCCESS",
  "status_code": "200",
  "message": "Login success",
  "data": {
    "token": "JWT_OR_CUSTOM_TOKEN",
    "user": {
      "id": "u001",
      "role": "ADMIN"
    }
  },
  "meta": {
    "time_request": "2026-01-02T09:36:10Z",
    "time_response": "2026-01-02T09:36:10Z",
    "processing_time": 0.234
  }
}
```

---

## 6. QUY TRÌNH TOKEN (BẮT BUỘC)

### 6.1 Sinh token

- Chỉ **Service AUTH** được sinh token
- Token chứa tối thiểu:
  - user_id
  - role
  - exp
  - signature

### 6.2 Gateway xử lý token

- Verify signature
- Check expire
- Cache session
- Map:
  ```
  socket_id ↔ token ↔ user_id
  ```

---

## 7. SOCKET CONNECT & VERIFY

### 7.1 Connect lần đầu (chưa login)

```text
Client connect → Gateway
→ KHÔNG verify
```

---

### 7.2 Connect / Reconnect (đã có token)

```json
{
  "token": "JWT_OR_CUSTOM_TOKEN",
  "meta": {
    "client_type": "DESKTOP_WEB",
    "ts": 1704179200
  }
}
```

Lúc này gateway sẽ verify token

---

## 8. Gateway verify CONNECT (Logic)

- Token hợp lệ → gán user cho socket
- Token sai / hết hạn → reject & yêu cầu login lại

---

## 9. Verify mỗi REQUEST từ Client

- Mọi request **sau login** bắt buộc có token
- Gateway kiểm tra:
  1. Token tồn tại
  2. Token hợp lệ
  3. Socket đã được map với token

---

## 10. RECONNECT SCENARIO

| Trường hợp    | Xử lý                 |
| ------------- | --------------------- |
| Mất mạng      | Reconnect + gửi token |
| F5 Web        | Reconnect + gửi token |
| App resume    | Reconnect + gửi token |
| Token expired | Trả 401 → login lại   |

---

## 11. NGUYÊN TẮC BẤT DI BẤT DỊCH

- ❌ Gateway không sinh token
- ❌ Gateway không đổi format
- ✅ Auth là 1 service độc lập
- ✅ Một format cho toàn hệ thống
- ✅ Token là định danh session duy nhất

---

## 12. KẾT LUẬN

Spec này:

- Dùng được cho PROD
- Phù hợp microservice
- Socket-first architecture
- Dễ audit, dễ scale

---

END OF DOCUMENT
