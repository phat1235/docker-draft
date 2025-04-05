# **Docker là gì?**

Docker là một nền tảng được thiết kế để giúp các nhà phát triển xây dựng, chia sẻ và chạy các ứng dụng container.

## **Tại sao lại chọn Docker?**
Vì sao lại sử dụng Docker trong khi chúng ta có thể sử dụng máy ảo (VM) để chạy ứng dụng?

Docker hiện nay được coi là phương pháp tiêu chuẩn để triển khai ứng dụng. Nó đã được chấp nhận rộng rãi với một hệ sinh thái mạnh mẽ được hỗ trợ bởi nhiều nhà cung cấp Linux.

Công cụ này nhằm mục đích tăng tốc và đơn giản hóa việc triển khai thông qua ảo hóa nhẹ.

### **Giải thích về ảo hóa nhẹ:**
Ảo hóa nhẹ là một công nghệ ảo hóa cho phép bạn đóng gói và cô lập các ứng dụng cùng với các phụ thuộc của chúng vào các đơn vị độc lập gọi là container. Những container này chia sẻ kernel của hệ điều hành host nhưng chạy trong môi trường cô lập. Cách tiếp cận này giảm thiểu chi phí overhead và nâng cao hiệu suất so với máy ảo (VMs).

![](https://img001.prntscr.com/file/img001/xQd7PWjuTmSdEmd3JzcJZg.png)
---

## **VMs vs Docker:**
Máy ảo yêu cầu chạy một hệ điều hành đầy đủ trên một hệ thống host vật lý, điều này thường dẫn đến việc tiêu tốn tài nguyên cao và thời gian khởi động lâu hơn. Ngược lại, Docker containers bao bọc ứng dụng và các phụ thuộc của nó, sử dụng kernel của hệ điều hành host và giảm thiểu overhead. Điều này giúp triển khai nhanh hơn, tối ưu tài nguyên và cải thiện khả năng mở rộng.

![](https://img001.prntscr.com/file/img001/UD_jpRypRjCjZZMYnVCmVw.png)
---

## **Các nguyên lý cơ bản của Docker: Container, Image và các khái niệm cốt lõi**

Docker hoạt động như một phần mềm client-server, tập trung vào các khái niệm về image và container:

### **Docker Image**:
Docker Image là một thành phần không hoạt động, là một gói meta chứa mã nguồn, các yếu tố tĩnh và phụ thuộc. Image này sẽ được lưu trữ cục bộ trên registry hoặc bộ nhớ cache cục bộ do Docker Engine quản lý. Image này được kích hoạt thành container. Một image có thể làm cơ sở cho một hoặc nhiều container.

![](https://img001.prntscr.com/file/img001/qaJMQzm8RQigCgW7AXnt_w.png)

### **Docker Container**:
Container là một môi trường cô lập hoặc một tập hợp các tiến trình ở cấp hệ thống. Trong các container này, các tiến trình thực thi mã được lưu trữ trong gói hoặc image.

![](https://img001.prntscr.com/file/img001/L4I4s7abTC64H5DsFJ7VVQ.png)
---

## **Container Runtime - Cấp cao và cấp thấp**:
Tất cả các công ty làm việc quanh container runtime cần phải đồng bộ hóa, và họ đã hợp tác để hình thành OCI (Open Container Initiative).

![](https://img001.prntscr.com/file/img001/_FQ9M7_PRjmQDaTI1-T1mg.png)
### **Open Container Initiative (OCI)**:
Mục tiêu của OCI là thiết lập các tiêu chuẩn đồng nhất cho các image và container, đơn giản hóa quá trình chuyển đổi giữa các runtime và tạo điều kiện cho việc sử dụng các image trên các nền tảng khác nhau, bao gồm Docker và các môi trường thay thế.

![](https://img001.prntscr.com/file/img001/GxoJQC1tRcmAHwgiMI_Xkg.png)

Những tiêu chuẩn này bao gồm ba thành phần quan trọng:
- **Layers**: Các lớp này định nghĩa các khối xây dựng của image, tạo thành một cấu trúc theo lớp. Mỗi lớp chứa các thay đổi hoặc bổ sung cụ thể vào image. Cách tiếp cận này giúp cải thiện hiệu quả trong việc lưu trữ và phân phối.
  
- **Manifests**: Manifests cho phép chỉ định các kiến trúc được hỗ trợ cho một image. Điều này đảm bảo tính tương thích trên các nền tảng phần cứng và kiến trúc khác nhau.
  
- **Configurations**: Các cấu hình bao gồm nhiều yếu tố, bao gồm các lệnh dòng lệnh cần thực thi trong image. Nó cũng bao gồm các tham số, đối số và thiết lập môi trường ảnh hưởng đến hành vi và tương tác của image.

---

## **Hiểu về các nguyên lý hoạt động của container**

Để hiểu rõ hơn về container và dễ dàng xử lý sự cố, việc hiểu rõ nguyên lý vận hành của container là rất quan trọng.

Bắt đầu với **Docker CLI**, đây là giao diện người dùng giúp tương tác với **dockerD API** (Docker Engine), được coi là container runtime cấp cao. Trong kiến trúc Docker, ‘dockerD’ sử dụng một container runtime cấp cao khác được gọi là **ContaineD** (cấp thấp hơn dockerD). ContaineD xử lý việc quản lý vòng đời container và chịu trách nhiệm về quản lý mạng, trong khi dưới lớp đó là container runtime cấp thấp **runC**, chịu trách nhiệm khởi chạy containers từ các images theo thông số kỹ thuật của **Open Container Initiative (OCI)**.
![](https://img001.prntscr.com/file/img001/WDRUJWh_Q0qQKxjj8-wzQA.png)
### **Lưu ý về hai yếu tố quan trọng trong việc khởi chạy container:**
- **Cgroups (Control Groups)**: Các cơ chế này cho phép giới hạn tài nguyên ở cấp hệ điều hành. Chúng điều chỉnh phân bổ tài nguyên cho CPU, bộ nhớ, đĩa...
- **Namespaces**: Cung cấp các lớp cô lập cho mỗi container, bao gồm PID namespace, cách ly mạng, cách ly volume mount, và tách biệt tên hệ thống của các file UTS hostnames.

Điều này đảm bảo mỗi container hoạt động trong môi trường cô lập của nó, giúp tăng cường bảo mật và tối ưu hóa tài nguyên.

---

### **Xử lý Container**

Để khởi tạo một container, sử dụng lệnh sau:
```bash
docker run -d --name Tên_container Tên_image
```

**Giải thích các thành phần trong lệnh:**

- `-d`: Tham số này chỉ định chế độ tách (detach), tức container sẽ được chạy ở chế độ nền.
- `--name`: Gán tên cụ thể cho container.
- `Tên_container`: Thay thế bằng tên bạn muốn đặt cho container.
- `Tên_image`: Thay thế bằng tên của image Docker sẽ được dùng làm nền tảng cho container.

**Một số tùy chọn bổ sung khi khởi chạy container:**

- `-ti`: Mở terminal tương tác, cho phép bạn tương tác trực tiếp với tiến trình bên trong container.
- `--rm`: Tự động xóa container khi container kết thúc, giúp môi trường luôn sạch.
- `--hostname`: Đặt hostname bên trong container.
- `--dns`: Chỉ định DNS server cho container (nên tránh trùng với DNS của máy host).

**Quản lý container:**

- Liệt kê các container đang hoạt động:  
  ```bash
  docker ps
  ```

- Liệt kê tất cả container (bao gồm đã thoát):  
  ```bash
  docker ps -a
  ```

- Trích xuất ID của các container đang hoạt động:  
  ```bash
  docker ps -q
  ```

- Hiển thị ID của tất cả container:  
  ```bash
  docker ps -qa
  ```

- **Xóa container**: không thể xóa container đang chạy trực tiếp. Trước tiên phải dừng nó:
  ```bash
  docker stop tên_container
  docker rm tên_container
  ```

- **Xóa cưỡng bức container đang chạy**:
  ```bash
  docker rm -f tên_container
  ```

- **Xóa tất cả container đang hoạt động**:
  ```bash
  docker rm -f $(docker ps -q)
  ```

- **Khởi động lại container đã dừng**:
  ```bash
  docker start tên_container
  ```


### **Mạng Docker**

Docker networking là yếu tố quan trọng cho hoạt động của container, đóng hai vai trò chính:

- Cho phép giao tiếp giữa các container.
- Quản lý luồng dữ liệu giữa container và môi trường bên ngoài.

**Các chế độ mạng (driver) trong Docker:**

- **Bridge**: Mặc định, sử dụng mạng `docker0` để giao tiếp giữa các container.
- **Host**: Container sử dụng mạng của máy chủ (host).
- **None**: Cô lập container hoàn toàn, không có giao tiếp ra bên ngoài.
- **Overlay**: Cho phép container trên nhiều máy khác nhau giao tiếp, phù hợp với mô hình swarm.

Bridge mode gán IP riêng cho mỗi container, dải IP phổ biến là `172.17.0.0/16`.

**Xem mạng docker0 (bridge):**
```bash
ifconfig
```
![](https://img001.prntscr.com/file/img001/Nt6UPmOXSI65mLlGhvCdkw.png)

**Liệt kê các mạng hiện có:**
```bash
docker network ls
```
![](https://img001.prntscr.com/file/img001/hF2gDudFSAGKzPTfZTWiWw.png)

Trên mỗi máy ảo hoặc máy vật lý thường có giao diện `eth1`, bridge là lớp logic kết nối vào giao diện này nhằm tạo một mạng nội bộ ảo riêng biệt trong máy chủ host.
![](https://img001.prntscr.com/file/img001/8bhmnyU9SoyArw12ZTmqQA.png)

Bridge này sẽ tự động sinh ra một interface ảo (veth) cho mỗi container, cho phép các container giao tiếp với nhau thông qua mạng bridge nhưng vẫn tách biệt với các mạng bên ngoài.

### **Expose và Publish cổng (Port)**

**Expose port** và **Publish port** là hai khái niệm liên quan đến cách container giao tiếp với bên ngoài.

#### **Expose port**

Expose port là cách khai báo metadata rằng container có thể lắng nghe trên cổng đó, nhưng **không đảm bảo** ứng dụng trong container sẽ thực sự sử dụng cổng đó.

Ví dụ:
```bash
docker run -d --name c1 --expose 8080 nginx:latest
```
Cổng 8080 được expose, tuy nhiên nginx mặc định lắng nghe cổng 80, dẫn đến việc container không phản hồi nếu không map đúng cổng.
![](https://img001.prntscr.com/file/img001/vSOLcQ9sTaufftPhob23qg.png)

![](https://img001.prntscr.com/file/img001/-3lYXSguTGSc2cnpOmkdtw.png)
#### **Publish port**

Publish port là việc ánh xạ (mapping) cổng trên máy chủ với cổng tương ứng trong container để ứng dụng bên trong container có thể được truy cập từ bên ngoài.

- **Cấu hình thủ công với `-p`**:
  ```bash
  docker run -d --name c1 -p 8080:80 nginx
  ```

- **Gán ngẫu nhiên cổng với `-P`**:
  ```bash
  docker run -d --name c1 -P nginx
  ```

Ví dụ: cổng 8080 trên host được ánh xạ đến cổng 80 trong container (dạng TCP).
![](https://img001.prntscr.com/file/img001/18gCCI10Rw2fZTiigRmdLw.png)
---

### **Quản lý mạng Docker nâng cao**

Mặc định, các container không có IP tĩnh – IP được cấp phát mỗi lần tạo và sẽ mất khi container bị xóa hoặc dừng.

**Giải pháp**: Tạo **mạng tùy chỉnh (custom network)**.

Khác với mạng bridge `docker0` không hỗ trợ DNS nội bộ, custom network cho phép các container giao tiếp bằng tên DNS của nhau.

#### **Tạo container trong custom network**

1. **Tạo mạng tùy chỉnh:**
   ```bash
   docker network create tên_mạng
   ```

2. **Khởi chạy container trong mạng đó:**
   ```bash
   docker run -d --network tên_mạng tên_image
   ```

Khi đó, container có thể gọi nhau bằng cú pháp: `tên_container.tên_mạng`
![](https://img001.prntscr.com/file/img001/bj5EFMXjSpGIgq1p5Te2gQ.png)
Hoặc dùng lệnh:
```bash
docker network connect tên_mạng tên_container
```
để kết nối một container có sẵn vào mạng tùy chỉnh.
![](https://img001.prntscr.com/file/img001/QaSFei_zTEKnb261VeWvmw.png)

### **Docker Volume**

Dữ liệu là yếu tố quan trọng với ứng dụng (như cơ sở dữ liệu, nội dung tĩnh...). **Volume** trong Docker dùng để lưu trữ dữ liệu **bền vững** và độc lập với vòng đời của container.

![](https://img001.prntscr.com/file/img001/Y_FB2yIwQGC72evo4N-wCQ.png)
Namespace giúp cô lập volume, đảm bảo dữ liệu không bị chia sẻ giữa các container nếu không cần thiết.

Volume thực tế là một thư mục trên máy chủ host.

Ví dụ: ứng dụng cần lưu dữ liệu vào `/data/`, bạn tạo volume và mount nó vào thư mục này.

#### **Quản lý volume:**

- Liệt kê volume:
  ```bash
  docker volume ls
  ```

- Tạo volume mới:
  ```bash
  docker volume create tên_volume
  ```

- Xóa volume:
  ```bash
  docker volume rm tên_volume
  ```

- Xem chi tiết volume:
  ```bash
  docker volume inspect tên_volume
  ```

- Gắn volume vào container:
  ```bash
  docker run -d --name c1 -v tên_volume:/path/in/container image:tag
  ```

Metadata của volume chứa thông tin `Mountpoint`, là nơi dữ liệu volume được lưu trên host.
![](https://img001.prntscr.com/file/img001/dsmX_CwuQqG0QeLwT0VReQ.png)

### **Các phương thức lưu trữ dữ liệu trong Docker**

![](https://img001.prntscr.com/file/img001/Kjz9TjFzRBy9t8yTaFMqug.png)
#### **1. Bind Mount**
Mount một thư mục từ host vào container. Dữ liệu trên host ghi đè dữ liệu trong image.
```bash
docker run -d --name c1 -v /data/:/usr/data/ image:tag
docker inspect --format "{{.Mounts}}" c1
```

#### **2. Docker Volume**
Sử dụng volume được Docker quản lý:
```bash
docker run -d --name c1 --mount type=volume,src=volume_name,destination=/usr/data image:tag
```

#### **3. TMPFS**
Tạo không gian tạm trong RAM, không lưu trữ sau khi container dừng:
```bash
docker run -d --name c1 --mount type=tmpfs,destination=/usr/data nginx:tag
```

Hai phương pháp gắn volume qua dòng lệnh:

- Dùng `-v` hoặc `--volume`
- Dùng `--mount` với loại cụ thể (volume, tmpfs, bind,...)


### **Kết luận**

Image, Container, Network và Volume là các thành phần cốt lõi của Docker giúp tạo ra môi trường phát triển và triển khai ứng dụng linh hoạt, dễ mang vác (portable) và hiệu quả. Việc hiểu và vận dụng tốt các thành phần này sẽ giúp bạn khai thác tối đa sức mạnh của Docker trong quản lý và triển khai ứng dụng.

