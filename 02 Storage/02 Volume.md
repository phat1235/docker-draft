## Volumes

**Volumes** (ổ lưu trữ khối lượng) là các kho lưu trữ dữ liệu bền vững dành cho container, được tạo và quản lý bởi Docker. Bạn có thể tạo volume một cách tường minh bằng lệnh `docker volume create`, hoặc Docker có thể tự động tạo volume trong quá trình tạo container hoặc service.

![](https://img001.prntscr.com/file/img001/w0WwmEoFQQW_oL-eX2Lwkg.png)

Khi bạn tạo một volume, nó được lưu trữ bên trong một thư mục trên máy chủ Docker (Docker host). Khi bạn mount volume này vào một container, thư mục đó chính là phần được mount vào container. Điều này tương tự như **bind mounts**, ngoại trừ việc volume được Docker quản lý và được tách biệt khỏi các chức năng cốt lõi của hệ điều hành máy chủ.

---

## Khi nào nên dùng volumes

Volumes là cơ chế ưu tiên để lưu trữ dữ liệu được sinh ra và sử dụng bởi các container Docker. Trong khi [bind mounts](bind-mounts.md) phụ thuộc vào cấu trúc thư mục và hệ điều hành của máy chủ, volumes hoàn toàn được Docker quản lý. Volumes là lựa chọn phù hợp cho các trường hợp sau:

- Dễ dàng sao lưu hoặc di chuyển hơn so với bind mounts.
- Có thể quản lý volumes bằng lệnh Docker CLI hoặc thông qua Docker API.
- Hoạt động trên cả container Linux và Windows.
- Có thể chia sẻ an toàn giữa nhiều container.
- Volumes mới có thể được khởi tạo sẵn nội dung bởi container hoặc build.
- Khi ứng dụng của bạn yêu cầu I/O hiệu năng cao.

**Không nên dùng volumes** nếu bạn cần truy cập trực tiếp tệp từ máy chủ host, vì volume được Docker hoàn toàn quản lý. Trong trường hợp đó, hãy dùng [bind mounts](bind-mounts.md).

Việc sử dụng volume cũng thường tốt hơn so với việc ghi trực tiếp vào layer ghi của container, vì volume không làm tăng kích thước container và có hiệu suất cao hơn. Ghi vào layer ghi của container cần trình điều khiển lưu trữ ([storage driver](/manuals/engine/storage/drivers/_index.md)) để quản lý hệ thống tập tin, điều này làm giảm hiệu suất so với việc ghi trực tiếp vào hệ thống tập tin của host qua volumes.

Nếu container của bạn tạo ra dữ liệu trạng thái tạm thời, hãy cân nhắc sử dụng [tmpfs mount](tmpfs.md) để tránh lưu dữ liệu vĩnh viễn và tăng hiệu suất bằng cách tránh ghi vào layer ghi của container.

Volumes sử dụng chính sách bind propagation là `rprivate` (đệ quy riêng tư) và bind propagation không thể cấu hình với volumes.

---

## Vòng đời của một volume

Nội dung của volume tồn tại **ngoài vòng đời** của một container cụ thể. Khi một container bị xóa, layer ghi của nó cũng bị xóa theo. Tuy nhiên, nếu dùng volume, dữ liệu vẫn được giữ lại ngay cả khi container đã bị xóa.

Một volume có thể được mount đồng thời vào nhiều container. Khi không có container nào đang sử dụng một volume, volume vẫn tồn tại và không tự động bị xóa. Bạn có thể dùng lệnh `docker volume prune` để xóa các volume không sử dụng.

---

## Mount volume lên thư mục đã có dữ liệu

Nếu bạn mount một **volume không rỗng** vào một thư mục trong container đã có sẵn tập tin hoặc thư mục, thì các tập tin hiện có sẽ **bị ẩn đi** bởi volume được mount. Điều này giống như bạn lưu tập tin vào `/mnt` trên hệ thống Linux, sau đó mount một ổ USB vào `/mnt` — nội dung ban đầu sẽ bị ẩn cho đến khi tháo ổ USB.

Với container, không có cách đơn giản nào để "tháo mount" để hiển thị lại nội dung cũ. Giải pháp tốt nhất là tạo lại container mà không mount volume đó.

Nếu bạn mount một **volume rỗng** vào một thư mục trong container có sẵn dữ liệu, các tập tin/thư mục đó sẽ được **copy mặc định** vào volume. Tương tự, nếu bạn khởi tạo một container và chỉ định một volume chưa tồn tại, Docker sẽ tạo một volume rỗng và copy dữ liệu từ thư mục mount vào volume.

Để ngăn Docker copy dữ liệu từ container vào volume rỗng, hãy dùng tùy chọn `volume-nocopy` (xem phần [Options for --mount](#options-for---mount)).

---

## Volume có tên và volume ẩn danh

Volume có thể là **được đặt tên** hoặc **ẩn danh**.

- Volume ẩn danh được gán một tên ngẫu nhiên đảm bảo duy nhất trên mỗi Docker host.
- Cũng như volume có tên, volume ẩn danh vẫn tồn tại sau khi container bị xóa, trừ khi bạn dùng cờ `--rm` khi tạo container — lúc này volume ẩn danh sẽ bị xóa theo.

Nếu bạn tạo nhiều container liên tiếp, mỗi container dùng volume ẩn danh riêng biệt. Volume ẩn danh không được tái sử dụng hay chia sẻ tự động giữa các container. Nếu muốn chia sẻ volume ẩn danh giữa nhiều container, bạn phải dùng ID của volume để mount thủ công.

---

## Cú pháp

Để mount volume khi dùng lệnh `docker run`, bạn có thể dùng một trong hai tùy chọn:

```bash
$ docker run --mount type=volume,src=<volume-name>,dst=<mount-path>
$ docker run --volume <volume-name>:<mount-path>
```

Thông thường, nên dùng `--mount` vì cú pháp rõ ràng hơn và hỗ trợ đầy đủ các tùy chọn.

Bạn **phải dùng `--mount`** nếu muốn:

- Chỉ định [tùy chọn volume driver](#use-a-volume-driver)
- Mount [một thư mục con trong volume](#mount-a-volume-subdirectory)
- Mount volume vào service trong Docker Swarm

---

### Các tùy chọn cho `--mount`

Cờ `--mount` bao gồm các cặp khóa-giá trị (key-value), phân cách bởi dấu phẩy, mỗi cặp là một `key=value`.

```bash
$ docker run --mount type=volume[,src=<volume-name>],dst=<mount-path>[,<key>=<value>...]
```

| Tùy chọn                      | Mô tả                                                                                                                                                                                                                       |
|------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `source`, `src`              | Tên của volume. Nếu là volume ẩn danh thì không cần chỉ định.                                                                                                                                                              |
| `destination`, `dst`, `target` | Đường dẫn trong container nơi volume sẽ được mount vào.                                                                                                                                                                   |
| `volume-subpath`             | Đường dẫn con trong volume để mount vào container. Thư mục con này phải tồn tại trước khi mount. Xem [Mount một thư mục con của volume](#mount-a-volume-subdirectory).                                                    |
| `readonly`, `ro`             | Nếu có, volume sẽ được mount ở chế độ chỉ đọc. Xem [Dùng volume ở chế độ chỉ đọc](#use-a-read-only-volume).                                                                                                                |
| `volume-nocopy`              | Nếu có, dữ liệu tại đích mount sẽ không được copy vào volume nếu volume rỗng. Mặc định thì dữ liệu sẽ được copy.                                                                                                            |
| `volume-opt`                 | Có thể chỉ định nhiều lần, mỗi lần là một cặp khóa-giá trị của tùy chọn volume.                                                                                                                                              |

Ví dụ:

```bash
$ docker run --mount type=volume,src=myvolume,dst=/data,ro,volume-subpath=/foo
```

---

### Các tùy chọn cho `--volume`

Cờ `--volume` hoặc `-v` gồm ba phần, phân cách bằng dấu hai chấm (`:`), theo thứ tự:

```bash
$ docker run -v [<volume-name>:]<mount-path>[:opts]
```

- Nếu là volume có tên: phần đầu là tên volume, duy nhất trên mỗi host.
- Nếu là volume ẩn danh: bỏ qua phần đầu.
- Phần thứ hai là đường dẫn mount trong container.
- Phần thứ ba là các tùy chọn (ví dụ: `ro`).


---

## Trường thứ ba (trong `-v`) là tùy chọn

Trường thứ ba là tùy chọn, là danh sách các tùy chọn phân tách bởi dấu phẩy. Các tùy chọn hợp lệ khi sử dụng `--volume` với volume dữ liệu bao gồm:

| Tùy chọn           | Mô tả                                                                                                                                                              |
|--------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `readonly`, `ro`   | Nếu được chỉ định, volume sẽ được [mount vào container ở chế độ chỉ đọc](#use-a-read-only-volume).                                                                 |
| `volume-nocopy`    | Nếu được chỉ định, dữ liệu tại điểm mount sẽ **không** được copy vào volume nếu volume đang rỗng. Mặc định, nội dung tại điểm mount sẽ được copy nếu volume rỗng. |

Ví dụ:

```bash
$ docker run -v myvolume:/data:ro
```

---

## Tạo và quản lý volume

Không giống như bind mount, bạn có thể tạo và quản lý volume độc lập mà không cần liên quan đến bất kỳ container nào.

**Tạo một volume:**

```bash
$ docker volume create my-vol
```

**Liệt kê các volume:**

```bash
$ docker volume ls

local               my-vol
```

**Kiểm tra thông tin một volume:**

```bash
$ docker volume inspect my-vol
[
    {
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
        "Name": "my-vol",
        "Options": {},
        "Scope": "local"
    }
]
```

**Xóa một volume:**

```bash
$ docker volume rm my-vol
```

---

## Khởi chạy container với volume

Nếu bạn khởi chạy container và chỉ định một volume chưa tồn tại, Docker sẽ tự động tạo volume đó. Ví dụ dưới đây mount volume `myvol2` vào thư mục `/app/` trong container:

### Dùng `--mount`

```bash
$ docker run -d \
  --name devtest \
  --mount source=myvol2,target=/app \
  nginx:latest
```

### Dùng `-v`

```bash
$ docker run -d \
  --name devtest \
  -v myvol2:/app \
  nginx:latest
```

> Lưu ý: Hai lệnh trên cho kết quả giống nhau. Không thể chạy cả hai cùng lúc trừ khi bạn xóa container `devtest` và volume `myvol2` trước đó.

**Kiểm tra container và volume được mount đúng hay chưa:**

```bash
$ docker inspect devtest
```

Tìm phần `"Mounts"`:

```json
"Mounts": [
    {
        "Type": "volume",
        "Name": "myvol2",
        "Source": "/var/lib/docker/volumes/myvol2/_data",
        "Destination": "/app",
        "Driver": "local",
        "Mode": "",
        "RW": true,
        "Propagation": ""
    }
],
```

Điều này cho thấy volume được mount đúng, đường dẫn nguồn/đích chính xác, và chế độ mount là ghi-đọc (`read-write`).

**Dừng và xóa container, sau đó xóa volume:**

```bash
$ docker container stop devtest
$ docker container rm devtest
$ docker volume rm myvol2
```

---

## Sử dụng volume với Docker Compose

Ví dụ sau sử dụng một service trong Docker Compose với volume:

```yaml
services:
  frontend:
    image: node:lts
    volumes:
      - myapp:/home/node/app

volumes:
  myapp:
```

Khi chạy `docker compose up` lần đầu, volume sẽ được tạo. Docker sẽ tái sử dụng volume này trong các lần chạy sau.

**Tạo volume ngoài Compose và sử dụng lại:**

```yaml
services:
  frontend:
    image: node:lts
    volumes:
      - myapp:/home/node/app

volumes:
  myapp:
    external: true
```

> Tham khảo thêm phần [Volumes](/reference/compose-file/volumes.md) trong tài liệu Docker Compose để biết chi tiết.

---

## Khởi chạy một dịch vụ có sử dụng volume

Khi bạn khai báo volume trong Docker Swarm service, mỗi bản sao (replica) của container sẽ có volume riêng biệt nếu dùng driver `local`. Chúng **không chia sẻ dữ liệu** trừ khi dùng volume driver hỗ trợ chia sẻ.

Ví dụ sau tạo một service `nginx` với 4 bản sao, mỗi bản sao dùng volume `myvol2`:

```bash
$ docker service create -d \
  --replicas=4 \
  --name devtest-service \
  --mount source=myvol2,target=/app \
  nginx:latest
```

**Kiểm tra trạng thái service:**

```bash
$ docker service ps devtest-service
```

Kết quả:

```text
ID                  NAME                IMAGE           NODE        DESIRED STATE   CURRENT STATE            ERROR       PORTS
4d7oz1j85wwn        devtest-service.1   nginx:latest    moby        Running         Running 14 seconds ago
```

**Xóa service:**

```bash
$ docker service rm devtest-service
```

> Xóa service **không tự động xóa volume**. Bạn cần xóa volume riêng.

---

## Nạp dữ liệu vào volume bằng container

Khi khởi chạy container với volume mới được tạo, nếu thư mục mount (ví dụ `/app/`) có dữ liệu sẵn trong container, Docker sẽ **copy nội dung** thư mục đó vào volume.

Sau khi volume được nạp dữ liệu, container sẽ mount volume đó và các container khác dùng cùng volume cũng có quyền truy cập vào dữ liệu đã được copy.

Ví dụ sau chạy một container `nginx` và copy nội dung thư mục mặc định `/usr/share/nginx/html` vào volume `nginx-vol`:

### Dùng `--mount`

```bash
$ docker run -d \
  --name=nginxtest \
  --mount source=nginx-vol,destination=/usr/share/nginx/html \
  nginx:latest
```

### Dùng `-v`

```bash
$ docker run -d \
  --name=nginxtest \
  -v nginx-vol:/usr/share/nginx/html \
  nginx:latest
```

**Dọn dẹp sau khi thử nghiệm:**

```bash
$ docker container stop nginxtest
$ docker container rm nginxtest
$ docker volume rm nginx-vol
```

---

## Sử dụng volume chỉ đọc (read-only)

Với một số ứng dụng phát triển, container cần ghi dữ liệu vào bind mount để các thay đổi được phản ánh trở lại máy chủ Docker. Trong các trường hợp khác, container chỉ cần quyền đọc dữ liệu. Nhiều container có thể mount cùng một volume. Bạn có thể mount một volume dưới dạng `read-write` cho một số container và `read-only` cho các container khác cùng lúc.

Ví dụ sau thay đổi từ ví dụ trước đó, mount thư mục dưới dạng volume chỉ đọc bằng cách thêm `ro` vào danh sách tùy chọn (mặc định là rỗng), sau điểm mount trong container. Khi có nhiều tùy chọn, phân tách chúng bằng dấu phẩy.

Cả cú pháp `--mount` và `-v` đều cho kết quả giống nhau.

**Sử dụng `--mount`:**
```bash
docker run -d \
  --name=nginxtest \
  --mount source=nginx-vol,destination=/usr/share/nginx/html,readonly \
  nginx:latest
```

**Sử dụng `-v`:**
```bash
docker run -d \
  --name=nginxtest \
  -v nginx-vol:/usr/share/nginx/html:ro \
  nginx:latest
```

Sử dụng `docker inspect nginxtest` để kiểm tra Docker đã mount volume dưới dạng chỉ đọc chưa. Tìm phần `Mounts`:

```json
"Mounts": [
    {
        "Type": "volume",
        "Name": "nginx-vol",
        "Source": "/var/lib/docker/volumes/nginx-vol/_data",
        "Destination": "/usr/share/nginx/html",
        "Driver": "local",
        "Mode": "",
        "RW": false,
        "Propagation": ""
    }
]
```

Dừng và xóa container, sau đó xóa volume (bước xóa volume là riêng biệt):

```bash
docker container stop nginxtest
docker container rm nginxtest
docker volume rm nginx-vol
```

---

## Mount thư mục con của volume

Khi mount một volume vào container, bạn có thể chỉ định thư mục con trong volume bằng tham số `volume-subpath` với cờ `--mount`. Thư mục con phải tồn tại trong volume trước khi mount, nếu không mount sẽ thất bại.

Điều này hữu ích nếu bạn chỉ muốn chia sẻ một phần cụ thể trong volume. Ví dụ: có nhiều container cùng ghi log vào một volume chung, bạn có thể tạo các thư mục con riêng cho từng container.

Ví dụ:

```bash
docker volume create logs
docker run --rm \
  --mount src=logs,dst=/logs \
  alpine mkdir -p /logs/app1 /logs/app2
docker run -d \
  --name=app1 \
  --mount src=logs,dst=/var/log/app1/,volume-subpath=app1 \
  app1:latest
docker run -d \
  --name=app2 \
  --mount src=logs,dst=/var/log/app2,volume-subpath=app2 \
  app2:latest
```

Kết quả: mỗi container ghi log vào thư mục con riêng của mình trong volume `logs`, và không thể truy cập log của container khác.

---

## Chia sẻ dữ liệu giữa các máy

Khi xây dựng ứng dụng chịu lỗi (fault-tolerant), bạn có thể cần nhiều bản sao của một dịch vụ truy cập cùng tập tin.

Có thể dùng các phương pháp sau:
- Tích hợp logic trong ứng dụng để ghi tập tin lên cloud như Amazon S3.
- Tạo volume với driver hỗ trợ hệ thống lưu trữ bên ngoài như NFS, Amazon S3.

Volume driver giúp bạn tách biệt hệ thống lưu trữ với logic ứng dụng. Bạn có thể thay đổi driver lưu trữ (ví dụ từ NFS sang S3) mà không cần sửa ứng dụng.

---

## Sử dụng volume driver

Khi tạo volume bằng `docker volume create`, hoặc khi container sử dụng một volume chưa tồn tại, bạn có thể chỉ định driver volume.

Ví dụ với driver `vieux/sshfs`:

### Cài plugin `vieux/sshfs`:

```bash
docker plugin install --grant-all-permissions vieux/sshfs
```

### Tạo volume với SSHFS:

```bash
docker volume create --driver vieux/sshfs \
  -o sshcmd=test@node2:/home/test \
  -o password=testpassword \
  sshvolume
```

### Dùng volume đó với container:

```bash
docker run -d \
  --name sshfs-container \
  --mount type=volume,volume-driver=vieux/sshfs,src=sshvolume,target=/app,volume-opt=sshcmd=test@node2:/home/test,volume-opt=password=testpassword \
  nginx:latest
```

---

## Tạo volume NFS trong dịch vụ

Ví dụ dùng NFSv3:

```bash
docker service create -d \
  --name nfs-service \
  --mount 'type=volume,source=nfsvolume,target=/app,volume-driver=local,volume-opt=type=nfs,volume-opt=device=:/var/docker-nfs,volume-opt=o=addr=10.0.0.10' \
  nginx:latest
```

Dùng NFSv4:

```bash
docker service create -d \
  --name nfs-service \
  --mount 'type=volume,source=nfsvolume,target=/app,volume-driver=local,volume-opt=type=nfs,volume-opt=device=:/var/docker-nfs,"volume-opt=o=addr=10.0.0.10,rw,nfsvers=4,async"' \
  nginx:latest
```

---

## Mount volume từ Samba/CIFS

Không cần mount trước trên host:

```bash
docker volume create \
  --driver local \
  --opt type=cifs \
  --opt device=//uxxxxx.your-server.de/backup \
  --opt o=addr=uxxxxx.your-server.de,username=uxxxxxxx,password=*****,file_mode=0777,dir_mode=0777 \
  --name cif-volume
```

---

## Thiết bị lưu trữ dạng khối (block storage)

Có thể mount thiết bị lưu trữ khối như ổ cứng gắn ngoài vào container.

> ⚠️ **Cảnh báo**: Không khuyến nghị dùng nếu không chắc chắn bạn đang làm gì.

Ví dụ dùng file làm thiết bị lưu trữ khối:

1. Tạo file 1GB:

   ```bash
   fallocate -l 1G disk.raw
   ```

2. Format ext4:

   ```bash
   mkfs.ext4 disk.raw
   ```

3. Gán loop device:

   ```bash
   losetup -f --show disk.raw
   ```

4. Mount vào container:

   ```bash
   docker run -it --rm \
     --mount='type=volume,dst=/external-drive,volume-driver=local,volume-opt=device=/dev/loop5,volume-opt=type=ext4' \
     ubuntu bash
   ```

5. Gỡ thiết bị sau khi dùng:

   ```bash
   losetup -d /dev/loop5
   ```

---

## Sao lưu, khôi phục, hoặc di chuyển volume dữ liệu

### Sao lưu:

```bash
docker run -v /dbdata --name dbstore ubuntu /bin/bash
docker run --rm --volumes-from dbstore -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata
```

### Khôi phục:

```bash
docker run -v /dbdata --name dbstore2 ubuntu /bin/bash
docker run --rm --volumes-from dbstore2 -v $(pwd):/backup ubuntu bash -c "cd /dbdata && tar xvf /backup/backup.tar --strip 1"
```

---

## Xóa volume

Volume không tự động xóa khi container bị xóa.

### Tự động xóa volume ẩn danh (anonymous):

```bash
docker run --rm -v /foo -v awesome:/bar busybox top
```

> Lưu ý: nếu container khác dùng `--volumes-from`, volume sẽ vẫn tồn tại.

### Xóa tất cả volume không sử dụng:

```bash
docker volume prune
```

---
