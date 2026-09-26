# BÀI TẬP 2

Bài tập thực hành xây dựng API kiểm tra độ mạnh mật khẩu bằng Node-RED, cấu hình Nginx để website có thể kết nối tới API và sử dụng JavaScript để gửi dữ liệu, nhận kết quả JSON và hiển thị trực tiếp trên giao diện.

## 1. Tạo API trên Node-RED

Trước khi thực hiện, kiểm tra trạng thái các container bằng lệnh:

```bash
docker compose ps
```

Các dịch vụ Nginx, Node-RED, MariaDB, phpMyAdmin và Cloudflared đều đang hoạt động.

![Kiểm tra trạng thái các container](./images/baitap2/docker-compose-ps.png)

Truy cập Node-RED tại địa chỉ:

```text
http://localhost:1880
```

![Giao diện Node-RED](./images/baitap2/nodered-interface.png)

### 1.1. Tạo luồng xử lý API

Sử dụng ba node `http in`, `function` và `http response` để xây dựng API.

```text
http in → function → http response
```

Trong đó:

- `http in`: nhận yêu cầu kiểm tra mật khẩu.
- `function`: xử lý và đánh giá độ mạnh mật khẩu.
- `http response`: trả kết quả về dưới dạng JSON.

![Luồng API trên Node-RED](./images/baitap2/nodered-api-flow.png)

### 1.2. Cấu hình HTTP In

Node `http in` sử dụng phương thức:

```text
GET
```

Đường dẫn API:

```text
/api/check-password
```

Mật khẩu được truyền vào thông qua tham số `password`.

Ví dụ:

```text
/api/check-password?password=Hello123!
```

![Cấu hình HTTP In](./images/baitap2/http-in-config.png)

### 1.3. Xây dựng thuật toán kiểm tra mật khẩu

Node `function` kiểm tra mật khẩu dựa trên 5 tiêu chí:

- Có ít nhất 8 ký tự.
- Có chữ cái viết hoa.
- Có chữ cái viết thường.
- Có chữ số.
- Có ký tự đặc biệt.

Mỗi tiêu chí thỏa mãn được cộng 1 điểm, tổng điểm tối đa là 5.

Mức độ mật khẩu:

- 0 - 2 điểm: Yếu.
- 3 - 4 điểm: Trung bình.
- 5 điểm: Mạnh.

![Cấu hình Function kiểm tra mật khẩu](./images/baitap2/function-config.png)

### 1.4. Kiểm tra API

Sau khi hoàn thành luồng xử lý, nhấn `Deploy` và kiểm tra API trên trình duyệt:

```text
http://localhost:1880/api/check-password?password=Hello123!
```

API trả về dữ liệu JSON:

```json
{
  "ok": 1,
  "score": 5,
  "level": "Mạnh",
  "message": "Mật khẩu có độ bảo mật tốt"
}
```

![Kết quả API Node-RED](./images/baitap2/api-result.png)


## 2. Cấu hình Nginx để website gọi API Node-RED

Để website có thể gọi API mà không cần truy cập trực tiếp cổng `1880`, Nginx được cấu hình để chuyển tiếp các request có đường dẫn `/api/` tới Node-RED.

Thêm cấu hình:

```nginx
location /api/ {
    proxy_pass http://nodered:1880;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
}
```

![Cấu hình Nginx cho API](./images/baitap2/nginx-api-config.png)

Kiểm tra cấu hình Nginx:

```bash
docker compose exec nginx nginx -t
```

![Kiểm tra cấu hình Nginx](./images/baitap2/nginx-test.png)

Sau khi cấu hình hợp lệ, reload Nginx:

```bash
docker compose exec nginx nginx -s reload
```

Khi đó request:

```text
/api/check-password
```

sẽ được Nginx chuyển tiếp tới API đang chạy trên Node-RED.


## 3. Sử dụng JavaScript trên website để gọi API

Website 2 được xây dựng thành giao diện kiểm tra độ mạnh mật khẩu.

Người dùng có thể:

- Nhập mật khẩu cần kiểm tra.
- Hiện hoặc ẩn mật khẩu.
- Nhấn nút `Kiểm tra`.
- Xem điểm và mức độ bảo mật của mật khẩu.

JavaScript sử dụng `fetch()` để gửi mật khẩu tới API:

```javascript
const response = await fetch(
    "/api/check-password?password=" +
    encodeURIComponent(password)
);

const data = await response.json();
```

`encodeURIComponent()` được sử dụng để mã hóa giá trị mật khẩu trước khi đưa vào URL.

Dữ liệu JSON trả về từ API được JavaScript xử lý và hiển thị trực tiếp trên giao diện.

![Code JavaScript gọi API](./images/baitap2/javascript-fetch.png)

### 3.1. Kiểm tra hoạt động của website

Truy cập Website 2 thông qua tên miền đã cấu hình.

Nhập mật khẩu và nhấn nút `Kiểm tra`. Website sẽ gửi request tới API và hiển thị kết quả trả về.

Quá trình hoạt động:

```text
Người dùng nhập mật khẩu
        ↓
JavaScript fetch()
        ↓
Nginx
        ↓
Node-RED API
        ↓
Function kiểm tra mật khẩu
        ↓
JSON
        ↓
Website hiển thị kết quả
```

![Website gọi API thành công](./images/baitap2/website-api-result.png)


## 4. Kết quả

Hoàn thành Bài tập 2 với ba nội dung chính:

- Xây dựng API kiểm tra độ mạnh mật khẩu trên Node-RED bằng `http in`, `function` và `http response`.
- Cấu hình Nginx để website có thể gửi request tới API Node-RED.
- Sử dụng JavaScript `fetch()` trên trang HTML để gọi API, nhận dữ liệu JSON và hiển thị kết quả.

API có khả năng kiểm tra mật khẩu dựa trên 5 tiêu chí và trả về số điểm, mức độ bảo mật cùng nội dung đánh giá.

Website có giao diện nhập mật khẩu, chức năng hiện/ẩn mật khẩu và hiển thị kết quả kiểm tra trực tiếp cho người dùng.
