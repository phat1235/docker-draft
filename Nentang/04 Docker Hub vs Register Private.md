## 🐳 **Docker Hub**

### 1. **Là gì?**
Docker Hub là **Docker Registry công khai mặc định** được vận hành bởi Docker Inc. Nó đóng vai trò là **kho lưu trữ các Docker image** mà bạn có thể:
- **Pull** (tải về) image để sử dụng.
- **Push** (đẩy lên) image đã build từ local.
- Tìm kiếm, chia sẻ và cộng tác qua namespace cá nhân hoặc tổ chức.

### 2. **Tính năng chính**
| Tính năng | Mô tả |
|----------|------|
| Public Repository | Miễn phí, ai cũng có thể pull về. |
| Private Repository | Có giới hạn nếu tài khoản miễn phí, không giới hạn với tài khoản trả phí. |
| Automated Build | Tự động build image từ GitHub/GitLab. |
| Web UI | Giao diện quản lý trực quan qua https://hub.docker.com |
| Official Images | Bộ image đã được kiểm định, bảo trì bởi Docker hoặc cộng đồng lớn. |

### 3. **Sử dụng**
```bash
# Pull image từ Docker Hub
docker pull nginx:latest

# Login vào Docker Hub
docker login

# Tag image để chuẩn bị đẩy lên
docker tag myimage dockerhub_username/myimage:tag

# Push image lên Docker Hub
docker push dockerhub_username/myimage:tag
```

---

## 🛠️ **Private Docker Registry (Registry nội bộ)**

### 1. **Là gì?**
Registry nội bộ là **hệ thống lưu trữ Docker image riêng**, cho phép bạn **kiểm soát hoàn toàn** việc lưu trữ, phân phối và bảo mật các image trong mạng nội bộ hoặc nội bộ doanh nghiệp.

> Docker cung cấp image chính thức để triển khai Registry riêng:  
> `registry:2`

### 2. **Tạo Registry nội bộ**
```bash
# Chạy container Docker Registry riêng
docker run -d -p 5000:5000 --name registry registry:2
```

Điều này tạo ra một registry hoạt động tại `http://localhost:5000`.

### 3. **Sử dụng registry nội bộ**
```bash
# Tag image với địa chỉ Registry nội bộ
docker tag myapp localhost:5000/myapp

# Push image vào Registry
docker push localhost:5000/myapp

# Pull image từ Registry
docker pull localhost:5000/myapp
```

> ⚠️ Lưu ý: Theo mặc định, Docker yêu cầu registry phải dùng HTTPS. Nếu bạn dùng HTTP, cần cấu hình **insecure registry** trong Docker daemon (file `/etc/docker/daemon.json`):
```json
{
  "insecure-registries" : ["localhost:5000"]
}
```
Sau đó restart Docker daemon:
```bash
sudo systemctl restart docker
```

---

## 🔐 Registry nội bộ có thể mở rộng với:
- **Authentication**: Sử dụng `htpasswd`, LDAP,...
- **TLS**: Kích hoạt SSL để bảo mật truyền tải.
- **UI quản lý**: Triển khai thêm Harbor, Portus, hoặc Docker Registry UI để dễ quản lý.
- **Storage backend**: Lưu trữ image trên filesystem, S3, GCS, Azure Blob Storage,...

---

## 🧾 So sánh nhanh: Docker Hub vs Private Registry

| Tiêu chí | Docker Hub | Registry nội bộ |
|---------|-------------|-----------------|
| Loại | Công khai (mặc định) | Riêng tư (tùy chọn triển khai) |
| URL | hub.docker.com | Tùy cấu hình, ví dụ `localhost:5000` |
| Bảo mật | Theo tài khoản Docker | Tùy chỉnh, toàn quyền kiểm soát |
| Phân phối | Qua internet | Qua mạng nội bộ hoặc internet riêng |
| Dễ triển khai | Không cần setup | Cần tự thiết lập và quản lý |
| Hạn chế | Hạn chế số lượng repo/image nếu dùng free | Không giới hạn, phụ thuộc vào tài nguyên máy chủ |

---

 **Harbor** làm private registry nâng cao!
