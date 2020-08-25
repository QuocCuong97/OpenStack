# Resize VM
- Có thể thay đổi cấu hình của một instance bằng cách thay đổi **flavor** của nó .
## **1) Resize instance boot từ local**
- Thực hiện resize instance bằng câu lệnh :
    ```
    # openstack server resize --flavor <flavor_name> <instance_name|instance_id>
    ```
- Đợi đến khi trạng thái máy ảo chuyển về `VERIFY_RESIZE` (dùng câu lệnh `openstack server show` để xem). Tiến hành xác nhận hoặc cancel việc resize instance. Tiến hành xác nhận bằng câu lệnh sau:
    ```
    # openstack server resize --confirm <instance_name|instance_id>
    ```
    hoặc cancel việc resize :
    ```
    # openstack server resize --revert <instance_name|instance_id>
    ```
- **VD :** Resize instance từ `Flavor-A` (`1GB-15GB-1vCPU`) sang `Flavor-B` (`2GB-20GB-2vCPUs`) :
    - **B1 :** Tạo `Flavor_B` (nếu chưa tạo sẵn) :
        ```
        # openstack flavor create --id auto \
        --ram 4096 \
        --disk 20 \
        --vcpus 2 \
        --public \
        Flavor-B
        ```
        <img src=https://i.imgur.com/JDOJT1z.png>
    - **B2 :** Resize instance :
        ```
        # openstack server resize --flavor Flavor-B VM05
        ```
    - 
## **2) Resize instance boot từ volume**