# Bind Mounts (Liên kết gắn kết)

Khi bạn sử dụng **bind mount**, một tệp hoặc thư mục trên máy chủ (host) sẽ được **gắn kết (mount)** vào trong container. Ngược lại, khi sử dụng **volume**, Docker sẽ tạo ra một thư mục mới trong thư mục lưu trữ nội bộ của Docker trên máy chủ, và Docker sẽ quản lý nội dung của thư mục đó.

---

## Khi nào nên sử dụng bind mount

Bind mount phù hợp trong các trường hợp sau:

- **Chia sẻ mã nguồn hoặc các tệp kết quả biên dịch** giữa môi trường phát triển trên máy chủ Docker và container.
  
- **Tạo hoặc sinh tệp trong container và cần lưu giữ các tệp đó trên hệ thống tập tin của máy chủ**.

- **Chia sẻ các tệp cấu hình từ máy chủ đến container**. Ví dụ: Docker cung cấp khả năng phân giải DNS mặc định cho container bằng cách mount tệp `/etc/resolv.conf` từ máy chủ vào mỗi container.

Ngoài ra, bind mount còn được dùng trong quá trình **build**, ví dụ: mount mã nguồn từ host vào container build để test, lint, hoặc compile dự án.

---

## Bind mount ghi đè dữ liệu có sẵn

Nếu bạn bind mount một tệp hoặc thư mục vào một thư mục trong container vốn đã có dữ liệu, các tệp/thư mục hiện có sẽ **bị che khuất (obscured)** bởi mount. Tình huống này tương tự như việc bạn lưu trữ dữ liệu vào `/mnt` trên Linux rồi gắn một USB vào `/mnt`. Nội dung gốc của `/mnt` sẽ bị ẩn đi cho đến khi bạn tháo gắn kết USB.

Trong Docker container, **không có cách đơn giản nào để gỡ bỏ bind mount và khôi phục lại dữ liệu bị che khuất**. Giải pháp tốt nhất là **tạo lại container mà không có bind mount** đó.

---

## Lưu ý và giới hạn

- **Bind mount mặc định có quyền ghi trên máy chủ**.

  Khi sử dụng bind mount, **container có thể thay đổi hệ thống tập tin của host**, bao gồm việc tạo, sửa đổi, hoặc xóa các tệp/thư mục quan trọng. Điều này có thể gây ra các **vấn đề bảo mật** ảnh hưởng đến cả hệ thống và các tiến trình không phải Docker.

  Để ngăn ghi từ container, có thể sử dụng tùy chọn `readonly` hoặc `ro`.

- **Bind mount áp dụng cho host của Docker daemon, không phải máy client**.

  Nếu bạn đang dùng **Docker daemon từ xa**, bạn **không thể bind mount** các tệp từ máy client vào container.

  Trên **Docker Desktop**, Docker daemon chạy bên trong một máy ảo Linux, không chạy trực tiếp trên máy chủ bản địa. Docker Desktop có các cơ chế tích hợp để xử lý bind mount một cách minh bạch, giúp chia sẻ đường dẫn từ hệ thống tập tin của host với container trong VM.

- **Container dùng bind mount sẽ phụ thuộc mạnh vào host**.

  Vì bind mount phụ thuộc vào **cấu trúc thư mục cụ thể trên máy chủ**, nếu chạy container đó trên một máy khác mà không có cấu trúc thư mục tương tự, container có thể bị lỗi.

---

## Cú pháp

Để tạo một bind mount, bạn có thể dùng tùy chọn `--mount` hoặc `--volume`.

```bash
docker run --mount type=bind,src=<đường-dẫn-host>,dst=<đường-dẫn-container>
docker run --volume <đường-dẫn-host>:<đường-dẫn-container>
```

Thông thường, nên dùng `--mount` vì **cú pháp rõ ràng hơn** và hỗ trợ đầy đủ các tùy chọn.

> Nếu bạn dùng `--volume` để bind mount một tệp hoặc thư mục chưa tồn tại trên host, Docker sẽ **tự động tạo thư mục đó**, và nó **luôn là thư mục** (dù bạn định mount một tệp).

> Ngược lại, `--mount` **không tự tạo** thư mục. Nếu mount path không tồn tại, Docker sẽ báo lỗi:

```bash
docker run --mount type=bind,src=/dev/noexist,dst=/mnt/foo alpine
docker: Error response from daemon: invalid mount config for type "bind": bind source path does not exist: /dev/noexist.
```

---

### Các tùy chọn cho `--mount`

Tùy chọn `--mount` gồm nhiều cặp `key=value`, phân tách bằng dấu phẩy, không bắt buộc theo thứ tự:

```bash
docker run --mount type=bind,src=<đường-dẫn-host>,dst=<đường-dẫn-container>[,<key>=<value>...]
```

| Tùy chọn                  | Mô tả                                                                                       |
|--------------------------|----------------------------------------------------------------------------------------------|
| `source`, `src`          | Đường dẫn tệp/thư mục trên host (có thể là tương đối hoặc tuyệt đối).                       |
| `destination`, `dst`, `target` | Đường dẫn tuyệt đối trong container nơi mount vào.                                        |
| `readonly`, `ro`         | Nếu có, mount sẽ ở chế độ chỉ đọc (read-only).                                               |
| `bind-propagation`       | Cấu hình [bind propagation](#configure-bind-propagation).                                    |

**Ví dụ:**

```bash
docker run --mount type=bind,src=.,dst=/project,ro,bind-propagation=rshared
```

---

### Các tùy chọn cho `--volume` hoặc `-v`

Cú pháp:

```bash
docker run -v <đường-dẫn-host>:<đường-dẫn-container>[:tùy-chọn]
```

- Trường 1: đường dẫn trên host.
- Trường 2: nơi mount vào container.
- Trường 3: tùy chọn (nếu có), phân cách bởi dấu phẩy.

| Tùy chọn        | Mô tả                                                                                     |
|----------------|---------------------------------------------------------------------------------------------|
| `readonly`, `ro` | Mount ở chế độ chỉ đọc (read-only).                                                       |
| `z`, `Z`         | Cấu hình nhãn SELinux. Xem [Cấu hình SELinux label](#configure-the-selinux-label).       |
| `rprivate`       | (mặc định) Bind propagation là `rprivate`.                                                |
| `private`        | Bind propagation là `private`.                                                            |
| `rshared`        | Bind propagation là `rshared`.                                                            |
| `shared`         | Bind propagation là `shared`.                                                             |
| `rslave`         | Bind propagation là `rslave`.                                                             |
| `slave`          | Bind propagation là `slave`.                                                              |

**Ví dụ:**

```bash
docker run -v .:/project:ro,rshared
```

---

## Khởi chạy container với bind mount

Giả sử bạn có thư mục `source`, khi build mã nguồn, sản phẩm sẽ được lưu vào thư mục `source/target/`. Bạn muốn mount thư mục `target/` vào container tại `/app/` và mỗi lần build mới, container sẽ thấy sản phẩm mới.

Chạy từ trong thư mục `source`:

> `$(pwd)` sẽ trả về đường dẫn thư mục hiện tại (trên Linux/macOS).

**Ví dụ dùng `--mount`:**

```bash
docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app \
  nginx:latest
```

**Ví dụ dùng `-v`:**

```bash
docker run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app \
  nginx:latest
```

---

Để kiểm tra mount có thành công không:

```bash
docker inspect devtest
```

Tìm phần `Mounts`:

```json
"Mounts": [
    {
        "Type": "bind",
        "Source": "/tmp/source/target",
        "Destination": "/app",
        "Mode": "",
        "RW": true,
        "Propagation": "rprivate"
    }
],
```

Điều này xác nhận:

- Mount là dạng bind
- Nguồn và đích đúng
- Có quyền ghi (`RW: true`)
- Propagation là `rprivate`

---

Để dừng và xóa container:

```bash
docker container rm -fv devtest
```


### Gắn kết vào thư mục không rỗng trong container

Nếu bạn bind-mount (gắn kết ràng buộc) một thư mục từ máy chủ vào một thư mục không rỗng trong container, nội dung hiện có trong thư mục của container sẽ bị che khuất bởi nội dung từ bind mount. Điều này có thể hữu ích, chẳng hạn khi bạn muốn kiểm thử một phiên bản mới của ứng dụng mà không cần build lại image. Tuy nhiên, hành vi này có thể gây bất ngờ và khác với [volumes](volumes.md).

Ví dụ sau là một tình huống cực đoan, trong đó thư mục `/usr/` trong container bị thay thế bởi thư mục `/tmp/` trên máy chủ. Trong hầu hết trường hợp, điều này khiến container không thể hoạt động.

Cả ví dụ dùng `--mount` và `-v` đều cho kết quả giống nhau.

#### Ví dụ với `--mount`
```bash
$ docker run -d \
  -it \
  --name broken-container \
  --mount type=bind,source=/tmp,target=/usr \
  nginx:latest

docker: Error response from daemon: oci runtime error: container_linux.go:262:
starting container process caused "exec: \"nginx\": executable file not found in $PATH".
```

#### Ví dụ với `-v`
```bash
$ docker run -d \
  -it \
  --name broken-container \
  -v /tmp:/usr \
  nginx:latest

docker: Error response from daemon: oci runtime error: container_linux.go:262:
starting container process caused "exec: \"nginx\": executable file not found in $PATH".
```

Container được tạo nhưng không khởi động được. Hãy xóa container này:
```bash
$ docker container rm broken-container
```

---

## Sử dụng bind mount chỉ đọc (read-only)

Với một số ứng dụng phát triển, container cần ghi dữ liệu vào bind mount để các thay đổi phản ánh ngược lại về máy chủ Docker. Tuy nhiên, có lúc container chỉ cần quyền đọc.

Ví dụ sau chỉnh sửa ví dụ trên, gắn kết thư mục dưới dạng chỉ đọc bằng cách thêm `readonly` (hoặc `ro`) vào danh sách tùy chọn (mặc định là rỗng). Nếu có nhiều tùy chọn, hãy phân tách chúng bằng dấu phẩy.

#### Ví dụ với `--mount`
```bash
$ docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app,readonly \
  nginx:latest
```

#### Ví dụ với `-v`
```bash
$ docker run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app:ro \
  nginx:latest
```

Sử dụng lệnh `docker inspect devtest` để kiểm tra bind mount được tạo đúng chưa. Kiểm tra phần `Mounts`:
```json
"Mounts": [
    {
        "Type": "bind",
        "Source": "/tmp/source/target",
        "Destination": "/app",
        "Mode": "ro",
        "RW": false,
        "Propagation": "rprivate"
    }
]
```

Dừng và xóa container:
```bash
$ docker container rm -fv devtest
```

---

## Gắn kết đệ quy (recursive mounts)

Khi bạn bind mount một đường dẫn chứa các mount bên trong, các submount này mặc định cũng sẽ được bind vào. Hành vi này có thể cấu hình được thông qua tùy chọn `bind-recursive` khi dùng `--mount`. Tùy chọn này **chỉ hỗ trợ** với `--mount`, **không hỗ trợ** với `-v` hay `--volume`.

Nếu bind mount là chỉ đọc, Docker Engine sẽ cố gắng đặt các submount cũng là chỉ đọc. Tính năng này gọi là "recursive read-only mount" và yêu cầu kernel Linux phiên bản 5.12 trở lên. Nếu bạn dùng kernel cũ hơn, các submount mặc định là ghi được. Nếu bạn cố thiết lập submount chỉ đọc trên kernel < 5.12, sẽ gặp lỗi.

Các giá trị được hỗ trợ cho tùy chọn `bind-recursive`:

| Giá trị            | Mô tả                                                                                                                |
|--------------------|----------------------------------------------------------------------------------------------------------------------|
| `enabled` (mặc định)| Nếu kernel ≥ v5.12, mount chỉ đọc sẽ đệ quy là chỉ đọc. Ngược lại, submount vẫn có thể ghi được.                   |
| `disabled`         | Các submount bị bỏ qua (không được bind vào container).                                                             |
| `writable`         | Submount có thể ghi.                                                                                                 |
| `readonly`         | Submount chỉ đọc. Yêu cầu kernel v5.12+.                                                                             |

---

## Cấu hình truyền dẫn bind mount (bind propagation)

Mặc định, bind propagation là `rprivate` với cả bind mount và volume. Bạn chỉ có thể cấu hình bind propagation cho **bind mount**, và **chỉ trên Linux**. Đây là khái niệm nâng cao, đa phần người dùng không cần thay đổi.

Bind propagation xác định việc một mount được tạo bên trong bind-mount có được nhân bản (replicate) đến nơi khác không. Ví dụ `/mnt` cũng được mount tại `/tmp`. Nếu bạn tạo mount tại `/tmp/a`, việc `/mnt/a` có tồn tại hay không phụ thuộc vào cài đặt propagation.

> **Lưu ý:** Bind propagation không hoạt động với Docker Desktop.

| Giá trị         | Mô tả                                                                                                                                               |
|----------------|------------------------------------------------------------------------------------------------------------------------------------------------------|
| `shared`       | Submount từ mount gốc được phản ánh sang mount bản sao, và ngược lại.                                                                               |
| `slave`        | Một chiều: submount từ gốc đến bản sao được truyền dẫn, nhưng ngược lại thì không.                                                                  |
| `private`      | Không có sự truyền dẫn nào giữa mount gốc và bản sao.                                                                                               |
| `rshared`      | Giống `shared` nhưng có tính đệ quy – áp dụng cho tất cả mount lồng nhau.                                                                           |
| `rslave`       | Giống `slave` nhưng đệ quy.                                                                                                                          |
| `rprivate`     | Mặc định – giống `private` nhưng đệ quy. Không có truyền dẫn giữa bất kỳ mount nào.                                                                 |

Trước khi cấu hình propagation, hệ thống tập tin của host cần hỗ trợ bind propagation.

Tài liệu thêm: [Linux kernel shared subtree](https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt)

Ví dụ sau gắn cùng một thư mục vào 2 vị trí trong container, một vị trí có thêm tùy chọn `readonly` và `rslave`.

#### Ví dụ với `--mount`
```bash
$ docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app \
  --mount type=bind,source="$(pwd)"/target,target=/app2,readonly,bind-propagation=rslave \
  nginx:latest
```

#### Ví dụ với `-v`
```bash
$ docker run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app \
  -v "$(pwd)"/target:/app2:ro,rslave \
  nginx:latest
```

Nếu bạn tạo `/app/foo/`, thì `/app2/foo/` cũng sẽ tồn tại.

---

## Cấu hình nhãn SELinux

Nếu bạn sử dụng SELinux, bạn có thể thêm tùy chọn `z` hoặc `Z` để thay đổi nhãn SELinux của tệp hoặc thư mục trên host được mount vào container. Thao tác này ảnh hưởng trực tiếp đến host và có thể gây hậu quả vượt ra ngoài phạm vi của Docker.

- `z`: nội dung bind mount được chia sẻ giữa nhiều container.
- `Z`: nội dung bind mount là riêng tư, không chia sẻ.

**Cảnh báo:** Cần hết sức cẩn thận với các tùy chọn này. Nếu bind mount một thư mục hệ thống như `/home` hoặc `/usr` với `Z`, bạn có thể khiến máy chủ không thể hoạt động. Phục hồi đòi hỏi relabel bằng tay.

> **Quan trọng:**  
> Khi dùng bind mount với service, các nhãn SELinux (`:Z`, `:z`) và `:ro` sẽ bị bỏ qua. Xem [moby/moby #32579](https://github.com/moby/moby/issues/32579).

Ví dụ dùng `z` để chia sẻ nội dung mount giữa các container:

```bash
$ docker run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app:z \
  nginx:latest
```

Lưu ý: Không thể cấu hình nhãn SELinux qua `--mount`.

---

## Sử dụng bind mount với Docker Compose

Một dịch vụ Docker Compose đơn giản sử dụng bind mount có thể như sau:

```yaml
services:
  frontend:
    image: node:lts
    volumes:
      - type: bind
        source: ./static
        target: /opt/app/static
volumes:
  myapp:
```

Để biết thêm về cách dùng volumes kiểu `bind` với Compose, xem:
- [Tài liệu Compose về volumes](/reference/compose-file/services.md#volumes)
- [Tài liệu Compose về cấu hình volume](/reference/compose-file/services.md#volumes)

---
