# QoS - Quality of Service
## **1) Giới thiệu**
- **QoS** được định nghĩa là *khả năng đảm bảo các yêu cầu mạng nhất định như **bandwidth**, **latency** (độ trễ), **jitter** (độ giật) và **reliability** (độ tin cậy) để đáp ứng thỏa thuận về **Service Level Agreement (SLA)** giữa các nhà cũng cấp dịch vụ và end users.*
- **VD :** Cần set bandwidth cho từng loại traffic (những traffic như VoIP, streaming,... cần được ưu tiên hơn)
- **QoS** policy có thể được áp dụng:
    - **Theo từng network**: Tất cả các ports được gán vào network nơi có QoS policies sẽ được áp dụng.
    - **Theo từng port**: Các port cụ thể sẽ được áp dụng các policy, khi port đã có policy rồi thì nó sẽ bị ghi đè.
- Trong **OpenStack**, **QoS** là một service plug-in mở rộng, được khai báo trong code của **neutron** và cung cấp thông qua `ml2` extension driver.
## **2) Các rule QoS hỗ trợ trong Neutron**
- Trong **Neutron** hiện đang hỗ trợ các rule **QoS** sau:
    - `banwidth_limit`: hỗ trợ giới hạn băng thông tối đa trên từng network, port và IP floating
    - `dscp_marking`: hỗ trợ giới hạn băng thông dựa trên DSCP value. - Với QoS. Marking là 1 task nhỏ trong Classtifycation, (và tất nhiên marking lúc này là DSCP cho Difserv). Classtifycation có 2 task là identify gói tin và marking gói tin, sau đó đẩy vào các queuing, dùng scheduling để quyết định gói nào ra trước, gói nào phải chờ.
    - `minimum_bandwidth`: giới hạn băng thông tối đa dựa lên kiểu kết nối.
## **3) Cấu hình QoS**
### **3.1) Cấu hình trên node `controller`**
- **B1 :** Chỉnh sửa file cấu hình `/etc/neutron/neutron.conf`, thêm plugin `qos` :
    ```ini
    [DEFAULT]
    ...
    service_plugins = router, qos
    ...
    ```
- **B2 :** Khai báo driver `qos` trong file `/etc/neutron/plugins/ml2/ml2_conf.ini` :
    ```ini
    [ml2]
    ...
    extension_drivers = port_security, qos
    ...
    ```
- **B3 :** Sửa file `/etc/neutron/plugins/ml2/<agent_name>_agent.ini`, tại đây là LinuxBridge (`/etc/neutron/plugins/ml2/linuxbridge_agent.ini`) :
    ```ini
    [agent]
    ...
    extensions = qos
    ...
    ```
    > Nếu là **OpenvSwitch** thì sẽ là file `/etc/neutron/plugins/ml2/openvswitch_agent.ini`
- **B4 :** Khởi động lại các dịch vụ :
    ```
    # systemctl restart neutron-server neutron-linuxbridge-agent neutron-dhcp-agent neutron-metadata-agent
    ```
### **3.2) Trên compute node**
- **B1 :** Sửa file `/etc/neutron/plugins/ml2/<agent_name>_agent.ini`, tại đây là LinuxBridge (`/etc/neutron/plugins/ml2/linuxbridge_agent.ini`) :
    ```ini
    [agent]
    ...
    extensions = qos
    ...
    ```
    > Nếu là **OpenvSwitch** thì sẽ là file `/etc/neutron/plugins/ml2/openvswitch_agent.ini`
- **B2 :** Khởi động lại các dịch vụ :
    ```
    # systemctl restart neutron-linuxbridge-agent neutron-metadata-agent neutron-dhcp-agent openstack-nova-compute
    ```
## **4) Các command thường dùng với QoS**
### **Tạo policy và rule**
- Tạo 1 **QoS** giới hạn băng thông :
    ```
    # openstack network qos policy create bw-limiter
    ```
    <img src=>
- Tạo các rule cả chiều ra (`egress`) và chiều vào (`ingress`) cho **QoS** :
    ```
    # openstack network qos rule create --type bandwidth-limit --max-kbps 3000 \
    --max-burst-kbits 2400 --egress bw-limiter
    # openstack network qos rule create --type bandwidth-limit --max-kbps 3000 \
    --max-burst-kbits 2400 --ingress bw-limiter
    ```
    <img src=https://i.imgur.com/FQ1sHdi.png>

    > **Chú ý :** **QoS** yêu cầu chỉ số `burst` để chắc chắn sự đúng đắn các các rule set bandwith trên các OpenvSwitch và Linux Bridge. Nếu không set trong quá trình đặt rule thì mặc định chỉ số này sẽ về 80% bandwidth của các gói TCP thông thường. Nếu giá trị burst quá thấp sẽ gây ra việc giảm băng thông so với thông số cấu hình
- Kiểm tra lại rule hiện có trong QoS policy vừa tạo:
    ```
    # openstack network qos rule list bw-limiter
    ```
    <img src=https://i.imgur.com/s1WYjFo.png>

- List các port đang có để lấy ID của port :
    ```
    # openstack port list
    ```
    <img src=https://i.imgur.com/7uCi2eh.png>

- Gán QoS policy vào port hoặc network cụ thể:
    ```
    # openstack port set --qos-policy bw-limiter d21bfe5d-4034-4119-aefb-96b8aadcc7d8
    ```
- Kiểm tra lại port lại xem đã được gán QoS rule chưa:
    ```
    # openstack port show d21bfe5d-4034-4119-aefb-96b8aadcc7d8
    ```
    <img src=https://i.imgur.com/gejz0Zs.png>

### **Detach port khỏi policy**
- Cú pháp :
    ```
    # openstack port unset --qos-policy <port_ID>
    ```
### **Sửa rule**



### **Xóa rule**
- Cú pháp :
    ```
    # openstack network qos rule delete <policy_name> <rule_ID>
    ```
## **5) Kiểm chứng**
- **B1 :** Tạo 2 instance :

    | VM02 | ubuntu20 | `192.168.1.196` |
    |------|----------|---------------|
    | **VM03** | **centos7** | **`192.168.1.124`** |

- **B2 :** Cài đặt package `iperf3` :
    ```
    # yum install iperf3 -y                    (centos)
    # sudo apt-get install iperf3 -y           (ubuntu)
    ```
- **B3 :** Kiểm tra bandwidth trước khi gán policy :
    - Trên **VM02**, sử dụng lệnh `iperf -s` để tạo iperf server :

        <img src=https://i.imgur.com/53560PN.png>

    - Trên **VM03**, sử dụng lệnh `iperf3 -c <IP_server>` để kết nối đến iperf server :

        <img src=https://i.imgur.com/uvLAusN.png>

    > Ta thấy tốc độ cả send và receive &asymp; `185Mbit`
- **B4 :** Thực hiện gán policy lên **VM03** :
    ```
    # openstack port set --qos-policy bw-limiter d21bfe5d-4034-4119-aefb-96b8aadcc7d8
    ```
- **B5 :** Kiểm tra lại kết nối giữa 2 VM qua `iperf` :
    - Trên **VM02** :

        <img src=https://i.imgur.com/vP757HN.png>

    - Trên **VM03** :

        <img src=https://i.imgur.com/xYNxPv2.png>

    > Ta thấy tốc độ cả send và receive &asymp; `3.5Mbit` (gần đúng với băng thông **QoS** : `3Mbit`)
    
    

------------
Tham khảo
- https://docs.openstack.org/neutron/latest/admin/config-qos.html