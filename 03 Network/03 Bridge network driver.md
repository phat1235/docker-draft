---

## Trình điều khiển mạng dạng cầu (Bridge network driver)
![](https://img001.prntscr.com/file/img001/3d42fWFlQwSLrEY2ItdRcw.png)

Về mặt mạng máy tính, **bridge network** là một thiết bị tầng liên kết (Link Layer) dùng để chuyển tiếp lưu lượng giữa các phân đoạn mạng khác nhau. Một cầu nối có thể là thiết bị phần cứng hoặc thiết bị phần mềm chạy trong nhân (kernel) của máy chủ.

Trong Docker, **bridge network** sử dụng cầu nối phần mềm cho phép các container kết nối đến cùng một mạng bridge có thể giao tiếp với nhau, đồng thời cách ly chúng với các container không kết nối cùng bridge đó. Trình điều khiển bridge của Docker sẽ tự động thêm các quy tắc vào máy chủ để các container trên các bridge khác nhau không thể giao tiếp trực tiếp.

Mạng bridge chỉ áp dụng cho các container chạy trên cùng một máy chủ Docker daemon. Để giao tiếp giữa các container trên các máy chủ Docker khác nhau, bạn có thể thiết lập định tuyến ở cấp độ hệ điều hành hoặc sử dụng [overlay network](overlay.md).

Khi Docker khởi động, một [mạng bridge mặc định](#use-the-default-bridge-network) (còn gọi là `bridge`) sẽ được tạo ra tự động, và các container mới sẽ kết nối vào mạng này nếu không chỉ định mạng khác. Bạn cũng có thể tạo các **mạng bridge do người dùng định nghĩa (user-defined)**. **Mạng bridge do người dùng định nghĩa có nhiều ưu điểm hơn mạng bridge mặc định.**

![](https://img001.prntscr.com/file/img001/HXG7tG1cTYuNo-0KSb8ZmA.png)

## So sánh giữa bridge mặc định và bridge do người dùng định nghĩa

- **Bridge do người dùng định nghĩa hỗ trợ phân giải DNS tự động giữa các container**  
  Trên mạng bridge mặc định, các container chỉ có thể truy cập nhau qua địa chỉ IP, trừ khi bạn sử dụng tùy chọn [`--link`](../links.md) (đã lỗi thời). Trên mạng bridge do người dùng định nghĩa, các container có thể phân giải tên container hoặc alias một cách tự động.

- **Cung cấp khả năng cách ly tốt hơn**  
  Các container không chỉ định `--network` sẽ được gắn vào bridge mặc định, điều này tạo rủi ro vì các container không liên quan vẫn có thể giao tiếp. Sử dụng mạng do người dùng định nghĩa giúp giới hạn phạm vi giao tiếp giữa các container.

- **Container có thể được gắn/kéo ra khỏi mạng trong thời gian chạy (on-the-fly)**  
  Bạn có thể kết nối hoặc ngắt kết nối container với mạng do người dùng định nghĩa mà không cần khởi động lại. Với bridge mặc định, bạn phải dừng và tạo lại container với cấu hình mạng mới.

- **Mỗi mạng do người dùng định nghĩa sẽ tạo một cầu nối có thể cấu hình được**  
  Cầu nối mặc định `docker0` có thể được cấu hình nhưng tất cả container đều dùng chung cấu hình này. Trong khi đó, mạng do người dùng định nghĩa có thể tùy chỉnh riêng biệt thông qua `docker network create`.

- **Các container được liên kết qua bridge mặc định có thể chia sẻ biến môi trường**  
  Điều này không hỗ trợ trên mạng do người dùng định nghĩa. Tuy nhiên, bạn có thể chia sẻ biến qua:
  - Docker volumes.
  - `docker-compose` với file khai báo biến.
  - Sử dụng dịch vụ swarm và các [secrets](/manuals/engine/swarm/secrets.md), [configs](/manuals/engine/swarm/configs.md).

**Lưu ý:** Các container trên cùng một mạng bridge do người dùng định nghĩa có thể truy cập mọi cổng của nhau. Để truy cập từ bên ngoài hoặc giữa các mạng, cần phải **publish** cổng bằng cờ `-p` hoặc `--publish`.

---

## Các tùy chọn

Bảng sau mô tả các tùy chọn dành riêng cho trình điều khiển `bridge`, dùng với `--opt` khi tạo mạng:

| Tùy chọn | Mặc định | Mô tả |
|---------|----------|-------|
| `com.docker.network.bridge.name` |  | Tên giao diện cầu nối khi tạo bridge Linux. |
| `com.docker.network.bridge.enable_ip_masquerade` | `true` | Bật chức năng IP masquerading. |
| `com.docker.network.bridge.gateway_mode_ipv4`, `gateway_mode_ipv6` | `nat` | Điều khiển kết nối ra bên ngoài. |
| `com.docker.network.bridge.enable_icc` | `true` | Bật/tắt khả năng giao tiếp giữa container. |
| `com.docker.network.bridge.host_binding_ipv4` | Tất cả địa chỉ IPv4/IPv6 | IP mặc định khi bind port. |
| `com.docker.network.driver.mtu` | `0` (không giới hạn) | Thiết lập MTU cho mạng container. |
| `com.docker.network.container_iface_prefix` | `eth` | Prefix giao diện mạng trong container. |
| `com.docker.network.bridge.inhibit_ipv4` | `false` | Ngăn Docker gán địa chỉ IP cho bridge. |

Một số tùy chọn cũng có thể cấu hình bằng cờ khi chạy `dockerd`:

| Tùy chọn | Cờ tương ứng |
|---------|--------------|
| `enable_ip_masquerade` | `--ip-masq` |
| `enable_icc` | `--icc` |
| `host_binding_ipv4` | `--ip` |
| `mtu` | `--mtu` |

Docker cũng hỗ trợ cờ `--bridge` để bạn định nghĩa cầu nối `docker0` riêng. Thường dùng khi chạy nhiều daemon Docker trên cùng một host. Xem thêm tại [Run multiple daemons](/reference/cli/dockerd.md#run-multiple-daemons).

---

### Địa chỉ bind mặc định của host

Khi không chỉ định địa chỉ host trong câu lệnh publish port (ví dụ: `-p 80`), Docker sẽ bind cổng 80 của container vào tất cả các địa chỉ của host (IPv4 & IPv6).

Bạn có thể thay đổi mặc định này bằng tùy chọn `com.docker.network.bridge.host_binding_ipv4`.

- Gán `::` → chỉ khả dụng qua IPv6.
- Gán `0.0.0.0` → khả dụng qua cả IPv4 và IPv6.
- Gán địa chỉ cụ thể → port chỉ khả dụng qua địa chỉ đó.

Ví dụ để chỉ publish cổng qua IPv4:
```bash
-p 0.0.0.0:8080:80
```

---

## Quản lý mạng bridge do người dùng định nghĩa

Tạo một mạng:
```bash
docker network create my-net
```

Bạn có thể chỉ định subnet, dải IP, gateway và các tùy chọn khác.

Xoá mạng (sau khi đã ngắt kết nối container):
```bash
docker network rm my-net
```

> **Chuyện gì thực sự xảy ra?**
>
> Khi bạn tạo hoặc xoá mạng bridge, hoặc kết nối/ngắt container khỏi mạng đó, Docker sẽ sử dụng các công cụ của hệ điều hành để quản lý hạ tầng mạng phía dưới (ví dụ: thêm/xoá bridge, chỉnh `iptables` trên Linux). Những thao tác này nên để Docker xử lý tự động.


## Kết nối container vào mạng bridge do người dùng định nghĩa

Khi bạn tạo một container mới, bạn có thể chỉ định một hoặc nhiều cờ `--network`.  
Ví dụ dưới đây kết nối một container Nginx vào mạng `my-net`. Đồng thời, nó publish cổng 80 trong container ra cổng 8080 trên host Docker, cho phép các client bên ngoài truy cập vào cổng này. Bất kỳ container nào khác kết nối vào mạng `my-net` đều có thể truy cập tất cả các cổng của container `my-nginx`, và ngược lại.

```bash
docker create --name my-nginx \
  --network my-net \
  --publish 8080:80 \
  nginx:latest
```

Để kết nối một container **đang chạy** vào mạng bridge do người dùng định nghĩa, sử dụng lệnh `docker network connect`.  
Lệnh sau sẽ kết nối container `my-nginx` đang chạy vào mạng `my-net`:

```bash
docker network connect my-net my-nginx
```

---

## Ngắt kết nối container khỏi mạng bridge do người dùng định nghĩa

Để ngắt kết nối một container đang chạy khỏi mạng bridge do người dùng định nghĩa, sử dụng lệnh `docker network disconnect`.  
Lệnh sau sẽ ngắt kết nối container `my-nginx` khỏi mạng `my-net`:

```bash
docker network disconnect my-net my-nginx
```

---

## Sử dụng IPv6 trong mạng bridge do người dùng định nghĩa

Khi tạo mạng, bạn có thể dùng cờ `--ipv6` để bật hỗ trợ IPv6:

```bash
docker network create --ipv6 --subnet 2001:db8:1234::/64 my-net
```

Nếu bạn không chỉ định tùy chọn `--subnet`, Docker sẽ tự động chọn một **prefix ULA** (Unique Local Address).

---

## Mạng bridge chỉ dùng IPv6

Để bỏ qua cấu hình địa chỉ IPv4 trên bridge và các container trong mạng, bạn tạo mạng với tùy chọn `--ipv4=false` và bật IPv6 bằng `--ipv6`:

```bash
docker network create --ipv6 --ipv4=false v6net
```

**Lưu ý:** Không thể vô hiệu hóa cấu hình địa chỉ IPv4 trên mạng bridge mặc định.

---

## Sử dụng mạng bridge mặc định

Mạng `bridge` mặc định được xem là chi tiết kế thừa (legacy) của Docker và **không được khuyến nghị dùng trong môi trường production**. Việc cấu hình mạng này là thủ công và nó có [nhiều hạn chế kỹ thuật](#differences-between-user-defined-bridges-and-the-default-bridge).

### Kết nối container vào mạng bridge mặc định

Nếu bạn **không chỉ định mạng** bằng cờ `--network`, container sẽ mặc định được kết nối vào mạng `bridge`.  
Các container kết nối vào mạng `bridge` mặc định có thể giao tiếp với nhau, **nhưng chỉ qua địa chỉ IP**, trừ khi bạn sử dụng cờ `--link` (đã lỗi thời).

### Cấu hình mạng bridge mặc định

Để cấu hình mạng `bridge` mặc định, bạn khai báo các tùy chọn trong file `daemon.json`.  
Ví dụ file `daemon.json` dưới đây chỉ định nhiều cấu hình khác nhau (chỉ nên cấu hình những gì cần thiết):

```json
{
  "bip": "192.168.1.1/24",
  "fixed-cidr": "192.168.1.0/25",
  "mtu": 1500,
  "default-gateway": "192.168.1.254",
  "dns": ["10.20.1.2","10.20.1.3"]
}
```

Trong ví dụ trên:

- Địa chỉ cầu nối là `192.168.1.1/24` (từ `bip`)
- Subnet của mạng bridge là `192.168.1.0/24` (cũng từ `bip`)
- Địa chỉ IP của container sẽ được phân phối từ dải `192.168.1.0/25` (từ `fixed-cidr`)

---

### Sử dụng IPv6 với mạng bridge mặc định

Bạn có thể bật IPv6 cho mạng bridge mặc định bằng cách thêm các tùy chọn sau vào `daemon.json` (hoặc sử dụng các cờ tương ứng khi chạy `dockerd`):

Lưu ý:
- Các tùy chọn này **chỉ áp dụng cho mạng bridge mặc định** (không áp dụng cho mạng do người dùng định nghĩa).
- Các địa chỉ bên dưới chỉ là ví dụ từ dải địa chỉ IPv6 dùng cho tài liệu.

```json
{
  "ipv6": true,
  "bip6": "2001:db8::1111/64",
  "fixed-cidr-v6": "2001:db8::/64",
  "default-gateway-v6": "2001:db8:abcd::89"
}
```

Giải thích:
- `ipv6`: Bắt buộc phải bật nếu muốn dùng IPv6.
- `bip6`: (tùy chọn) địa chỉ IPv6 của bridge, sẽ dùng làm default gateway cho container.
- `fixed-cidr-v6`: (tùy chọn) dải địa chỉ IPv6 Docker có thể gán cho container.
  - Prefix nên là `/64` hoặc nhỏ hơn (ví dụ `/60`, `/56`,...).
  - Nếu chạy thử nghiệm trong mạng cục bộ, nên dùng địa chỉ **ULA** (`fd00::/8`) thay vì **Link Local** (`fe80::/10`).
- `default-gateway-v6`: (tùy chọn) gateway mặc định. Nếu không chỉ định, địa chỉ đầu tiên trong subnet của `fixed-cidr-v6` sẽ được sử dụng.

> Sau khi chỉnh sửa `daemon.json`, bạn cần **khởi động lại Docker** để thay đổi có hiệu lực.

---

## Giới hạn số kết nối trong mạng bridge

Do giới hạn từ nhân Linux, khi có **1000 container trở lên** kết nối vào cùng một mạng bridge, mạng có thể trở nên không ổn định và mất khả năng giao tiếp giữa các container.

Tham khảo chi tiết tại: [moby/moby#44973](https://github.com/moby/moby/issues/44973#issuecomment-1543747718)

---

## Bỏ qua cấu hình địa chỉ IP của bridge

Thông thường, bridge được gán địa chỉ gateway của mạng (`--gateway`), địa chỉ này được dùng làm default route để container trong mạng giao tiếp ra ngoài.

Tùy chọn `com.docker.network.bridge.inhibit_ipv4` cho phép bạn **tạo mạng mà không gán địa chỉ IPv4 gateway cho bridge**. Trường hợp sử dụng:

- Khi bạn muốn **tự gán địa chỉ gateway cho bridge** bằng tay.
- Khi bạn **gắn thêm giao diện vật lý** vào bridge và cần gán gateway cụ thể cho nó.

Với cấu hình này, lưu lượng **north-south** (từ trong ra ngoài hoặc ngược lại) sẽ không hoạt động **nếu bạn chưa cấu hình thủ công địa chỉ gateway cho bridge** hoặc cho thiết bị gắn vào bridge.

> **Tùy chọn này chỉ dùng được với mạng bridge do người dùng định nghĩa**.


