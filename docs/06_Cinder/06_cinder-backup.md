# Cấu hình `cinder-backup` với NFS Backend
## **1) Cấu hình NFS cho node `controller`**
- **B1 :** Tạo và chỉnh sửa file `/etc/cinder/nfs_shares` :
    ```
    # vi /etc/cinder/nfs_shares
    ```
    - Thêm vào file thư mục chứa volume NFS và các volume backups :
        ```
        controller:/var/lib/nfs-share
        ```
- **B2 :** Phân quyền cho file `nfs_shares` vừa tạo trên :
    ```
    # chown root:cinder /etc/cinder/nfs_shares
    # chmod 0640 /etc/cinder/nfs_shares
    ```
- **B3 :** Cấu hình **cinder** sử dụng `/etc/cinder/nfs_shares` file đã được tạo phía trên :
    ```
    # crudini --set /etc/cinder/cinder.conf backend_defaults nfs_shares_config /etc/cinder/nfs_shares
    # crudini --set /etc/cinder/cinder.conf backend_defaults volume_driver cinder.volume.drivers.nfs.NfsDriver
    ```
- **B4 :** Khởi động lại các dịch vụ :
    ```
    # systemctl enable rpcbind nfs-server
    # systemctl restart openstack-cinder-volume target rpcbind nfs-server
    ```
- **B5 :** Cài đặt `nfs-utils` :
    ```
    # yum install nfs-utils -y
    # mkdir /var/lib/nfs-share
    # echo "/var/lib/nfs-share 10.10.230.0/24(rw,no_root_squash)" > /etc/exports 
    # systemctl restart rpcbind nfs-server
    ```
- **B6 :** Chỉnh sửa cấu hình **cinder** :
    ```
    # vi /etc/cinder/cinder.conf
    ```
    - Thêm `nfs` vào `enabled_backends` :
        ```ini
        [DEFAULT]
        ...
        enabled_backends = lvm,nfs
        ...
        ```
    - Thêm đoạn sau vào cuối file :
        ```ini
        [nfs]
        volume_driver = cinder.volume.drivers.nfs.NfsDriver
        nfs_shares_config = /etc/cinder/nfs_shares
        volume_backend_name = nfsdriver-1
        nfs_mount_point_base = $state_path/mnt_nfs
        ```
- **B7 :**
    ```
    # systemctl restart openstack-cinder-volume
    # chown -R cinder. /var/lib/cinder/mnt_nfs/
    ```
- **B8 :**
    ```
    # systemctl restart openstack-cinder-volume target rpcbind nfs-server
    ```
- **B9 :** Tạo type volume :
    ```
    # openstack volume type create nfs
    # openstack volume type set nfs --property volume_backend_name=nfsdriver-1
    ```
    - Kiểm tra lại :
        ```
        # openstack volume type list --long
        ```
        <img src=https://i.imgur.com/zDLOdzm.png>
- **B10 :** Khởi tạo NFS Disk :
    ```
    # openstack volume create --type nfs --size 30 disk_nfs
    ```
- **B11 :** Khởi động dịch vụ **`cinder-backup`** :
    ```
    # systemctl start openstack-cinder-backup
    # systemctl enable openstack-cinder-backup
    ```
- **B12 :** Kiểm tra lại các volume service :
    ```
    # openstack volume service list
    ```
    <img src=https://i.imgur.com/HpAtZUA.png>

- **B13 :** Chỉnh sửa file cấu hình **cinder** :
    ```
    # crudini --set /etc/cinder/cinder.conf DEFAULT backup_driver cinder.backup.drivers.nfs.NFSBackupDriver
    # crudini --set /etc/cinder/cinder.conf DEFAULT backup_share controller:/var/lib/nfs-share
    ```
- **B14 :** Khởi động lại dịch vụ **`cinder-backup`** :
    ```
    # systemctl restart openstack-cinder-backup
    ```
## **2) Backup và Restore**
- Để có thể tạo backup từ một volume thì volume đó phải đang ở trạng thái `available`. Nếu volume đang được attach vào instance thì cần phải gỡ nó ra hoặc chuyển đổi status của nó bằng tài khoản admin, sau khi backup xong thì attach lại.
### **2.1) Backup**
- Cú pháp :
    ```
    # openstack volume backup create --name <tên_bản_backup> [--incremental] [--force] <VOLUME>
    ```
    - Trong đó:
        - `<VOLUME>` : tên hoặc ID của volume
        - `incremental` : để chỉ ra kiểu backup muốn thực hiện là incremental backup (không có cờ này thì mặc định sẽ thực hiện full backup)
        - `force` : là cờ chỉ ra sự cho phép hoặc không cho phép thực hiện backup volume trong khi đang được attach vào máy ảo. Khi không có cờ `force`, volume sẽ chỉ được back up khi nó ở trạng thái available (nghĩa là đang không được gắn vào máy ảo). Khi trạng thái của volume là `in-use`, nó đang được gán vào một máy ảo nào đó, muốn backup nó cần bật cờ `force`, khi đó dữ liệu trong volume sẽ bị ngắt quãng. Mặc định cờ `force` sẽ được thiết lập bằng `FALSE`.
> **Chú ý:** Qúa trình backup tiêu tốn khá nhiều tài nguyên, vì vậy trước khi basckup cần chú ý tăn RAM, CPU nếu có thể, hoặc có thể tạo swap hỗ trợ RAM
### **2.2) Restore**
- Cú pháp :
    ```
    # openstack volume backup restore BACKUP_ID VOLUME_ID
    ```
> Sau khi restore cần reboot lại VM.
- Khi restore từ full backup nó sẽ gọi là full restore.
- Khi restore từ incremental backup, danh sách backup được liệt kê dựa trên IDs của parent backups. Một full restore được thực hiện dựa trên full backup đầu tiên và các lần incremental backup sau đó được đặt theo đúng thứ tự
### **2.3) Kiểm chứng**
- **B1 :** Ta tạo 1 volume từ image cirros và tạo 1 vm với volume đó:
    ```
    # openstack volume create --type nfs --size 10 --image cirros vlm_nfs_cirros
    # openstack server create --flavor Flavor-A --volume vlm_nfs_cirros --nic net-id=cdaf9772-ad08-471d-a3b6-332a9a3bbaa6 VM05
    ```
- **B2 :** Console và tạo 1 file bên trong instance :
    ```
    $ touch test_file
    ```
    <img src=https://i.imgur.com/xwgpm5t.png>
- **B3 :** Tắt instance và chuyển về volume về trạng thái `available` :
    ```
    # openstack server stop VM05
    # openstack volume set --state available vlm_nfs_cirros
    ```
    <img src=https://i.imgur.com/VLLtsEC.png>
- **B4 :** Backup volume :
    ```
    # openstack volume backup create --name backup_vm05 vlm_nfs_cirros
    ```
    <img src=https://i.imgur.com/d9nH3qb.png>
- **B5 :** Kiểm tra lại trạng thái volume backup :
    ```
    # openstack volume backup list
    ```
    <img src=https://i.imgur.com/VM2AKNA.png>
- **B6 :** Khởi động instance :
    ```
    # openstack server start VM05
    ```
    - Thực hiện xóa file `test_file` đã tạo lúc trước trên instance :
        ```
        # rm test_file
        ```
        <img src=https://i.imgur.com/k24MdQd.png>
- **B7 :** Thực hiện restore bản backup đã tạo :
    ```
    # openstack volume backup restore 6508532c-349a-4ff5-9519-70783910201a 0bcf6f96-ad7f-4a33-8903-10f03242f960
    ```
    - Trong đó :
        - `6508532c-349a-4ff5-9519-70783910201a` : volume backup ID
        - `0bcf6f96-ad7f-4a33-8903-10f03242f960` : volume ID hiện tại

    <img src=https://i.imgur.com/RGD0lbZ.png>

    > Trạng thái volume lúc này sẽ chuyển thành `restoring-backup` .
- **B8 :** Reboot instance và kiểm chứng :
    ```
    $ ls -ll
    ```
    <img src=https://i.imgur.com/xMzDgoa.png>
    
    > Ta thấy instance đã quay trở lại thời điểm backup, khi file `test_file` chưa bị xóa .
- **B9 :** Thay đổi lại trạng thái cho volume về `in-use` :
    ```
    # openstack volume set --state in-use vlm_nfs_cirros
    ```
    <img src=https://i.imgur.com/iIcRiFN.png>

> Hoàn tất quá trình restore!
