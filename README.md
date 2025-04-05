## 🧱 PHẦN 1: NỀN TẢNG & CƠ BẢN

### 🔍 Kiến thức:
- [ ] Docker là gì? So sánh với máy ảo (VM)
- [ ] Image vs Container vs Volume vs Network
- [ ] Kiến trúc Docker: Daemon, CLI, Registry
- [ ] Docker Hub và Registry nội bộ

### 🧪 Thực hành:
- [ ] Cài đặt Docker (Linux / WSL2 / Mac)
- [ ] Chạy container đầu tiên (`hello-world`, `nginx`, `ubuntu`)
- [ ] Làm việc với images (`pull`, `build`, `tag`, `push`)
- [ ] Quản lý container (`run`, `start`, `stop`, `rm`, `exec`, `logs`)
- [ ] Mount volumes & bind mounts
- [ ] Viết Dockerfile đơn giản (base image, CMD, EXPOSE...)

---

## ⚙️ PHẦN 2: VẬN HÀNH & QUẢN LÝ

### 🔍 Kiến thức:
- [ ] Docker Compose: Cấu trúc và cách dùng
- [ ] Docker Volumes & Bind Mount chi tiết
- [ ] Docker Networks: bridge, host, overlay
- [ ] Xem logs, thống kê, tài nguyên
- [ ] Docker Registry nội bộ

### 🧪 Thực hành:
- [ ] Viết `docker-compose.yml` chạy multi-container (nginx + php + db)
- [ ] Mount volume để giữ dữ liệu DB
- [ ] Theo dõi log và stats container
- [ ] Tạo & sử dụng Docker Registry nội bộ

---

## 🔐 PHẦN 3: BẢO MẬT & QUẢN LÝ TÀI NGUYÊN

### 🔍 Kiến thức:
- [ ] Bảo mật Docker Daemon (TLS, socket)
- [ ] Chạy container không dùng root
- [ ] Docker Secrets & Configs
- [ ] Giới hạn tài nguyên (`--memory`, `--cpus`)
- [ ] Chính sách khởi động lại container
- [ ] Healthchecks cho container

### 🧪 Thực hành:
- [ ] Tạo container có giới hạn RAM/CPU
- [ ] Viết `HEALTHCHECK` trong Dockerfile
- [ ] Dùng `--restart` trong `docker run` hoặc compose
- [ ] Tạo user trong container để tránh root
- [ ] Dùng secrets trong `docker-compose`

---

## ☁️ PHẦN 4: NÂNG CAO – ORCHESTRATION & CLUSTER

### 🔍 Kiến thức:
- [ ] Docker Swarm (node, manager, worker)
- [ ] Stack deployment và rolling update
- [ ] Overlay network nhiều node
- [ ] Load balancing nội bộ
- [ ] Auto-scaling và constraint

### 🧪 Thực hành:
- [ ] Tạo cụm Swarm (1 manager + 2 worker)
- [ ] Deploy stack sử dụng Compose
- [ ] Rolling update dịch vụ
- [ ] Gán labels và constraint cho node

---

## 🛠 PHẦN 5: CÔNG CỤ QUẢN LÝ HỖ TRỢ

### 🔍 Công cụ:
- [ ] Portainer – giao diện web quản lý Docker
- [ ] LazyDocker – CLI/TUI đẹp và tiện
- [ ] Docker Slim – tối ưu image nhỏ gọn
- [ ] Docker Scout – phân tích bảo mật image

### 🧪 Thực hành:
- [ ] Cài và cấu hình Portainer
- [ ] Quản lý toàn bộ container qua web
- [ ] Phân tích image với Docker Scout
- [ ] Giảm size image bằng Docker Slim

---

## 🎯 MỤC TIÊU CUỐI CÙNG (PROJECT GỢI Ý)
- [ ] Deploy ứng dụng thực tế bằng Docker Compose (VD: WordPress, GitLab, Nextcloud)
- [ ] Triển khai hệ thống phát triển (dev/test) bằng Docker
- [ ] Tạo CI/CD pipeline đơn giản có dùng Docker (GitHub Actions hoặc GitLab CI)
