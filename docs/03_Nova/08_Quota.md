# Quota
## **1) Giới thiệu**
- Để tránh việc hệ thống **OpenStack** bị cạn tài nguyên mà không có thông báo, cần setup **quota**. **Quota** là giới hạn tài nguyên được sử dụng. 
- **Quota** có thể được áp dụng trên cả mức project và mức user.
- Sử dụng CLI, có thể quản lý **quota** cho **OpenStack compute service**, **OpenStack Block Storage**, và **OpenStack Networking service**
## **2) Quản lý quota trên `Nova`**
- Một số loại **quota** :

    | Tên quota | Mô tả |
    |-----------|-------|
    | `cores` | Số lượng core (VCPUs) cho phép trong mỗi project |
    | `instances` | Số lượng instance cho phép trong mỗi project |
    | `key_pairs` | Số lượng key_pair cho phép mỗi user |
    | `metadata_items` | Số lượng metadata item cho phép trên mỗi instance |
    | `ram` | Số lượng MB RAM cho phép trên mỗi instance trong project |
    | `server_groups` | Số lượng server group cho phép trên mỗi project |
    | `server_group_members` | Số lượng server cho phép trên mỗi server group |

- Hiển thị **quota** mặc định cho các project :
    ```
    # openstack quota show --default
    ```
    <img src=https://i.imgur.com/CrasNqI.png>

- Chỉnh sửa **quota** mặc định cho các project :
    ```
    # openstack quota set <--parameters> --class default
    ```
    > Hiện tại `nova` chỉ đang hỗ trợ class `default`
    - **VD :** Chỉnh sửa tăng số lượng instance tối đa lên `20` :
        ```
        # openstack quota set --instances 20 --class default
        ```
        <img src=https://i.imgur.com/dzx9Z6q.png>

- Hiển thị **quota** của project :
    ```
    # openstack quota show <project_name>
    ```
    - Hiển thị **quota** của project trên dashboard **Horizon** :

        <img src=https://i.imgur.com/TdewxMR.png>

    - Cũng có thể hiển thị **quota** của project bằng nova-cli :
        ```
        # nova limits --tenant default
        ```
        <img src=https://i.imgur.com/pk3Rct5.png>
- Set giá trị **quota** cho project :
    ```
    # openstack quota set <project_name> --parameters
    ```
- Hiển thị **quota** của user trong project :
    ```
    # nova quota-show --user <user_id|user_name> --tenant <project_id>
    ```
    - **VD :** Hiển thị quota cho user `admin` trong project `admin` :
        ```
        # nova quota-show --user admin --tenant 64e04e0a977f43b99198744098d6fa63
        ```
        <img src=https://i.imgur.com/0jJmKrA.png>

- Set giá trị **quota** cho user trong project :
    ```
    # nova quota-set --user <user_id|user_name> --tenant <project_id> --parameters
    ```
-----------------------
[Tham khảo](https://docs.openstack.org/nova/train/admin/quotas.html)