

## I. GIỚI THIỆU

Docker là một nền tảng tiêu chuẩn mở dùng để xây dựng, xuất bản và chạy các ứng dụng. Docker cũng có thể được xem như một chương trình dòng lệnh với một daemon chạy nền và một tập hợp các tiện ích từ xa, áp dụng phương pháp hệ thống để giải quyết các vấn đề phần mềm phổ biến, đồng thời đơn giản hóa trải nghiệm người dùng trong việc cài đặt, chạy, xuất bản và gỡ bỏ ứng dụng.

Docker cung cấp khả năng tách biệt ứng dụng khỏi hạ tầng, giúp triển khai phần mềm nhanh chóng. Một container Docker là một “thùng chứa” phần mềm bao gồm toàn bộ các thành phần phụ thuộc cần thiết để chạy ứng dụng một cách độc lập. Ngoài ra, tính năng cách ly và bảo mật trong Docker cho phép chạy nhiều container đồng thời trên cùng một máy chủ vật lý.

Bài viết này cung cấp cái nhìn sâu về công nghệ Docker và phân tích cách Docker đã thay đổi cục diện của toàn bộ ngành ảo hóa. Nội dung tài liệu được hệ thống như sau:  
- Phần 2: Giải thích Docker thực chất là gì  
- Phần 3: Tập trung vào kiến trúc Docker và các thành phần của nó  
- Phần 4: So sánh ảo hóa nhẹ bằng Docker và ảo hóa truyền thống  
- Phần 5 và 6: Phân tích lý do nên hoặc không nên sử dụng Docker  
- Phần 7: So sánh giữa container và máy ảo  
- Phần 8: Kết luận và triển vọng tương lai của công nghệ Docker  

---

## II. KHÁI NIỆM DOCKER

Docker là một nền tảng được thiết kế nhằm đơn giản hóa quá trình tạo, triển khai và chạy các container ứng dụng ảo hóa trên một hệ điều hành chung, với một hệ sinh thái công cụ hỗ trợ đi kèm. Ban đầu, Docker được phát triển để chạy trên nền tảng Linux, nhưng sau đó đã được mở rộng để hỗ trợ các hệ điều hành khác như Mac OSX và Microsoft Windows. Các phiên bản Docker cho Amazon Web Services (AWS) và Microsoft Azure cũng đã được phát hành.

Trước khi có Docker, việc triển khai phần mềm lên các môi trường khác nhau đòi hỏi nhiều công sức. Dù các nỗ lực đó được gói gọn trong máy ảo, nhưng vẫn tốn thời gian để quản lý, triển khai, cài đặt, khởi động và xử lý overhead tài nguyên mà máy ảo tạo ra.

Với Docker, các image có thể được tải về từ kho chứa công khai hoặc riêng tư và sẵn sàng để chạy, tiêu thụ ít tài nguyên hơn và được cô lập để không ảnh hưởng đến các môi trường khác.

Người dùng không cần lo lắng container của mình sẽ chạy trên Ubuntu, CentOS hay hệ điều hành nào khác, miễn là máy chủ đó đã cài Docker.

---

## III. KIẾN TRÚC DOCKER

Docker có thể được xem như một ứng dụng dựa trên kiến trúc client-server. Trên máy chủ vật lý, Docker được chia thành hai phần:

- **Docker Daemon (Server)** với REST API  
- **Docker CLI (Client)** – giao tiếp với daemon

REST API sử dụng các phương thức HTTP tiêu chuẩn như `GET`, `POST`, `DELETE`, cho phép chương trình giao tiếp với Docker daemon.

![](https://img001.prntscr.com/file/img001/zmiXJ54kSseY_18tKrRYpQ.png)

Docker daemon là thành phần lõi trong kiến trúc Docker, chịu trách nhiệm nhận các lệnh từ client để xây dựng, chạy và phân phối các container. Docker cho phép chạy client và daemon trên cùng một thiết bị, hoặc client có thể kết nối từ xa đến daemon qua giao diện mạng, sử dụng REST API.

### A. Docker Daemon

Docker daemon (xem hình 3) là trung tâm xử lý các tương tác với Docker. Nó chờ nhận các yêu cầu từ API và quản lý trạng thái của các đối tượng Docker tương ứng.
![](https://img001.prntscr.com/file/img001/4tefPDc5Ra-HH8HJtuSxow.png)

### B. Docker Client

Docker client (xem hình 4) là thành phần đơn giản nhất trong kiến trúc Docker, giúp người dùng giao tiếp với hệ thống. Khi người dùng gõ các lệnh như `docker run` hoặc `docker pull`, Docker client sẽ gửi yêu cầu HTTP tới Docker daemon.
![](https://img001.prntscr.com/file/img001/utPLbYRbRiWTtE6Fog01_A.png)

### C. Docker Registries

Khi người dùng tạo ra các image riêng và muốn chia sẻ chúng với người khác, **Docker Registry** là nơi thực hiện điều đó. Docker registry cung cấp nền tảng để lưu trữ các Docker image.

- **Docker Hub** và **Docker Cloud** là hai registry công khai cho phép mọi người tải lên hoặc tải về các image.  
- Mặc định, Docker sẽ tìm image trên Docker Hub.  
- Các doanh nghiệp thường triển khai registry riêng để lưu trữ và chia sẻ image nội bộ.

Dùng lệnh `docker pull` hoặc `docker run` để tải image từ registry. Dùng `docker push` để đẩy image lên registry đã cấu hình.

![](https://img001.prntscr.com/file/img001/pWtpjK3AT5eEGL_7ZemKdw.png)
### D. Docker Images

Trong Docker, mọi thứ đều bắt nguồn từ **image**. Image là một template chỉ đọc, dùng để tạo container mới. Cơ sở để tạo một image là **base image** – ví dụ, Ubuntu 16.04 LTS đã cài sẵn Apache và ứng dụng web.

Quá trình tạo image từ base image gọi là “committing a change”. Người dùng có thể tạo image bằng cách định nghĩa các bước trong **Dockerfile**. Mỗi lệnh trong Dockerfile tạo ra một **lớp (layer)** trong image.

Khi người dùng chỉnh sửa Dockerfile và xây dựng lại image, chỉ những lớp thay đổi mới được cập nhật – điều này làm cho Docker image nhẹ, nhỏ gọn, nhanh và đáng tin cậy hơn so với công nghệ ảo hóa truyền thống.

![](https://img001.prntscr.com/file/img001/sNjwRiTsTf2UHlRPnZzF0w.png)

### E. Docker Containers

Nhìn sơ qua, Docker container có vẻ giống một phiên bản nhẹ của máy ảo. Container được tạo từ image và có thể xem là một instance chạy được của image đó.

Container chứa đầy đủ tất cả các thành phần phụ thuộc mà ứng dụng cần để chạy trong một môi trường cô lập. Thông qua Docker CLI hoặc API, người dùng có thể dễ dàng tạo, chạy, di chuyển, dừng hoặc xóa container.

Docker container luôn nằm trong một trong bốn trạng thái chính như minh họa trong hình dưới.

![](https://img001.prntscr.com/file/img001/7dU60jpoTROblJjGVmERag.png)

---


### IV. DOCKER SO VỚI MÁY ẢO (VIRTUAL MACHINE)

Trước khi so sánh Docker với máy ảo, trước hết chúng ta cần hiểu các khái niệm “ảo hóa nặng” (heavyweight virtualization) và “ảo hóa nhẹ” (lightweight virtualization).

**Ảo hóa nặng** nghĩa là mỗi máy ảo đều có hệ điều hành độc lập riêng. Với sự hỗ trợ của phần mềm ảo hóa, bạn có thể cài đặt bất kỳ hệ điều hành khách nào (guest OS) như Windows, Linux, hoặc macOS lên hệ điều hành chủ (host OS). Các phần mềm ảo hóa như VMware Workstation, KVM, Oracle VirtualBox và Hyper-V thuộc loại ảo hóa Type-2, tức là chúng không thể chạy trực tiếp trên phần cứng hệ thống. Vì vậy, một công nghệ ảo hóa mới gọi là **hypervisor** đã ra đời, có thể cài đặt trực tiếp lên phần cứng hệ thống – đây là **ảo hóa Type-1**.

Tuy nhiên, vấn đề của các phần mềm này là bạn phải cài đặt toàn bộ hệ điều hành khách chỉ để thực hiện các thao tác đơn giản như kiểm tra kết nối mạng giữa hai máy ảo bằng lệnh `ping`, điều này dẫn đến lãng phí tài nguyên không cần thiết.

Hiện nay, xu hướng đang chuyển sang công nghệ ảo hóa nhẹ dựa trên container, được biết đến với tên gọi **Docker** như được mô tả trong dưới. Với Docker, ứng dụng có thể được đóng gói bên trong container, cung cấp nền tảng “Container như một Dịch vụ” (CaaS – Container as a Service). Các container chứa ứng dụng rất dễ quản lý và có thể triển khai trên bất kỳ nền tảng nào, kể cả đám mây. Ngoài ra, các công nghệ nhẹ kiểu này có tốc độ cao và tiêu thụ rất ít tài nguyên.
![](https://img001.prntscr.com/file/img001/goMk1uC8S_2jRrhyh95STw.png)

Ở giai đoạn đầu, nếu so sánh Docker với máy ảo, ta thấy container Docker có khả năng chia sẻ nhân hệ điều hành (kernel) và thư viện ứng dụng. Bên cạnh đó, Docker có **độ trễ hệ thống thấp hơn** (ví dụ: thời gian xử lý, bộ nhớ,…), do đó hiệu năng của ứng dụng chạy trong container **thường tốt hơn** so với ứng dụng chạy trong máy ảo.

Khả năng của máy ảo có thể được tăng cường nhờ các công nghệ như **Intel VT-x/EPT** và **VT-d** – những công nghệ đã được VMware tích hợp. VT-x/EPT và VT-d giúp hệ điều hành khách nhận được sức mạnh xử lý tương đương với hệ điều hành chủ và cải thiện hiệu suất mạng. Ngoài ra, các công nghệ này còn giúp máy ảo đạt được **cách ly phần cứng mức ring-1**. “Ring” hoạt động như một lớp bảo vệ để bảo vệ dữ liệu và chức năng nhạy cảm khỏi lỗi và hành vi bất thường, tăng khả năng chịu lỗi và bảo mật máy tính.

Theo Gupta V. (2017), **thời gian tạo và khởi động container Docker ngắn hơn nhiều** so với máy ảo. Đồng thời, container chỉ sử dụng tài nguyên đúng với nhu cầu của dịch vụ hoặc ứng dụng cụ thể. Nhờ vậy, có thể chạy được **nhiều container hơn trên hệ thống có cấu hình không quá mạnh**.

---

### V. ƯU ĐIỂM CỦA CONTAINER DOCKER

Docker đã trở thành một phần cốt lõi trong thế giới DevOps kể từ khi xuất hiện vào năm 2013. Lý do chính nằm ở các lợi ích vượt trội mà container Docker mang lại. Dưới đây là một số đặc điểm nổi bật:

#### A. Miễn phí và Mã nguồn mở  
Thông thường, khi sử dụng các phần mềm ảo hóa như VMware để tạo máy ảo, người dùng cần mua giấy phép thương mại khi dùng trong doanh nghiệp. Trong khi đó, Docker hoàn toàn **miễn phí và mã nguồn mở**, có thể tải về và cài đặt dễ dàng mà không cần mua giấy phép để chạy ứng dụng.

#### B. Tính nhất quán cao  
Máy ảo cần một lượng tài nguyên lớn, nếu không đủ sẽ hoạt động không ổn định. Ngược lại, Docker **tiêu tốn ít tài nguyên hơn** nên có thể đạt được **tính nhất quán cao hơn**.

#### C. Triển khai nhanh  
Trước đây, việc triển khai phần cứng mới mất vài ngày. Sau đó, máy ảo đã rút ngắn thời gian này xuống vài phút. Với Docker, quá trình này chỉ mất **vài giây**, đồng thời **chi phí tạo và xóa container là rất thấp**.

#### D. Tính di động cao  
Ứng dụng trong Docker container có **tính di động và nhẹ**, có thể đóng gói thành một đơn vị độc lập, dễ dàng di chuyển mà không ảnh hưởng đến hiệu suất [10].

#### E. Cô lập tài nguyên  
Mỗi container có tập tài nguyên riêng và hoạt động **độc lập**, không ảnh hưởng đến container khác. Việc tạo và xóa ứng dụng trở nên dễ dàng vì mỗi ứng dụng chạy trong container riêng biệt. **Việc xóa container không để lại file cấu hình dư thừa**. Docker đảm bảo mỗi ứng dụng **chỉ sử dụng tài nguyên được cấp cho nó**.

#### F. Bảo mật  
Docker **bảo mật cao** vì mỗi container và dịch vụ trong đó hoàn toàn **độc lập**. Không có sự chia sẻ tài nguyên giữa các container, do đó nếu một container gặp sự cố, **các container khác không bị ảnh hưởng**.

---

### VI. NHƯỢC ĐIỂM CỦA CONTAINER DOCKER

Trước khi chuyển sang sử dụng Docker, cần lưu ý một số hạn chế sau:

#### A. Lưu trữ dữ liệu lâu dài phức tạp  
Khi container bị tắt, **toàn bộ dữ liệu bên trong sẽ biến mất**, trừ khi bạn lưu ở nơi khác. Docker có hỗ trợ các giải pháp lưu trữ như **Docker Volumes**, nhưng việc sử dụng hiệu quả vẫn còn là **thách thức**.

#### B. Không tương thích với máy 32-bit  
Docker hiện **chỉ hoạt động trên máy 64-bit**, **không hỗ trợ** máy 32-bit.

#### C. Không đạt hiệu suất như phần cứng gốc  
Dù Docker tiết kiệm tài nguyên hơn máy ảo, **hiệu suất vẫn thấp hơn phần cứng vật lý**, chủ yếu do sử dụng overlay network.

#### D. Phụ thuộc vào nền tảng  
Docker ban đầu được thiết kế cho Linux. Ngày nay đã có thể chạy trên Windows, macOS,... nhưng để đạt tính **đa nền tảng**, cần thêm một **lớp trung gian giữa hệ điều hành chủ và Docker**.

---

### VII. SO SÁNH ẢO HÓA TRUYỀN THỐNG VÀ CONTAINER

Việc lựa chọn công nghệ ảo hóa hay container phụ thuộc vào nhu cầu người dùng. Môi trường container thường cho phép **triển khai nhanh, hiệu quả cao và tiêu tốn ít tài nguyên**. Nếu bạn cần **cách ly hệ thống tốt hơn**, công nghệ máy ảo vẫn là lựa chọn phù hợp. Tuy nhiên, Docker đang dần **thay đổi cục diện**.

Theo Gupta V. (2017), **Bảng 1** so sánh giữa VirtualBox (ảo hóa nặng) và Docker (ảo hóa nhẹ) dựa trên các yếu tố như kích thước, bộ nhớ, lưu trữ, thời gian cài đặt và khởi động.

## **Bảng 1:** So sánh Ảo hóa nặng và nhẹ  
### So sánh giữa Ảo hóa nặng (VirtualBox) và Ảo hóa nhẹ (Docker)

| **Tiêu chí**                     | **Ảo hóa nặng (VirtualBox)**           | **Ảo hóa nhẹ (Docker)**               |
|----------------------------------|----------------------------------------|---------------------------------------|
| **Dung lượng ISO (Ubuntu 16.04)**   | 667 MB                                 | 130 MB                                |
| **RAM tiêu thụ**                  | 1 GB                                   | 568 KB                                |
| **Dung lượng lưu trữ**            | 10 GB                                  | 104 KB                                |
| **Thời gian cài đặt máy ảo**       | 21 phút                                | 0.9 giây (tạo container)              |
| **Dung lượng phần mềm**           | 108 MB (VirtualBox)                    | 19.4 MB (Docker Engine)               |
| **Thời gian khởi động**           | 35 giây                                | Dưới 1 giây                           |

## **Bảng 2 (JC Wang, 2015):** Container khởi động trong vài giây, nhanh hơn và tiết kiệm không gian hơn [13].  
### So sánh Ảo hóa và Container

| **Tiêu chí**                     | **Ảo hóa (Virtualization – ví dụ: KVM, Xen)**                         | **Container (ví dụ: LXC, Docker)**                               |
|----------------------------------|------------------------------------------------------------------------|-------------------------------------------------------------------|
| **Triển khai điển hình trên máy chủ** | 10 – 100 máy ảo (VMs)                                                  | 100 – 1000 container                                              |
| **Thời gian khởi động (Boot Time)**  | Dưới 1 phút                                                             | Trong vài giây                                                    |
| **Tài nguyên vật lý (Physical Resources)** | Mỗi máy ảo được cấp tài nguyên riêng biệt                              | Tài nguyên được chia sẻ giữa các container                        |

### **Bảng 3 (Dua et al., 2014):** So sánh giữa VM và Container dựa trên hiệu năng, bảo mật, mạng, lưu trữ,…
### So sánh Máy Ảo và Container

| **Tiêu chí**         | **Máy Ảo (Virtual Machines)**                                                                 | **Container (Docker)**                                                                 |
|----------------------|-----------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------|
| **Hệ điều hành khách (Guest OS)** | Mỗi máy ảo chạy trên phần cứng ảo riêng và có kernel được nạp vào vùng nhớ riêng biệt.        | Tất cả container dùng chung một hệ điều hành và kernel. Kernel được nạp vào bộ nhớ vật lý. |
| **Giao tiếp giữa các tiến trình (Communication)** | Giao tiếp qua các thiết bị Ethernet ảo.                                                    | Sử dụng các cơ chế IPC chuẩn như: signal, pipe, socket,…                              |
| **Bảo mật (Security)**          | Phụ thuộc vào cách triển khai của Hypervisor.                                               | Có thể áp dụng kiểm soát truy cập bắt buộc (Mandatory Access Control).                |
| **Hiệu năng (Performance)**     | Có độ trễ nhỏ do lệnh máy phải được chuyển dịch từ Hệ điều hành khách sang Hệ điều hành chủ. | Hiệu năng gần như tương đương với hệ điều hành chủ.                                   |
| **Cô lập (Isolation)**          | Không thể chia sẻ thư viện, tập tin,… giữa các máy ảo hoặc giữa máy ảo và máy chủ.          | Có thể gắn (mount) thư mục con và chia sẻ một cách trong suốt.                        |
| **Thời gian khởi động (Start-up Time)** | Mất vài phút để khởi động.                                                                | Khởi động rất nhanh do dùng chung hệ điều hành nền.                                   |
| **Lưu trữ (Storage)**           | Tốn nhiều dung lượng vì cần cài đặt toàn bộ kernel và các chương trình liên quan.           | Dung lượng ít hơn vì dùng chung hệ điều hành nền.                                     |




### VIII. KẾT LUẬN VÀ HƯỚNG PHÁT TRIỂN

Kiến trúc **Microservices** là nền tảng cốt lõi mà Docker xây dựng trên đó. Kiến trúc này giúp **giảm thời gian xây dựng và triển khai container**. Đồng thời, nhu cầu tài nguyên của container rất thấp nhưng vẫn đạt hiệu năng cao. Docker cung cấp các tính năng mạnh mẽ, đảm bảo **tính linh hoạt, dễ sử dụng và khả năng mở rộng**.

Mặc dù còn một số vấn đề bảo mật, nhưng có thể cải thiện bằng cách sử dụng các công cụ hỗ trợ. Mục tiêu của bài viết là cung cấp cái nhìn tổng quan về các thành phần quan trọng của Docker và sự khác biệt giữa Docker và công nghệ ảo hóa truyền thống.

