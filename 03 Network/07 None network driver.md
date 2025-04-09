## Trình điều khiển mạng `none` (None Network Driver)

Nếu bạn muốn **cách ly hoàn toàn ngăn xếp mạng (network stack)** của một container, bạn có thể sử dụng tùy chọn `--network none` khi khởi chạy container. Với cấu hình này, **chỉ interface loopback (127.0.0.1)** được tạo bên trong container.

Container sẽ không thể giao tiếp với bất kỳ mạng nào bên ngoài, kể cả với host hoặc internet.

---

### Ví dụ

Ví dụ dưới đây minh họa lệnh `ip link show` được thực thi bên trong container Alpine sử dụng driver mạng `none`:

```bash
$ docker run --rm --network none alpine:latest ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

---

### Không có địa chỉ IPv6 loopback

Khi container sử dụng driver mạng `none`, **không có địa chỉ loopback IPv6 (`::1`)** được cấu hình.

Ví dụ kiểm tra địa chỉ IP với `ip addr show`:

```bash
$ docker run --rm --network none --name no-net-alpine alpine:latest ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
```

---

### Khi nào nên sử dụng `--network none`?

- Khi bạn muốn container **không có quyền truy cập mạng hoàn toàn**, ví dụ để kiểm tra các hành vi xử lý cục bộ.
- Khi container không cần giao tiếp ra ngoài, chỉ chạy các tác vụ nội bộ.
- Dùng trong môi trường an toàn hoặc mô phỏng tấn công, khi cần hạn chế tối đa quyền truy cập mạng.

