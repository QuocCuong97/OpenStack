# GRE Tunnel
## **1) Giới thiệu**
- **GRE - *Generic Routing Encapsulation*** là giao thức  cho phép đóng gói dữ liệu của nhiều loại giao thức khác nhau như **IP** , **IPX** , **AppleTalk** , các giao thức định tuyến ,... để truyền tải qua 1 mạng IP. 
- Khi thực hiện đóng gói , mặc định GRE thêm vào 1 overhead 24 byte gồm 20 byte IP header và 4 byte GRE header .
    <img src=https://i.imgur.com/LN8w7Kb.png>

- **GRE** là giao thức sử dụng kết nối point-to-point .
- Phần header của packet **GRE** chứa `32bit` tunnel-ID được sử dụng để xác định các luồng tunnel riêng biệt trong mạng .
- **GRE** không thực hiện bất kỳ cơ chế bảo mật nào cho dữ liệu được đóng gói ( không mã hóa , không xác thực ,... ) .
## **2) Cấu hình Neutron sử dụng GRE Tunnel**
- **B1 :** Sao lưu file cấu hình `ml2` :
    ```
    # cp /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugins/ml2/ml2_conf.ini.bak
    ```
- **B2 :** Cấu hình ML2 driver trong file `/etc/neutron/plugins/ml2/ml2_conf.ini` :
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
- **VD :**
    - Tạo mạng GRE tunnel :
        ```
        # openstack network create --project-domain admin --provider-network-type gre private2
        ```
        <img src=https://i.imgur.com/fRxl9Hv.png>