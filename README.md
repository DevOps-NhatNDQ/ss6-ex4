## 1. Các bước thực hiện

### Bước 1: Tạo kịch bản shell script
Tạo tệp `loop-monitor.sh` và cấp quyền thực thi:

```bash
cat << 'EOF' > loop-monitor.sh
#!/bin/bash
while true; do
    echo "System time: $(date)" >> /tmp/monitor.log
    sleep 5
done
EOF

chmod +x loop-monitor.sh

```

### Bước 2: Khởi chạy tiến trình nền độc lập bằng `nohup`

Thực thi script ở chế độ nền độc lập (vẫn tiếp tục chạy sau khi thoát SSH session):

```bash
nohup ./loop-monitor.sh > /dev/null 2>&1 &

```

> Giải thích lệnh:
> * `nohup`: Bỏ qua tín hiệu `SIGHUP` (không bị tắt khi đóng Terminal).
> * `> /dev/null 2>&1`: Đẩy cả standard output (stdout) và standard error (stderr) vào `/dev/null` để tránh tạo file `nohup.out`.
> * `&`: Đưa tiến trình vào chạy ẩn dưới nền (background process).
> 
> 

### Bước 3: Tìm PID của tiến trình

Kiểm tra tiến trình đang chạy và xác định Process ID (PID):

```bash
ps aux | grep loop-monitor.sh
# Hoặc dùng pgrep
pgrep -f loop-monitor.sh

```

Kết quả:

```text
ubuntu     14582  0.0  0.1   7236  3120 pts/0    S    21:35   0:00 /bin/bash ./loop-monitor.sh

```

*(PID của tiến trình trong trường hợp này là `14582`)*

---

## 3. Kiểm tra kết quả & Dừng tiến trình

### 3.1. Theo dõi dữ liệu ghi vào tệp log (`tail -f`)

Kiểm tra xem dữ liệu có liên tục được ghi vào file `/tmp/monitor.log` cứ mỗi 5 giây hay không:

```bash
tail -n 10 /tmp/monitor.log

```

Kết quả đầu ra (Output):

```text
System time: Wed Oct  7 21:35:10 UTC 2026
System time: Wed Oct  7 21:35:15 UTC 2026
System time: Wed Oct  7 21:35:20 UTC 2026
System time: Wed Oct  7 21:35:25 UTC 2026
System time: Wed Oct  7 21:35:30 UTC 2026

```

---

### 3.2. Dừng tiến trình bằng tín hiệu `SIGTERM (15)`

Dừng tiến trình một cách an toàn bằng câu lệnh:

```bash
kill -15 14582

```

### 3.3. Xác nhận tiến trình đã bị hủy

Kiểm tra lại danh sách tiến trình để đảm bảo `loop-monitor.sh` đã tắt:

```bash
ps aux | grep loop-monitor.sh

```

Kết quả đầu ra (Output):

```text
ubuntu     14610  0.0  0.0   6480   720 pts/0    S+   21:36   0:00 grep --color=auto loop-monitor.sh

```

*(Tiến trình `14582` đã biến mất hoàn toàn khỏi danh sách)*
