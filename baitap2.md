# BÀI TẬP 2

Bài tập thực hành tạo API bằng Node-RED, cấu hình Nginx để website có thể truy cập API và sử dụng JavaScript trên trang HTML để gọi API.

## 1. Sử dụng Node-RED tạo API đơn giản

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

### 1.1. Tạo luồng API

Trên Node-RED, sử dụng các node để xây dựng API:

```text
http in → function → http response
```

Trong đó:

- `http in`: nhận request gửi đến API.
- `function`: xử lý dữ liệu.
- `http response`: trả kết quả về cho client.

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

Mật khẩu cần kiểm tra được truyền thông qua tham số `password`.

Ví dụ:

```text
/api/check-password?password=Hello123!
```

![Cấu hình HTTP In](./images/baitap2/http-in-config.png)

Sau khi hoàn thành các node, nhấn `Deploy` để API bắt đầu hoạt động.


## 2. Cấu hình Nginx và xây dựng thuật toán cho API

### 2.1. Xây dựng thuật toán kiểm tra độ mạnh mật khẩu

API được xây dựng với chức năng kiểm tra độ mạnh của mật khẩu.

Node `function` nhận mật khẩu từ request và kiểm tra 5 tiêu chí:

- Có ít nhất 8 ký tự.
- Có chữ cái viết hoa.
- Có chữ cái viết thường.
- Có chữ số.
- Có ký tự đặc biệt.

Mỗi tiêu chí thỏa mãn được cộng 1 điểm, tổng điểm tối đa là 5.

Kết quả được phân loại:

- 0 - 2 điểm: Yếu.
- 3 - 4 điểm: Trung bình.
- 5 điểm: Mạnh.

![Cấu hình Function kiểm tra mật khẩu](./images/baitap2/function-config.png)

Sau khi xử lý, API trả về dữ liệu dưới dạng JSON.

Ví dụ khi kiểm tra mật khẩu `Hello123!`:

```json
{
  "ok": 1,
  "score": 5,
  "level": "Mạnh",
  "message": "Mật khẩu có độ bảo mật tốt"
}
```

Kiểm tra API trực tiếp trên trình duyệt:

```text
http://localhost:1880/api/check-password?password=Hello123!
```

![Kết quả API Node-RED](./images/baitap2/api-result.png)

### 2.2. Cấu hình Nginx kết nối tới Node-RED

Để website có thể gọi API Node-RED thông qua đường dẫn `/api/`, cấu hình Nginx chuyển tiếp request tới dịch vụ Node-RED.

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

Khi đó, request từ website tới:

```text
/api/check-password
```

sẽ được Nginx chuyển tiếp tới Node-RED.


## 3. Code JavaScript trên trang HTML để gọi API

Website 2 được sử dụng làm giao diện kiểm tra độ mạnh mật khẩu.

Giao diện cho phép người dùng:

- Nhập mật khẩu cần kiểm tra.
- Hiện hoặc ẩn mật khẩu.
- Nhấn nút `Kiểm tra`.
- Xem điểm và mức độ bảo mật của mật khẩu.

JavaScript sử dụng `fetch()` để gọi API:

```javascript
const response = await fetch(
    "/api/check-password?password=" +
    encodeURIComponent(password)
);

const data = await response.json();
```

Trong đó, `encodeURIComponent()` được sử dụng để mã hóa mật khẩu trước khi đưa vào URL.

Sau khi nhận dữ liệu JSON từ API, JavaScript lấy các giá trị `score`, `level` và `message` để hiển thị kết quả trên trang HTML.

![Code JavaScript gọi API](./images/baitap2/javascript-fetch.png)

### 3.1. Kiểm tra kết quả trên website

Người dùng nhập mật khẩu và nhấn nút `Kiểm tra`.

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
Function xử lý
        ↓
JSON
        ↓
JavaScript hiển thị kết quả
```

Kết quả kiểm tra được hiển thị trực tiếp trên website.

![Website gọi API thành công](./images/baitap2/website-api-result.png)


## 4. Kết quả

Hoàn thành các yêu cầu của Bài tập 2:

- Sử dụng Node-RED với `http in` và `http response` để tạo API.
- Xây dựng thuật toán kiểm tra độ mạnh mật khẩu và cấu hình Nginx để website có thể truy cập API Node-RED.
- Sử dụng JavaScript trên trang HTML để gọi API, nhận dữ liệu JSON và hiển thị kết quả.

API kiểm tra mật khẩu dựa trên 5 tiêu chí và trả về số điểm, mức độ bảo mật cùng nội dung đánh giá.

Website có thể nhập mật khẩu, hiện hoặc ẩn mật khẩu và hiển thị kết quả kiểm tra trực tiếp cho người dùng.
