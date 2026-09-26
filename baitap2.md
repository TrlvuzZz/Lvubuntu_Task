# BÀI TẬP 2

Bài tập thực hành xây dựng API đơn giản bằng Node-RED, cấu hình Nginx làm trung gian giữa website và Node-RED, sau đó sử dụng JavaScript trên trang HTML để gửi yêu cầu đến API và xử lý dữ liệu JSON trả về.

## 1. Kiểm tra hệ thống Docker Compose

Trước khi thực hiện bài tập, kiểm tra trạng thái các container đã được triển khai từ Bài tập 1.

```bash
docker compose ps
```

Các dịch vụ Nginx, Node-RED, MariaDB, phpMyAdmin và Cloudflared đều đang hoạt động.

![Kiểm tra trạng thái các container](./images/baitap2/docker-compose-ps.png)

## 2. Tạo API trên Node-RED

Truy cập giao diện Node-RED thông qua địa chỉ:

```text
http://localhost:1880
```

Node-RED đang hoạt động và có thể truy cập từ trình duyệt trên máy Ubuntu.

![Giao diện Node-RED](./images/baitap2/nodered-interface.png)

### 2.1. Tạo luồng xử lý API

Sử dụng các node `http in`, `function` và `http response` để xây dựng một API đơn giản.

Luồng xử lý của API:

```text
http in → function → http response
```

Trong đó:

- `http in`: tiếp nhận yêu cầu từ website.
- `function`: xử lý dữ liệu và tạo kết quả trả về.
- `http response`: gửi kết quả từ Node-RED về cho phía website.

![Luồng API trên Node-RED](./images/baitap2/nodered-api-flow.png)

### 2.2. Cấu hình HTTP In

Node `http in` được cấu hình để nhận yêu cầu HTTP từ người dùng.

Phương thức:

```text
GET
```

Đường dẫn API:

```text
/api/xep-loai
```

API nhận điểm của sinh viên thông qua tham số `diem`.

Ví dụ:

```text
/api/xep-loai?diem=8.5
```

![Cấu hình HTTP In](./images/baitap2/http-in-config.png)

### 2.3. Xử lý dữ liệu bằng Function

Node `function` nhận giá trị điểm từ request, kiểm tra dữ liệu và thực hiện xếp loại.

Quy tắc xếp loại:

- Điểm từ 8 trở lên: Giỏi.
- Điểm từ 6.5 đến dưới 8: Khá.
- Điểm từ 5 đến dưới 6.5: Trung bình.
- Điểm dưới 5: Yếu.

Kết quả sau khi xử lý được chuyển thành dữ liệu JSON và gửi tới node `http response`.

![Cấu hình Function](./images/baitap2/function-config.png)

### 2.4. Kiểm tra API Node-RED

Sau khi hoàn thành luồng xử lý, tiến hành Deploy và kiểm tra API trên trình duyệt.

Ví dụ:

```text
http://localhost:1880/api/xep-loai?diem=8.5
```

API trả về dữ liệu JSON chứa điểm và kết quả xếp loại tương ứng.

Ví dụ:

```json
{
  "ok": 1,
  "diem": 8.5,
  "xepLoai": "Giỏi"
}
```

![Kết quả API Node-RED](./images/baitap2/api-result.png)

## 3. Cấu hình Nginx kết nối với Node-RED

Để website có thể gọi API của Node-RED, cấu hình Nginx chuyển tiếp các request có đường dẫn `/api/` tới dịch vụ Node-RED.

Sau khi chỉnh sửa cấu hình, kiểm tra lại cú pháp Nginx:

```bash
docker compose exec nginx nginx -t
```

Nếu cấu hình hợp lệ, Nginx sẽ thông báo kiểm tra thành công.

![Kiểm tra cấu hình Nginx](./images/baitap2/nginx-test.png)

Sau đó tiến hành reload Nginx để áp dụng cấu hình mới.

![Cấu hình Nginx cho API](./images/baitap2/nginx-api-config.png)

## 4. Viết JavaScript gọi API từ website

Trang web được bổ sung giao diện cho phép nhập điểm của sinh viên và gửi yêu cầu tới API.

JavaScript sử dụng `fetch()` để gọi API thông qua đường dẫn:

```text
/api/xep-loai?diem=...
```

Dữ liệu JSON nhận được từ API sẽ được xử lý và hiển thị trực tiếp trên giao diện website.

![Code JavaScript gọi API](./images/baitap2/javascript-fetch.png)

## 5. Kiểm tra website gọi API

Truy cập website thông qua tên miền đã cấu hình ở Bài tập 1.

Nhập điểm cần kiểm tra và nhấn nút xếp loại. Website gửi request đến Nginx, Nginx chuyển tiếp request đến Node-RED và nhận kết quả JSON trả về.

Kết quả xếp loại sau đó được JavaScript hiển thị trên giao diện.

![Website gọi API thành công](./images/baitap2/website-api-result.png)

## 6. Kết quả

Hoàn thành việc xây dựng API đơn giản bằng Node-RED với các node `http in`, `function` và `http response`.

Nginx được cấu hình để chuyển tiếp request từ website tới Node-RED. JavaScript trên trang HTML có thể gọi API, nhận dữ liệu JSON và hiển thị kết quả lên giao diện.

Luồng hoạt động của hệ thống:

```text
Website
   ↓
JavaScript fetch()
   ↓
Nginx
   ↓
Node-RED API
   ↓
Xử lý dữ liệu
   ↓
JSON
   ↓
Website hiển thị kết quả
```
