# Resize VM
- Có thể thay đổi cấu hình của một instance bằng cách thay đổi **flavor** của nó, có thể thực hiện trên cùng host hoặc resize đến host khác .
- Các đặc điểm cần chú ý:
    - Phải resize VM đến flavor khác.
    - Không thể resize VM nếu VM đó đang ở trạng thái được resized.
    - Không thể resize VM đến disk flavor có kích thước nhỏ hơn disk flavor hiện tại của VM.
    - Nếu resize đến cùng host, nhưng host đó không đủ tài nguyên cho flavor mới, VM sẽ không thay đổi flavor.
    - Có thể giảm CPU, nhưng không thể giảm kích thước disk của bất kỳ VM nào.
    - Resize đến host khác yêu cầu cấu hình SSH Tunnel giữa 2 host.
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
    - **B1 :** Tạo `Flavor-B` (nếu chưa tạo sẵn) :
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
        # openstack server resize --flavor Flavor-B VM03
        ```
    - **B3 :** Kiểm tra trạng thái instance (lúc này đang ở trạng thái `VERIFY_RESIZE`):
        ```
        # openstack server list | grep VM03
        ```
        <img src=https://i.imgur.com/WB5aSpu.png>

        - Hiển thị trên giao diện **Horizon** :

            <img src=https://i.imgur.com/gH0uxub.png>
    - **B4 :** Xác nhận việc resize :
        ```
        # openstack server resize confirm VM03
        ```
    - **B5 :** Kiểm tra lại flavor của instance :
        ```
        # openstack server show VM03
        ```
        <img src=https://i.imgur.com/YvV6I75.png>
        
        > Tại đây, ta thấy instance đã được bầu chọn và migrate sang node compute khác 
    - **B6 :** Đăng nhập vào instance, kiểm tra lại cấu hình :
        
        <img src=https://i.imgur.com/ftpF53q.png>

        > Cấu hình đã đúng với Flavor sau khi resize
## **2) Resize instance boot từ volume**
> Thực hiện resize instance `VM05` cấu hình RAM 1G, 1 core, 10GB disk volume thành :<br>
    - Flavor-B : RAM 4G, 2 core
    - Disk : 20GB
- **B1 :** Tắt instance :
    ```
    # openstack server stop VM05
    ```
- **B2 :** Kiểm tra volume mà instance đang sử dụng :
    ```
    # openstack volume list | grep VM05
    ```
    <img src=https://i.imgur.com/fZQ4FIv.png>

- **B3 :** Thực hiện resize disk :
    ```
    # openstack volume set --state available vlm_cirros
    # openstack volume set vlm_cirros --size 20
    ```
    - Kiểm tra lại trạng thái volume :
        ```
        # openstack volume list
        ```
        <img src=https://i.imgur.com/fL5fiCl.png>

- **B4 :** Khởi động instance :
    ```
    # openstack server start VM05
    ```
- **B4 :** Thực hiện resize Flavor :
    ```
    # openstack server resize --flavor Flavor-B VM05
    ```
    - Kiểm tra lại Flavor của instance :
        ```
        # openstack server list | grep VM05
        ```
