## Trình điều khiển mạng `overlay` (Overlay network driver)

Trình điều khiển mạng `overlay` tạo ra một mạng phân tán giữa nhiều máy chủ Docker daemon. Mạng này hoạt động như một lớp phủ (overlay) lên các mạng cụ thể của từng host, cho phép các container kết nối với nó có thể giao tiếp một cách an toàn (khi bật mã hóa). Docker sẽ xử lý định tuyến gói tin đến đúng host Docker daemon và container đích một cách minh bạch.

Bạn có thể tạo mạng `overlay` do người dùng định nghĩa bằng cách sử dụng `docker network create`, tương tự như cách tạo mạng `bridge`. Một dịch vụ hoặc container có thể kết nối với nhiều mạng cùng lúc, nhưng chỉ có thể giao tiếp qua các mạng mà chúng được kết nối.

Mạng `overlay` thường được dùng để kết nối giữa các dịch vụ Swarm, nhưng bạn cũng có thể dùng nó để kết nối các container độc lập trên các host khác nhau. Khi dùng với container độc lập, bạn vẫn cần bật Swarm mode để thiết lập kết nối giữa các host.

Trang này mô tả mạng `overlay` nói chung và cách sử dụng với container độc lập. Với dịch vụ Swarm, xem thêm tại: [Quản lý mạng dịch vụ Swarm](/manuals/engine/swarm/networking.md).

### Tạo mạng `overlay`

Trước khi bắt đầu, bạn cần đảm bảo các node tham gia có thể giao tiếp với nhau qua mạng. Bảng sau liệt kê các cổng cần mở:

| Cổng                 | Mô tả                                                                                                                                                   |
|----------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------|
| `2377/tcp`           | Cổng điều khiển Swarm mặc định, có thể cấu hình bằng [`docker swarm join --listen-addr`]                          |
| `4789/udp`           | Cổng mặc định cho lưu lượng `overlay`, cấu hình được bằng [`docker swarm init --data-path-addr`]                   |
| `7946/tcp`, `7946/udp` | Dùng để giao tiếp giữa các node, không thể cấu hình                                                                                                     |

Tạo mạng `overlay` cho phép container trên các host khác nhau kết nối:

```bash
$ docker network create -d overlay --attachable my-attachable-overlay
```

Tùy chọn `--attachable` cho phép cả container độc lập và dịch vụ Swarm kết nối vào mạng. Nếu không có cờ này, chỉ dịch vụ Swarm mới có thể kết nối.

Bạn có thể chỉ định dải IP, subnet, gateway, và các tùy chọn khác. Xem chi tiết qua lệnh `docker network create --help`.

### Mã hóa lưu lượng trong mạng `overlay`

Sử dụng cờ `--opt encrypted` để bật mã hóa dữ liệu ứng dụng truyền qua mạng `overlay`:

```bash
$ docker network create \
  --opt encrypted \
  --driver overlay \
  --attachable \
  my-attachable-multi-host-network
```

Tùy chọn này sử dụng mã hóa IPsec ở cấp độ Virtual Extensible LAN (VXLAN). Việc mã hóa có thể gây ảnh hưởng hiệu suất đáng kể, nên bạn nên kiểm thử kỹ trước khi dùng trong môi trường production.

> ⚠️ **Cảnh báo:**
>
> Không nên kết nối container Windows vào mạng `overlay` được mã hóa. Mạng mã hóa `overlay` không được hỗ trợ trên Windows. Swarm sẽ không báo lỗi, nhưng kết nối mạng trên container Windows sẽ gặp vấn đề:
>
> - Container Windows không thể giao tiếp với container Linux trên cùng mạng.
> - Dữ liệu giữa các container Windows trên mạng sẽ không được mã hóa.

### Gắn một container vào mạng `overlay`

Gắn container vào mạng `overlay` giúp chúng giao tiếp được với nhau mà không cần thiết lập định tuyến giữa các Docker daemon. Điều kiện tiên quyết là các host đã tham gia cùng một Swarm.

Ví dụ, để gắn container `busybox` vào mạng `multi-host-network`:

```bash
$ docker run --network multi-host-network busybox sh
```

> ℹ️ **Lưu ý:** Chỉ hoạt động nếu mạng được tạo với cờ `--attachable`.

### Khám phá container

Khi một container trên mạng `overlay` được publish port, nó có thể được truy cập bởi các container khác trên cùng mạng. Container có thể được khám phá thông qua tra cứu DNS bằng tên container.

| Giá trị cờ                         | Mô tả                                                                                                                                 |
|------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------|
| `-p 8080:80`                       | Ánh xạ cổng TCP 80 trong container ra cổng `8080` trên mạng overlay.                                                                   |
| `-p 8080:80/udp`                   | Ánh xạ cổng UDP 80 trong container ra cổng `8080` trên mạng overlay.                                                                   |
| `-p 8080:80/sctp`                  | Ánh xạ cổng SCTP 80 trong container ra cổng `8080` trên mạng overlay.                                                                  |
| `-p 8080:80/tcp -p 8080:80/udp`    | Ánh xạ cả TCP và UDP cổng 80 trong container ra cổng `8080` tương ứng trên mạng overlay.                                              |

### Giới hạn kết nối trong mạng `overlay`

Do giới hạn của nhân Linux, mạng `overlay` có thể trở nên không ổn định nếu có hơn **1000 container cùng tồn tại trên một host**. Khi đó, giao tiếp giữa các container có thể bị gián đoạn.

Xem thêm thông tin tại: [moby/moby#44973](https://github.com/moby/moby/issues/44973#issuecomment-1543747718)
