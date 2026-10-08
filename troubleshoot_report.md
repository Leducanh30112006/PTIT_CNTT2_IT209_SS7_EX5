# Báo Cáo Chẩn Đoán và Xử Lý Xung Đột Cổng Mạng (Troubleshoot Report)

## 1. Mô tả sự cố
Khi khởi chạy dịch vụ `spring-app.service`, hệ thống báo lỗi không thể bind vào cổng **8082** do lỗi `java.net.BindException: Address already in use`.

---

## 2. Quy trình chẩn đoán (Troubleshooting)

### 2.1. Xác định tiến trình đang chiếm dụng cổng 8082
Sử dụng lệnh `ss` để kiểm tra tiến trình đang lắng nghe trên cổng 8082:
```bash
$ sudo ss -tlnp | grep 8082
LISTEN 0      5            0.0.0.0:8082      0.0.0.0:*    users:(("python3",pid=21543,fd=3))
```
Hoặc kiểm tra chi tiết bằng `lsof`:
```bash
$ sudo lsof -i :8082
COMMAND     PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
python3   21543 root    3u  IPv4  65432      0t0  TCP *:8082 (LISTEN)
```

**Kết quả phân tích:**
- Tiến trình gây xung đột: `python3` (chạy lệnh `python3 -m http.server 8082`).
- **PID (Process ID)**: `21543`.

---

## 3. Xử lý và Giải phóng cổng

### 3.1. Chấm dứt tiến trình chiếm dụng
Gửi tín hiệu dừng tiến trình để giải phóng tài nguyên:
```bash
$ sudo kill -9 21543
```

Kiểm tra lại để đảm bảo cổng đã được giải phóng:
```bash
$ sudo ss -tlnp | grep 8082
# (Không có tiến trình nào trả về)
```

### 3.2. Khởi động lại dịch vụ Spring Boot
```bash
$ sudo systemctl restart spring-app.service
```

---

## 4. Xác nhận kết quả

Kiểm tra lại cổng 8082 sau khi khởi động lại dịch vụ:
```bash
$ sudo ss -tlnp | grep 8082
LISTEN 0      100          *:8082            *:*    users:(("java",pid=22014,fd=42))
```

Kiểm tra trạng thái dịch vụ `spring-app.service`:
```bash
$ sudo systemctl status spring-app.service
● spring-app.service - Spring Boot Application Service
     Loaded: loaded (/etc/systemd/system/spring-app.service; disabled; vendor preset: enabled)
     Active: active (running) since Thu 2026-10-08 14:45:10 UTC; 8s ago
   Main PID: 22014 (java)
```

**Kết luận:** Xung đột cổng 8082 đã được giải quyết triệt để. Cổng 8082 hiện đã được cấp phát cho tiến trình `java` của ứng dụng Spring Boot.
