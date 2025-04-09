# Hiểu về Docker Mounts: Volumes, Bind Mounts và tmpfs  
**Tổng quan ngắn gọn về Docker mounts cùng các trường hợp sử dụng và lợi ích của từng loại**

![](https://img001.prntscr.com/file/img001/mfFczFQTSViYBsffX0pLdw.png)

Hiện nay, Docker 🐳 đã trở thành tiêu chuẩn mặc định cho việc container hóa và được sử dụng rộng rãi trong nhiều lĩnh vực kỹ thuật phần mềm như Backend, Frontend, DevOps, và nhiều hơn nữa.

Nếu bạn đã từng sử dụng Docker – và mình tin rằng bạn đã từng – thì hẳn bạn sẽ nhận ra rằng mọi dữ liệu được ghi vào hệ thống tệp của container sẽ hoàn toàn **bị mất** nếu container bị khởi động lại (cố ý hoặc vô tình) 🔄. Đây là một vấn đề lớn, bởi vì chúng ta – những kỹ sư – có trách nhiệm nặng nề trong việc **đảm bảo dữ liệu ứng dụng được lưu trữ bền vững**.

Để giải quyết vấn đề này, Docker cung cấp một cơ chế gọi là **Mounts** nhằm hỗ trợ lưu trữ dữ liệu một cách lâu dài hơn.

Trong bài viết này, chúng ta sẽ cùng tìm hiểu từng phương pháp mount, khi nào nên dùng và cách sử dụng thông qua cả Docker CLI và Docker Compose. 🚀

Tính đến hiện tại, Docker hỗ trợ **3 loại mount chính**, mỗi loại phù hợp với các mục đích và tình huống sử dụng khác nhau:

- **Bind Mounts**
- **Volume Mounts**
- **Tmpfs Mounts**

Cuối bài còn có các lệnh để quản lý mount.  

Bạn sẵn sàng chưa? Cùng bắt đầu khám phá nhé! ✨

---

## 1. **Bind Mounts**
![](https://img001.prntscr.com/file/img001/Rg3MEvGFRUybRTBqLXJOYw.png)
Mặc định, các thư mục của hệ thống máy chủ **không thể truy cập từ bên trong container**. Nhưng với bind mounts, chúng ta có thể truy cập vào hệ thống tệp của host.  
**Bind mount** là cách để **kết nối thư mục hoặc tập tin từ hệ thống máy chủ** (host) đến một **vị trí cụ thể bên trong container**.

Bạn có thể dễ dàng cập nhật tập tin từ máy chủ, và thay đổi sẽ được phản ánh ngay bên trong container **mà không cần build lại container**.

Tuy nhiên, bind mount gắn kết rất chặt với hệ thống host. Điều này có nghĩa là các tiến trình bên trong container có thể **đọc, ghi, xóa** các tập tin trên máy chủ. Vì vậy, cần hết sức cẩn thận với quyền truy cập để **tránh rủi ro bảo mật hoặc xung đột**.

###  Ưu điểm của Bind Mounts:

- Chia sẻ tệp giữa host và container dễ dàng
- Cập nhật dữ liệu theo thời gian thực
- Dữ liệu được lưu bền vững (persistent)
- Có thể chia sẻ giữa nhiều container
- Có thể truy cập các tập tin hệ thống nhạy cảm (lợi và hại)

###  Khi nào nên dùng Bind Mounts:

- Môi trường phát triển (development)
- Nạp các tệp cấu hình và tự động reload khi thay đổi
- Truy cập các tài nguyên đặc thù của máy chủ mà container không có sẵn (thiết bị, tệp hệ thống...)

###  Cách sử dụng Bind Mount:

**Docker CLI:**
```bash
docker run -v /duongdan/host:/duongdan/trong/container image:tag
```

**Docker Compose:**
```yaml
services:
  myservice:
    image: image:tag
    volumes:
      - /duongdan/host:/duongdan/trong/container
```

---

## 2. **Volume Mounts**
![](https://img001.prntscr.com/file/img001/LNhmrFIuRHWo9katuTRLew.png)

**Volume mounts** tương tự bind mounts nhưng **được Docker quản lý hoàn toàn**. Dữ liệu volume **lưu trữ bên trong máy ảo Linux** chứ không phải hệ thống host, vì vậy tốc độ đọc/ghi **nhanh hơn và hiệu suất cao hơn**.

Volume là cách để **tách biệt dữ liệu khỏi container**, giúp dữ liệu không bị mất ngay cả khi container bị xóa hay tạo lại.

Volume cũng hỗ trợ **chia sẻ dữ liệu giữa nhiều container**. Ví dụ, bạn có thể tạo một volume và gắn nó vào nhiều container để chia sẻ dữ liệu chung. Điều này giúp hệ thống Docker linh hoạt và có tính mô-đun cao hơn.

###  Khuyến nghị khi dùng volumes:

- Gán tên tùy chỉnh cho volume
- Ghi chú cấu hình volume rõ ràng
- Thường xuyên backup các volume quan trọng

###  Ưu điểm của Volume Mounts:

- Hiệu suất cao nhờ sử dụng driver và hệ thống tệp tối ưu
- Được Docker quản lý – dễ backup, khôi phục, di chuyển
- Tính di động và tương thích đa nền tảng cao

###  Khi nào nên dùng Volume Mounts:

- Lưu trữ dữ liệu database
- Xử lý upload / download file
- Lưu log hệ thống
- Nạp / cập nhật file cấu hình
- Phục hồi dữ liệu (backup/restore)

###  Cách sử dụng Volume Mount:

**Docker CLI:**
```bash
docker run -v myvolume:/duongdan/trong/container image:tag
```

**Docker Compose:**
```yaml
services:
  myservice:
    image: image:tag
    volumes:
      - myvolume:/duongdan/trong/container

volumes:
  myvolume:
```

---

## 3. **Tmpfs Mounts**
![](https://img001.prntscr.com/file/img001/b9QU5e4-Q463qXirIgduQg.png)

**Tmpfs (temporary file system)** là một loại mount đặc biệt giúp tạo ra **hệ thống tập tin tạm thời trong RAM**. Tưởng tượng tmpfs như một ổ đĩa ảo, Docker dành một phần bộ nhớ để lưu trữ tạm.

Tmpfs tồn tại **chỉ khi container đang chạy**. Khi container bị dừng hoặc khởi động lại, dữ liệu tmpfs **bị mất hoàn toàn**. Vì vậy **không nên dùng tmpfs để lưu dữ liệu lâu dài**.

### Ưu điểm của Tmpfs Mounts:

- Tốc độ cực nhanh vì lưu trữ trên RAM
- Không cần ghi/đọc đĩa vật lý
- Rất nhẹ, lý tưởng cho các ứng dụng cần tốc độ cao

###  Khi nào nên dùng Tmpfs Mounts:

- Dữ liệu tạm không cần lưu sau khi container kết thúc
- Làm vùng nhớ trung gian (scratch space) như cache hoặc xử lý tạm thời
- Các ứng dụng cần xử lý nhanh, độ trễ thấp
- Lưu trữ dữ liệu nhạy cảm chỉ trong thời gian ngắn

###  Cách sử dụng Tmpfs Mount:

**Docker CLI:**
```bash
docker run --tmpfs /duongdan/trong/container image:tag
```

**Docker Compose:**
```yaml
services:
  myservice:
    image: image:tag
    tmpfs:
      - /duongdan/trong/container
```

---

##  Quản lý Mounts và Volumes

###  Kiểm tra thông tin mounts của container:
```bash
docker inspect [container_id] --format='{{ .Mounts }}'
```

### Tạo mới volume:
```bash
docker volume create [ten-volume]
```

###  Xem chi tiết volume:
```bash
docker volume inspect [ten-volume]
```

###  Xóa volume:
```bash
docker volume rm [ten-volume]
```
