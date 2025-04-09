# Tổng quan về Mạng trong Docker (Networking Overview)

**Mạng của container** (container networking) đề cập đến khả năng mà các container có thể kết nối và giao tiếp với nhau hoặc với các tác vụ không phải Docker.

Container mặc định được bật chức năng mạng và có thể tạo kết nối ra ngoài. Một container không có thông tin về loại mạng mà nó đang kết nối, hay liệu các đối tượng ngang hàng (peer) của nó có phải cũng là các container Docker hay không. Một container chỉ nhìn thấy một giao diện mạng với địa chỉ IP, gateway, bảng định tuyến, dịch vụ DNS và các thông tin mạng khác — trừ khi container sử dụng trình điều khiển mạng `none`.

Trang này mô tả mạng từ góc nhìn của container và các khái niệm xoay quanh mạng của container. Trang này không đi sâu vào chi tiết hệ điều hành cụ thể về cách Docker hoạt động với mạng. Để tìm hiểu cách Docker thao tác với các quy tắc `iptables` trên Linux, xem [Lọc gói và tường lửa (Packet filtering and firewalls)](https://docs.docker.com/engine/network/packet-filtering-firewalls/).

---

## Mạng do người dùng định nghĩa (User-defined networks)

Bạn có thể tạo các mạng tùy chỉnh (user-defined networks) và kết nối nhiều container vào cùng một mạng. Một khi đã kết nối vào mạng do người dùng định nghĩa, các container có thể giao tiếp với nhau bằng địa chỉ IP hoặc tên container.

Ví dụ sau tạo một mạng sử dụng trình điều khiển `bridge` và chạy một container trong mạng đó:

```bash
$ docker network create -d bridge my-net
$ docker run --network=my-net -itd --name=container3 busybox
```

---

### Các loại trình điều khiển mạng (Drivers)

Các trình điều khiển mạng có sẵn mặc định cung cấp chức năng mạng cốt lõi:

| Trình điều khiển (Driver) | Mô tả |
|---------------------------|-------|
| `bridge`  | Trình điều khiển mặc định. |
| `host`    | Gỡ bỏ sự cô lập mạng giữa container và máy chủ Docker. |
| `none`    | Cô lập hoàn toàn container khỏi host và các container khác. |
| `overlay` | Kết nối nhiều Docker daemon lại với nhau. |
| `ipvlan`  | Cung cấp toàn quyền kiểm soát địa chỉ IPv4 và IPv6. |
| `macvlan` | Gán địa chỉ MAC cho container. |

Chi tiết thêm về các driver tại [Network drivers overview](https://docs.docker.com/engine/network/drivers/).

---

### Kết nối tới nhiều mạng (Connecting to multiple networks)

Một container có thể kết nối đến nhiều mạng khác nhau.

Ví dụ: một container giao diện người dùng (frontend) có thể kết nối tới một mạng `bridge` có truy cập ra ngoài, và một mạng [`--internal`](https://docs.docker.com/reference/cli/docker/network/create/#internal) để giao tiếp với các container backend không cần truy cập mạng bên ngoài.

Một container cũng có thể kết nối đến các loại mạng khác nhau. Ví dụ: một mạng `ipvlan` để truy cập internet và một mạng `bridge` để truy cập dịch vụ nội bộ.

Khi gửi gói tin, nếu đích nằm trong mạng kết nối trực tiếp, gói sẽ được gửi trực tiếp. Nếu không, gói sẽ được gửi qua gateway mặc định. Gateway mặc định được Docker chọn tự động và có thể thay đổi khi kết nối mạng của container thay đổi.

Bạn có thể đặt độ ưu tiên của gateway bằng `gw-priority`. Mặc định là `0`, mạng có giá trị `gw-priority` cao nhất sẽ được chọn làm gateway mặc định.

```bash
$ docker run --network name=gwnet,gw-priority=1 --network anet1 --name myctr myimage
$ docker network connect anet2 myctr
```

---

## Mạng container (Container networks)

Bên cạnh mạng do người dùng định nghĩa, bạn có thể gán container vào ngăn xếp mạng của container khác bằng cú pháp `--network container:<name|id>`.

Khi sử dụng chế độ `container:`, các tham số sau sẽ không được hỗ trợ:

- `--add-host`
- `--hostname`
- `--dns`, `--dns-search`, `--dns-option`
- `--mac-address`
- `--publish`, `--publish-all`, `--expose`

Ví dụ: chạy một container Redis chỉ lắng nghe trên `localhost`, sau đó dùng `redis-cli` từ container khác để kết nối:

```bash
$ docker run -d --name redis example/redis --bind 127.0.0.1
$ docker run --rm -it --network container:redis example/redis-cli -h 127.0.0.1
```

---

## Cổng được công bố (Published ports)

Mặc định, khi bạn tạo hoặc chạy container bằng `docker run`, các cổng trong container không được công bố ra ngoài. Dùng cờ `--publish` hoặc `-p` để công bố cổng cho các dịch vụ bên ngoài.

Ví dụ:

| Giá trị cờ | Mô tả |
|------------|-------|
| `-p 8080:80` | Ánh xạ cổng 8080 trên host tới cổng TCP 80 trong container |
| `-p 192.168.1.100:8080:80` | Ánh xạ cổng 8080 trên IP cụ thể của host tới TCP 80 trong container |
| `-p 8080:80/udp` | Ánh xạ cổng UDP 8080 trên host tới UDP 80 trong container |
| `-p 8080:80/tcp -p 8080:80/udp` | Ánh xạ cả TCP và UDP từ host tới container |

> ⚠️ **Quan trọng:** Khi bạn publish cổng container, nó sẽ có thể được truy cập từ bên ngoài — không chỉ từ host. Để giới hạn chỉ cho host, dùng địa chỉ `127.0.0.1`:

```bash
$ docker run -p 127.0.0.1:8080:80 -p '[::1]:8080:80' nginx
```

> ⚠️ **Cảnh báo:** Các host trong cùng phân đoạn L2 (VD: cùng switch mạng) vẫn có thể truy cập các cổng chỉ định `localhost`. Tham khảo thêm: [moby/moby#45610](https://github.com/moby/moby/issues/45610)

Nếu bạn chỉ muốn các container giao tiếp với nhau, **không cần publish cổng**. Chỉ cần kết nối chúng cùng một mạng (thường là `bridge`).

---

## Địa chỉ IP và hostname

Khi tạo mạng, Docker mặc định bật cấp phát địa chỉ IPv4. Bạn có thể tắt nó bằng `--ipv4=false`, hoặc bật IPv6 bằng `--ipv6`.

```bash
$ docker network create --ipv6 --ipv4=false v6net
```

Container sẽ nhận địa chỉ IP từ mạng mà nó kết nối vào. Docker daemon quản lý việc cấp phát địa chỉ IP động.

Bạn có thể dùng `--ip` và `--ip6` để chỉ định địa chỉ cụ thể.

Hostname của container mặc định là ID của nó, có thể ghi đè bằng `--hostname`.

Khi dùng `docker network connect`, bạn có thể dùng `--alias` để đặt thêm tên phụ cho container trong mạng đó.

---

## Dịch vụ DNS

Container mặc định sử dụng DNS của host (lấy từ `/etc/resolv.conf`). Container trong mạng `bridge` mặc định sẽ được sao chép file này.

Container trong mạng tùy chỉnh sẽ dùng DNS nhúng của Docker, DNS này sẽ chuyển tiếp các truy vấn ra ngoài.

Bạn có thể điều chỉnh cấu hình DNS từng container bằng các cờ:

| Cờ | Mô tả |
|----|------|
| `--dns` | Chỉ định IP máy chủ DNS |
| `--dns-search` | Định nghĩa tên miền để tìm khi không có FQDN |
| `--dns-opt` | Thêm tùy chọn DNS như trong `resolv.conf` |
| `--hostname` | Đặt hostname cho container |

---

## Hosts tùy chỉnh

File `/etc/hosts` trong container chứa `localhost`, hostname của chính nó và một vài mục mặc định. Để thêm host tùy chỉnh, dùng `--add-host`. Xem thêm tại: [docker run – add-host](https://docs.docker.com/reference/cli/docker/container/run/#add-host)

---

## Máy chủ proxy

Nếu container cần dùng proxy, tham khảo: [Sử dụng proxy trong Docker](https://docs.docker.com/manuals/engine/daemon/proxy/)

