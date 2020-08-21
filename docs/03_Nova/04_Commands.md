# Các command thường sử dụng trong Nova
## **1) Flavor**
#### **List các flavor**
- Liệt kê các flavor đang có :
    ```
    # openstack flavor list
    ```
    <img src=https://i.imgur.com/d0xwkoA.png>
#### **Tạo mới Flavor**
- Cú pháp :
    ```
    # openstack flavor create
        [--id <id>]
        [--ram <size-mb>]
        [--disk <size-gb>]
        [--ephemeral-disk <size-gb>]
        [--swap <size-mb>]
        [--vcpus <num-cpu>]
        [--rxtx-factor <factor>]
        [--public | --private]
        [--property <key=value> [...] ]
        [--project <project>]
        [--project-domain <project-domain>]
        <flavor-name>
    ```
    > Tên flavor nên đặt viết liền, không dấu
- **VD :**
    ```
    # openstack flavor create --id auto \
    --ram 1024 \
    --disk 15 \
    --vcpus 1 \
    --public \
    Flavor-A
    ```
    <img src=https://i.imgur.com/nTbbIpq.png>

#### **Hiển thị thông tin Flavor**
- Cú pháp :
    ```
    # openstack flavor show <flavor_id|flavor_name>
    ```
- **VD :**
    ```
    # openstack flavor show Flavor-C
    ```
    <img src=https://i.imgur.com/d3RcxCi.png>

#### **Xóa Flavor**
- Cú pháp :
    ```
    # openstack flavor delete <flavor_id|flavor_name>
    ```
- **VD :**
    ```
    # openstack flavor delete Flavor-C
    ```
#### **Update thuộc tính Flavor**
- Cú pháp :
    ```
    # openstack flavor set
        [--no-property]
        [--property <key=value> [...] ]
        [--project <project>]
        [--project-domain <project-domain>]
        <flavor_id|flavor_name>
    ```
    > `--no-property` : xóa hết các thuộc tính đang có
- **VD :** Gán project 'admin' cho flavor 'Flavor-A':
    ```
    # openstack flavor set --project admin Flavor-A
    ```
    > Điều kiện : Flavor gán không có thuộc tính `--public`
#### **Xóa thuộc tính Flavor**
- Cú pháp :
    ```
    # openstack flavor unset
        [--property <key> [...] ]
        [--project <project>]
        [--project-domain <project-domain>]
        <flavor_id|flavor_name>
    ```
- **VD :** Bỏ gán project 'admin' cho flavor 'Flavor-A':
    ```
    # openstack flavor unset --project admin Flavor-A
    ```
## **2) Instance**
#### **Tạo instance từ image hoặc volume**
- Cú pháp :
    ```
    # openstack server create
        (--image <image> | --volume <volume>)
        --flavor <flavor>
        [--security-group <security-group>]
        [--key-name <key-name>]
        [--property <key=value>]
        [--file <dest-filename=source-filename>]
        [--user-data <user-data>]
        [--availability-zone <zone-name>]
        [--block-device-mapping <dev-name=mapping>]
        [--nic <net-id=net-uuid,v4-fixed-ip=ip-addr,v6-fixed-ip=ip-addr,port-id=port-uuid,auto,none>]
        [--network <network>]
        [--port <port>]
        [--hint <key=value>]
        [--config-drive <config-drive-volume>|True]
        [--min <count>]
        [--max <count>]
        [--wait]
        <server-name>
    ```
- **VD :**
    ```
    # openstack server create --flavor Flavor-A --image cirros VM01
    ```
    <img src=https://i.imgur.com/sjNdfvx.png>

#### **List các server đang có**
- Cú pháp :
    ```
    # openstack server list
    ```
    <img src=https://i.imgur.com/CVkJqlH.png>

#### **Show thông tin chi tiết một instance**
- Cú pháp :
    ```
    # openstack server show <instance_name|instance_id>
    ```
- **VD1 :**
    ```
    # openstack server show VM01
    ```
    <img src=https://i.imgur.com/a8ptjjh.png>

- **VD2 :** Hiển thị thông tin phân tích ram và disk của instance :
    ```
    # openstack server show VM01 --diagnostics
    ```
    <img src=https://i.imgur.com/6dR5kMb.png>

#### **Start, Stop, Reboot, Suspend, Resume**
- Cú pháp :
    ```
    # openstack server <start|stop|reboot|suspend|resume> <instance_name|instance_ID>
    ```
#### **Xóa Instance**
- Cú pháp :
    ```
    # openstack server delete [--wait] <instance_name|instance_ID>
    ```
    > `--wait` : đợi lệnh cho đến khi quá trình xóa hoàn tất
#### **List các Hypervisor (các compute)**
- Cú pháp :
    ```
    # openstack hypervisor list
    ```
    <img src=https://i.imgur.com/ZMnJbnK.png>

#### **Show thông tin chi tiết một Hypervisor**
- Cú pháp :
    ```
    # openstack hypervisor show <hostname> --fit-width
    ```
    > `--fit-width` : hiển thị vừa màn hình terminal

    <img src=https://i.imgur.com/dLIE7x4.png>