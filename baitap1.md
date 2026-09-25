# BÀI TẬP 1

## 1. Giả lập hệ điều hành Linux

Sử dụng **VMware Workstation** để tạo máy ảo và cài đặt hệ điều hành **Ubuntu Linux**.

### Kiểm tra phiên bản hệ điều hành

```bash
cat /etc/os-release
```

Kết quả Ubuntu đã được cài đặt và hoạt động trên VMware:

<p align="center">
  <img src="images/baitap1/ubuntu_ver.png" width="850">
</p>

---

## 2. Cài đặt Docker Compose trên Ubuntu

Cập nhật danh sách các gói phần mềm:

```bash
sudo apt update
```

Cài đặt Docker và Docker Compose:

```bash
sudo apt install docker.io docker-compose-v2 -y
```

### Kiểm tra phiên bản

```bash
docker --version
docker compose version
```

Kết quả Docker và Docker Compose đã được cài đặt thành công:

<p align="center">
  <img src="images/baitap1/docker_compose_version.png" width="850">
</p>
---

## 3. Triển khai các dịch vụ bằng Docker Compose

Sử dụng Docker Compose để triển khai các dịch vụ cần thiết cho hệ thống gồm **Nginx, Node-RED, MariaDB và phpMyAdmin**. Cloudflared sẽ được cấu hình ở bước tiếp theo.

### 3.1. Khởi chạy các dịch vụ

Các dịch vụ được cấu hình trong file:

```text
compose.yaml
```

Khởi chạy toàn bộ container:

```bash
docker compose up -d
```

Kiểm tra trạng thái hoạt động:

```bash
docker compose ps
```

Kết quả cho thấy các container **Nginx, Node-RED, MariaDB và phpMyAdmin** đã được khởi chạy thành công.

<p align="center">
  <img src="images/baitap1/docker_compose_service.png" width="850">
</p>

### 3.2. Kiểm tra Nginx

Nginx được sử dụng làm Web Server và chạy trên cổng `8080`.

Truy cập:

```text
http://localhost:8080
```

Kết quả cho thấy Nginx đã hoạt động và có thể phục vụ nội dung website.

<p align="center">
 <img src="images/baitap1/nginx.png" width="850">
</p>
### 3.3. Kiểm tra Node-RED

Node-RED được triển khai bằng Docker Compose và chạy trên cổng `1880`.

Truy cập:

```text
http://localhost:1880
```

Giao diện Node-RED sau khi khởi động thành công:

<p align="center">
  <img src="images/baitap1/nodered.png" width="850">
</p>

### 3.4. Kiểm tra MariaDB và phpMyAdmin

MariaDB được sử dụng để lưu trữ và quản lý dữ liệu. phpMyAdmin cung cấp giao diện web giúp quản lý cơ sở dữ liệu MariaDB.

phpMyAdmin được chạy trên cổng `8081`.

Truy cập:

```text
http://localhost:8081
```

Sau khi đăng nhập, phpMyAdmin kết nối thành công với MariaDB và có thể truy cập cơ sở dữ liệu đã cấu hình.

<p align="center">
  <img src="images/baitap1/phpmyadmin-mariadb.png" width="850">
</p>

### 3.5. Cloudflared

Cloudflared được sử dụng để tạo Cloudflare Tunnel, giúp đưa các dịch vụ đang chạy trong máy ảo ra Internet thông qua domain.

Phần cấu hình Cloudflared và domain sẽ được thực hiện ở bước tiếp theo.
