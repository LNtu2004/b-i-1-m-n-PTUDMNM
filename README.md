# Môn: Phát triển ứng dụng với mã nguồn mở-TEE0421
Lê Ngọc Tú msv K225480106069
### A. Đăng ký tên miền xịn cho cá nhân:
![d232fd45-1acf-4a4a-af1e-f1fc50e049f8](https://github.com/user-attachments/assets/e347912d-71dd-4b34-a462-f43b0d62d907)
1.em đăng ký tên miền là ngoctu123.id.vn trên web pavietnam.vn
<img width="1834" height="384" alt="image" src="https://github.com/user-attachments/assets/c5d3578b-0181-4701-a059-dc4ffc39bc9c" />
2.Đã đăng ký cloudflare 
<img width="1484" height="739" alt="image" src="https://github.com/user-attachments/assets/b840af93-19b5-4a96-b117-6d9eee87345d" />
3.Đã thêm domain đăng ký vô cloudflare và nhận 2 namespace là : brenda.ns.cloudflare.com và yoxall.ns.cloudflare.com
<img width="1379" height="850" alt="image" src="https://github.com/user-attachments/assets/1896f3f4-20b4-47e4-b652-f9b7b7354bc1" />
4.Đã nhập 2 dòng namespace của cloudflare vào trong trang quản lý DNS
### Câu hỏi về bài làm?
1. Tại sao phải dùng Nginx làm Reverse Proxy mà không trỏ thẳng Tunnel vào Node-RED?
2. Sự khác biệt giữa việc Mount file và Mount thư mục trong Docker là gì?
3. Nếu thay đổi file index.html ở máy Ubuntu, nội dung trên web có thay đổi ngay không? Tại sao?
4. docker-compose.yml khai báo các services có phần **restart: always** hoặc **restart: unless-stopped** : chúng để làm gì?
5. Cách khai báo để tất cả các services đều dùng chung 1 network? lợi ích của việc khai báo này là gì? Sửa đổi file docker-compose để tất cả các service đều dùng chung 1 network.
6. Tìm cách đưa Cloudflare **Token** vào trong file .env rồi sau đó thêm .env vào file .gitignore trước khi push code lên github. Tại sao nói đây là điều quan trọng về bảo mật mã nguồn?
7. Tại sao chúng ta nên thêm hậu tố :ro khi mount file cấu hình Nginx?
8. Khi dùng Cloudflare Tunnel: có cần thiết phải mở cổng cho các service nữa không?
# Trả lời câu hỏi
Được, mình sẽ trả lời theo kiểu **ngắn gọn – rõ ràng – phù hợp bài kiểm tra**:

---

## **1. Tại sao dùng Nginx Reverse Proxy mà không trỏ thẳng Tunnel vào Node-RED?**

* **Bảo mật**: Nginx giúp ẩn backend (Node-RED), tránh bị truy cập trực tiếp.
* **Linh hoạt**: Có thể định tuyến nhiều service khác nhau trên cùng một domain.
* **Quản lý tập trung**: Xử lý SSL, logging, giới hạn truy cập tại một điểm.
* **Mở rộng**: Dễ thêm service mới mà không cần thay đổi cấu trúc hệ thống.

→ Trỏ thẳng vào Node-RED chỉ phù hợp cho demo nhỏ, không phù hợp thực tế.

---

## **2. Sự khác biệt giữa Mount file và Mount thư mục trong Docker**

* **Mount file**:

  * Chỉ liên kết một file cụ thể
  * Ít rủi ro, dùng cho file cấu hình

* **Mount thư mục**:

  * Liên kết toàn bộ thư mục
  * Dùng cho source code hoặc dữ liệu

→ Mount thư mục có thể ghi đè toàn bộ nội dung trong container.

---

## **3. Nếu thay đổi file index.html ở Ubuntu, web có thay đổi ngay không? Tại sao?**

* **Có**, nếu file được mount từ host vào container.

* Vì container sử dụng trực tiếp file từ hệ thống host nên thay đổi sẽ phản ánh ngay.

* **Không**, nếu file được copy vào image khi build (không dùng mount).

---

## **4. restart: always và restart: unless-stopped dùng để làm gì?**

* **restart: always**

  * Container luôn tự khởi động lại khi bị dừng hoặc khi hệ thống reboot

* **restart: unless-stopped**

  * Tự khởi động lại trừ khi người dùng đã dừng thủ công

→ Dùng để đảm bảo service luôn hoạt động ổn định.

---

## **5. Khai báo để tất cả services dùng chung 1 network và lợi ích**

### Cách khai báo:

```yaml
version: "3.8"

services:
  nginx:
    image: nginx
    networks:
      - mynetwork

  app:
    image: node
    networks:
      - mynetwork

networks:
  mynetwork:
    driver: bridge
```

### Lợi ích:

* Các container giao tiếp bằng **tên service** (DNS nội bộ)
* Tăng bảo mật (không cần mở port ra ngoài)
* Quản lý hệ thống dễ hơn

---

## **6. Đưa Cloudflare Token vào .env và .gitignore, tại sao quan trọng?**

### .env:

```env
CLOUDFLARE_TOKEN=your_token
```

### docker-compose:

```yaml
environment:
  - CLOUDFLARE_TOKEN=${CLOUDFLARE_TOKEN}
```

### .gitignore:

```
.env
```

### Ý nghĩa:

* Tránh lộ thông tin nhạy cảm (token, mật khẩu)
* Nếu bị lộ, người khác có thể chiếm quyền hệ thống
* Đây là nguyên tắc bảo mật cơ bản trong phát triển phần mềm

---

## **7. Tại sao thêm :ro khi mount file cấu hình Nginx?**

* `:ro` = read-only (chỉ đọc)
* Ngăn container chỉnh sửa file cấu hình
* Tránh lỗi và tăng bảo mật

## **8. Dùng Cloudflare Tunnel có cần mở cổng không?**

* **Không cần mở cổng**

Vì:

* Tunnel tạo kết nối outbound từ server đến Cloudflare
* Không cần mở port inbound trên firewall

→ Giúp hệ thống an toàn hơn và dễ triển khai.

