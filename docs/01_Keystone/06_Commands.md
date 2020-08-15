# Các lệnh cơ bản trong Keystone
## **1) Khởi tạo biến môi trường**
- Là cách tạo ra một file chứa thông tin đăng nhập
- Nên tạo tên file là tên user cho dễ nhớ :
    ```
    # vi admin-openrc
    ```
    - Thêm vào nội dung file :
        ```
        export OS_USERNAME=admin
        export OS_PASSWORD=Password123
        export OS_PROJECT_NAME=admin
        export OS_USER_DOMAIN_NAME=Default
        export OS_PROJECT_DOMAIN_NAME=Default
        export OS_AUTH_URL=http://10.10.230.10:5000/v3
        export OS_IDENTITY_API_VERSION=3
        export OS_IMAGE_API_VERSION=2
        ```
        > Với `10.10.230.10` là IP MGNT của node `controller`
- Khởi chạy môi trường :
    ```
    # source <path_to_file>/admin-openrc
    ```
- Kiểm tra biến môi trường :
    ```
    # env | grep OS
    ```
    <img src=https://i.imgur.com/Io0KBHW.png>

- Lấy token :
    ```
    # openstack token issue
    ```
    <img src=https://i.imgur.com/QDZeRZG.png>

## **2) Các lệnh làm việc với user, project, groups, roles, domain**
### **2.1) User**
#### **List các user**
- List các user đang có :
    ```
    # openstack user list
    ```
    <img src=https://i.imgur.com/pWqPg8N.png>

- List user trong group cụ thể :
    ```
    # openstack user list --group <group_name>
    ```
- List user trong project cụ thể :
    ```
    # openstack user list --project <project_name>
    ```
- List user trong domain cụ thể :
    ```
    # openstack user list --domain <domain_name>
    ```
#### **Tạo user**
- Tạo user mới :
    ```
    # openstack user create <username> --password <password>
    ```
    - **VD :** Tạo user `cuongnq` :
        
        <img src=https://i.imgur.com/Ec75B3p.png>

#### **Update thông tin user**
> Có thể cập nhật tên, email, và enable status cho user
- Show thông tin user :
    ```
    # openstack user show <username>
    ```
    <img src=https://i.imgur.com/xm0bdiW.png>

- Vô hiệu hóa user tạm thời :
    ```
    # openstack user set <username> --disable
    ```
- Enable lại user :
    ```
    # openstack user set <username> --enable
    ```
- Thay đổi username của user :
    ```
    # openstack user set <username> --name <username-new>
    ```
    <img src=https://i.imgur.com/S1Hff6a.png>

**Xóa user**
- Xóa user :
    ```
    # openstack user delete <username>
    ```
### **2.2) Project**
#### **List các project**
- Liệt kê các project :
    ```
    # openstack project list
    ```
    <img src=https://i.imgur.com/QG5iAmV.png>
#### **Tạo project**
- Tạo một project :
    ```
    # openstack project create --description '<Description_Project>' <project_name> --domain <domain_name>
    ```
#### **Update thông tin project**
> Để update thông tin của project, cần có ID hoặc tên của project
- Disable project :
    ```
    # openstack project set <project_ID || project_name> --disable
    ```
- Enable project :
    ```
    # openstack project set <project_ID || project_name> --enable
    ```
- Thay đổi tên project :
    ```
    # openstack project set <project_ID || project_name> --name <project_newname>
    ```
- Hiển thị thông tin project :
    ```
    # openstack project show <PROJECT_ID || project_name>
    ```
    <img src=https://i.imgur.com/wdR6fvm.png>
#### **Xóa project**
- Xóa một project :
    ```
    # openstack project delete <project_ID || project_name>
    ```
### **2.3) Group**
#### **List các group**
- Liệt kê các group đang có :
    ```
    # openstack group list
    ```
#### **Tạo group**
- Tạo một group :
    ```
    # openstack group create [--domain <domain>] [--description <description>] [--or-show] <group-name>
    ```
    <img src=https://i.imgur.com/Fbad1aT.png>

#### **Add user vào group**
- Thêm một hoặc nhiều user vào group :
    ```
    # openstack group add user [--group-domain <group-domain>] [--user-domain <user-domain>] <group_name> <username> [<username> ...]
    ```
    - **VD :**
        ```
        # openstack group add user --group-domain default --user-domain default sysadmin cuongnqc
        ```
#### **Kiểm tra user có trong group không**
- Sử dụng lệnh sau để kiểm tra :
    ```
    # openstack group contains user [--group-domain <group-domain>] [--user-domain <user-domain>] <group_name> <username>
    ```
    <img src=https://i.imgur.com/FDXQ7j0.png>
#### **Xóa user khỏi group**
- Xóa một hoặc nhiều user khỏi một group :
    ```
    # openstack group remove user [--group-domain <group-domain>] [--user-domain <user-domain>] <group_name> <username> [<username> ...]
    ```
#### **Update lại thông tin group**
- Đặt lại các thuộc tính của group :
    ```
    # openstack group set [--domain <domain>] [--name <name>] [--description <description>] <group_name>
    ```
#### **Hiển thị thông tin group**
- Hiển thị thông tin group :
    ```
    # openstack group show [--domain <domain>] <group_name>
    ```
    <img src=https://i.imgur.com/RJN5uo9.png>
### **2.4) Role**
#### **List các roles**
- Liệt kê các role đang có :
    ```
    # openstack role list
    ```
    <img src=https://i.imgur.com/7FEWpWa.png>
#### **Tạo một role mới**
- Để gán người dùng cho nhiều project, hay xác định vai role cho user với các project :
    ```
    # openstack role create <role_name>
    ```
    > Nếu sử dụng Identity v3 thì cần có option `--domain` với một domain cụ thể
#### **Gán role cho user**
- Để gán user cho project, ta cần gán role theo cặp ***user-project***. Để làm điều đó, ta cần user ID, role và projectID .
- **B1 :** List user và để lấy userID cần sử dụng :
    ```
    # openstack user list
    ```
    <img src=https://i.imgur.com/9Ev5o2d.png>

- **B2 :** List các Role để lấy roleID :
    ```
    # openstack role list
    ```
    <img src=https://i.imgur.com/8jUJ4AC.png>
- **B3 :** List các project để lấy projectID :
    ```
    # openstack project list
    ```
    <img src=https://i.imgur.com/7F0urmc.png>
- **B4 :** Gán role :
    ```
    # openstack role add --user <username> --project <project_name> <role_name>
    ```
- **B5 :** Xác nhận lại :
    ```
    # openstack role assignment list --user <username> --project <project_id> --names
    ```
    - **VD :**

        <img src=https://i.imgur.com/5yYGRZm.png>
#### **Xem thông tin chi tiết role**
- Show thông tin chi tiết role :
    ```
    # openstack role show <role_name>
    ```
    <img src=https://i.imgur.com/vpZn4pm.png>
#### **Xóa role**
- **B1 :** Xóa role :
    ```
    # openstack role remove --user <username> --project <project_name> <role_name>
    ```
- **B2 :** Xác nhận lại việc xóa role :
    ```
    # openstack role assignment list --user <user> --project <project_name|project_ID> --names
    ```
## **3) Câu lệnh `keystone-manage`**
- `keystone-manage` là công cụ dòng lệnh tương tác với **Keystone** để thiết lập và cập nhật dữ liệu trong việc quản lý các dịch vụ của **keystone**.
- `keystone-manage` chỉ được sử dụng để hoạt động mà không thể thực hiện thông qua các API của HTTP - như là import/export dữ liệu và di chuyển database.
- Cú pháp :
    ```
    # keystone-manage [options] <action> [additional args]
    ```
    - **Options :**
        - `--help` : hiển thị menu trợ giúp
    - **Actions :**
        - `bootstrap` : khởi tạo thông tin tài khoản admin
        - `credential_migrate` : mã hóa thông tin đăng nhập theo private key mới
        - `credential_rotate` :
        - `credential_setup` :
        - `db_sync` : đồng bộ database
        - `db_version` : hiển thị phiên bản DB hiện tại
        - `doctor` :
        - `domain_config_upload` : upload file cấu hình domain
        - `fernet_rotate` : xoay khóa trong thư mục chứa fernet key
        - `fernet_setup` : setup fernet key repo để mã hóa token
        - `mapping_populate` :
        - `mapping_purge` :
        - `mapping_engine` :
        - `saml_idp_metadata` :
        - `token_flush` :
---------------------------
Tham khảo thêm
- https://docs.openstack.org/python-openstackclient/latest/cli/command-objects/role-assignment.html
- https://docs.openstack.org/python-openstackclient/latest/cli/command-objects/group.html
- https://docs.openstack.org/keystone/train/admin/cli-manage-projects-users-and-roles.html