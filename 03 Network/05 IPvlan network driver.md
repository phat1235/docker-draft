## Trình điều khiển mạng IPvlan

Trình điều khiển **IPvlan** cung cấp cho người dùng quyền kiểm soát hoàn toàn đối với định địa chỉ **IPv4 và IPv6**. Trình điều khiển VLAN được xây dựng trên nền IPvlan, cung cấp thêm cho người quản trị quyền kiểm soát toàn diện ở tầng 2 với khả năng gắn thẻ VLAN (VLAN tagging) và thậm chí cả định tuyến IPvlan ở tầng 3 (IPvlan L3 routing) – rất hữu ích cho các trường hợp tích hợp mạng tầng dưới (underlay). Nếu bạn cần triển khai dạng phủ (overlay) để trừu tượng hóa các ràng buộc vật lý, hãy xem trình điều khiển [mạng overlay nhiều máy chủ](https://docs.docker.com/manuals/engine/network/tutorials/overlay).

IPvlan là một biến thể mới của kỹ thuật ảo hóa mạng truyền thống. Các triển khai trên Linux rất nhẹ vì thay vì sử dụng **Linux bridge** truyền thống để cách ly, chúng liên kết trực tiếp với một **giao diện Ethernet** hoặc **giao diện con** của Linux để thực thi việc tách biệt giữa các mạng và kết nối đến mạng vật lý.

IPvlan cung cấp nhiều tính năng độc đáo và dư địa mở rộng thông qua các chế độ khác nhau. Hai lợi ích lớn của phương pháp này là:
- **Hiệu năng cao** do bỏ qua bridge của Linux.
- **Cấu hình đơn giản hơn** vì có ít thành phần phải quản lý.

Bằng việc loại bỏ bridge truyền thống giữa NIC của host và giao diện container, hệ thống trở nên đơn giản: các giao diện của container gắn trực tiếp vào giao diện của host. Điều này đặc biệt hữu ích cho các dịch vụ cần công khai ra bên ngoài vì không cần ánh xạ cổng (port mapping).

---

## Tùy chọn

Bảng dưới mô tả các tùy chọn riêng của trình điều khiển IPvlan có thể sử dụng với `--opt` khi tạo mạng.

| Tùy chọn        | Mặc định    | Mô tả                                                                 |
|-----------------|-------------|-----------------------------------------------------------------------|
| `ipvlan_mode`   | `l2`        | Thiết lập chế độ hoạt động của IPvlan: `l2`, `l3`, hoặc `l3s`         |
| `ipvlan_flag`   | `bridge`    | Thiết lập cờ chế độ IPvlan: `bridge`, `private`, hoặc `vepa`         |
| `parent`        | (không có)  | Xác định giao diện cha sẽ được sử dụng                               |

---

## Ví dụ

### Điều kiện tiên quyết

- Các ví dụ dưới đây áp dụng cho **một máy chủ đơn**.
- Có thể dùng giao diện chính như `eth0`, hoặc giao diện con như `eth0.10`. Giao diện con được tạo động khi có dấu `.`.
- Nếu không chỉ rõ `-o parent`, Docker sẽ tạo giao diện `dummy` để hỗ trợ các ví dụ.
- Yêu cầu kernel:
  - Linux kernel v4.2+ (các phiên bản trước có hỗ trợ nhưng không ổn định). Kiểm tra phiên bản kernel bằng `uname -r`.

---

### Ví dụ sử dụng chế độ IPvlan L2

Hình minh họa sau cho thấy topology chế độ `L2` của IPvlan:

![Mô hình đơn giản IPvlan L2](https://img001.prntscr.com/file/img001/Cn_YMwotTJKL_bINLqpgjQ.png)

Ví dụ sau sử dụng giao diện cha `eth0` với cấu hình:

```bash
$ ip addr show eth0
3: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    inet 192.168.1.250/24 brd 192.168.1.255 scope global eth0
```

Sử dụng mạng của giao diện host làm `--subnet`:

```bash
# Tạo mạng IPvlan (mặc định là chế độ L2 nếu không chỉ rõ)
$ docker network create -d ipvlan \
    --subnet=192.168.1.0/24 \
    --gateway=192.168.1.1 \
    -o ipvlan_mode=l2 \
    -o parent=eth0 db_net

# Chạy container gắn vào mạng vừa tạo
$ docker run --net=db_net -it --rm alpine /bin/sh

# Lưu ý: container KHÔNG thể ping tới host do Linux cố tình chặn vì lý do bảo mật
```

> Mặc định chế độ của IPvlan là `l2`, nếu không chỉ rõ `-o ipvlan_mode=`, Docker sẽ dùng mặc định. Tương tự, nếu không chỉ rõ `--gateway`, Docker sẽ chọn IP usable đầu tiên (ví dụ: `192.168.1.1`).

---

Hình sau mô tả cách hai máy Docker host cùng nằm trong một segment tầng 2 (Layer 2) khi dùng IPvlan L2:

![Nhiều host dùng IPvlan](https://img001.prntscr.com/file/img001/6GhgJrJsSSyNGC1qpLAh5A.png)

Tạo mạng tương tự `db_net` với các tham số mặc định:

```bash
$ docker network create -d ipvlan \
    --subnet=192.168.1.0/24 \
    -o parent=eth0 db_net_ipv

# Chạy container đầu tiên
$ docker run --net=db_net_ipv --name=ipv1 -itd alpine /bin/sh

# Chạy container thứ hai và ping container đầu tiên qua tên
$ docker run --net=db_net_ipv --name=ipv2 -it --rm alpine /bin/sh
$ ping -c 4 ipv1

# Lưu ý: container vẫn không thể ping trực tiếp tới host
```

---

### Cô lập mạng với `--internal` hoặc không khai báo `-o parent=`

Trình điều khiển IPvlan hỗ trợ cờ `--internal`, giúp cách ly hoàn toàn container trên một mạng khỏi bên ngoài.

Nếu không khai báo `-o parent=`, Docker sẽ tạo giao diện cha `dummy`, tạo hiệu ứng cô lập giống `--internal`.

Ví dụ:

```bash
# Không khai báo '-o parent=' -> mạng cô lập
$ docker network create -d ipvlan \
    --subnet=192.168.10.0/24 isolated1

# Hoặc sử dụng '--internal'
$ docker network create -d ipvlan \
    --subnet=192.168.11.0/24 --internal isolated2

# Hoặc thậm chí không khai báo subnet, Docker sẽ dùng 172.18.0.0/16 mặc định
$ docker network create -d ipvlan isolated3

# Chạy container trong các mạng cô lập
$ docker run --net=isolated1 --name=cid1 -it --rm alpine /bin/sh
$ docker run --net=isolated2 --name=cid2 -it --rm alpine /bin/sh
$ docker run --net=isolated3 --name=cid3 -it --rm alpine /bin/sh

# Truy cập container để test
$ docker exec -it cid1 /bin/sh
$ docker exec -it cid2 /bin/sh
$ docker exec -it cid3 /bin/sh
```

### Ví dụ sử dụng chế độ IPvlan L2 với trunk 802.1Q

Về mặt kiến trúc, chế độ trunk của IPvlan L2 tương tự như Macvlan về mặt cổng gateway và cách cô lập đường truyền lớp 2 (L2). Tuy nhiên, IPvlan có một số khác biệt có thể mang lại lợi ích như giảm áp lực lên bảng CAM trong các switch ToR (Top of Rack), sử dụng một địa chỉ MAC cho mỗi cổng, và tránh hiện tượng cạn kiệt địa chỉ MAC trên card mạng vật lý của máy chủ.

Tình huống sử dụng trunk 802.1Q về mặt hình thức là giống nhau giữa IPvlan và Macvlan. Cả hai đều tuân thủ chuẩn đánh thẻ VLAN và tích hợp mượt mà với mạng vật lý để hỗ trợ underlay cũng như plugin từ nhà cung cấp phần cứng.

Các máy chủ trên cùng một VLAN thường nằm trong cùng một subnet và được nhóm lại dựa trên chính sách bảo mật. Trong đa số các tình huống, các ứng dụng đa tầng sẽ chia thành các subnet khác nhau do mỗi tầng yêu cầu mức độ cô lập khác nhau. Ví dụ, việc đặt hệ thống xử lý thẻ tín dụng cùng mạng với máy chủ frontend là vi phạm tiêu chuẩn tuân thủ và chống lại nguyên tắc bảo mật theo lớp (defense-in-depth).

Việc chia VLAN (hoặc VNI nếu dùng Overlay driver) là bước đầu trong việc cô lập lưu lượng người dùng (tenant traffic).

![](https://img001.prntscr.com/file/img001/loJNnLy5TlWAqKI4W7a9ng.png)

#### Sub-interface VLAN

Các sub-interface gắn thẻ VLAN trên Linux có thể đã tồn tại hoặc sẽ được tạo ra khi bạn chạy `docker network create`. Khi dùng `docker network rm`, các sub-interface này sẽ bị xóa. Tuy nhiên, các interface gốc như `eth0` sẽ không bị xóa – chỉ có sub-interface (có chỉ số netlink > 0) mới bị xóa.

Để driver có thể tạo hoặc xóa các sub-interface VLAN, bạn cần dùng cú pháp `interface_name.vlan_tag`. Nếu dùng định danh khác cho sub-interface, Docker sẽ không tự xóa khi `docker network rm`.

Bạn có thể tự quản lý các interface hoặc để Docker tự động tạo và xóa sub-interface với lệnh `ip link`.

Ví dụ: `eth0.10` là sub-interface của `eth0` với VLAN ID là `10`, tương đương với lệnh:
```bash
ip link add link eth0 name eth0.10 type vlan id 10
```

---

### Ví dụ cấu hình VLAN ID 20

Sub-interface `eth0.20` được gắn thẻ VLAN ID `20`, được truyền vào Docker bằng tùy chọn `-o parent=eth0.20`.

```bash
docker network create -d ipvlan \
    --subnet=192.168.20.0/24 \
    --gateway=192.168.20.1 \
    -o parent=eth0.20 ipvlan20

docker run --net=ipvlan20 -it --name ivlan_test1 --rm alpine /bin/sh
docker run --net=ipvlan20 -it --name ivlan_test2 --rm alpine /bin/sh
```

---

### Ví dụ cấu hình VLAN ID 30

```bash
docker network create -d ipvlan \
    --subnet=192.168.30.0/24 \
    --gateway=192.168.30.1 \
    -o parent=eth0.30 \
    -o ipvlan_mode=l2 ipvlan30

docker run --net=ipvlan30 -it --name ivlan_test3 --rm alpine /bin/sh
docker run --net=ipvlan30 -it --name ivlan_test4 --rm alpine /bin/sh
```

Cấu hình gateway bên trong container:
```bash
ip route
default via 192.168.30.1 dev eth0
192.168.30.0/24 dev eth0  src 192.168.30.2
```

---

### Ví dụ IPvlan L2 nhiều subnet

Để subnet `192.168.114.0/24` có thể truy cập `192.168.116.0/24`, cần một router bên ngoài định tuyến giữa chúng.

```bash
docker network create -d ipvlan \
    --subnet=192.168.114.0/24 --subnet=192.168.116.0/24 \
    --gateway=192.168.114.254 --gateway=192.168.116.254 \
    -o parent=eth0.114 \
    -o ipvlan_mode=l2 ipvlan114

docker run --net=ipvlan114 --ip=192.168.114.10 -it --rm alpine /bin/sh
docker run --net=ipvlan114 --ip=192.168.114.11 -it --rm alpine /bin/sh
```

---

### Ánh xạ mạng vật lý sang Docker Network

Quản trị viên mạng (NetOps) có thể cấp trunk 802.1Q trên Docker host. Mỗi mạng Docker sẽ được ánh xạ tương ứng với VLAN/Subnet cụ thể. Các ví dụ:

- VLAN 10 – Subnet: `172.16.80.0/24`, Gateway: `172.16.80.1`  
  → `--subnet=172.16.80.0/24 --gateway=172.16.80.1 -o parent=eth0.10`

- VLAN 20 – Subnet: `172.16.50.0/22`, Gateway: `172.16.50.1`  
  → `--subnet=172.16.50.0/22 --gateway=172.16.50.1 -o parent=eth0.20`

- VLAN 30 – Subnet: `10.1.100.0/16`, Gateway: `10.1.100.1`  
  → `--subnet=10.1.100.0/16 --gateway=10.1.100.1 -o parent=eth0.30`

---

### Ví dụ IPvlan chế độ L3

Trong chế độ IPvlan L3, các tuyến (routes) cần được phân phối đến từng container. Driver chỉ gắn container vào interface IPvlan, còn việc phân phối tuyến qua cluster là ngoài phạm vi.
![](https://img001.prntscr.com/file/img001/AQtww0AoR_-ooUSJaEMXpA.png)

Đặc điểm:

- Mỗi subnet phải khác với subnet mặc định của host.
- Chế độ này không hỗ trợ broadcast/multicast → rất phù hợp cho môi trường cần mở rộng lớn và ổn định cao.
- Cần chỉ định `-o ipvlan_mode=l3` vì mặc định là `l2`.
- Có thể không cần chỉ định `-o parent=`, driver sẽ tạo interface kiểu dummy.

```bash
docker network create -d ipvlan \
    --subnet=192.168.214.0/24 \
    --subnet=10.1.214.0/24 \
    -o ipvlan_mode=l3 ipnet210

docker run --net=ipnet210 --ip=192.168.214.10 -itd alpine /bin/sh
docker run --net=ipnet210 --ip=10.1.214.10 -itd alpine /bin/sh

docker run --net=ipnet210 --ip=192.168.214.9 -it --rm alpine ping -c 2 10.1.214.10
docker run --net=ipnet210 --ip=10.1.214.9 -it --rm alpine ping -c 2 192.168.214.10
```

> 📝 **Lưu ý:**  
> Trong chế độ IPvlan L3, không cần cấu hình `--gateway`. Container sử dụng `eth0` làm gateway:
> ```bash
> ip route
> default dev eth0
> 192.168.214.0/24 dev eth0  src 192.168.214.10
> ```

Muốn các container trong mạng này có thể ping từ/to host khác thì cần có tuyến định tuyến trong router vật lý hoặc host từ xa chỉ đến địa chỉ IP Docker host.




### Chế độ IPvlan L2 hỗ trợ Dual Stack IPv4/IPv6

- Không chỉ hỗ trợ kiểm soát hoàn toàn địa chỉ IPv4, Libnetwork còn cho phép bạn kiểm soát đầy đủ địa chỉ IPv6, đồng thời cung cấp tính năng tương đương cho cả hai họ địa chỉ.

- Ví dụ sau đây sẽ bắt đầu với cấu hình chỉ IPv6. Khởi tạo hai container trong cùng VLAN `139` và thực hiện lệnh ping giữa chúng. Do không chỉ định subnet IPv4, IPAM mặc định sẽ cấp một subnet IPv4 mặc định. Subnet này sẽ bị cô lập trừ khi mạng phía trên được cấu hình định tuyến rõ ràng trên VLAN `139`.

```bash
# Tạo mạng IPv6
$ docker network create -d ipvlan \
    --ipv6 --subnet=2001:db8:abc2::/64 --gateway=2001:db8:abc2::22 \
    -o parent=eth0.139 v6ipvlan139

# Khởi chạy container trên mạng vừa tạo
$ docker run --net=v6ipvlan139 -it --rm alpine /bin/sh
```

Xem thông tin interface `eth0` và bảng định tuyến IPv6 bên trong container:

```bash
$$ ip a show eth0
$$ ip -6 route
```

Khởi tạo container thứ hai và ping địa chỉ IPv6 của container đầu tiên:

```bash
$ docker run --net=v6ipvlan139 -it --rm alpine /bin/sh
$$ ping6 2001:db8:abc2::1
```

> **Kết quả:** Ping thành công xác nhận kết nối L2 qua IPv6 hoạt động.

---

### Cấu hình mạng Dual Stack IPv4/IPv6 với VLAN ID `140`

Tạo mạng có hai subnet IPv4 và một subnet IPv6, mỗi subnet có gateway riêng:

```bash
$ docker network create -d ipvlan \
    --subnet=192.168.140.0/24 --subnet=192.168.142.0/24 \
    --gateway=192.168.140.1 --gateway=192.168.142.1 \
    --subnet=2001:db8:abc9::/64 --gateway=2001:db8:abc9::22 \
    -o parent=eth0.140 \
    -o ipvlan_mode=l2 ipvlan140
```

Khởi chạy container, kiểm tra `eth0` và bảng định tuyến IPv4 & IPv6:

```bash
$ docker run --net=ipvlan140 --ip6=2001:db8:abc2::51 -it --rm alpine /bin/sh
$ ip a show eth0
$ ip route
$ ip -6 route
```

Khởi chạy container thứ hai với địa chỉ `--ip` cụ thể và ping container đầu:

```bash
$ docker run --net=ipvlan140 --ip=192.168.140.10 -it --rm alpine /bin/sh
```

> [!CHÚ Ý]
>
> Các subnet khác nhau trên cùng một interface vật lý trong chế độ IPvlan `L2` **không thể ping nhau**. Điều này cần một router thực hiện proxy ARP. Ngược lại, chế độ `L3` có thể định tuyến lưu lượng unicast giữa các subnet khác nhau miễn là chúng dùng cùng một interface cha (`-o parent`).

---

### Chế độ IPvlan L3 hỗ trợ Dual Stack IPv4/IPv6

**Ví dụ:** Dual Stack IPv4/IPv6 sử dụng IPvlan chế độ L3 với nhiều subnet và VLAN Tag `118`.

> Interface VLAN không bắt buộc — bạn có thể dùng `eth0`, `eth1`, `bond0`, v.v.

Điểm khác biệt chính ở chế độ L3: Docker không tạo route mặc định với next-hop, thay vào đó dùng `default dev eth0` vì các gói ARP/Broadcast/Multicast bị lọc bởi Linux. Interface cha hoạt động như router nên phải nằm ngoài subnet của container.

```bash
# Tạo mạng IPvlan L3 Dual Stack
$ docker network create -d ipvlan \
    --subnet=192.168.110.0/24 \
    --subnet=192.168.112.0/24 \
    --subnet=2001:db8:abc6::/64 \
    -o parent=eth0 \
    -o ipvlan_mode=l3 ipnet110

# Khởi chạy các container
$ docker run --net=ipnet110 -it --rm alpine /bin/sh
$ docker run --net=ipnet110 --ip6=2001:db8:abc6::10 -it --rm alpine /bin/sh
$ docker run --net=ipnet110 --ip=192.168.112.30 -it --rm alpine /bin/sh
$ docker run --net=ipnet110 --ip6=2001:db8:abc6::50 --ip=192.168.112.50 -it --rm alpine /bin/sh
```

Xem thông tin interface và bảng định tuyến:

```bash
$ ip a show eth0
$ ip route
$ ip -6 route
```

> [!CHÚ Ý]
>
> Có thể tồn tại lỗi khi dùng `--ip6=`. Khi xoá một container với địa chỉ IPv6 cụ thể và khởi tạo lại với cùng địa chỉ đó, Docker có thể báo lỗi "Address already in use" và không unmount được container.

---

### Tạo thủ công liên kết VLAN 802.1Q

#### VLAN ID 40

Nếu không muốn Docker tự tạo sub-interface VLAN, bạn phải tạo nó thủ công trước khi tạo mạng Docker.

```bash
# Tạo sub-interface cho VLAN 40
$ ip link add link eth0 name eth0.40 type vlan id 40
$ ip link set eth0.40 up

# Tạo mạng Docker sử dụng sub-interface vừa tạo
$ docker network create -d ipvlan \
    --subnet=192.168.40.0/24 \
    --gateway=192.168.40.1 \
    -o parent=eth0.40 ipvlan40

# Khởi tạo container để kiểm tra ping qua VLAN
$ docker run --net=ipvlan40 -it --name ivlan_test5 --rm alpine /bin/sh
$ docker run --net=ipvlan40 -it --name ivlan_test6 --rm alpine /bin/sh
```

**Ví dụ khác:** tạo sub-interface với tên tùy ý:

```bash
# Tạo sub-interface tên "foo" cho VLAN 40
$ ip link add link eth0 name foo type vlan id 40
$ ip link set foo up

# Tạo mạng Docker với interface "foo"
$ docker network create -d ipvlan \
    --subnet=192.168.40.0/24 --gateway=192.168.40.1 \
    -o parent=foo ipvlan40

# Khởi chạy container
$ docker run --net=ipvlan40 -it --name ivlan_test5 --rm alpine /bin/sh
$ docker run --net=ipvlan40 -it --name ivlan_test6 --rm alpine /bin/sh
```

Xóa interface được tạo thủ công:

```bash
$ ip link del foo
```

> **Ghi chú:** Tất cả driver của Libnetwork có thể kết hợp lẫn nhau, kể cả các driver bên thứ ba để tăng sự linh hoạt cho người dùng Docker.

