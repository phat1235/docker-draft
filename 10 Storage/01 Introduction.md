# **Lưu trữ trong Docker (Docker Storage)**

Mặc định, tất cả các tập tin được tạo ra bên trong container sẽ được lưu trữ trong **lớp ghi (writable layer)** — lớp này nằm trên cùng của các lớp ảnh chỉ đọc (read-only, immutable image layers).

Dữ liệu được ghi vào lớp ghi **sẽ không được giữ lại** khi container bị xóa. Điều này có nghĩa là rất khó để trích xuất dữ liệu từ container nếu một tiến trình khác cần sử dụng nó.

Mỗi container có lớp ghi riêng biệt, không chia sẻ với container khác. Việc trích xuất dữ liệu từ lớp ghi này ra máy chủ vật lý (host) hoặc chuyển sang container khác **không dễ dàng và không được khuyến nghị**.

---

## **Tùy chọn gắn kết lưu trữ (Storage mount options)**

Docker hỗ trợ các loại gắn kết lưu trữ sau, giúp lưu trữ dữ liệu **bên ngoài lớp ghi của container**:

* [Gắn kết volume (Volume mounts)](#volume-mounts)
* [Gắn kết bind (Bind mounts)](#bind-mounts)
* [Gắn kết tmpfs (tmpfs mounts)](#tmpfs-mounts)
* [Named pipes (ống đặt tên)](#named-pipes)

Dù sử dụng loại gắn kết nào, từ bên trong container, dữ liệu vẫn được nhìn thấy như là một thư mục hoặc tập tin trong hệ thống tệp của container.

---

### **Gắn kết Volume (Volume mounts)**

**Volume** là một cơ chế lưu trữ bền vững (persistent), do **Docker daemon quản lý**. Dữ liệu trong volume **vẫn được giữ lại** ngay cả khi container sử dụng nó bị xóa.

Dữ liệu của volume được lưu trên hệ thống tệp của máy chủ (host), nhưng để truy cập hoặc thao tác với dữ liệu đó, bạn cần **gắn volume vào một container**. Truy cập trực tiếp vào volume từ ngoài Docker **không được hỗ trợ và có thể gây lỗi** hoặc hư hỏng dữ liệu không lường trước.

Volumes rất thích hợp cho các tác vụ cần hiệu suất cao hoặc lưu trữ lâu dài. Vì dữ liệu được lưu trên máy chủ host, volume cho phép đạt hiệu suất tương đương với việc truy cập trực tiếp vào hệ thống tệp của host.

---

### **Gắn kết Bind (Bind mounts)**

Bind mount tạo ra một liên kết trực tiếp giữa một **đường dẫn trên máy chủ vật lý** và một thư mục bên trong container, cho phép container truy cập tập tin hoặc thư mục ở **bất kỳ vị trí nào trên host**.

Do bind mount **không được cô lập bởi Docker**, nên cả tiến trình Docker và các tiến trình bên ngoài Docker **đều có thể đọc/ghi dữ liệu cùng lúc**, có thể gây xung đột nếu không kiểm soát tốt.

Sử dụng bind mounts trong các trường hợp bạn cần chia sẻ dữ liệu giữa host và container, chẳng hạn như chia sẻ mã nguồn trong quá trình phát triển phần mềm.

---

### **Gắn kết tmpfs (tmpfs mounts)**

tmpfs là một loại gắn kết đặc biệt, trong đó dữ liệu **được lưu trực tiếp vào bộ nhớ RAM** của máy chủ, **không ghi ra đĩa**.

Lưu trữ bằng tmpfs là **tạm thời** (ephemeral): dữ liệu sẽ **mất hoàn toàn** khi container dừng, khởi động lại hoặc khi host bị khởi động lại.

Gắn kết tmpfs phù hợp với các tình huống cần lưu trữ tạm thời trong bộ nhớ, chẳng hạn như:

* Lưu trữ dữ liệu trung gian,
* Xử lý thông tin nhạy cảm (ví dụ: thông tin đăng nhập),
* Giảm tải I/O đĩa.

Chỉ nên sử dụng tmpfs khi bạn **không cần giữ dữ liệu sau khi container kết thúc**.

---

### **Named Pipes (Ống đặt tên)**

[Named pipes](https://docs.microsoft.com/en-us/windows/desktop/ipc/named-pipes) được sử dụng để thiết lập **giao tiếp giữa Docker host và container**.

Một trường hợp phổ biến là khi bạn muốn chạy một công cụ của bên thứ ba bên trong container và cần kết nối đến **Docker Engine API** thông qua named pipe.

---

## **Hướng đi tiếp theo**

* Tìm hiểu thêm về [Volumes](https://docs.docker.com/engine/storage/volumes/)
* Tìm hiểu thêm về [Bind mounts](https://docs.docker.com/engine/storage/bind-mounts/)
* Tìm hiểu thêm về [tmpfs mounts](https://docs.docker.com/engine/storage/tmpfs/)
* Tìm hiểu thêm về [Storage Drivers](https://docs.docker.com/engine/storage/drivers/) – không liên quan đến volumes hoặc bind mounts, nhưng quyết định cách Docker lưu trữ dữ liệu trong lớp ghi của container.
* Tham khảo thêm về [containerd image store](https://docs.docker.com/engine/storage/containerd/) với Docker Engine.

---
