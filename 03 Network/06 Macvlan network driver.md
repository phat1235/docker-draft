## Trình điều khiển mạng Macvlan (Macvlan Network Driver)

Một số ứng dụng, đặc biệt là các ứng dụng kế thừa (legacy) hoặc các ứng dụng giám sát lưu lượng mạng, yêu cầu phải được kết nối trực tiếp với mạng vật lý. Trong những tình huống như vậy, bạn có thể sử dụng **trình điều khiển mạng `macvlan`** để gán một địa chỉ MAC cho mỗi interface mạng ảo của container, khiến nó xuất hiện như một interface mạng vật lý được kết nối trực tiếp với mạng vật lý.

Khi sử dụng Macvlan, bạn cần chỉ định một **interface vật lý trên máy chủ Docker** sẽ được dùng cho mạng Macvlan, cũng như thông tin **subnet** và **gateway** của mạng. Bạn thậm chí có thể **tách biệt các mạng Macvlan** bằng cách sử dụng các interface vật lý khác nhau.

### Lưu ý quan trọng:

- Bạn có thể **vô tình làm suy giảm hiệu suất mạng** nếu cạn kiệt địa chỉ IP hoặc gặp tình trạng **“VLAN spread”** — xảy ra khi mạng có quá nhiều địa chỉ MAC duy nhất không cần thiết.

- Thiết bị mạng của bạn cần phải hỗ trợ **chế độ promiscuous**, cho phép một interface vật lý được gán nhiều địa chỉ MAC.

- Nếu ứng dụng của bạn có thể hoạt động tốt với **bridge (trên một Docker host)** hoặc **overlay (giao tiếp giữa nhiều Docker host)** thì nên ưu tiên các giải pháp này về lâu dài.

---

## Các tùy chọn cấu hình (Options)

Bảng sau mô tả các tùy chọn đặc trưng cho trình điều khiển `macvlan` mà bạn có thể truyền vào thông qua tham số `--opt` khi tạo mạng:

| Tùy chọn (`option`) | Mặc định    | Mô tả                                                                 |
|---------------------|-------------|-----------------------------------------------------------------------|
| `macvlan_mode`      | `bridge`    | Xác định chế độ hoạt động của macvlan: `bridge`, `vepa`, `passthru`, `private` |
| `parent`            | *(bắt buộc)*| Chỉ định interface vật lý sẽ được sử dụng làm interface cha (parent). |

---

## Tạo mạng Macvlan

Khi tạo một mạng `macvlan`, bạn có thể sử dụng **chế độ bridge** hoặc **chế độ trunk 802.1Q**.

- **Chế độ bridge**: Lưu lượng Macvlan đi qua interface vật lý trên host.
- **Chế độ trunk 802.1Q**: Lưu lượng đi qua một **sub-interface 802.1Q** mà Docker tạo động, cho phép kiểm soát định tuyến và lọc chi tiết hơn.

---

### Chế độ Bridge

Để tạo mạng `macvlan` kết nối bridge với một interface mạng vật lý cụ thể, dùng `--driver macvlan` cùng lệnh `docker network create`. Bạn cần chỉ định `parent`, là interface mà lưu lượng sẽ đi qua trên host.

```bash
$ docker network create -d macvlan \
  --subnet=172.16.86.0/24 \
  --gateway=172.16.86.1 \
  -o parent=eth0 pub_net
```

#### Loại trừ địa chỉ IP không được dùng

Nếu cần loại trừ một số IP (do đã được dùng hoặc vì lý do bảo trì), bạn có thể dùng `--aux-address`:

```bash
$ docker network create -d macvlan \
  --subnet=192.168.32.0/24 \
  --ip-range=192.168.32.128/25 \
  --gateway=192.168.32.254 \
  --aux-address="my-router=192.168.32.129" \
  -o parent=eth0 macnet32
```

---

### Chế độ trunk 802.1Q

Nếu bạn chỉ định `parent` có dấu chấm (ví dụ: `eth0.50`), Docker sẽ hiểu đó là một sub-interface và **tự động tạo sub-interface** này nếu nó chưa tồn tại.

```bash
$ docker network create -d macvlan \
    --subnet=192.168.50.0/24 \
    --gateway=192.168.50.1 \
    -o parent=eth0.50 macvlan50
```

---

### Sử dụng IPvlan thay vì Macvlan

Ví dụ trên vẫn dùng bridge L3. Nếu muốn sử dụng bridge L2, bạn có thể dùng `ipvlan` thay thế và chỉ định `-o ipvlan_mode=l2`.

```bash
$ docker network create -d ipvlan \
    --subnet=192.168.210.0/24 \
    --subnet=192.168.212.0/24 \
    --gateway=192.168.210.254 \
    --gateway=192.168.212.254 \
    -o ipvlan_mode=l2 -o parent=eth0 ipvlan210
```

---

## Sử dụng IPv6

Nếu bạn đã [cấu hình Docker daemon để bật IPv6](/manuals/engine/daemon/ipv6.md), bạn có thể tạo mạng `macvlan` hỗ trợ **dual-stack IPv4/IPv6**:

```bash
$ docker network create -d macvlan \
    --subnet=192.168.216.0/24 --subnet=192.168.218.0/24 \
    --gateway=192.168.216.1 --gateway=192.168.218.1 \
    --subnet=2001:db8:abc8::/64 --gateway=2001:db8:abc8::10 \
    -o parent=eth0.218 \
    -o macvlan_mode=bridge macvlan216
```

