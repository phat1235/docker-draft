# Lưu trữ trong Docker (Storage)

Mặc định, tất cả các tệp được tạo bên trong một container sẽ được lưu trữ trên **lớp ghi (writable layer)** của container. Lớp này nằm phía trên các lớp ảnh (image layers) chỉ đọc (read-only), không thể thay đổi (immutable).

Dữ liệu ghi vào lớp ghi của container **sẽ không được lưu giữ** khi container bị hủy. Điều này khiến việc trích xuất dữ liệu ra khỏi container trở nên khó khăn nếu một tiến trình khác cần sử dụng nó.

Lưu ý: **Lớp ghi là duy nhất cho từng container**. Bạn **không thể dễ dàng trích xuất dữ liệu** từ lớp ghi ra máy chủ host hoặc sang một container khác.

---

## Các tùy chọn mount lưu trữ (Storage Mount Options)

Docker hỗ trợ các kiểu mount sau để lưu trữ dữ liệu bên ngoài lớp ghi của container:

- [Volume mounts](#volume-mounts)
- [Bind mounts](#bind-mounts)
- [tmpfs mounts](#tmpfs-mounts)
- [Named pipes](#named-pipes)

Bất kể bạn chọn loại mount nào, dữ liệu bên trong container **sẽ hiển thị giống nhau**, dưới dạng một thư mục hoặc một tệp đơn lẻ trong hệ thống tập tin (filesystem) của container.

---

### Volume mounts (Gắn ổ đĩa Docker - Volume)

Volume là cơ chế lưu trữ bền vững (persistent storage) được quản lý bởi Docker daemon. Dữ liệu trong volume vẫn được **giữ nguyên ngay cả khi container sử dụng nó bị xóa**.

Dữ liệu của volume được lưu trữ trên hệ thống tập tin của máy chủ host. Tuy nhiên, để tương tác với dữ liệu trong volume, bạn **phải mount volume vào container**. Việc truy cập trực tiếp vào dữ liệu volume trên host **không được hỗ trợ**, có thể gây lỗi hoặc làm hỏng volume.

Volume đặc biệt phù hợp cho:

- Các tác vụ xử lý dữ liệu hiệu năng cao
- Lưu trữ dữ liệu lâu dài

Do Docker quản lý vị trí lưu trữ volume, hiệu suất truy xuất tệp có thể tương đương với việc truy cập trực tiếp vào hệ thống tập tin host.

---

### Bind mounts (Gắn thư mục/thư viện từ Host)

Bind mount tạo liên kết trực tiếp giữa một đường dẫn trên máy chủ host và container, cho phép truy cập đến các tệp hoặc thư mục **bất kỳ vị trí nào trên host**.

Bind mount **không được cô lập bởi Docker**, nên cả container và các tiến trình ngoài Docker trên host **có thể đồng thời sửa đổi dữ liệu**.

Sử dụng bind mount trong các trường hợp cần:

- Truy cập song song dữ liệu từ cả container và host
- Kiểm thử, phát triển ứng dụng theo thời gian thực (hot-reload)

---

### tmpfs mounts (Gắn hệ thống tập tin ảo trong RAM)

tmpfs mount lưu trữ dữ liệu **trực tiếp trong bộ nhớ RAM** của máy chủ, đảm bảo **không ghi dữ liệu xuống ổ cứng**.

Đặc điểm:

- Dữ liệu **mất đi hoàn toàn** khi container dừng, khởi động lại hoặc khi host reboot.
- Không lưu lại dữ liệu trên host hay trong container sau khi kết thúc phiên làm việc.

tmpfs phù hợp với các tình huống:

- Lưu trữ tạm thời (cache)
- Xử lý dữ liệu nhạy cảm (credentials, token,...)
- Giảm I/O đĩa (tăng hiệu năng)

Chỉ sử dụng tmpfs khi dữ liệu **không cần giữ lại sau khi container kết thúc**.

---

### Named pipes (Đường ống đặt tên)

[Named pipes](https://docs.microsoft.com/en-us/windows/desktop/ipc/named-pipes) cho phép giao tiếp giữa máy chủ Docker và container.

Một ví dụ điển hình là:

- Chạy công cụ của bên thứ ba bên trong container
- Giao tiếp với Docker Engine API thông qua named pipe

Lưu ý: Named pipe thường áp dụng trên Windows, trong các môi trường phát triển hoặc tích hợp đặc biệt.

---
