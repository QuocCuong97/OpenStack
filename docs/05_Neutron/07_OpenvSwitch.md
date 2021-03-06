# OpenvSwitch
## **1) Giới thiệu về SDN và Openflow**
### **1.1) SDN - Sofware Define Networking**
- **SDN** hay ***mạng điều khiển bằng phần mềm*** (**Software Defined Networking**) được dựa trên cơ chế tách riêng việc kiểm soát một luồng mạng với luồng dữ liệu (**control plane** và **data plane**). 
- **SDN** dựa trên giao thức luồng mở (**OpenFlow**) và là kết quả nghiên cứu của **Đại học Stanford** và **California Berkeley**. SDN tách định tuyến và chuyển các luồng dữ liệu riêng rẽ và chuyển kiểm soát luồng sang thành phần mạng riêng có tên gọi là thiết bị kiểm soát luồng (**Flow Controller**). Điều này cho phép luồng các gói dữ liệu đi qua mạng được kiểm soát theo lập trình. - Trong **SDN**, **control plane** được tách ra từ các thiết bị vật lý và chuyển đến các bộ điều khiển. Bộ điều khiển này có thể nhìn thấy toàn bộ mạng lưới, và do đó cho phép các kỹ sư mạng làm cho chính sách chuyển tiếp tối ưu dựa trên toàn bộ mạng. Các bộ điều khiển tương tác với các thiết bị mạng vật lý thông qua một giao thức chuẩn **OpenFlow**. 
- Kiến trúc của **SDN** gồm 3 lớp riêng biệt: lớp ứng dụng, lớp điều khiển, và lớp cơ sở hạ tầng (lớp chuyển tiếp) :

    <p align=center><img src=https://i.imgur.com/JDGdNvn.png width=70%></p>

    - **Lớp ứng dụng:** Là các ứng dụng kinh doanh được triển khai trên mạng, được kết nối tới lớp điều khiển thông qua các API, cung cấp khả năng cho phép lớp ứng dụng lập trình lại (cấu hình lại) mạng (điều chỉnh các tham số trễ, băng thông, định tuyến, …) thông qua lớp điều khiển.
    - **Lớp điều khiển:** Là nơi tập trung các bộ điều khiển thực hiện việc điều khiển cấu hình mạng theo các yêu cầu từ lớp ứng dụng và khả năng của mạng. Các bộ điều khiển này có thể là các phần mềm được lập trình.
    - **Lớp cơ sở hạ tầng:** Là các thiết bị mạng thực tế (vật lý hay ảo hóa) thực hiện việc chuyển tiếp gói tin theo sự điều khiển của lớp điểu khiển. Một thiết bị mạng có thể hoạt động theo sự điều khiển của nhiều bộ điều khiển khác nhau, điều này giúp tăng cường khả năng ảo hóa của mạng.
### **1.2) OpenFlow**
- **OpenFlow** là tiêu chuẩn đầu tiên, cung cấp khả năng truyền thông giữa các giao diện của lớp điều khiển và lớp chuyển tiếp trong kiến trúc **SDN**. **OpenFlow** cho phép truy cập trực tiếp và điều khiển mặt phẳng chuyển tiếp của các thiết bị mạng như switch và router, cả thiết bị vật lý và thiết bị ảo, do đó giúp di chuyển phần điều khiển mạng ra khỏi các thiết bị chuyển mạch thực tế tới phần mềm điều khiển trung tâm. Các quyết định về các luồng traffic sẽ được quyết định tập trung tại OpenFlow Controller giúp đơn giản trong việc quản trị cấu hình trong toàn hệ thống. Một thiết bị OpenFlow bao gồm ít nhất 3 thành phần:
    - Secure Channel: kênh kết nối thiết bị tới bộ điều khiển (controller), cho phép các lệnh và các gói tin được gửi giữa bộ điều khiển và thiết bị.
    - OpenFlow Protocol: giao thức cung cấp phương thức tiêu chuẩn và mở cho một bộ điều khiển truyền thông với thiết bị.
    - Flow Table: một liên kết hành động với mỗi luồng, giúp thiết bị xử lý các luồng thế nào.

    <p align=center><img src=https://i.imgur.com/olqoDcw.png width=70%></p>

- Một số lợi ích khi sử dụng OpenFlow:
    - Công nghệ SDN trên cơ sở OpenFlow cho phép nhân viên IT giải quyết các ứng dụng băng thông cao và biến đổi động hiện nay, khiến cho mạng thích ứng với các nhu cầu kinh doanh thay đổi, và làm giảm đáng kể các hoạt động và quản lý phức tạp. Những lợi ishc mà các doanh nghiệp cà nhà khai thác mạng có thể đạt được thông qua kiến trúc SDN trên cơ sở OpenFlow bao gồm:
    - Tập trung hóa điều khiển trong môi trường nhiều nhà cung cấp thiết bị: phần mềm điều khiển SDN có thể điều khiển bất kỳ thiết bị mạng nào cho phép OpenFlow từ bất kỳ nhà cung cấp thiết bị nào, bao gồm switch, router, và các switch ảo.
    - Giảm sự phức tạp thông qua việc tự động hóa: kiến trúc SDN trên cơ sở OpenFlow cung cấp một framework quản lý mạng tự động và linh hoạt. Từ framework này có thể phát triển các công cụ tự động hóa các nhiệm vụ hiện đang được thực hiện bằng tay.
    - Tốc độ đổi mới cao hơn: việc áp dụng OpenFlow cho phép các nhà khai thác mạng lập trình lại mạng trong thời gian thực để đạt được các nhu cầu kinh doanh và yêu cầu người dùng cụ thể khi có sự thay đổi.
    - Gia tăng độ tin cậy và khả năng an ninh của mạng: các nhân viên IT có thể định nghĩa các trạng thái cấu hình và chính sách ở mức cao, và áp dụng tới cơ sở hạ tầng thông qua OpenFlow. Kiến trúc SDN trên cơ sở OpenFlow cung cấp điều khiển và tầm nhìn hoàn chỉnh trên mạng, nên có thể đảm bảo điều khiển truy nhập, định hình lưu lượng, QoS, an ninh, và các chính sách khác được thực thi nhất quán trên toàn bộ cơ sở hạ tầng mạng không dây và có dây, bao gồm cả các văn phòng chi nhánh, các cơ sở chính và DC.
    - Điều khiển mạng chi tiết hơn: mô hình điều khiển trên cơ sở flow của OpenFlow cho phép nhân viên IT áp dụng các chính sách tại mức chi tiết, bao gồm phiên, người dùng, thiết bị, và các mức ứng dụng, trong một sự trừu tượng hóa cao, tự động điều chỉnh thích hợp.
    - Tốt hơn với trải nghiệm người dùng: bằng việc tập trung hóa điều khiển mạng và tạo ra trạng thái thông tin có sẵn cho các ứng dụng mức cao hơn, kiến trúc SDN trên cơ sở OpenFlow có thể đáp ứng tốt hơn cho các nhu cầu thay đổi của người dùng
## **2) OpenvSwitch**
### **2.1) Giới thiệu**
- **OpenvSwitch (OVS)** là một dự án về chuyển mạch ảo đa lớp (multilayer). Mục đích chính của OpenvSwitch là cung cấp lớp chuyển mạch cho môi trường ảo hóa phần cứng, trong khi hỗ trợ nhiều giao thức và tiêu chuẩn được sử dụng trong hệ thống chuyển mạch thông thường. OpenvSwitch hỗ trợ nhiều công nghệ ảo hóa dựa trên nền tảng Linux như Xen/XenServer, KVM, và VirtualBox.

    <p align=center><img src=https://i.imgur.com/eyqJNtq.png width=70%></p>

- OpenvSwitch hỗ trợ các tính năng sau: 
    - VLAN tagging & 802.1q trunking 
    - Standard Spanning Tree Protocol (802.1D) 
    - LACP 
    - Port Mirroring (SPAN/RSPAN) 
    - Tunneling Protocols 
    - QoS
- Các thành phần chính của OpenvSwitch:
    - `ovs-vswitchd`: thực hiện chuyển đổi các luồng chuyển mạch.
    - `ovsdb-server`: là một lightweight database server, cho phép ovs-vswitchd thực hiện các truy vấn đến cấu hình.
    - `ovs-dpctl`: công cụ để cấu hình các switch kernel module.
    - `ovs-vsctl`: tiện ích để truy vấn và cập nhật cấu hình ovs-vswitchd.
    - `ovs-appctl`: tiện ích gửi command để chạy OpenvSwitch.
### **2.2) So sánh LinuxBridge và OpenvSwitch**
- **OpenvSwitch**
    - Ưu điểm :
        - Dễ quản lý mạng hơn : với **OpenvSwitch**, sẽ thuận tiện hơn cho người quản trị quản lý và giám sát trạng thái mạng và luồng dữ liệu trong môi trường cloud .
        - Hỗ trợ nhiều giao thức tunnel : **OVS** hỗ trợ **GRE**, **VXLAN**, **IPSec**,...
        - Được tích hợp trong **SDN** : **OVS** được tích hợp trong **SDN** vì vật nó có thể được cài bằng cách sử dụng một **OpenStack plug-in** hoặc trực tiếp từ **SDN controller**, như **OpenDaylight**
    - Nhược điểm :
        - Thiếu ổn định : **OpenvSwitch** có một số vấn đề về tính ổn định như Kernetl panics, ovs-switched segfaults, and data corruption.
        - Vận hành phức tạp : **OpenvSwitch** là một giải pháp phức tạp, nhiều chức năng, khó học, cài đặt và vận hành
- **LinuxBridge**
    - Ưu điểm :
        - Ổn định và tin cậy : **LinuxBridge** đã được sử dụng nhiều năm, sự ổn định và tin tưởng đã được chứng minh .
        - Dễ cài đặt : **LinuxBridge** là một phần của việc cài đặt **Linux** và không cần cài thêm gói hỗ trợ nào khác .
        - Thuận tiện cho việc troubleshoot : **LinuxBridge** là một giải pháp đơn giản và dễ vận hành hơn **OpenvSwitch**
    - Nhược điểm :
        - Ít chức năng : **LinuxBridge** không hỗ trợ **Neutron DVR**,VXLan và nhiều chức năng khác .
        - Ít được support : **LinuxBridge** sẽ ít cộng đồng hỗ trợ hơn **OpenvSwitch**
### **2.3) Các lệnh thường dùng trong OpenvSwitch**
- `ovs-` : Chỉ cần nhập vào `ovs` rồi ấn tab 2 lần là có thể xem tất cả các câu lệnh đối với OpenvSwitch :

    <img src=https://i.imgur.com/Aa65th0.png>

- `ovs-vsctl` : là câu lệnh để cài đặt và thay đổi một số cấu hình ovs. Nó cung cấp interface cho phép người dùng tương tác với Database để truy vấn và thay đổi dữ liệu :
    - `ovs-vsctl show` : Hiển thị cấu hình hiện tại của switch.
    - `ovs-vsctl list-br`: Hiển thị tên của tất cả các bridges.
    - `ovs-vsctl list-ports` : Hiển thị tên của tất cả các port trên bridge.
    - `ovs-vsctl list interface` : Hiển thị tên của tất cả các interface trên bridge.
    - `ovs-vsctl add-br` : Tạo bridge mới trong database.
    - `ovs-vsctl add-port` : Gán interface (card ảo hoặc card vật lý) vào Open vSwitch bridge.
- `ovs-ofctl` và `ovs-dpctl` : Dùng để quản lí và kiểm soát các flow entries. OVS quản lý 2 loại flow:
    - OpenFlows : flow quản lí control plane
    - Datapath : là kernel flow.
    - `ovs-ofctl` giao tiếp với OpenFlow module, ovs-dpctl giao tiếp với Kernel module.
- `ovs-ofctl show` : hiển thị thông tin ngắn gọn về switch bao gồm port number và port mapping.
- `ovs-ofctl dump-flows` : Dữ liệu trong OpenFlow tables
- `ovs-dpctl show` : Thông tin cơ bản về logical datapaths (các bridges) trên switch.
- `ovs-dpctl dump-flows` : Hiển thị flow cached trong datapath.
- `ovs-appctl bridge/dumpflows` : thông tin trong flow tables và offers kết nối trực tiếp cho VMs trên cùng hosts.
- `ovs-appctl fdb/show` : Hiển thị các cặp mac/vlan trên bridge.
### **2.4) Nguyên lý hoạt động trong OpenvSwitch**
<p align=center><img src=https://i.imgur.com/CLnrwZU.png></p>

- Theo mô hình trên, NIC của VM kết nối đến tap interface (`vnet`) của LinuxBridge . Lý do duy nhất mà **linux bridge** còn được sử dụng là vì các iptables rule bên trên các **bridge** này được sử dụng để  thực thi security group rule cho VM. **Linux bridge** được kết nối thông qua veth pairt tới **integration bridge** `br-int` của **OvS**. . **Integration bridge** `br-int` sẽ gắn VLAN tag vào các gói tin tới từ các VM. VLAN tag này là riêng biệt trên từng network trên mỗi compute node. **Integration bridge** được kết nối đến bridge `br-tun` trong trường hợp sử dụng các giao thức tunnel như **VXLAN** hay **GRE** .
- Ta có thể dễ dàng kiểm tra được sự xuất hiện của các **tap interface** trong các compute node bằng lệnh `ip a` hoặc `ifconfig`, nhưng kết quả trả về sẽ hoàn toàn không có các **linux bridge** . Đó là vì **ML2/ODL** không dựa vào Iptable để theo dõi các trạng thái vì vậy thậm chí nó có thể không cần đến **linux bridge** . Thậm chí, nếu sử dụng `firewall_driver` là `openvswitch` thay cho `iptables_hybrid` mặc định, sẽ chả có **linux bridge** nào được tạo ra . Theo các đánh giá hiệu năng, `openvswitch firewall_driver` cho thấy hiệu quả tốt hơn .
- Để thay đổi cài đặt này, chỉnh sửa trong file `/etc/neutron/plugins/ml2/openvswitch_agent.ini` :
    ```ini
    [securitygroup]
    ...
    firewall_driver = openvswitch
    ...
    ```
### **Cách kiểm tra tap interface nào gắn vào VM nào**
- Trước hết kiểm tra xem VM đang ở trên node nào :
    ```
    # openstack server list --long
    ```
    <img src=https://i.imgur.com/1YE7oCe.png>

    > Các VM đang được đặt ở `compute1`

- Trên node compute1 kiểm tra các tap interface đang có :
    ```
    # ovs-vsctl list-ports br-int
    ```
    <img src=https://i.imgur.com/jshjXuZ.png>

- Kiểm tra lại các port đang có trên các VM :
    ```
    # openstack port list
    ```
    <img src=https://i.imgur.com/3Le6wVj.png>

    - Tại đây ta có thể thấy rõ ID của các port được sử dụng để đặt tên cho **tap interface** kết nối với nó :
        - Port ID : `1f7f9831-3183-4c17-ada8-3d687b2437a3` <=> Interface : `tap1f7f9831-31`
        - Port ID : `6cf796e9-f3af-43f2-b92c-217bf040b90e` <=> Interface : `tap6cf796e9-f3`
        - Port ID : `78ed0e52-2374-441d-85ba-84ff9fb90a7d` <=> Interface : `tap78ed0e52-23`
- Việc cuối cùng ta chỉ cần xem port ID trên gắn trên VM nào bằng lệnh :
    ```
    # openstack port show <port_ID>
    ```
    <img src=https://i.imgur.com/oSl8fNg.png>
### **2.5) Triển khai OpenvSwitch**
#### **2.5.1) Trên node controller**
- **B1 :** Tạo database cho `Neutron` :
    ```
    # mysql -u root -pPassword123
    > CREATE DATABASE neutron;
    > GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'Password123';
    > GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'Password123';
    > FLUSH PRIVILEGES;
    > exit
    ```
- **B2 :** Tạo project, user, endpoint cho `Neutron` :
    ```
    # source /root/admin-openrc
    # openstack user create neutron --domain default --password Password123
    # openstack role add --project service --user neutron admin
    # openstack service create --name neutron --description "OpenStack Networking" network
    # openstack endpoint create --region RegionOne network public http://controller:9696
    # openstack endpoint create --region RegionOne network internal http://controller:9696
    # openstack endpoint create --region RegionOne network admin http://controller:9696
    ```
- **B3 :** Cài đặt **Neutron** và **OpenvSwitch** :
    ```
    # yum install -y openstack-neutron openstack-neutron-ml2 openstack-neutron-openvswitch ebtables
    ```
- **B4 :** Sao lưu file cấu hình của **OpenvSwitch** :
    ```
    # cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.bak
    # cp /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugins/ml2/ml2_conf.ini.bak
    # cp /etc/neutron/dhcp_agent.ini /etc/neutron/dhcp_agent.ini.bak
    # cp /etc/neutron/metadata_agent.ini /etc/neutron/metadata_agent.ini.bak
    # cp /etc/neutron/plugins/ml2/openvswitch_agent.ini /etc/neutron/plugins/ml2/openvswitch_agent.ini.bak
    ```
- **B5 :** Cấu hình file `/etc/nova/nova.conf` :
    ```
    # crudini --set /etc/nova/nova.conf DEFAULT use_neutron true
    # crudini --set /etc/nova/nova.conf DEFAULT firewall_driver nova.virt.firewall.NoopFirewallDriver
    # crudini --set /etc/nova/nova.conf neutron url http://controller:9696
    # crudini --set /etc/nova/nova.conf neutron auth_url http://controller:5000
    # crudini --set /etc/nova/nova.conf neutron region_name RegionOne
    # crudini --set /etc/nova/nova.conf neutron auth_type password
    # crudini --set /etc/nova/nova.conf neutron project_domain_name default
    # crudini --set /etc/nova/nova.conf neutron user_domain_name default
    # crudini --set /etc/nova/nova.conf neutron project_name service
    # crudini --set /etc/nova/nova.conf neutron username neutron
    # crudini --set /etc/nova/nova.conf neutron password Password123
    # crudini --set /etc/nova/nova.conf neutron service_metadata_proxy True
    # crudini --set /etc/nova/nova.conf neutron metadata_proxy_shared_secret Password123
    ```
- **B6 :** Chỉnh sửa file cấu hình `neutron.conf` :
    ```
    # crudini --set /etc/neutron/neutron.conf DEFAULT core_plugin ml2
    # crudini --set /etc/neutron/neutron.conf DEFAULT service_plugins router
    # crudini --set /etc/neutron/neutron.conf DEFAULT allow_overlapping_ips True
    # crudini --set /etc/neutron/neutron.conf DEFAULT transport_url rabbit://openstack:Password123@controller
    # crudini --set /etc/neutron/neutron.conf DEFAULT auth_strategy keystone
    # crudini --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_status_changes True
    # crudini --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_data_changes True
    # crudini --set /etc/neutron/neutron.conf DEFAULT dhcp_agents_per_network 2
    # crudini --set /etc/neutron/neutron.conf database connection mysql+pymysql://neutron:Password123@controller/neutron
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken www_authenticate_uri http://controller:5000
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken auth_url http://controller:5000
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken memcached_servers controller:11211
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken auth_type password
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken project_domain_name default
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken user_domain_name default
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken project_name service
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken username neutron
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken password Password123
    # crudini --set /etc/neutron/neutron.conf nova auth_url http://controller:5000
    # crudini --set /etc/neutron/neutron.conf nova auth_type password
    # crudini --set /etc/neutron/neutron.conf nova project_domain_name default
    # crudini --set /etc/neutron/neutron.conf nova user_domain_name default
    # crudini --set /etc/neutron/neutron.conf nova region_name RegionOne
    # crudini --set /etc/neutron/neutron.conf nova project_name service
    # crudini --set /etc/neutron/neutron.conf nova username nova
    # crudini --set /etc/neutron/neutron.conf nova password Password123
    # crudini --set /etc/neutron/neutron.conf oslo_concurrency lock_path /var/lib/neutron/tmp
    ```
- **B7 :** Sửa file cấu hình `/etc/neutron/plugins/ml2/ml2_conf.ini` :
    ```
    # crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 type_drivers flat,vlan,vxlan
    # crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 tenant_network_types vxlan
    # crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 mechanism_drivers openvswitch,l2population
    # crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 extension_drivers port_security
    # crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2_type_flat flat_networks provider
    # crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2_type_vxlan vni_ranges 1:1000
    # crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini securitygroup enable_ipset True
    ```
- **B8 :** Khai báo `sysctl` :
    ```
    # echo 'net.bridge.bridge-nf-call-iptables = 1' >> /etc/sysctl.conf
    # echo 'net.bridge.bridge-nf-call-ip6tables = 1' >> /etc/sysctl.conf
    # modprobe br_netfilter
    # /sbin/sysctl -p
    ```
- **B9 :** Tạo liên kết file :
    ```
    # ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
    ```
- **B10 :** Khởi động dịch vụ **OvS** :
    ```
    # systemctl enable openvswitch
    # systemctl start openvswitch
    ```
- **B11 :** Tạo script tạo bridge `br-provider` :
    ```
    # vi ovs.sh
    ```
    - Thêm vào nội dung sau :
        ```bash
        #!/bin/bash
        ovs-vsctl add-br br-provider
        ovs-vsctl add-port br-provider eth0
        ip a flush eth0
        ip a add 10.5.11.210/22 dev br-provider
        ip link set br-provider up
        ip r add default via 10.5.8.1
        ```
        > Trong đó `eth0` là interface provider, `10.5.11.210/22` là IP Provider
    - Phân quyền và chạy script :
        ```
        # chmod +x ovs.sh
        # ./ovs.sh
        ```
- **B12 :** Sửa file cấu hình `openvswitch_agent.ini` :
    ```
    # crudini --set /etc/neutron/plugins/ml2/openvswitch_agent.ini ovs bridge_mappings provider:br-provider
    # crudini --set /etc/neutron/plugins/ml2/openvswitch_agent.ini ovs local_ip $(ip addr show dev eth2 scope global | grep "inet " | sed -e 's#.*inet ##g' -e 's#/.*##g')
    # crudini --set /etc/neutron/plugins/ml2/openvswitch_agent.ini securitygroup firewall_driver iptables_hybrid
    # crudini --set /etc/neutron/plugins/ml2/openvswitch_agent.ini agent tunnel_types vxlan
    # crudini --set /etc/neutron/plugins/ml2/openvswitch_agent.ini agent l2_population True
    ```
- **B13 :** Sửa file cấu hình `l3_agent.ini` :
    ```
    # crudini --set /etc/neutron/l3_agent.ini DEFAULT interface_driver openvswitch
    ```
- **B14 :** Thiết lập database :
    ```
    # su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
    --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
    ```
- **B15 :** Khởi động dịch vụ `neutron` :
    ```
    # systemctl enable neutron-server neutron-openvswitch-agent neutron-metadata-agent neutron-l3-agent
    # systemctl start neutron-server neutron-openvswitch-agent neutron-metadata-agent neutron-l3-agent
    ```
- **B16 :** Kiểm tra lại trạng thái dịch vụ :
    ```
    # openstack network agent list
    ```
    <img src=https://i.imgur.com/Izb0LJu.png>
### **2.5.2) Trên node compute**
- **B1 :** Khai báo bổ sung cho `Nova` :
    ```
    # crudini --set /etc/nova/nova.conf DEFAULT use_neutron true
    # crudini --set /etc/nova/nova.conf DEFAULT firewall_driver nova.virt.firewall.NoopFirewallDriver
    # crudini --set /etc/nova/nova.conf neutron url http://controller:9696
    # crudini --set /etc/nova/nova.conf neutron auth_url http://controller:5000
    # crudini --set /etc/nova/nova.conf neutron auth_type password
    # crudini --set /etc/nova/nova.conf neutron project_domain_name default
    # crudini --set /etc/nova/nova.conf neutron user_domain_name default
    # crudini --set /etc/nova/nova.conf neutron project_name service
    # crudini --set /etc/nova/nova.conf neutron username neutron
    # crudini --set /etc/nova/nova.conf neutron password Password123
    ```
- **B2 :** Cài đặt **Neutron** và **OpenvSwitch** :
    ```
    # yum install -y openstack-neutron openstack-neutron-ml2 openstack-neutron-openvswitch ebtables ipset
    ```
- **B3 :** Sao lưu file cấu hình của **OpenvSwitch** :
    ```
    # cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.bak
    # cp /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugins/ml2/ml2_conf.ini.bak
    # cp /etc/neutron/dhcp_agent.ini /etc/neutron/dhcp_agent.ini.bak
    # cp /etc/neutron/metadata_agent.ini /etc/neutron/metadata_agent.ini.bak
    # cp /etc/neutron/plugins/ml2/openvswitch_agent.ini /etc/neutron/plugins/ml2/openvswitch_agent.ini.bak
    ```
- **B4 :** Chỉnh sửa file cấu hình `/etc/neutron/neutron.conf` :
    ```
    # crudini --set /etc/neutron/neutron.conf DEFAULT auth_strategy keystone
    # crudini --set /etc/neutron/neutron.conf DEFAULT core_plugin ml2
    # crudini --set /etc/neutron/neutron.conf DEFAULT transport_url rabbit://openstack:Password123@controller
    # crudini --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_status_changes true
    # crudini --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_data_changes true
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken www_authenticate_uri http://controller:5000
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken auth_url http://controller:5000
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken memcached_servers controller:11211
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken auth_type password
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken project_domain_name default
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken user_domain_name default
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken project_name service
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken username neutron
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken password Password123
    # crudini --set /etc/neutron/neutron.conf oslo_concurrency lock_path /var/lib/neutron/tmp
    ```
- **B5 :** Khai báo `sysctl` :
    ```
    # echo 'net.bridge.bridge-nf-call-iptables = 1' >> /etc/sysctl.conf
    # echo 'net.bridge.bridge-nf-call-ip6tables = 1' >> /etc/sysctl.conf
    # modprobe br_netfilter
    # /sbin/sysctl -p
    ```
- **B6 :** Sửa file cấu hình `/etc/neutron/plugins/ml2/openvswitch_agent.ini` :
    ```
    # crudini --set /etc/neutron/plugins/ml2/openvswitch_agent.ini ovs local_ip $(ip addr show dev eth2 scope global | grep "inet " | sed -e 's#.*inet ##g' -e 's#/.*##g')
    # crudini --set /etc/neutron/plugins/ml2/openvswitch_agent.ini agent tunnel_types vxlan
    # crudini --set /etc/neutron/plugins/ml2/openvswitch_agent.ini agent l2_population True
    ```
- **B7 :** Khai báo cho file `/etc/neutron/dhcp_agent.ini` :
    ```
    # crudini --set /etc/neutron/dhcp_agent.ini DEFAULT interface_driver openvswitch
    # crudini --set /etc/neutron/dhcp_agent.ini DEFAULT enable_isolated_metadata True
    # crudini --set /etc/neutron/dhcp_agent.ini DEFAULT dhcp_driver neutron.agent.linux.dhcp.Dnsmasq
    # crudini --set /etc/neutron/dhcp_agent.ini DEFAULT force_metadata True
    ```
- **B8 :** Khai báo trong file `/etc/neutron/metadata_agent.ini` :
    ```
    # crudini --set /etc/neutron/metadata_agent.ini DEFAULT nova_metadata_host controller
    # crudini --set /etc/neutron/metadata_agent.ini DEFAULT metadata_proxy_shared_secret Password123
    ```
- **B9 :** Khởi động dịch vụ **OvS** :
    ```
    # systemctl enable openvswitch
    # systemctl start openvswitch
    ```
- **B10 :** Tạo script tạo bridge `br-provider` :
    ```
    # vi ovs.sh
    ```
    - Thêm vào nội dung sau :
        ```bash
        #!/bin/bash
        ovs-vsctl add-br br-provider
        ovs-vsctl add-port br-provider eth0
        ip a flush eth0
        ip a add 10.5.11.201/22 dev br-provider
        ip link set br-provider up
        ip r add default via 10.5.8.1
        ```
        > Trong đó `eth0` là interface provider, `10.5.11.201/22` là IP Provider của `compute1`

        > Làm tương tự và thay IP với `compute2`
    - Phân quyền và chạy script :
        ```
        # chmod +x ovs.sh
        # ./ovs.sh
        ```
- **B11 :** Khởi động dịch vụ `Neutron` :
    ```
    # systemctl enable neutron-openvswitch-agent neutron-metadata-agent neutron-dhcp-agent
    # systemctl start neutron-openvswitch-agent neutron-metadata-agent neutron-dhcp-agent
    ```
- **B12 :** Khởi động lại dịch vụ `nova-compute` :
    ```
    # systemctl restart openstack-nova-compute
    ```
-----------------------------------------
Tham khảo
- https://docs.openstack.org/neutron/train/admin/deploy-ovs-provider.html
- https://docs.openstack.org/neutron/train/admin/deploy-ovs-selfservice.html
- https://thesaitech.wordpress.com/2017/09/24/how-to-trace-the-tap-interfaces-and-linux-bridges-on-the-hypervisor-your-openstack-vm-is-on/
