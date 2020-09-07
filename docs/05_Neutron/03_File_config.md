# File cấu hình trong Neutron
## **1) File `/etc/neutron/neutron.conf`**
- Là file cấu hình chính của **Neutron**
- Section `[DEFAULT]` :
    ```ini
    [DEFAULT]
    ...
    auth_strategy = keystone
    core_plugin = ml2
    transport_url = rabbit://openstack:Password123@10.10.230.10
    ...
    ```
    - Trong đó :
        - `auth_strategy` : loại xác thực
        - `core_plugin` : plugin lõi
        - `transport_url` : thông tin kết nối **RabbitMQ**
- Section `[database]` :
    ```ini
    [database]
    ...
    connection = mysql+pymysql://neutron:Password123@10.10.230.10/neutron
    ...
    ```
    - Trong đó :
        - `connection` : thông tin kết nối database
- Section `[keystone_authtoken]` : khai báo thông tin kết nối đến **keystone** :
    ```ini
    [keystone_authtoken]
    www_authenticate_uri = http://10.10.230.10:5000
    auth_url = http://10.10.230.10:5000
    memcached_servers = 10.10.230.10:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = neutron
    password = Password123
    ```
- Section `[nova]` : khai báo thông tin kết nối đến **nova** :
    ```ini
    [nova]
    ...
    auth_url = http://10.10.230.10:5000
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    region_name = RegionOne
    project_name = service
    username = nova
    password = Password123
    ...
    ```
## **2) File `/etc/neutron/plugins/ml2/ml2_conf.ini`**
- Section `[ml2]` :
    ```ini
    [ml2]
    ...
    type_drivers = flat,vlan,vxlan
    tenant_network_types = vxlan
    mechanism_drivers = linuxbridge
    extension_drivers = port_security
    ...
    ```
    - Trong đó :
        - `type_drivers : local,flat,vlan,gre,vxlan,geneve` : các loại driver mạng được sử dụng
        - `tenant_network_types` : Danh sách theo thứ tự các kiểu mạng cho tenant network.
        - `mechanism_drivers` : cơ chế network sử dụng. Có thể là **OpenvSwitch** hoặc **LinuxBridge**
        - `extension_drivers` : các driver mở rộng thêm
- Section `[ml2_type_flat]` :
    ```ini
    [ml2_type_flat]
    ...
    flat_networks = provider
    ...
    ```
    - Trong đó :
        - `flat_networks` : Tên mạng vật lý dùng làm flat network
- Section `[ml2_type_vxlan]` :
    ```ini
    [ml2_type_vxlan]
    vni_ranges = 1:1000
    ```
- Section `[securitygroup]` :
    ```ini
    [securitygroup]
    enable_security_group = True
    firewall_driver = None
    enable_ipset = True
    ```
    - Trong đó :
        - `enable_security_group` : kích hoạt security group API
        - `firewall_driver` : driver cho security groups firewall trong l2 agent
        - `enable_ipset` : Tăng tốc các security groups
## **3) File `/etc/neutron/plugins/ml2/linuxbridge_agent.ini`**
- Section `[linux_bridge]` :
    ```ini
    [linux_bridge]
    ...
    physical_interface_mappings =
    ...
    ```
    - Trong đó :
        -` physical_interface_mappings = <physical_network>:<physical_interface>` : các mạng vật lý có ánh xạ với các interface
- Section `[vxlan]` :
    ```ini
    [vxlan]
    ...
    enable_vxlan = True
    local_ip = 10.10.230.10
    ...
    ```
    - Trong đó :
        - `enable_vxlan` : bật mode vxlan
        - `local_ip` : địa chỉ ip của overlay (tunnel) network endpoint
- Section `[securitygroup]` :
    ```ini
    [securitygroup]
    ...
    enable_security_group = True
    firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
    ...
    ```
    - Trong đó :
        - `enable_security_group` : kích hoạt security group API
        - `firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver` : driver cho security groups firewall trong l2 agent
## **4) File `/etc/neutron/metadata_agent.ini` (compute)**
- Section `[DEFAULT]` :
    ```ini
    [DEFAULT]
    nova_metadata_host = 10.10.230.10
    metadata_proxy_shared_secret = Password123
    ...
    ```
    - Trong đó :
        - `nova_metadata_host` : Địa chỉ IP hoặc DNS name của Nova metadata
        - `metadata_proxy_shared_secret` : Secret key để xác minh. Nó cần khớp với config key password của Nova trong section `[neutron]` của Nova
## **5) File `/etc/neutron/dhcp_agent.ini` (compute)**
- Section `[DEFAULT]` :
    ```ini
    [DEFAULT]
    interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
    enable_isolated_metadata = True
    dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
    force_metadata = True
    ```
    - Trong đó :
        - `interface_driver` : Driver được sử dụng để quản lý virtual interface
        - `enable_isolated_metadata` : Máy chủ DHCP có thể hỗ trợ cung cấp metadata cho các mạng isolated. Tùy chọn này không có tác dụng nào khi `force_metadata` được đặt giá trị là `True`
        - `dhcp_driver` : Driver sử dụng để quản lý DHCP server
        - `force_metadata` : Đặt giá trị này sẽ buộc DHCP server phải kết nối tới các host routes cụ thể vào DHCP request. Nếu tùy chọn này được đặt, thì metadata service sẽ được kích hoạt cho tất cả các mạng.
------------------------
Tham khảo
- https://docs.openstack.org/neutron/train/configuration/config.html
- https://docs.openstack.org/neutron/train/configuration/ml2-conf.html
- https://docs.openstack.org/neutron/train/configuration/linuxbridge-agent.html