# GRE Tunnel
## **1) Giới thiệu**
## **2) Cấu hình Neutron sử dụng GRE Tunnel**
- **B1 :** Sao lưu file cấu hình `ml2` :
    ```
    # cp /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugins/ml2/ml2_conf.ini.bak
    ```
- **B2 :** Cấu hình ML2 driver :
    ```ini
    [ml2]
    ...
    type_drivers = gre
    tenant_network_types = gre
    mechanism_drivers = linuxbridge
    ```
    > `mechanism_drivers = openvswitch` nếu sử dụng **OpenvSwitch**
- **B3 :** Cấu hình dải tunnel ID cho GRE :
    ```ini
    [ml2_type_gre]
    ...
    tunnel_id_ranges = 1:1000
    ```
- **B4 :** Trong file `/etc/neutron/plugins/ml2/linuxbridge_agent.ini` (đối với **OvS** là `/etc/neutron/plugins/ml2/openvswitch_agent.ini`), cấu hình `local_ip` là địa chỉ IP dải data VM :
    ```ini
    [gre]
    ...
    local_ip = 10.10.240.10
    ```
    > Bước này thực hiện trên tất cả các node