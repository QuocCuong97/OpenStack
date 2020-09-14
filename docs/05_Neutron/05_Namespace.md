# Namespace
## **1) Giới thiệu**
- **OpenStack** cung cấp môi trường **multitenancy**, cung cấp cho user khả năng tạo vả quản lý các compute và tài nguyên network. **Neutron** hỗ trợ mỗi **tenant** (hay **project**) có thể có nhiều **private network**, **router**, **firewall**, **load balancer** và các tài nguyên mạng khác. Khả năng cô lập các object này là dựa vào **namespace** .
- Một **namespace** được định nghĩa là một bản copy logic của 1 hệ thống mạng với **route** riêng, **firewall rule** riêng, interface riêng . Mỗi khi sử dụng một plugin hay driver open-source, các **router**, **network** hay **load balancer** được tạo ra bởi user đều đại diện cho 1 **network namespace** . Khi **network namespace** được bật, **Neutron** có thể cô lập môi trường DHCP và dịch vụ routing tới các mạng khác. Các dịch vụ này cho phép user tạo nên các mạng overlap với các user khác trong các project khác và thậm chí là các mạng khác trên cùng project
- Để xem các namespace đang được sử dụng, sử dụng lệnh :
    ```
    # ip netns list
    ```
    > Nếu chưa thêm bất kỳ **namespace** nào, output sẽ trống. **Namespace** mặc định thì không được tính vào output của lệnh `ip netns list`

- Các namespace hiển thị dưới dạng
    - `qdhcp-*` (DHCP namespace)
    - `qrouter-*` (Router namespace)
    - `qlbaas-*` (Load Balancing namespace)
- **VD :**
    - Khi cài đặt plugin `router` :

        <img src=https://i.imgur.com/6nAxmjN.png>
    
    - Khi cài đặt service `dhcp_agent` :

        <img src=https://i.imgur.com/pYRDlRR.png>

        <img src=https://i.imgur.com/8Q81RzM.png>

## **2) Thực thi trong namespace**
- Sử dụng lệnh `ip netns exec <namespace>`, có thể thực thi được lệnh trong namespace. Các lệnh hữu ích như `ip`, `netstat`, `ps`, `iptables` sẽ cung cấp chi tiết về scope của namespace đang sử dụng .
### **List các namespace đang có**
- Khi cài đặt plugin `router` :

    <img src=https://i.imgur.com/6nAxmjN.png>

- Khi cài đặt service `dhcp_agent` :

    <img src=https://i.imgur.com/pYRDlRR.png>

    <img src=https://i.imgur.com/8Q81RzM.png>

### **Truy cập shell namespace**
- Cú pháp :
    ```
    # ip netns exec <namespace> /bin/bash
    ```
- **VD1 :** Truy cập namespace `qrouter` :
    ```
    # ip netns exec qrouter-045c5928-2a51-4b3e-a5fb-9a1e4386b13b /bin/bash
    ```
    <img src=https://i.imgur.com/moCdcU4.png>

    > Sử dụng lệnh `exit` để thoát khỏi shell
- **VD2 :** Truy cập namespace `qdhcp` :
    ```
    # ip netns exec qdhcp-4a9229fc-9d72-449d-b908-b8ca5c5be852 /bin/bash
    ```
    <img src=https://i.imgur.com/ajrdBtS.png>

    > Sử dụng lệnh `exit` để thoát khỏi shell
### **Xem bảng định tuyến trong namespace**
- **VD1 :** namespace `qrouter` :
    ```
    # ip netns exec qrouter-045c5928-2a51-4b3e-a5fb-9a1e4386b13b route -n
    ```
    <img src=https://i.imgur.com/eCci6e0.png>
- **VD2 :** namespace `qdhcp` :
    ```
    # ip netns exec qdhcp-4a9229fc-9d72-449d-b908-b8ca5c5be852 route -n
    ```
    <img src=https://i.imgur.com/hBeIrB8.png>