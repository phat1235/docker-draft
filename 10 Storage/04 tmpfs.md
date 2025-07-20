**tmpfs mounts (Liên kết bộ nhớ tạm thời)**

[Volumes](volumes.md) và [bind mounts](bind-mounts.md) là các cơ chế giúp chia sẻ tệp giữa máy chủ (host) và container, cho phép lưu trữ dữ liệu bền vững ngay cả sau khi container bị dừng.

Nếu bạn đang chạy Docker trên hệ điều hành Linux, bạn có thêm một tùy chọn thứ ba: **tmpfs mounts**. Khi bạn tạo container sử dụng tmpfs mount, container có thể tạo tệp bên ngoài lớp ghi (writable layer) của chính nó.

Khác với volumes và bind mounts, tmpfs mount là loại lưu trữ **tạm thời**, chỉ tồn tại trong **bộ nhớ RAM** của máy chủ. Khi container dừng, phần lưu trữ tmpfs sẽ bị gỡ bỏ và mọi dữ liệu trong đó **không được giữ lại**.

tmpfs mounts thích hợp trong các tình huống mà dữ liệu **không cần tồn tại lâu dài**, dù là trên máy chủ hay bên trong container. Điều này thường áp dụng vì lý do **bảo mật**, hoặc để **tối ưu hiệu năng**, đặc biệt khi ứng dụng cần ghi một lượng lớn dữ liệu trạng thái không bền vững.

> **Lưu ý quan trọng:** tmpfs trong Docker ánh xạ trực tiếp đến [tmpfs](https://en.wikipedia.org/wiki/Tmpfs) trong nhân Linux. Vì vậy, dữ liệu tạm có thể bị ghi xuống ổ đĩa thông qua file hoán đổi (swap), dẫn đến việc có thể lưu lại trên hệ thống tệp.

---

### Ghi đè dữ liệu có sẵn

Nếu bạn mount tmpfs vào một thư mục bên trong container đã chứa tệp hoặc thư mục, những nội dung sẵn có sẽ bị **che khuất** bởi dữ liệu từ tmpfs. Tương tự như khi bạn gắn một ổ USB vào thư mục `/mnt` trên máy Linux — nội dung cũ sẽ không thể truy cập cho đến khi bạn gỡ ổ USB.

Trong container, **không có cách đơn giản để gỡ mount và truy lại dữ liệu đã bị che**. Cách tốt nhất là **tạo lại container mà không gắn tmpfs**.

---

### Hạn chế của tmpfs mounts

* Không giống volumes hay bind mounts, tmpfs mount **không thể chia sẻ giữa nhiều container**.
* Tính năng này **chỉ khả dụng trên hệ điều hành Linux**.
* Phân quyền cho tmpfs mount có thể **bị thiết lập lại sau khi khởi động lại container**. Có thể khắc phục bằng cách **đặt thủ công `uid`/`gid`**.

---

### Cú pháp sử dụng

Bạn có thể gắn tmpfs mount bằng cách sử dụng cờ `--mount` hoặc `--tmpfs` trong lệnh `docker run`.

```bash
docker run --mount type=tmpfs,dst=<đường-dẫn-gắn>
docker run --tmpfs <đường-dẫn-gắn>
```

Trong thực tế, nên sử dụng `--mount` vì cú pháp rõ ràng, dễ hiểu hơn. Cờ `--tmpfs` đơn giản hơn, nhưng cho phép thiết lập nhiều tùy chọn linh hoạt.

**Lưu ý:** `--tmpfs` **không sử dụng được** với dịch vụ dạng swarm. Bắt buộc dùng `--mount`.

---

### Tùy chọn cho `--tmpfs`

Cờ `--tmpfs` bao gồm 2 phần, ngăn cách bởi dấu hai chấm `:`:

```bash
docker run --tmpfs <đường-dẫn-gắn>[:tùy-chọn]
```

Danh sách các tùy chọn hợp lệ:

| Tùy chọn     | Mô tả                                                         |
| ------------ | ------------------------------------------------------------- |
| `ro`         | Gắn tmpfs ở chế độ chỉ đọc.                                   |
| `rw`         | Gắn tmpfs ở chế độ đọc/ghi (mặc định).                        |
| `nosuid`     | Không cho phép thực thi bit `setuid` và `setgid`.             |
| `suid`       | Cho phép thực thi `setuid`/`setgid` (mặc định).               |
| `nodev`      | Cho phép tạo tệp thiết bị, nhưng không thể truy cập (bị lỗi). |
| `dev`        | Cho phép tạo và truy cập tệp thiết bị.                        |
| `exec`       | Cho phép thực thi các tệp nhị phân.                           |
| `noexec`     | Không cho phép thực thi các tệp nhị phân.                     |
| `sync`       | Tất cả I/O đều thực hiện đồng bộ.                             |
| `async`      | Tất cả I/O thực hiện bất đồng bộ (mặc định).                  |
| `dirsync`    | Đồng bộ hóa việc cập nhật thư mục.                            |
| `atime`      | Cập nhật thời gian truy cập mỗi lần truy cập tệp.             |
| `noatime`    | Không cập nhật thời gian truy cập tệp.                        |
| `diratime`   | Cập nhật thời gian truy cập thư mục mỗi lần truy cập.         |
| `nodiratime` | Không cập nhật thời gian truy cập thư mục.                    |
| `size`       | Kích thước của tmpfs, ví dụ `size=64m`.                       |
| `mode`       | Quyền tệp của tmpfs, ví dụ `mode=1777`.                       |
| `uid`        | UID của chủ sở hữu tmpfs, ví dụ `uid=1000`.                   |
| `gid`        | GID của chủ sở hữu tmpfs, ví dụ `gid=1000`.                   |
| `nr_inodes`  | Giới hạn số lượng inode, ví dụ `nr_inodes=400k`.              |
| `nr_blocks`  | Giới hạn số lượng block, ví dụ `nr_blocks=1024`.              |

**Ví dụ:**

```bash
docker run --tmpfs /data:noexec,size=1024,mode=1777
```

Một số tính năng nâng cao của tmpfs trong lệnh `mount` của Linux **không được hỗ trợ** qua `--tmpfs`. Nếu cần thiết, bạn có thể dùng container đặc quyền (privileged) hoặc cấu hình bên ngoài Docker.

> **Cảnh báo:** Sử dụng container với `--privileged` cấp quyền truy cập cao, có thể gây nguy cơ bảo mật. Chỉ sử dụng trong môi trường tin cậy và khi thực sự cần thiết.

```bash
docker run --privileged -it debian sh
/# mount -t tmpfs -o <tùy-chọn> tmpfs /data
```

---

### Tùy chọn cho `--mount`

Cờ `--mount` sử dụng cú pháp dạng nhiều cặp khóa-giá trị, cách nhau bởi dấu phẩy:

```bash
docker run --mount type=tmpfs,dst=<đường-dẫn>[,<khóa>=<giá-trị>...]
```

Các tùy chọn hợp lệ:

| Tùy chọn                       | Mô tả                                                                                         |
| ------------------------------ | --------------------------------------------------------------------------------------------- |
| `destination`, `dst`, `target` | Đường dẫn bên trong container được mount tmpfs.                                               |
| `tmpfs-size`                   | Kích thước tmpfs tính bằng byte. Mặc định là 50% tổng RAM của hệ thống.                       |
| `tmpfs-mode`                   | Quyền truy cập tmpfs ở dạng bát phân, ví dụ `700`, `0770`. Mặc định là `1777` (ghi toàn cục). |

**Ví dụ:**

```bash
docker run --mount type=tmpfs,dst=/app,tmpfs-size=21474836480,tmpfs-mode=1770
```

---

### Sử dụng tmpfs trong container

Có hai cách:

#### Sử dụng `--mount`

```bash
docker run -d \
  -it \
  --name tmptest \
  --mount type=tmpfs,destination=/app \
  nginx:latest
```

Xác minh tmpfs bằng lệnh:

```bash
docker inspect tmptest --format '{{ json .Mounts }}'
```

Kết quả ví dụ:

```json
[{"Type":"tmpfs","Source":"","Destination":"/app","Mode":"","RW":true,"Propagation":""}]
```

#### Sử dụng `--tmpfs`

```bash
docker run -d \
  -it \
  --name tmptest \
  --tmpfs /app \
  nginx:latest
```

Xác minh:

```bash
docker inspect tmptest --format '{{ json .Mounts }}'
```

Kết quả:

```json
{"/app":""}
```

Dừng và xóa container:

```bash
docker stop tmptest
docker rm tmptest
```

---

### Tài liệu liên quan

* [Volumes](volumes.md)
* [Bind mounts](bind-mounts.md)
* [Storage drivers](/engine/storage/drivers/)
