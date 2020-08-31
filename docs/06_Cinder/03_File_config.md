# File cấu hình của Cinder
- File cấu hình chính : `/etc/cinder/cinder.conf`

    <img src=https://i.imgur.com/HLNE8uQ.png>

- Các phần cần chú ý trong file :
    - Khai báo database :
        ```ini
        [database]
        ...
        connection = mysql+pymysql://cinder:Password123@controller/cinder
        ...
        ```
    - Cấu hình **RabbitMQ** :
        ```ini
        [DEFAULT]
        ...
        transport_url = rabbit://openstack:Password123@controller
        ...
        ```
    - Cấu hình identity service :
        ```ini
        [DEFAULT]
        ...
        auth_strategy = keystone
        ...
        [keystone_authtoken]
        www_authenticate_uri = http://controller:5000
        auth_url = http://controller:5000
        memcached_servers = controller:11211
        auth_type = password
        project_domain_name = default
        user_domain_name = default
        project_name = service
        username = cinder
        password = Password123
        ...
        ```
    - Cấu hình management IP :
        ```ini
        [DEFAULT]
        ...
        my_ip = 10.10.230.10
        ...
        ```
    - Cấu hình kết nối **Glance** :
        ```ini
        [DEFAULT]
        ...
        glance_api_servers = http://controller:9292
        ...
        ```
    - Cấu hình `oslo` :
        ```ini
        ...
        [oslo_concurrency]
        lock_path = /var/lib/cinder/tmp
        ...
        ```
### **Cấu hình Storage Backend**
#### **LVM**
- Cấu hình **LVM** backend :
    ```ini
    ...
    [DEFAULT]
    enabled_backends = lvm
    ...
    [lvm]
    volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
    volume_group = cinder-volumes
    target_protocol = iscsi
    target_helper = lioadm
    ...
    ```
    - Trong đó :
        - `enabled_backends = lvm` : Sử dụng backend là **LVM**. Đối với multiple backend chỉ cần dấu phẩy giữa các backend (**VD :** `enable_backends = lvm,nfs,glusterfs`)
        - `volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver` : Chỉ định driver mà **LVM** sử dụng
        - `volume_group = cinder-volumes` : Chỉ định `vgroup` mà tạo lúc cài đặt. Sử dụng câu lệnh `vgs` hoặc `vgdisplay` để xem thông tin về `vgroup` đã tạo.
        - `target_protocol = iscsi` : Xác định giao thức **iSCSI** cho **iSCSI** volumes mới, được tạo ra với `tgtadm` hoặc `lioadm`. Để kích hoạt **RDMA** , tham số nên được đặt là "`iser`" . Hỗ trợ cho giao thức **iSCSI** giá trị là "`iscsi`" và "`iser`"
        - `target_helper = lioadm` : Chỉ ra **iSCSI** sử dụng. Mặc định là `tgtadm`. Có các tùy chọn sau:
            - `lioadm` hỗ trợ **LIO iSCSI**
            - `scstadmin` cho **SCST**
            - `iseradm` cho **ISER**
            - `ietadm` cho **iSCSI**
#### **GlusterfFS**
- Cấu hình **GlusterFS** backend :
    ```ini
    [DEFAULT]
    ...
    enabled_backends = glusterfs
    ...
    [glusterfs]
    volume_driver = cinder.volume.drivers.glusterfs.GlusterfsDriver
    glusterfs_shares_config = /etc/cinder/glusterfs_shares
    glusterfs_mount_point_base = $state_path/mnt_gluster
    ...
    ```
    - Trong đó :
        - `enabled_backends = glusterfs` : Sử dụng backend là **GlusterFS**
        - `volume_driver = cinder.volume.drivers.glusterfs.GlusterfsDriver` : Chỉ định driver mà **GlusterFS** sử dụng
        - `glusterfs_shares_config = /etc/cinder/glusterfs_shares` : File cấu hình để kết nối tới **GlusterFS**.
            - Nội dung file `glusterfs_shares` có dạng : `<địa_chỉ_IP>:/cinder-vol` :
                - `<địa_chỉ_IP>` : IP của **GlusterFS Pool**.
                - `cinder-vol` : Tên volume đã tạo ở GlusterFS
        - `glusterfs_mount_point_base = $state_path/mnt_gluster` : Mount point tới **GlusterFS**
---------------
Tham khảo
- [Các Volume Driver hỗ trợ](https://docs.openstack.org/cinder/train/configuration/block-storage/volume-drivers.html)
- [Các Backup Driver hỗ trợ](https://docs.openstack.org/cinder/train/configuration/block-storage/backup-drivers.html)