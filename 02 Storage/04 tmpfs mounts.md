## tmpfs mounts

Các **volume** và **bind mounts** cho phép bạn chia sẻ tập tin giữa máy chủ (host) và container, giúp dữ liệu có thể được duy trì ngay cả khi container bị dừng.

Nếu bạn đang chạy Docker trên hệ điều hành **Linux**, bạn có thêm một tùy chọn thứ ba: **tmpfs mounts**.

Khi bạn tạo container với một tmpfs mount, container có thể tạo các tập tin bên ngoài lớp ghi (writable layer) của container.

Khác với volume và bind mount, **tmpfs mount là tạm thời** và chỉ tồn tại trong bộ nhớ RAM của máy chủ. Khi container dừng, tmpfs mount sẽ bị xóa, và tất cả tập tin bên trong đó sẽ bị mất.

### Khi nào nên dùng tmpfs mount?

tmpfs mounts phù hợp với các trường hợp **không cần lưu trữ dữ liệu lâu dài**, dù trên máy chủ hay trong container. Điều này có thể vì lý do bảo mật hoặc để **tăng hiệu năng** của container khi ứng dụng cần ghi lượng lớn dữ liệu không cần lưu giữ.

> ⚠️ **Lưu ý quan trọng:**  
> tmpfs mount trong Docker ánh xạ trực tiếp với tmpfs của nhân Linux ([tmpfs - Wikipedia](https://en.wikipedia.org/wiki/Tmpfs)).  
> Dữ liệu tạm thời có thể được ghi ra **swap file**, và do đó có thể bị ghi xuống đĩa.

## Ghi đè dữ liệu sẵn có

Nếu bạn mount tmpfs vào một thư mục trong container vốn đã có sẵn tập tin hoặc thư mục, các nội dung gốc sẽ **bị ẩn** bởi tmpfs mount.

Tương tự như khi bạn lưu tập tin vào `/mnt` trên Linux, sau đó gắn ổ USB vào `/mnt`, thì nội dung của thư mục gốc `/mnt` sẽ bị ẩn cho tới khi bạn tháo ổ USB.

Với container, không có cách đơn giản nào để gỡ mount và xem lại các tập tin gốc đã bị ẩn. Cách khả thi nhất là **tạo lại container mà không sử dụng mount**.

---

## Giới hạn của tmpfs mounts

- Không giống volume và bind mount, **tmpfs mount không thể chia sẻ giữa nhiều container.**
- Chỉ khả dụng khi chạy Docker trên **Linux**.
- Khi thiết lập quyền trên tmpfs, các quyền này **có thể bị reset** sau khi container khởi động lại. Có thể khắc phục bằng cách thiết lập **UID/GID** cụ thể ([link tham khảo](https://github.com/docker/compose/issues/3425#issuecomment-423091370)).

---

## Cú pháp sử dụng

### Dùng `docker run`

Bạn có thể sử dụng hai cách sau để mount tmpfs:

```bash
docker run --mount type=tmpfs,dst=<đường-dẫn-trong-container>
docker run --tmpfs <đường-dẫn-trong-container>
```

- **`--mount`** được khuyến khích vì rõ ràng và cấu hình chi tiết hơn.
- **`--tmpfs`** ngắn gọn hơn, cho phép cấu hình tùy chọn linh hoạt.

> ⚠️ **Chú ý:**  
> `--tmpfs` **không hỗ trợ Swarm Services**. Với Swarm, bạn **phải** dùng `--mount`.

---

## Tùy chọn cho `--tmpfs`

```bash
docker run --tmpfs <đường-dẫn-trong-container>[:tùy-chọn]
```

Các tùy chọn hợp lệ:

| Tùy chọn        | Mô tả                                                                 |
|----------------|------------------------------------------------------------------------|
| `ro`           | Mount ở chế độ chỉ đọc.                                                |
| `rw`           | Mount ở chế độ đọc/ghi (mặc định).                                     |
| `nosuid`       | Không cho phép setuid/setgid khi thực thi.                            |
| `suid`         | Cho phép setuid/setgid (mặc định).                                     |
| `nodev`        | Tập tin thiết bị có thể tạo nhưng không dùng được.                    |
| `dev`          | Tập tin thiết bị được tạo và sử dụng được.                            |
| `exec`         | Cho phép thực thi tập tin nhị phân trong mount.                       |
| `noexec`       | Không cho phép thực thi tập tin nhị phân.                             |
| `sync`         | Ghi/đọc đồng bộ.                                                       |
| `async`        | Ghi/đọc bất đồng bộ (mặc định).                                        |
| `size`         | Giới hạn dung lượng tmpfs, ví dụ `size=64m`.                          |
| `mode`         | Thiết lập quyền tập tin (ví dụ `mode=1777`).                          |
| `uid`          | ID người dùng sở hữu tmpfs (ví dụ `uid=1000`).                        |
| `gid`          | ID nhóm sở hữu tmpfs (ví dụ `gid=1000`).                              |
| `nr_inodes`    | Số lượng inode tối đa, ví dụ `nr_inodes=400k`.                        |
| `nr_blocks`    | Số lượng block tối đa, ví dụ `nr_blocks=1024`.                        |

**Ví dụ:**

```bash
docker run --tmpfs /data:noexec,size=1024,mode=1777
```

> ⛔ **Cảnh báo:**  
> Nếu cần dùng nhiều tùy chọn nâng cao không được hỗ trợ bởi `--tmpfs`, bạn có thể cần chạy container dưới quyền **privileged**:

```bash
docker run --privileged -it debian sh
# mount -t tmpfs -o <tùy-chọn> tmpfs /data
```

---

## Tùy chọn cho `--mount`

```bash
docker run --mount type=tmpfs,dst=<đường-dẫn>[,<key>=<value>...]
```

Tùy chọn hợp lệ:

| Tùy chọn            | Mô tả                                                                                     |
|---------------------|--------------------------------------------------------------------------------------------|
| `destination`, `dst`, `target` | Đường dẫn trong container.                                                |
| `tmpfs-size`         | Kích thước tmpfs (byte). Mặc định là 50% tổng RAM của host nếu không khai báo.          |
| `tmpfs-mode`         | Quyền tập tin tmpfs ở dạng bát phân, ví dụ `1777`, `0700`.                             |

**Ví dụ:**

```bash
docker run --mount type=tmpfs,dst=/app,tmpfs-size=21474836480,tmpfs-mode=1770
```

---

## Sử dụng tmpfs trong container

Cách 1: Dùng `--mount`  
```bash
docker run -d \
  -it \
  --name tmptest \
  --mount type=tmpfs,destination=/app \
  nginx:latest
```

Kiểm tra mount:  
```bash
docker inspect tmptest --format '{{ json .Mounts }}'
```

Kết quả:
```json
[{"Type":"tmpfs","Source":"","Destination":"/app","Mode":"","RW":true,"Propagation":""}]
```

---

Cách 2: Dùng `--tmpfs`  
```bash
docker run -d \
  -it \
  --name tmptest \
  --tmpfs /app \
  nginx:latest
```

Kiểm tra mount:
```bash
docker inspect tmptest --format '{{ json .Mounts }}'
```

Kết quả:
```json
{"/app":""}
```

---

## Xóa container

```bash
docker stop tmptest
docker rm tmptest
```

---
