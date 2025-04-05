# Hướng Dẫn Cài Đặt Docker và Docker Compose Trên Windows, Linux, và macOS

Docker và Docker Compose là hai công cụ quan trọng trong việc phát triển và triển khai ứng dụng hiện đại. Docker cho phép đóng gói ứng dụng cùng các phụ thuộc vào container, đảm bảo khả năng chạy ổn định trên mọi môi trường. Docker Compose giúp định nghĩa và quản lý các ứng dụng nhiều container thông qua một tệp cấu hình YAML.

Bài viết này sẽ hướng dẫn bạn cài đặt Docker và Docker Compose trên ba hệ điều hành phổ biến: Windows, Linux và macOS, kèm theo các bước kiểm tra và tối ưu sau khi cài đặt.

---

## Docker và Docker Compose là gì?

- **Docker**: Là một nền tảng mã nguồn mở giúp đóng gói, phân phối và chạy các ứng dụng trong môi trường container hóa. Docker đảm bảo môi trường phát triển nhất quán và có thể triển khai trên bất kỳ hệ thống nào hỗ trợ Docker.

- **Docker Compose**: Là công cụ dùng để định nghĩa và chạy các ứng dụng Docker đa container. Sử dụng tệp `docker-compose.yml`, người dùng có thể cấu hình nhiều dịch vụ và khởi động toàn bộ hệ thống bằng một lệnh duy nhất.

---

## 1. Cài đặt Docker và Docker Compose trên Windows

### Bước 1: Tải Docker Desktop cho Windows
- Truy cập trang chính thức của Docker: [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
- Chọn phiên bản phù hợp với Windows 10/11 Pro hoặc Enterprise (ưu tiên có hỗ trợ WSL 2).

### Bước 2: Cài đặt Docker Desktop
- Mở tệp cài đặt vừa tải về và làm theo hướng dẫn.
- Trong quá trình cài đặt, chọn sử dụng WSL 2 (nếu được hỏi).
- Khởi động lại máy sau khi cài đặt.

### Bước 3: Bật WSL 2 nếu chưa có
- Mở PowerShell với quyền Administrator và chạy:
```bash
wsl --install
```
- Sau khi hoàn tất, khởi động lại máy tính.

### Bước 4: Kiểm tra cài đặt
Mở PowerShell hoặc Command Prompt và chạy:
```bash
docker --version
docker-compose --version
```
Nếu mọi thứ cài đặt đúng, bạn sẽ thấy thông tin phiên bản hiển thị.

---

## 2. Cài đặt Docker và Docker Compose trên Linux (Ubuntu/Debian)

### Bước 1: Cập nhật hệ thống
```bash
sudo apt update && sudo apt upgrade -y
```

### Bước 2: Cài đặt các gói phụ thuộc
```bash
sudo apt install -y ca-certificates curl gnupg
```

### Bước 3: Thêm GPG key và kho lưu trữ Docker
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Bước 4: Cài đặt Docker Engine
```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

### Bước 5: Thêm người dùng vào nhóm `docker`
```bash
sudo usermod -aG docker $USER
```
Đăng xuất và đăng nhập lại để thay đổi có hiệu lực.

### Bước 6: Cài đặt Docker Compose
```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" \
-o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
```

### Bước 7: Kiểm tra Docker Compose
```bash
docker-compose --version
```

---

## 3. Cài đặt Docker và Docker Compose trên macOS

### Bước 1: Tải Docker Desktop cho macOS
- Truy cập [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
- Chọn phiên bản dành cho Intel hoặc Apple Silicon.

### Bước 2: Cài đặt Docker Desktop
- Mở tệp `.dmg` vừa tải.
- Kéo biểu tượng Docker vào thư mục `Applications`.

### Bước 3: Khởi động Docker
- Vào `Applications` và chạy Docker.
- Sau khi Docker khởi động, mở Terminal và kiểm tra:

```bash
docker --version
docker-compose --version
```

---

## Sau khi cài đặt: Các bước khuyến nghị

### 1. Bật Docker khi khởi động cùng hệ thống
- Trên Linux:
```bash
sudo systemctl enable docker
```
- Trên Windows/macOS: mặc định Docker Desktop tự động khởi động.

### 2. Sử dụng Docker không cần `sudo` (Linux)
Đảm bảo bạn đã thêm người dùng vào nhóm `docker` và đăng nhập lại.

### 3. Kiểm tra tương thích giữa Docker và Docker Compose
Tham khảo các phiên bản tương thích trong phần phát hành chính thức.

### 4. Xử lý sự cố
Truy cập tài liệu chính thức tại [docs.docker.com](https://docs.docker.com) hoặc tìm kiếm trên cộng đồng như Stack Overflow.

---

## Kết luận

Docker và Docker Compose giúp đơn giản hóa việc phát triển, kiểm thử và triển khai phần mềm trong môi trường cô lập và nhất quán. Việc cài đặt đúng cách trên hệ điều hành của bạn là bước đầu tiên để khai thác sức mạnh của container hóa. Hãy làm theo hướng dẫn từng bước để bắt đầu hành trình phát triển với Docker một cách hiệu quả.
