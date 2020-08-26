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
