# Router
## **1) Giới thiệu**
- Mặc định, các instance trong mạng nào thì chỉ giao tiếp với các instance gắn vào mạng đó . Để kết nối giữa 2 mạng layer 2 khác nhau, ta sử dụng Router .
- Router trong OpenStack cung cấp cho VM kết nối định tuyến dựa trên layer 3
- Một router sẽ hoạt động như một default gateway cho các mạng tenant kết nối vào nó :
    <p align=center><img src=https://i.imgur.com/Tp4VDmg.png width=50%></p>
- Khi một đường mạng external cắm vào router, nó có thể giúp forward các gói tin từ các mạng tenant ra ngoài internet :
    <p align=center><img src=https://i.imgur.com/zL0hnw2.png width=50%></p>
- **Outbound traffic**
    - Mặc định, Neutron router sẽ sử dụng **SNAT - *Source Network Address Translation*** cho các traffic ra ngoài từ các mạng tenant. Có nghĩa, tất cả các traffic đi ra từ router, router sẽ thay đổi địa chỉ nguồn của chúng thành địa chỉ nguồn của external interface. Điều này đảm bảo lượng traffic trả về sẽ về đúng cho router, sau đó các địa chỉ đích của traffic sẽ được thay đổi về đúng như cũ :
        <p align=center><img src=https://i.imgur.com/6GA95zD.png width=60%></p>
- **Inbound traffic**
    - Khi sử dụng **SNAT**, các inbound traffic (kết nối trả về) sẽ không thể có cách quay về đúng instance . Một floating IP là địa chỉ được sử dụng để cung cấp kết nối static NAT 1:1 map tới 1 IP cố định. Floating IP sẽ cung cấp 1 kết nối outbound và inbound duy nhất, cho phép traffic trả về đúng cho instance :
        <p align=center><img src=https://i.imgur.com/ehOtFEC.png width=60%></p>

## **2) Các loại router trong Neutron**
- Có 3 loại router trong Neutron:
    - Standalone
    - Highly available
    - Distributed
- **Standalone router** là một router logic khi được tạo sẽ tự tạo ra một network namespace trên node chạy **Neutron L3 agent** (thường sẽ chạy trên một node network hoặc trên chính node controller luôn) . Nếu node chứa namespace gặp sự cố, các kết nối thông qua namespace có thể bị hạn chế hoặc hoàn toàn không sử dụng được nữa . Đây là loại router mặc định phát hành cùng bản **Folsom** và được cả **LinuxBridge** và **OpenvSwitch** hỗ trợ .
- **HA (*High Availability*) router** là một router logic khi được tạo sẽ tự tạo ra 1 hoặc nhiều network namespace trên node chạy **Neutron L3 agent** . Một **HA router** sẽ tối ưu dịch vụ `keepalived` và **VRRP (*Virtual Routing Redundancy Protocol*)** giữa các network namespace để cung cấp cơ chế dự phòng . Chỉ có 1 namespace sẽ hoạt động ở chế độ **master virtual router**, còn lại sẽ ở trạng thái backup chờ **master** fail. Nếu **active router** fail, **backup router** sẽ chiếm quyền điều khiển ngay lập tức .
**HA router** cung cấp chế độ dự phòng không có trên **standalone router**, giúp tránh được hiện tượng nghẽn cổ chai khiến cho hiệu suất mạng bị kém . **HA router** đã có mặt từ bản **Juno**, và được cả **LinuxBridge** và **OpenvSwitch** hỗ trợ .
- **Distributed virtual router (DVR)** là một router logic khi được tạo ra sẽ tự tạo ra nhiều network namespace trên node network hoặc note compute . Mô hình phân phối virtual router đi khắc các compute node tương tự như tính năng multihost của Nova network cũ . Nó sẽ cung cấp tính dự phòng tốt hợn việc bằng việc giới hạn các điểm chết trên các compute node thay vì ở 1 network node . **Distributed virtual router** xuất hiện từ bản **Juno**, được hỗ trợ bởi **OpenvSwitch** từ bản **Liberty** .
- Để tạo mới :
    - **HA router** : thêm option `--ha {true | false}` khi tạo router
    - **Distributed router** : thêm option `--distributed {true | false}` khi tạo router
## **3) Cấu hình Router**
#### **Cấu hình ban đầu**
> Để cấu hình tạo router cần phải cấu hình plugin `router` và `l3-agent`
- **B1 :** Cấu hình plugin :
    ```
    # crudini --set /etc/neutron/neutron.conf DEFAULT service_plugins router
    ```
- **B2 :** Khởi động dịch vụ `neutron-l3-agent` :
    ```
    # systemctl enable neutron-l3-agent
    # systemctl start neutron-l3-agent
    ```
- **B3 :** Sửa file cấu hình `/etc/neutron/l3_agent.ini` :
    ```
    # crudini --set /etc/neutron/l3_agent.ini DEFAULT interface_driver linuxbridge
    ```
- **B4 :** Kiểm tra lại trạng thái dịch vụ :
    ```
    # systemctl status neutron-l3-agent
    ```
    <img src=https://i.imgur.com/4rPabC2.png>

    ```
    # openstack network agent list
    ```

    <img src=https://i.imgur.com/OX9iQmM.png>

#### **Tạo router**
- Cú pháp :
    ```
    # openstack router create <router_name>
    ```
    - **VD :**
        ```
        # openstack router create router1
        ```
        <img src=https://i.imgur.com/Q2UHjns.png>
#### **Gán interface cho router**
> Mục đích : Gắn external network làm gateway để truy cập internet , còn các mạng self-service cắm vào các interface để tham gia định tuyến
- Cú pháp :
    - Set gateway :
        ```
        # openstack router set <router_name> --external-gateway <external_network_name>
        ```
    - Add interface :
        ```
        # openstack router add subnet <router_name> <subnet_name>
        ```
- **VD :**
    ```
    # openstack router set router1 --external-gateway public
    ```
    ```
    # openstack router add subnet router1 sub1private1
    ```
    > Topology mạng sau khi gắn interface thành công :

    <img src=https://i.imgur.com/eWeaMzW.png>

    - Kiểm tra các instance chỉ sử dụng private network đã có thể ping thông qua internet :

        <img src=https://i.imgur.com/rHkEjmo.png>

#### **Xóa interface đã gán cho router**
- Kiểm tra ID của subnet interface :
    ```
    # openstack router show <router_name>
    ```
    <img src=https://i.imgur.com/TnML2dF.png>

- Xóa interface :
    ```
    # openstack router remove subnet <router_name> <subnet_ID>
    ```
#### **Xóa router** 
> Để xóa router, trước hết, cần gỡ hết tất cả các interface ra khỏi router .
- Cú pháp :
    ```
    # openstack router delete <router_name>
    ```