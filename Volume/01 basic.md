---

# Chia sẻ dữ liệu giữa Docker Host và Container

Trong Docker, việc chia sẻ dữ liệu giữa máy chủ (Docker Host) và Container, cũng như giữa các Container với nhau là một phần quan trọng trong triển khai ứng dụng. Docker cung cấp nhiều cách để thực hiện việc chia sẻ này thông qua **Bind Mounts** và **Volumes**.

Bài viết này tập trung hướng dẫn các kỹ thuật:

- Gắn thư mục từ Docker Host vào Container (Bind Mount)
- Chia sẻ dữ liệu giữa các Container
- Quản lý Volumes
- So sánh giữa `-v` và `--mount`

---

## 1. Bind Mount: Gắn thư mục máy Host vào Container

### 1.1. Mô hình

- **Thư mục trên Host**: `/path/in/host`
- **Đường dẫn trong Container**: `/path/in/container`

Khi khởi tạo container, dùng tham số `-v` để ánh xạ thư mục từ máy chủ vào container:

```bash
docker run -it -v /path/in/host:/path/in/container ubuntu
```

### 1.2. Ví dụ

```bash
docker run -it -v /home/sitesdata:/home/data ubuntu
```

Khi đó, thư mục `/home/sitesdata` trên máy Host sẽ được gắn vào `/home/data` bên trong container. Container có thể đọc, ghi dữ liệu vào thư mục này như bình thường.

---

## 2. Chia sẻ dữ liệu giữa các Container

Một Container có thể chia sẻ Volume hoặc Bind Mount với Container khác thông qua tham số `--volumes-from`.

### 2.1. Ví dụ

Giả sử có container `container_first` đang chạy và đã có volume hoặc mount thư mục host.

Tạo container mới sử dụng lại mount từ `container_first`:

```bash
docker run -it --volumes-from container_first ubuntu
```

Container mới sẽ có quyền truy cập cùng dữ liệu với container `container_first`.

---

## 3. Quản lý Volume trong Docker

Docker Volume là cách tốt nhất để lưu trữ dữ liệu bền vững, độc lập với vòng đời container. Dữ liệu trong volume sẽ không bị mất khi container bị xóa.

### 3.1. Các lệnh cơ bản

- **Liệt kê volume hiện có**:

```bash
docker volume ls
```

- **Tạo volume mới**:

```bash
docker volume create volume_name
```

- **Xem thông tin chi tiết về volume**:

```bash
docker volume inspect volume_name
```

- **Xóa volume**:

```bash
docker volume rm volume_name
```

- **Xóa toàn bộ volume không còn sử dụng bởi container nào**:

```bash
docker volume prune
```

---

## 4. Mount Volume vào Container

Docker hỗ trợ 2 cách gắn volume: `--mount` (cú pháp mới, rõ ràng) và `-v` (cú pháp ngắn, cũ hơn).

### 4.1. Gắn volume với `--mount`

#### 4.1.1. Ví dụ:

Tạo volume:

```bash
docker volume create firstdisk
```

Chạy container và mount volume vào:

```bash
docker run -it --mount source=firstdisk,target=/home/firstdisk ubuntu
```

### 4.2. Gắn volume với `-v`

```bash
docker run -it -v firstdisk:/home/firstdisk ubuntu
```

---

## 5. Bind Volume với thư mục máy Host

Có thể tạo volume liên kết trực tiếp với thư mục máy chủ bằng tùy chọn `--opt`.

### 5.1. Ví dụ:

Tạo volume gắn với thư mục `/home/mydata` trên Host:

```bash
docker volume create \
  --opt type=none \
  --opt device=/home/mydata \
  --opt o=bind \
  mydisk
```

Mount volume này vào container bằng cú pháp `-v`:

```bash
docker run -it -v mydisk:/home/sites ubuntu
```

---

## 6. So sánh `-v` và `--mount`

| Thuộc tính               | `-v`                               | `--mount`                                |
|--------------------------|------------------------------------|-------------------------------------------|
| Cú pháp                  | Ngắn gọn                           | Rõ ràng, dễ đọc                            |
| Hỗ trợ nhiều tùy chọn    | Hạn chế                            | Đầy đủ, linh hoạt hơn                      |
| Phiên bản hỗ trợ         | Từ trước Docker 17.06             | Docker 17.06 trở lên                       |
| Khuyến nghị sử dụng      | Các tác vụ đơn giản               | Dành cho cấu hình phức tạp và rõ ràng     |

---

## 7. Tổng kết

| Tình huống                        | Giải pháp                     |
|----------------------------------|-------------------------------|
| Gắn thư mục máy Host vào container | Dùng `-v /host/path:/container/path` |
| Chia sẻ dữ liệu giữa containers   | Dùng `--volumes-from container_name` |
| Quản lý dữ liệu bền vững          | Sử dụng Docker Volume         |
| Mount volume cấu hình chi tiết   | Dùng `--mount`                |



## 8.Docker Volume và dữ liệu

Ngoài những kiến thức cơ bản đã trình bày ở trên, người dùng có thể tìm hiểu thêm các chủ đề sau để phục vụ các hệ thống phức tạp hơn, chuyên nghiệp hơn:

### 8.1. Sử dụng Volume Plugin để mount ổ đĩa mạng (NFS, CIFS, S3,...)

Docker hỗ trợ các plugin volume bên thứ ba giúp bạn:

- Gắn thư mục chia sẻ qua NFS
- Mount thư mục SMB/CIFS (Windows Share)
- Lưu trữ dữ liệu lên cloud (Amazon S3, Azure Blob Storage,...)

#### Ví dụ: Mount ổ đĩa NFS

```bash
docker volume create \
  --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.100,rw \
  --opt device=:/sharedfolder \
  nfsvolume
```

```bash
docker run -it -v nfsvolume:/data ubuntu
```

---

### 8.2. Volume Encryption và bảo mật dữ liệu

Một số plugin như [rexray](https://rexray.readthedocs.io/) hoặc các giải pháp như Docker + LUKS (Linux Unified Key Setup) cho phép:

- Mã hóa volume (encrypt-at-rest)
- Bảo vệ dữ liệu khỏi truy cập trái phép khi container bị tấn công hoặc bị chiếm quyền điều khiển
- Kết hợp với cơ chế ACL, SELinux, AppArmor để tăng cường bảo mật

---

### 8.3. So sánh hiệu năng giữa Bind Mount và Docker Volume

| Tiêu chí                  | Bind Mount                     | Docker Volume                |
|---------------------------|--------------------------------|------------------------------|
| Hiệu năng                 | Tùy thuộc ổ đĩa và hệ thống file host | Tối ưu với Docker Engine     |
| Tính di động              | Kém, phụ thuộc cấu trúc thư mục host | Tốt, có thể backup dễ dàng   |
| Bảo mật                   | Rủi ro cao hơn                 | Tốt hơn, ít phụ thuộc host   |
| Quản lý qua API           | Không                          | Có hỗ trợ Docker API         |
| Backup/Restore dễ dàng    | Khó, thủ công                  | Có thể tích hợp công cụ tự động hóa |

---

### 8.4. Gắn volume ở chế độ readonly hoặc tùy chỉnh quyền truy cập

Giới hạn quyền truy cập vào volume để tăng cường bảo mật hoặc tránh ghi đè dữ liệu.

#### Ví dụ:

```bash
docker run -it -v mydata:/app/data:ro ubuntu
```

Thư mục `/app/data` trong container sẽ chỉ được đọc, không ghi.

---

### 8.5. Tự động backup và phục hồi dữ liệu volume

Sử dụng kết hợp `docker cp`, `rsync`, `tar`, hoặc các công cụ như `Restic`, `Velero`, `Volume-backup` container:

- Tự động snapshot dữ liệu volume
- Lên lịch backup (cron)
- Push dữ liệu lên cloud hoặc remote storage

---

### 8.6. Dữ liệu volume trong môi trường Docker Swarm hoặc Kubernetes

- Volume cần được hỗ trợ bởi driver phân tán (Distributed Volume)
- Dùng các plugin như **GlusterFS**, **Ceph**, hoặc hệ thống lưu trữ cloud-native (Longhorn, Rook, Portworx)
- Tương thích với PVC (Persistent Volume Claim) trong Kubernetes

---

### 8.7. Dữ liệu container và CI/CD

Trong quá trình build/test ứng dụng:

- Volume có thể dùng để chia sẻ build cache
- Mount thư mục chứa mã nguồn để test trực tiếp
- Dễ dàng rollback khi có lỗi xảy ra


