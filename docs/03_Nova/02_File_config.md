# File cấu hình của Nova
- File cấu hình chính : `/etc/nova/nova.conf`

    <img src=https://i.imgur.com/LJKDPmP.png>

- Các phần cần chú ý trong file :
    - Cấu hình management IP :
        ```ini
        [DEFAULT]
        ...
        my_ip = 10.10.230.10
        ...
        ```
    - Cấu hình hỗ trợ `neutron` :
        ```ini
        [DEFAULT]
        ...
        use_neutron = true
        firewall_driver = nova.virt.firewall.NoopFirewallDriver
        ...
        ```
    - Cấu hình identity service :
        ```ini
        [api]
        ...
        auth_strategy = keystone
        ...
        [keystone_authtoken]
        ...
        www_authenticate_uri = http://controller:5000/    # public identity endpoint
        auth_url = http://controller:5000/                # admin identity endpoint
        memcached_servers = controller:11211
        auth_type = password
        project_domain_name = Default
        user_domain_name = Default
        project_name = service
        username = nova
        password = Password123
        ...
    - Cấu hình database :
        ```ini
        [api_database]
        ...
        connection = mysql+pymysql://nova:Password123@controller/nova_api
        ...
        [database]
        ...
        connection = mysql+pymysql://nova:Password123@controller/nova
        ...
        ```
    - Cấu hình **RabbitMQ** :
        ```ini
        [DEFAULT]
        ...
        transport_url = rabbit://openstack:Password123@controller
        ...
        ```
    - Cấu hình **VNC proxy** :
        ```ini
        [vnc]
        ...
        enabled = true
        server_listen = $my_ip
        server_proxyclient_address = $my_ip
        ...
        ```
    - Cấu hình Image Service :
        ```ini
        [glance]
        ...
        api_servers = http://controller:9292
        ...
        ```
----------------
[Tham khảo thêm các option cấu hình khác](https://docs.openstack.org/nova/train/configuration/config.html)