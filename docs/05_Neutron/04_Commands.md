# Các command thường sử dụng trong Neutron
## **1) Network**
#### **List các network**
- Cú pháp :
    ```
    # openstack network list
    ```
    <img src=https://i.imgur.com/5FSMxAu.png>
#### **List các subnet**
- Cú pháp :
    ```
    # openstack subnet list
    ```
    <img src=https://i.imgur.com/1O3OaM4.png>
#### **Tạo network mới**
- Tạo một network provider :
    ```
    # openstack network create \
    --share --external --project-domain <project_name> \
    --provider-physical-network <provider_net_name> \
    --provider-network-type <net_type> <net_name>
    ```
    - **VD :**
        ```
        # openstack network create \
        --share --external --project-domain admin \
        --provider-physical-network provider \
        --provider-network-type flat public
        ```
- Tạo một network self-service :
    ```
    # openstack network create --project-domain <project_name> <net_name>
    ```
    - **VD :**
        ```
        # openstack network create --project-domain admin private2
        ```
        <img src=https://i.imgur.com/m8rooDQ.png>

        - Lưu ý :
            - **port security** mặc định được bật (disable bằng option `--disable-port-security`)
            - network type mặc định là `vxlan`
            - private network sẽ ở mode `Internal` thay vì `External` như provider
            - Không dùng tùy chọn `share`
#### **Tạo subnet**
- Tại đây, tạo dải mạng, default gateway, DNS cho network
    > **Lưu ý:** đối với **external network** thì dải mạng khai báo phải trùng với dải provider để máy ảo ra ngoài.
- Cú pháp :
    ```
    # openstack subnet create --network <net_name> \
    --allocation-pool start=<start_IP>,end=<end_IP> \
    --dns-nameserver <DNS_server> --gateway <IP_gateway> \
    --subnet-range <range/subnet_mask> \
    <subnet_name>
    ```
- **VD :**
    ```
    # openstack subnet create --network private2 \
    --allocation-pool start=172.16.1.100,end=172.16.1.200 \
    --dns-nameserver 8.8.8.8 --gateway 172.16.0.1 \
    --subnet-range 172.16.1.0/24 \
    sub1private2
    ```
    <img src=https://i.imgur.com/Ogergkm.png>
#### **Show thông tin chi tiết một network**
- Cú pháp :
    ```
    # openstack network show <net_name>
    ```
- **VD :** 
    ```
    # openstack network show public
    ```
    <img src=https://i.imgur.com/pyw5QFi.png>

#### **Show thông tin chi tiết một subnet**
- Cú pháp :
    ```
    # openstack subnet show <subnet_name>
    ```
- **VD :**
    ```
    # openstack subnet show sub1private1
    ```
    <img src=https://i.imgur.com/aMtVZwk.png>

#### **Xóa network**
- Cú pháp :
    ```
    # openstack network delete <network_name|network_ID>
    ```
#### **Xóa subnet**
- Cú pháp :
    ```
    # openstack subnet delete <subnet_name|subnet_ID>
    ```
## **2) Security group**
#### **List các security group**
- Cú pháp :
    ```
    # openstack security group list
    ```
    <img src=https://i.imgur.com/QHz9QlH.png>

#### **Tạo security group**
- Cú pháp :
    ```
    # openstack security group rule create 
        [--remote-ip <ip-address> | --remote-group <group>]
        [--dst-port <port-range>]
        [--protocol <protocol>]
        [--description <description>]
        [--icmp-type <icmp-type>]
        [--icmp-code <icmp-code>]
        [--ingress | --egress]
        [--ethertype <ethertype>]
        [--project <project>]
        [--project-domain <project-domain>]
        <sec_group_name|sec_group_ID>
    ```
- **VD :** Tạo 1 security group cho phép giao thức SSH:
    ```
    # openstack security group rule create --protocol ssh --ingress default
    ```
#### **Hiển thị các rule đang có của một security group**
- Cú pháp :
    ```
    # openstack security group rule list <sec_group_name|sec_group_ID>
    ```
    - **VD :**
        ```
        # openstack security group rule list caea3729-3a73-4b94-b60e-3593ab257f80
        ```
        <img src=https://i.imgur.com/IWlf3jZ.png>

#### **Xóa 1 security group**
- Cú pháp :
    ```
    # openstack security group delete <sec_group_name|sec_group_ID>
    ```
#### **Gán security group vào instance**
- Cú pháp :
    ```
    # nova add-secgroup <instance_name|instance_ID> <sec_group_name|sec_group_ID>
    ```
- **VD :**
    ```
    # nova add-secgroup VM05 caea3729-3a73-4b94-b60e-3593ab257f80
    ```
#### **Gỡ security group khỏi instance**
- Cú pháp :
    ```
    # nova remove-secgroup <instance_name|instance_ID> <sec_group_name|sec_group_ID>
    ```
- **VD :**
    ```
    # nova remove-secgroup VM05 caea3729-3a73-4b94-b60e-3593ab257f80
    ```
## **3) Cấu hình IP cho instance**
#### **List các port đang có trên instance**
- Cú pháp :
    ```
    # openstack port list --server <instance_name>
    ```
- **VD :**
    ```
    # openstack port list --server VM05
    ```
    <img src=https://i.imgur.com/FhnlWat.png>

#### **Gán IP mới cho instance**
- **B1 :** Ngắt kết nối và xóa port đang gắn với instance :
    ```
    # nova interface-detach <instance_name> <port_ID>
    ```
    - **VD :**
        ```
        # nova interface-detach VM05 d384049e-04b3-4dd9-b3ea-1e8b9b689ee2
        ```
- **B2 :** Tạo 1 port mới :
    - Lấy theo DHCP :
        ```
        # openstack port create --fixed-ip subnet=<subnet_name> --network <net_name> <port_name>
        ```
    - Gán IP tĩnh :
        ```
        # openstack port create --fixed-ip subnet=<subnet_name>,ip-address=<IP>  --network <net_name> <port_name>
        ```
        > Chú ý : Kiểm tra kỹ địa chỉ IP đã được sử dụng chưa, tránh conflict IP
    - **VD :**
        ```
        # openstack port create --fixed-ip subnet=sub1private1,ip-address=192.168.1.105 --network private01 port01
        ```
        <img src=https://i.imgur.com/GvseyaB.png>

        > Copy lại `port_ID` tạo ra cho bước tiếp theo
- **B3 :** Gán port cho instance :
    ```
    # nova interface-attach --port-id <port_ID> <instance_name>
    ```
    - **VD :**
        ```
        # nova interface-attach --port-id 8cf718d7-ed15-410c-aca2-4b6184d9e47d VM05
        ```
        <img src=https://i.imgur.com/ixmvMPQ.png>
- **B4 :** Khởi động lại instance :
    ```
    # openstack server reboot <instance_name>
    ```
- **B5 :** Console instance và kiểm tra IP đã nhận chưa :
    
    <img src=https://i.imgur.com/gXO9f2P.png>

    > Instance đã nhận đúng IP

    <img src=https://i.imgur.com/KOSnV8m.png>

    > Instance có thể ping đến các IP khác

## **4) Router**