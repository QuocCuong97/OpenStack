# Port Security
## **1) Giới thiệu**
- Mặc định, **neutron** sẽ kích hoạt **port security** đi kèm mỗi port được tạo ra trong **OpenStack** . Các rule sau sẽ được kích hoạt theo mặc định :
    - Tất cả các traffic đi vào và đi ra sẽ bị block bởi port khi kết nối với instance (khi có **Security group**)
    - Chỉ traffic bắt nguồn từ cặp địa chỉ IP/MAC được coi như đặc thù từ **OpenStack** (như IP của compute, hoặc các IP đã được allow), sẽ được allow trên network

        <p align=center><img src=https://i.imgur.com/jelFYer.png></p>

## **2) Các command với port security**
- Kiểm tra port security đã được kích hoạt chưa :
    ```
    # openstack port set \
        [--description <description>]
        [--fixed-ip subnet=<subnet>,ip-address=<ip-address>]
        [--no-fixed-ip]
        [--device <device-id>]
        [--device-owner <device-owner>]
        [--vnic-type <vnic-type>]
        [--binding-profile <binding-profile>]
        [--no-binding-profile]
        [--host <host-id>]
        [--enable | --disable]
        [--name <name>]
        [--security-group <security-group>]
        [--no-security-group]
        [--enable-port-security | --disable-port-security]
        [--dns-name <dns-name>]
        [--allowed-address ip-address=<ip-address>[,mac-address=<mac-address>]]
        [--no-allowed-address]
        <port>
    ```
    <img src=https://i.imgur.com/sZUr5Ek.png>

    - Tại thời điểm này, security group sẽ được áp dụng :
        - Các IP ở ngoài sẽ không thể kết nối đến instance có port này
        - Dù là port của mạng public, tuy nhiên port sẽ chặn việc instance truy cập ra ngoài
        - Chỉ có compute chứa instance có thể kết nối đến port
### **Thêm alllowed IP**
- Cú pháp :
    ```
    # openstack set port <port_ID|port_name> --allowed-address ip-address=<IP_ADDR>,mac-address=<MAC_ADDR>
    ```
    > Phần `mac-address` có thể không khai báo
- **VD :**
    ```
    # openstack set port port02 --allowed-address ip-address=10.5.11.210
    ```
### **Xóa tất cả allowed IP**
- Cú pháp :
    ```
    # openstack set port <port_ID|port_name> --no-allowed-address
    ```
--------------------
Tham khảo
- https://superuser.openstack.org/articles/managing-port-level-security-openstack/
- https://www.packetflow.co.uk/openstack-neutron-port-security-explained/#fn1
- http://kimizhang.com/neutron-ml2-port-security/