# Các trình điều khiển mạng (Network drivers)

Hệ thống mạng của Docker được thiết kế dạng **có thể cắm thêm (pluggable)**, sử dụng các **trình điều khiển mạng (network drivers)**. Một số trình điều khiển có sẵn mặc định và cung cấp các chức năng mạng cốt lõi:

- **`bridge`**: Đây là trình điều khiển mạng mặc định. Nếu bạn không chỉ định trình điều khiển khi tạo mạng, Docker sẽ sử dụng kiểu mạng này. Bridge thường được dùng khi ứng dụng của bạn chạy trong container và cần giao tiếp với các container khác trên **cùng một host**.  
  Tham khảo thêm tại: [Bridge network driver](bridge.md)

- **`host`**: Gỡ bỏ sự cô lập mạng giữa container và máy chủ Docker, container sẽ sử dụng trực tiếp mạng của máy chủ.  
  Tham khảo thêm tại: [Host network driver](host.md)

- **`overlay`**: Kết nối nhiều Docker daemon với nhau, cho phép các dịch vụ trong Swarm và container giao tiếp qua các node khác nhau. Cách tiếp cận này loại bỏ nhu cầu định tuyến ở cấp hệ điều hành.  
  Tham khảo thêm tại: [Overlay network driver](overlay.md)

- **`ipvlan`**: Cho phép người dùng kiểm soát hoàn toàn việc gán địa chỉ IPv4 và IPv6. Trình điều khiển VLAN được xây dựng trên nền IPvlan cho phép quản trị viên kiểm soát hoàn toàn việc gắn thẻ VLAN lớp 2, thậm chí là định tuyến IPvlan lớp 3, hữu ích khi tích hợp với mạng vật lý (underlay network).  
  Tham khảo thêm tại: [IPvlan network driver](ipvlan.md)

- **`macvlan`**: Cho phép gán địa chỉ MAC riêng cho từng container, khiến chúng xuất hiện như thiết bị vật lý trên mạng. Docker daemon định tuyến lưu lượng dựa trên địa chỉ MAC của container. Trình điều khiển `macvlan` đặc biệt phù hợp với các ứng dụng cũ cần kết nối trực tiếp với mạng vật lý, thay vì thông qua mạng của host.  
  Tham khảo thêm tại: [Macvlan network driver](macvlan.md)

- **`none`**: Cô lập hoàn toàn container khỏi máy chủ và các container khác. Chế độ `none` không khả dụng với các dịch vụ Swarm.  
  Tham khảo thêm tại: [None network driver](none.md)

- **[Plugin mạng bên thứ ba](https://docs.docker.com/engine/extend/plugins_network/)**: Bạn có thể cài đặt và sử dụng các plugin mạng từ bên thứ ba với Docker.

---

### Tóm tắt các trình điều khiển mạng (Network driver summary)

- **Bridge mặc định**: Phù hợp cho container không cần khả năng mạng đặc biệt.
- **Bridge do người dùng định nghĩa**: Cho phép các container trên cùng Docker host giao tiếp với nhau. Mạng do người dùng định nghĩa thường được dùng để cô lập nhóm container thuộc cùng một dự án hoặc thành phần.
- **Host**: Container dùng chung mạng với máy chủ. Khi sử dụng driver này, container không còn bị cô lập mạng với host.
- **Overlay**: Phù hợp khi bạn cần các container trên nhiều Docker host giao tiếp với nhau, hoặc khi nhiều ứng dụng phối hợp thông qua các dịch vụ Swarm.
- **Macvlan**: Thích hợp khi bạn muốn chuyển đổi từ môi trường máy ảo sang container, hoặc cần container hoạt động như các máy chủ vật lý riêng biệt với địa chỉ MAC riêng.
- **IPvlan**: Tương tự Macvlan nhưng không gán địa chỉ MAC riêng. Dùng IPvlan khi bạn bị giới hạn số lượng MAC address có thể gán cho một interface hoặc port mạng.
- **Plugin mạng bên thứ ba**: Cho phép tích hợp Docker với các nền tảng mạng chuyên dụng khác.
