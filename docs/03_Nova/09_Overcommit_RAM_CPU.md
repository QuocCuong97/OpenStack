# Overcommit RAM & CPU
## **1) Giới thiệu**
- **OpenStack** cho phép khai báo vượt quá (*overcommit*) CPU và RAM trên các node compute.Điều này cho phép tăng số instance chạy trên cloud .
- Theo mặc định, Compute service cho phép tỉ lệ sau:
    - CPU : `16:1`
    - RAM : `1.5:1`
- Công thức tính số lượng các máy ảo trên compute node là `(OR*PC)/VC`, trong đó:
    - `OR`: CPU overcommit ratio (virtual cores tương ứng với mỗi physical core)
    - `PC`: Số physical cores.
    - `VC`: Số virtual cores mỗi instance.
> ***Chú ý 1 :*** Cho dù ti lệ ratio là bao nhiêu, instance không thể được tạo trên một node compute có lượng tài nguyên thô (cpu, ram, disk ban đầu) nhỏ hơn lượng tài nguyên mà flavor của instance yêu cầu

> ***Chú ý 2 :*** Cần thay đổi quota cho thích hợp khi sử dụng overcommit
## **2) Cấu hình**
- Cấu hình chỉ số ratio cho CPU và RAM ở file `/etc/nova/nova.conf` trên node `controller` :
    ```ini
    [DEFAULT]
    cpu_allocation_ratio = 0.0
    ram_allocation_ratio = 0.0
    disk_allocation_ratio = 0.0
    ```
    > Nếu để ratio là `0.0` thì chỉ số ratio mặc định của cpu, ram, disk là `16.0`, `1.5`, `1.0`
- Sau khi cấu hình, restart lại dịch vụ của `nova`:
    ```
    # systemctl restart openstack-nova-api.service openstack-nova-scheduler openstack-nova-novncproxy openstack-nova-conductor
    ```
## **3) Kiểm chứng**
> ### **Thực hiện tạo 3 VM có `6` cores trên 2 compute chỉ có `8` cores**
- **B1 :** Tạo flavor trên node **controller** :
    ```
    # openstack flavor create --id auto \
    --ram 1024 \
    --disk 15 \
    --vcpus 6 \
    --public \
    Flavor-C
    ```
- **B2 :** Tạo instance :
    ```
    # openstack server create --flavor Flavor-C --image centos7 --nic net-id=cdaf9772-ad08-471d-a3b6-332a9a3bbaa6 --user-data=cloud_config VM04
    # openstack server create --flavor Flavor-C --image centos7 --nic net-id=cdaf9772-ad08-471d-a3b6-332a9a3bbaa6 --user-data=cloud_config VM05
    # openstack server create --flavor Flavor-C --image centos7 --nic net-id=cdaf9772-ad08-471d-a3b6-332a9a3bbaa6 --user-data=cloud_config VM06
    ```
    > Cần chú ý quota của project. Quota mặc định chỉ cho phép `20` core trong 1 project. Tăng quota bằng lệnh `openstack quota set`
- **B3 :** Truy cập console vào các instance để kiểm tra kết quả :

    <img src=https://i.imgur.com/A8PqhhF.png>
    <img src=https://i.imgur.com/aKUZkTa.png>
    <img src=https://i.imgur.com/wpxxJp7.png>

> ***Chú ý :*** Các thống kê trên OpenStack chỉ thống kê tài nguyên thực, không thống kê tài nguyên ảo :

<img src=https://i.imgur.com/YTCYkiG.png>

> Khi đạt giới hạn quota, user sẽ không thể tạo instance nữa :

<img src=https://i.imgur.com/2quBi1c.png>