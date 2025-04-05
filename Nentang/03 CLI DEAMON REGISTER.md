

## **Kiến trúc Docker: Daemon, CLI, Registry**

![](https://img001.prntscr.com/file/img001/6yxRxGQbR4iIQrOOS71xyw.png)
---
---
---

![](https://img001.prntscr.com/file/img001/0348uQL2Qx2Dap4jGL9l9w.png)
Kiến trúc của Docker được thiết kế theo mô hình client-server bao gồm ba thành phần chính:

### 1. **Docker Daemon (dockerd)**

Docker Daemon là thành phần chạy nền chịu trách nhiệm chính trong việc:

- Quản lý các đối tượng Docker như: **container**, **image**, **volume**, **network**, v.v.
- Lắng nghe các yêu cầu từ Docker CLI hoặc Docker API.
- Giao tiếp với các daemon Docker khác để quản lý container từ xa (ví dụ trong môi trường Swarm).

#### Chức năng chính:
- Xử lý các lệnh từ client (CLI hoặc API).
- Tạo và quản lý container.
- Quản lý image và volume.
- Xây dựng image từ Dockerfile.
- Kết nối và giao tiếp với registry để pull/push image.

### 2. **Docker CLI (Command Line Interface)**

Docker CLI là công cụ dòng lệnh mà người dùng sử dụng để tương tác với Docker Daemon.

Cú pháp thường thấy:
```bash
docker [OPTIONS] COMMAND [ARG...]
```

Ví dụ:
```bash
docker run -d --name web nginx
docker build -t myapp .
docker push myrepo/myimage
```

CLI sử dụng **REST API** để gửi yêu cầu đến Docker Daemon.

### 3. **Docker Registry**

Docker Registry là nơi lưu trữ và phân phối các **Docker images**.

- Docker sử dụng mặc định **Docker Hub** là registry công khai.
- Bạn có thể sử dụng registry riêng (private registry) bằng cách triển khai dịch vụ Docker Registry.

#### Các thao tác chính với Registry:
- **Pull** image từ registry:
  ```bash
  docker pull nginx:latest
  ```
- **Push** image lên registry:
  ```bash
  docker push myregistry.com/myimage:tag
  ```

#### Các thành phần của image trong Registry:
- Tên image (ví dụ: `nginx`)
- Tag (ví dụ: `latest`, `1.25`)
- Digest (SHA256 hash đại diện duy nhất cho nội dung image)

---

## **Tổng quan luồng hoạt động**

1. Người dùng gửi lệnh qua Docker CLI.
2. Docker CLI chuyển yêu cầu đến Docker Daemon thông qua Docker API.
3. Docker Daemon xử lý yêu cầu (ví dụ: tạo container, build image).
4. Nếu cần image, Docker Daemon sẽ truy vấn đến Docker Registry.
5. Container/image được tạo và quản lý bởi Daemon.

---
