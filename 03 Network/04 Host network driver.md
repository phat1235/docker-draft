## Trình điều khiển mạng `host` (Host network driver)

Khi bạn sử dụng chế độ mạng `host` cho một container, ngăn xếp mạng của container đó **không được cách ly** khỏi máy chủ Docker (container chia sẻ không gian tên mạng – *network namespace* – với máy chủ), và container **không được cấp phát địa chỉ IP riêng**.  
Ví dụ: nếu bạn chạy một container lắng nghe ở cổng 80 và sử dụng mạng `host`, ứng dụng trong container đó sẽ khả dụng tại cổng 80 trên địa chỉ IP của máy chủ.

> **Lưu ý:**  
> Vì container **không có địa chỉ IP riêng** khi dùng chế độ `host`, nên cơ chế [ánh xạ cổng](overlay.md#publish-ports) sẽ **không có hiệu lực**, và các tuỳ chọn `-p`, `--publish`, `-P`, `--publish-all` sẽ **bị bỏ qua** và hiển thị cảnh báo:
>
> ```console
> WARNING: Published ports are discarded when using host network mode
> ```

### Một số trường hợp sử dụng phù hợp với chế độ `host`:

- Tối ưu hiệu năng
- Khi container cần xử lý nhiều cổng cùng lúc

Chế độ này hoạt động tốt vì **không cần biên dịch địa chỉ mạng (NAT)** và không tạo "userland-proxy" cho mỗi cổng.

Trình điều khiển mạng `host` được hỗ trợ trên:

- **Docker Engine (chỉ Linux)**
- **Docker Desktop phiên bản 4.34 trở lên**

Bạn cũng có thể dùng `host` cho một dịch vụ trong Swarm bằng cách thêm tuỳ chọn `--network host` vào lệnh `docker service create`. Trong trường hợp này, lưu lượng điều khiển (traffic điều phối swarm và dịch vụ) vẫn đi qua mạng **overlay**, nhưng container của dịch vụ sẽ dùng mạng và cổng của máy chủ Docker (host network). Điều này tạo ra một số **hạn chế**, ví dụ: nếu một container dịch vụ sử dụng cổng 80, thì **chỉ một container duy nhất** có thể chạy trên mỗi node swarm.

---

## Docker Desktop

Chế độ mạng `host` được hỗ trợ từ **Docker Desktop phiên bản 4.34 trở lên**.

### Cách bật chế độ host networking:

1. Đăng nhập vào Docker Desktop bằng tài khoản Docker.
2. Vào **Settings (Cài đặt)**.
3. Chọn tab **Resources** → chọn **Network**.
4. Tích vào tuỳ chọn **Enable host networking**.
5. Nhấn **Apply and restart**.

Chế độ này hoạt động **hai chiều**:

- Bạn có thể truy cập dịch vụ trong container từ máy chủ (host).
- Và ngược lại, container cũng có thể truy cập dịch vụ chạy trên host.

Cả hai giao thức TCP và UDP đều được hỗ trợ.

---

### Ví dụ

Lệnh sau sẽ chạy `netcat` trong container lắng nghe cổng `8000`:

```bash
docker run --rm -it --net=host nicolaka/netshoot nc -lkv 0.0.0.0 8000
```

Khi đó, cổng `8000` sẽ có sẵn trên host và bạn có thể kết nối từ một terminal khác:

```bash
nc localhost 8000
```

Những gì bạn gõ tại đây sẽ hiển thị ở terminal đang chạy container.

---

### Truy cập dịch vụ chạy trên host từ trong container

Khởi chạy container với host networking:

```bash
docker run --rm -it --net=host nicolaka/netshoot
```

Ví dụ, nếu bạn có một web server chạy trên cổng `80` của host, bạn có thể truy cập từ trong container như sau:

```bash
nc localhost 80
```

---

### Các hạn chế

- Các tiến trình trong container **không thể bind trực tiếp tới IP của host**, vì container không truy cập được trực tiếp vào các giao diện mạng của host.
- Tính năng host networking trong Docker Desktop hoạt động ở **tầng 4 (Layer 4)**, nên **các giao thức mạng thấp hơn TCP/UDP sẽ không được hỗ trợ** (khác với Docker trên Linux).
- Tính năng này **không tương thích** với chế độ **Enhanced Container Isolation**, vì việc cô lập container với host mâu thuẫn với việc cho container dùng mạng host.
- **Chỉ hỗ trợ container Linux**. Không hoạt động với container Windows.

---
