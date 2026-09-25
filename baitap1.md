
# BÀI TẬP 1

## 1. Giả lập hệ điều hành Linux

Sử dụng **VMware Workstation** để tạo máy ảo và cài đặt hệ điều hành **Ubuntu Linux**.

### Kiểm tra phiên bản hệ điều hành

```bash
cat /etc/os-release
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
