# Port Security
## **1) Giới thiệu**
- Mặc định, **neutron** sẽ kích hoạt **port security** đi kèm mỗi port được tạo ra trong **OpenStack** . Các rule sau sẽ được kích hoạt theo mặc định :
    - Tất cả các traffic đi vào và đi ra sẽ bị block bởi port khi kết nối với instance 
    - Chỉ traffic bắt nguồn từ cặp địa chỉ IP/MAC mặc định (như IP của compute, hoặc các IP gốc của port), sẽ được allow trên network

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
## **3) Lab kiểm chứng**
- **B1 :** Kiểm tra các cặp default IP/MAC :
    ```
    # openstack port list
    ```
    <img src=https://i.imgur.com/Ksrilds.png>
    
    > Đây là cặp IP/MAC mặc định được sử dụng khi tạo port. **Port security** sẽ ngăn chặn việc thay đổi cặp IP/MAC này
- **B2 :** Kiểm tra **port security** đã được bật trên port :
    ```
    # openstack port show 35147cdc-1a22-4b7b-b136-7d3e8bead504 | grep 'port_security_enabled'
    ```
    <img src=https://i.imgur.com/uxgz1XH.png>

- **B3 :** Thực hiện ping tới gateway trên instance :
    ```
    # ping 192.168.1.1
    ```
    <img src=https://i.imgur.com/tTVMhW8.png>

- **B4 :** Thực hiện thay đổi thử IP trên instance :
    ```
    # ifconfig eth0 192.168.1.200/24
    # route add default gw 192.168.1.1 eth0
    ```
    <img src=https://i.imgur.com/30zhc6h.png>

- **B5 :** Trên instance, thực hiện ping lại đến gateway :
    ```
    # ping 192.168.1.1
    ```
    <img src=https://i.imgur.com/rjYhxIS.png>

    > Traffic đã hoàn toàn bị chặn bởi **port security** 
- **B6 :** Tắt port security trên port :
    ```
    # openstack port set 35147cdc-1a22-4b7b-b136-7d3e8bead504 --disable-port-security
    ```
    - Ngay lập tức, các traffic trở lại bình thường trên instance :

        <img src=https://i.imgur.com/Fxb4wI5.png>

> Lưu ý :
- **Port security** được thực hiện như trên là do rule trong **Iptable**. Các rule này được sắp xếp từ trước nhằm tránh việc bị tấn công **DHCP Snooping** hay giả mạo địa chỉ MAC trong mạng :
    - Kiểm tra **iptables** (thực hiện trên node chứa instance) :
        ```
        # iptables -L
        ```
        <img src=https://i.imgur.com/dJ8nKLu.png>
        
    - Xem chi tiết các rule bằng cách sử dụng :
        ```
        # iptables-save
        ```
        <img src=https://i.imgur.com/c6sbKyi.png>
    

--------------------
Tham khảo
- https://superuser.openstack.org/articles/managing-port-level-security-openstack/
- https://www.packetflow.co.uk/openstack-neutron-port-security-explained/#fn1
- http://kimizhang.com/neutron-ml2-port-security/