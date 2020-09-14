# Security group
## **1) Giới thiệu**
- **Security group** là bộ các quy tắc để filter các IP, được áp dụng cho tất cả các instance để định nghĩa lưu lượng mạng truy cập vào các instance. **Group rules** được xác định cho các projects cụ thể, các user thuộc vào project nào thì có thể chỉnh sửa, thêm, xóa các rule của group tương ứng.
- Tất cả các project đều có một security-groups mặc định là default được áp dụng cho bất kỳ một instance nào không được định nghĩa một security group nào khác. Nếu không thay đổi gì thì mặc định security group sẽ chặn tát cả các incoming traffic với instance của bạn.
## **2) Các command thường dùng**
#### **List các security group**
- Cú pháp :
    ```
    # openstack security group list
    ```
    <img src=https://i.imgur.com/QHz9QlH.png>

#### **List các security group trên project cụ thể**
- Cú pháp :
    ```
    # openstack security group list --project <project_name>
    ```
    <img src=https://i.imgur.com/jSBkab6.png>

#### **Tạo security group (rule)**
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
- **VD :** Tạo 1 security group cho phép giao thức TCP:
    ```
    # openstack security group rule create --protocol tcp --ingress default
    ```
#### **Xóa 1 rule trong security group**
- Cú pháp :
    ```
    # openstack security group rule delete <rule_ID>
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
## **3) Kiểm chứng**
### **3.1) Cho phép ping tới instance**
- Mặc định, sử dụng security group `Default` của project, nó sẽ chặn toàn bộ các kết nối từ ngoài vào instance 
- Thử ping từ ngoài vào instance. Kết quả sẽ là fail :
    <img src=https://i.imgur.com/rQ4mR1Z.png>
- Cấu hình cho phép luồng ICMP (ping) đến từ IP `10.5.11.210` :
    - **B1 :** Hiển thị security group của project :
        ```
        # openstack security group list --project admin
        ```
        <img src=https://i.imgur.com/TeIwR7h.png>

    - **B2 :** Thêm rule cho **security group** :
        ```
        # openstack security group rule create 5d948f95-3d5d-4752-b4b6-17fae0f8a082 --protocol icmp --remote-ip 10.5.11.210
        ```
        > Nếu muốn allow tất cả IP thì không cần option `--remote-ip`
    - **B3 :** Kiểm tra kết quả :
        - IP `10.5.11.210` từ ngoài có thể ping được vào instance :

            <img src=https://i.imgur.com/JRFmMvu.png>

        - Các IP khác đều không thể ping được instance (ngoại trừ compute node):
            
            <img src=https://i.imgur.com/tZ17q2Y.png>
### **3.2) Cho phép SSH tới instance**
- Thử SSH từ ngoài vào instance. Kết quả sẽ là :
    <img src=https://i.imgur.com/LDm1srH.png>
- Cấu hình cho phép luồng SSH đến từ IP `10.5.11.210` :
    - **B1 :** Thêm rule cho **security group** :
        ```
        # openstack security group rule create 5d948f95-3d5d-4752-b4b6-17fae0f8a082 --protocol tcp --dst-port 22:22 --remote-ip 10.5.11.210
        ```
    > Nếu muốn allow tất cả IP thì không cần option `--remote-ip`
    - **B2 :** Kiểm tra kết quả :
        - IP `10.5.11.210` có thể SSH được vào instance :

            <img src=https://i.imgur.com/OssZ6OH.png>
        
        - Các IP khác đều không thể SSH vào instance :

            <img src=https://i.imgur.com/QRTYtB4.png>

-----------------------
Tham khảo
- https://docs.openstack.org/python-openstackclient/latest/cli/command-objects/security-group-rule.html
- https://docs.openstack.org/ocata/user-guide/cli-nova-configure-access-security-for-instances.html