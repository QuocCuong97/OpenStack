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
