# Rescue VM
## **1) Giới thiệu**
- **OpenStack** cung cấp chức năng để rescue các instance trong các trường hợp lỗi hệ thống, mất mát filesystem, mất SSH key, cấu hình network bị sai hoặc dùng để khôi phục mật khẩu.
    > **Lưu ý:** Trong quá trình boot thì instance disk và recuse disk có thể trùng UUID , vì thế trong 1 một số trường hợp thì instance đã vào mode rescue nhưng vẫn boot từ local disk
- Một vài trường hợp sửa chữa một hệ thống bị lỗi bằng cách sử dụng rescue boot:
    - Mất ssh key và muốn kích hoạt tàm thời chế độ đăng nhập bằng password
    - Cấu hình mạng gặp lỗi
    - Cấu hình boot lỗi
- Để rescue instance bằng image được chỉ định sẵn, sử dụng lệnh :
    ```
    # openstack server rescue --image <image_name|image_id> <instance>
    ```
    > Lúc này trạng thái của instance sẽ là `RESCUE`
- Để gỡ bỏ trạng thái `RESCUE` cho instance, sử dụng lệnh :
    ```
    # openstack server unrescue <instance>
    ```
    > Instance sẽ reboot lại
## **2) Các bước thực hiện Rescue**
### **2.1) Rescue instance boot từ local disk**
#### **2.1.1) Thực hiện reset password VM sử dụng image SystemRescueCD**
> Trang chủ : https://www.system-rescue-cd.org/

**Tạo image SystemRescueCD**
- **B1 :** Download file `.iso` **SystemRescueCD** :
    ```
    # wget https://osdn.net/projects/systemrescuecd/storage/releases/6.1.7/systemrescuecd-amd64-6.1.7.iso
    ```
- **B2 :** Trên node **controller**, download `syslinux`. Đây là 1 package các lightweight MBR để khởi động OS :
    ```
    # yum install -y syslinux
    ```
- **B3 :** Sử dụng công cụ `isohybrid`, cho phép ISO image có thể được boot qua BIOS từ USB :
    ```
    # isohybrid systemrescuecd-amd64-6.1.7.iso
    ```
- **B4 :** Tạo image **SystemRescueCD** lên **Glance** :
    ```
    # openstack image create "system-rescue" --file systemrescuecd-amd64-6.1.7.iso --public
    ```
- **B5 :** Set các metadata cho image :
    ```
    # openstack image set --property hw_cdrom_bus=ide --property hw_disk_bus=ide system-rescue
    ```
**Cấu hình trên OpenStack**
- Để thực hiện việc reset password máy ảo, ta sẽ sử dụng 1 tính năng của nova là Rescue VM. Tính năng này sẽ đặt máy ảo vào trạng thái Rescue, trong đó VM sẽ boot vào 1 image được chỉ định.
- Khi thực hiện **rescue**, VM sẽ được soft reboot trước khi boot vào **SystemRescueCD**, nếu thời gian ***soft reboot*** lâu hơn thời gian mặc định của nova (`60s`), VM sẽ bị ***hard reboot***. Để quá trình reset password thành công, phải đảm bảo quá trình ***soft reboot*** này của VM thành công tốt đẹp. Để thực hiện điều đó, cấu hình giá trị **shutdown timeout** cao hơn mức mặc định của `Nova`.
- **B6 :** Sửa file `/etc/nova/nova.conf`, đặt thời gian timeout là `5p` (`300s`):
    ```
    # crudini --set /etc/nova/nova.conf DEFAULT shutdown_timeout 300
    # systemctl restart openstack-nova-api openstack-nova-scheduler openstack-nova-conductor openstack-nova-novncproxy
    ```
**Thực hiện reset password VM**
- **B7 :** List danh sách các instance để kiểm tra trạng thái :
    ```
    # openstack server list
    ```
    <img src=https://i.imgur.com/TwTxmjv.png>

    > ***Lưu ý :*** máy ảo phải đang ở trạng thái `ACTIVE` (Running) trước khi chuyển về `RESCUE`.
- **B8 :** Đưa instance về trạng thái `RESCUE` :
    ```
    # openstack server rescue --image system-rescue VM02
    ```
    - Trong đó :
        - `system-rescue` : tên image **SystemRescueCD** đã tạo trước đó
        - `VM02` : tên instance cần reset password
- **B9 :** Truy cập horizon, sẽ thấy trạng thái `Rescue` trên `VM02` :
    
    <img src=https://i.imgur.com/BJX6A42.png>

- **B10 :** Truy cập console của `VM02`, sẽ thấy giao diện của **SystemRescueCD** :
    
    <img src=https://i.imgur.com/TIOkbDb.png>

- **B11 :** Lựa chọn option đầu tiên ***Boot SystemRescueCD using default options***. VM sẽ boot vào cmd của **SystemRescueCD** (quá trình này có thể mất vài phút):

    <img src=https://i.imgur.com/gKqneQD.png>

- **B12 :** List các disk của instance :
    ```
    # fdisk -l
    ```
    <img src=https://i.imgur.com/QRdSHQP.png>

    > Ở đây có thể thấy `/dev/sdb1` là disk chứa OS instance
- **B13 :** Mount disk chứa OS vào thư mục `/mnt/root`
    ```
    # mkdir /mnt/root
    # mount /dev/sdb1 /mnt/root
    ```
- **B14 :** Thay đổi root directory từ **SystemRescueCD** sang thư mục `/mnt/root` :
    ```
    # chroot /mnt/root /bin/bash
    ```
- **B15 :** Đổi password cho user (ở đây sẽ là user `ubuntu`) :
    ```
    # passwd ubuntu
    ```
    > Có thể thay đổi password user nào tùy ý (kể cả user `root`)
- **B16 :** Gỡ bỏ trạng thái `RESCUE` của instance :
    ```
    # openstack server unrescue VM02
    ```
- **B17 :** Truy cập console lại vào instance, đăng nhập và kiểm tra :

    <img src=https://i.imgur.com/hEteuaR.png>
### **2.2) Rescue instance boot từ volume**