# Triển khai SOC Lab: Tích hợp Wazuh SIEM và Khắc phục sự cố mạng trên VirtualBox

## Tổng quan

Dự án này ghi lại quá trình triển khai một môi trường Security Operations Center (SOC) Lab thu nhỏ nhằm phục vụ việc thu thập log tập trung, giám sát Endpoint và phân tích sự kiện an ninh bằng nền tảng mã nguồn mở Wazuh SIEM.

### Kiến trúc hệ thống

```text
+------------------------+
|    Wazuh SIEM Server   |
|    Ubuntu Server       |
|      Wazuh 4.9.2       |
+-----------+------------+
            |
            |
     Host-Only Network
      192.168.56.0/24
            |
            |
+-----------+------------+
|      Windows 10 VM     |
|      Wazuh Agent       |
|         4.9.2          |
+------------------------+
```

### Thành phần triển khai

| Thành phần      | Mô tả                                |
| --------------- | ------------------------------------ |
| SIEM Server     | Ubuntu Server 22.04 chạy Wazuh 4.9.2 |
| Wazuh Manager   | Tiếp nhận log và quản lý Agent       |
| Wazuh Indexer   | Lưu trữ và lập chỉ mục log           |
| Wazuh Dashboard | Giao diện giám sát và phân tích      |
| Endpoint        | Windows 10 Enterprise LTSC           |
| Wazuh Agent     | Thu thập và gửi log về Manager       |

---

# Nhật ký xử lý sự cố

Trong quá trình triển khai, hệ thống gặp nhiều vấn đề liên quan đến mạng và khả năng tương thích. Dưới đây là các lỗi thực tế đã gặp cùng nguyên nhân và cách khắc phục.

## Sự cố 1: Mất file `ossec.conf` sau khi cài đặt Silent Install

### Hiện tượng

* Wazuh Agent cài đặt thành công.
* Dịch vụ `WazuhSvc` hoạt động bình thường.
* Dashboard hiển thị:

```text
No agents were added
```

* Không tìm thấy file cấu hình `ossec.conf`.

### Nguyên nhân

Việc sử dụng tham số cài đặt ẩn `/q` khiến Windows Installer hoàn tất quá trình cài đặt nhưng không tạo đầy đủ các file cấu hình cần thiết. Trong một số trường hợp, file có thể bị chuyển hướng vào VirtualStore hoặc bị lỗi do dữ liệu từ lần cài đặt trước.

### Giải pháp

1. Gỡ bỏ hoàn toàn Wazuh Agent.
2. Xóa các thư mục còn sót lại.
3. Tiến hành cài đặt lại bằng giao diện Wizard.
4. Kiểm tra thư mục:

```text
C:\Program Files (x86)\ossec-agent\
```

để xác nhận file cấu hình đã được tạo.

---

## Sự cố 2: Không thể đăng ký Agent do cấu hình Localhost

### Hiện tượng

Log Agent xuất hiện lỗi:

```text
Unable to connect to enrollment service at [127.0.0.1]:1515
```

### Nguyên nhân

Trong cấu hình Port Forwarding của VirtualBox, trường:

```text
Host IP = 127.0.0.1
```

được thiết lập, khiến VirtualBox chỉ lắng nghe trên loopback của máy chủ vật lý.

Máy ảo Windows không thể truy cập dịch vụ Wazuh Manager thông qua địa chỉ này.

### Giải pháp

1. Xóa giá trị Host IP trong cấu hình Port Forwarding.
2. Xác định địa chỉ Gateway của mạng NAT:

```text
10.0.2.2
```

3. Cập nhật file cấu hình Agent:

```xml
<address>10.0.2.2</address>
```

4. Thực hiện đăng ký Agent lại.

---

## Sự cố 3: Agent đăng ký thành công nhưng không kết nối

### Hiện tượng

* Agent đã được đăng ký.
* Dashboard hiển thị Agent.
* Trạng thái luôn ở:

```text
Never connected
```

hoặc

```text
Disconnected
```

### Nguyên nhân

File `docker-compose.yml` mặc định chỉ mở:

```yaml
1514-1515:1514-1515/tcp
```

Trong khi Wazuh Agent có thể sử dụng UDP 1514 để gửi gói tin Keep-Alive.

Do Docker chưa mở UDP 1514 nên các gói tin bị loại bỏ trước khi tới Wazuh Manager.

### Giải pháp

Chỉnh sửa phần cấu hình của Wazuh Manager:

```yaml
ports:
  - "1514:1514/tcp"
  - "1514:1514/udp"
  - "1515:1515/tcp"
```

Khởi động lại hệ thống:

```bash
docker compose down
docker compose up -d
```

---

## Sự cố 4: Chuyển đổi sang mạng Host-Only

### Hiện tượng

Mặc dù Agent đăng ký thành công nhưng kết nối thông qua NAT thường xuyên mất ổn định và Agent liên tục bị ngắt kết nối.

### Nguyên nhân

Mạng NAT và Port Forwarding làm tăng độ phức tạp trong việc định tuyến gói tin và gây khó khăn cho quá trình vận hành Lab.

### Giải pháp

### Chuyển đổi hạ tầng mạng

Cấu hình cả hai máy ảo sử dụng:

```text
Host-Only Adapter
```

Thông số:

```text
Network: 192.168.56.0/24
Promiscuous Mode: Allow All
```

### Chuẩn hóa giao thức truyền thông

Cập nhật file `ossec.conf`:

```xml
<protocol>tcp</protocol>
```

Việc sử dụng TCP giúp tăng độ tin cậy trong quá trình truyền log và đơn giản hóa việc xử lý sự cố.

---

# Kết quả đạt được

Sau khi hoàn tất quá trình cấu hình:

* Wazuh Agent kết nối thành công tới Manager.
* Kênh truyền thông hoạt động ổn định thông qua:

```text
192.168.56.101:1514/tcp
```

* Trạng thái Agent chuyển sang:

```text
Active
```

* Hệ thống bắt đầu thu thập thành công:

  * Security Logs
  * System Logs
  * Application Logs

Môi trường đã sẵn sàng để triển khai thêm các nguồn telemetry như Sysmon trong các giai đoạn tiếp theo.

> ![alt text](<Screenshot 2026-06-01 203625.png>)

---

# Bài học kinh nghiệm

## 1. Mạng máy tính là nền tảng của giám sát an ninh

Việc hiểu rõ cách gói tin di chuyển giữa các hệ thống là điều kiện tiên quyết khi xây dựng và vận hành hạ tầng giám sát.

Thông qua bài Lab này, đã có cơ hội thực hành với:

* NAT
* Port Forwarding
* Loopback Address (127.0.0.1)
* Host-Only Network
* TCP và UDP

---

## 2. Phân tích Log là kỹ năng quan trọng nhất

Mọi quyết định xử lý sự cố đều được đưa ra dựa trên việc phân tích log thay vì thử sai ngẫu nhiên.

Tệp log quan trọng nhất trong quá trình triển khai là:

```text
ossec.log
```

Việc đọc và phân tích chính xác các thông báo lỗi đã giúp xác định nhanh nguyên nhân và đưa ra hướng xử lý phù hợp.

---

## 3. Không nên phụ thuộc hoàn toàn vào cấu hình mặc định

Các cấu hình mặc định thường được thiết kế cho môi trường triển khai phổ biến và có thể không phù hợp với môi trường Lab ảo hóa.

Để hệ thống hoạt động ổn định, cần thực hiện:

* Điều chỉnh Port Mapping của Docker.
* Tối ưu cấu trúc mạng.
* Chuẩn hóa giao thức truyền thông.
* Kiểm tra khả năng tương thích giữa các thành phần.

Khả năng hiểu và tùy chỉnh hệ thống là kỹ năng quan trọng đối với SOC Analyst và Security Engineer.

---

# Công nghệ sử dụng

* Ubuntu Server 22.04 LTS
* Windows 10 Enterprise LTSC
* Wazuh 4.9.2
* Docker Engine
* Docker Compose V2
* VirtualBox
* Host-Only Networking
* TCP/IP
* Sysmon (triển khai ở giai đoạn tiếp theo)
