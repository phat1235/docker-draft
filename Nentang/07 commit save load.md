

# 🐳 Lệnh `docker commit`: Lưu container thành image mới

## Mục tiêu
Lệnh `docker commit` cho phép bạn **lưu trạng thái hiện tại của container** thành một Docker image mới. Điều này rất hữu ích khi:

- Bạn chỉnh sửa một container trực tiếp (cài thêm gói, chỉnh cấu hình, v.v)
- Bạn muốn "đóng băng" trạng thái hiện tại để dùng lại sau
- Bạn muốn tạo image từ container đang chạy thay vì viết Dockerfile


## Cú pháp

```bash
docker commit [OPTIONS] <container_id|container_name> <repository>:<tag>
```

### Một số tùy chọn hữu ích:

| Tùy chọn | Giải thích |
|---------|------------|
| `-a` hoặc `--author` | Tác giả của image |
| `-m` hoặc `--message` | Ghi chú ngắn về thay đổi |
| `--change` | Gán thêm lệnh Dockerfile (CMD, ENV, EXPOSE,...) |


## Ví dụ cơ bản

### Bước 1: Tạo container từ Ubuntu

```bash
docker run -it ubuntu bash
```

Trong container, bạn cài một số phần mềm:

```bash
apt update && apt install -y nginx
exit
```

### Bước 2: Dùng `docker commit` để lưu lại

```bash
docker ps -a
```

Giả sử container ID là `abc123`, bạn commit như sau:

```bash
docker commit -a "admin" -m "Cài nginx trên Ubuntu" abc123 ubuntu-nginx:v1
```
![](https://img001.prntscr.com/file/img001/YQqs7nQQTmeTGrvdhT6FkQ.png)
Kiểm tra image mới:

```bash
docker images
```


## Khởi động lại từ image mới

```bash
docker run -it ubuntu-nginx:v1 bash
```

→ Nginx đã được cài sẵn.

![](https://img001.prntscr.com/file/img001/7pK-QeaPQOW-yCPf_WfUgA.png)
## So sánh `docker commit` vs `docker build`

| Tiêu chí         | docker commit                         | docker build (Dockerfile)         |
|------------------|----------------------------------------|-----------------------------------|
| Kiểm soát         | Ít, khó tái hiện lại cùng kết quả     | Rõ ràng, dễ quản lý phiên bản     |
| Lặp lại           | Không đảm bảo                         | Dễ lặp lại, dễ CI/CD              |
| Tốt cho          | Thử nghiệm nhanh, chỉnh trực tiếp     | Dự án chuyên nghiệp, lâu dài     |


## Khi nào nên dùng `docker commit`?

- Khi bạn đang thử nghiệm nhanh và muốn lưu lại trạng thái container
- Khi chưa kịp viết Dockerfile
- Khi bạn muốn tạo image từ container cũ (backup, khôi phục)

**Lưu ý:** Dù `docker commit` tiện lợi, nhưng với môi trường production, bạn nên dùng `Dockerfile` để tạo image một cách rõ ràng, có thể kiểm soát và dễ tái tạo hơn.



Dưới đây là **bài viết kỹ thuật** hướng dẫn về cách **lưu file (container image)** và **nạp lại file (load image)** trong Docker, thường dùng để **chia sẻ**, **sao lưu** hoặc **triển khai hệ thống không có internet**.

---

# 🐳 Hướng dẫn lưu file và nạp lại file Docker Image

Trong nhiều trường hợp, bạn cần lưu lại Docker Image dưới dạng file `.tar` để:

- Sao lưu image phục vụ phục hồi sau này
- Chuyển image sang hệ thống khác (không có internet)
- Phân phối offline trong môi trường bảo mật
- Làm cache để cài đặt nhanh sau này

Docker hỗ trợ tính năng này qua hai lệnh: `docker save` và `docker load`.

---

## 1. Lưu Docker Image ra file `.tar` với `docker save`

Lệnh `docker save` cho phép bạn lưu một image (hoặc nhiều image) thành một file `.tar`.

### Cú pháp:
```bash
docker save -o <tên-file.tar> <tên-image>:<tag>
```

### Ví dụ:
```bash
docker save -o nginx-latest.tar nginx:latest
```
![](https://img001.prntscr.com/file/img001/1vIw5toFRw6swJ6cxLh9pA.png)

Lệnh trên sẽ tạo file `nginx-latest.tar` chứa toàn bộ image `nginx:latest`.

Bạn có thể lưu nhiều image cùng lúc (ít dùng):
```bash
docker save -o backup-images.tar nginx:latest alpine:3.18 ubuntu:20.04
```

---

## 2. Nạp lại Docker Image từ file `.tar` với `docker load`

Lệnh `docker load` giúp nạp image từ file `.tar` trở lại Docker.

### Cú pháp:
```bash
docker load -i <tên-file.tar>
```

### Ví dụ:
```bash
docker load -i nginx-latest.tar
```

Sau khi nạp xong, bạn có thể kiểm tra:
```bash
docker images
```
![](https://img001.prntscr.com/file/img001/-X_JQc46SIK4_qNhVWUPUg.png)
---

## 3. Trường hợp sử dụng điển hình

- **Di chuyển Docker Image giữa các máy tính:**  
  Máy A:
  ```bash
  docker save -o myapp.tar myapp:latest
  ```
  → chép `myapp.tar` sang máy B (bằng USB, SCP, v.v)

  Máy B:
  ```bash
  docker load -i myapp.tar
  ```

- **Sao lưu định kỳ image nội bộ để tránh phải build lại:**
  ```bash
  docker save -o backup-$(date +%F).tar myapp:latest
  ```

---

## 4. Lưu ý khi dùng

- File `.tar` có thể rất lớn nếu image chứa nhiều lớp (layer).
- `docker save` không lưu container, chỉ lưu image.
- Nếu bạn cần xuất **container**, hãy dùng `docker export` và `docker import` (trường hợp đặc biệt).

---

## So sánh nhanh: `docker save` vs `docker export`

| Lệnh            | Lưu gì?              | Dùng để khôi phục gì?         |
|-----------------|----------------------|-------------------------------|
| `docker save`   | Docker image (đầy đủ) | Image gốc với metadata        |
| `docker export` | Container filesystem  | Container (không có metadata) |

