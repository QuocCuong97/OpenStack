# Migration
## **1) Giới thiệu**
- **Migration** là quá trình di chuyển máy ảo từ host vật lí này sang một host vật lí khác. **Migration** được sinh ra để hỗ trợ việc bảo trì nâng cấp hệ thống. Ngày nay tính năng này đã được phát triển để thực hiện nhiều tác vụ hơn:
    - Cân bằng tải: Di chuyển VMs tới các host khác khi phát hiện host đang chạy có dấu hiệu quá tải.
    - Bảo trì, nâng cấp hệ thống: Di chuyển các VMs ra khỏi host trước khi tắt nó đi.
    - Khôi phục lại máy ảo khi host gặp lỗi: Restart máy ảo trên một host khác.
- Trong **OpenStack**, việc **migrate** được thực hiện giữa các node compute với nhau hoặc giữa các project trên cùng 1 node compute.
- **Openstack** cung cấp 1 số phương pháp khác nhau đề **migrate** VM. Các phương pháp này có các hạn chế khác nhau:
    - Không thể live-migrate với shared storage.
    - Không thể select target host nếu sử dụng nova migrate.
- **OpenStack** hỗ trợ hai kiểu **migration** đó là:
    - **Cold migration**: Non-live migration
    - **Live migration**:
        - ***True live migration*** (shared storage hoặc volume-based)
        - ***Block live migration***
## **2) So sánh các kiểu migrate**
### **2.1) Cold migrate**
- Ưu điểm :
    - Đơn giản, dễ thực hiện
    - Thực hiện được với mọi loại máy ảo
- Hạn chế:
    - Thời gian downtime lớn
    - Không thể chọn host muốn migrate đến
    - Quá trình migrate có thể mất một khoảng thời gian dài.
### **2.2) Live migrate**
- Ưu điểm:
    - Có thể chọn host muốn migrate đến
    - Tiết kiệm chi phí, tăng sự linh hoạt trong khâu quản lý và vận hành
    - Giảm thời gian downtime và gia tăng khả năng "cứu hộ" khi gặp sự cố.
- Nhược điểm:
    - Dù chọn được host nhưng vẫn có những giới hạn nhất định
    - Quá trình migrate có thể fails nếu host bạn chọn không có đủ tài nguyên.
    - Không được can thiệp vào bất cứ tiến trình nào trong quá trình live migrate.
    - Khó migrate với những máy ảo có dung lượng bộ nhớ lớn và trường hợp hai host khác CPU.
- Trong **live-migrate** chia thành 2 loại ***True live migration*** và ***Block live migration***. Dưới đây là mô tả một số các loại storage hỗ trợ hai loại **migration** này:
    
    | Loại | Local Storage | Volume | Shared Storage |
    |------|---------------|--------|----------------|
    | ***True LM*** |  | [x] | [x] |
    | ***Block LM*** | [x] |  |  |
    | ***True LM with read-only devices*** |  |  | [x] |
    | ***Block LM with read-only devices*** |  |  |  |

## **3) Migrate workflow**
- **Cold migrate**
    - Tắt máy ảo (giống với virsh destroy) và ngắt kết nối với volume
    - Di chuyển thư mục hiện tại của máy ảo (instance_dir -> instance_dir_resize)
    - Nếu sử dụng QCOW2 với backing files (chế độ mặc định) thì image sẽ được convert thành dạng flat
    - Với shared storage, di chuyển thư mục chứa máy ảo. Nếu không, copy toàn bộ thông qua SCP.
- Live migrate
    - Kiểm tra lại xem storage backend có phù hợp với loại migrate sử dụng không
        - Thực hiện check shared storage với chế độ migrate thông thường
        - Không check khi sử dụng block migrations
        - Việc kiểm tra thực hiện trên cả 2 node gửi và nhận, chúng được điều phối bởi RPC call từ scheduler.
    - Đối với nơi nhận
        - Tạo các kết nối càn thiết với volume.
        - Nếu dùng block migration, tạo thêm thư mục chứa máy ảo, truyền vào đó những backing files còn thiếu từ Glance và tạo disk trống.
    - Tại nơi gửi, bắt đầu quá trình migration (qua url)
    - Khi hoàn thành, generate lại file XML và define lại nó ở nơi chứa máy ảo mới.
## **4) Các bước migrate**
### **4.1) Cold migrate**
> Thực hiện migrate instance từ **compute1** (`10.10.230.11`) sang **compute2** (`10.10.230.12`)

> **Lưu ý:** sử dụng SSH tunnel giữa các compute để  migrate hoặc resize. Ở trường hợp này chỉ có 2 compute, đối với các trường hợp khác, các compute cần có liên kết SSH tunnel đến tất cả các node khác .

- **B1 :** Thực hiện trên 2 node **compute** :
    - Tạo usermod cho `nova` :
        ```
        # usermod -s /bin/bash nova
        ```
    - Tạo keypair cho user `nova` :
        ```
        # su nova
        # ssh-keygen
        # echo 'StrictHostKeyChecking no' >> /var/lib/nova/.ssh/config
        ```
- **B2 :** Thực hiện copy key :
    - Thực hiện copy public key của **compute1** (file `/var/lib/nova/.ssh/id_rsa.pub`) sang **compute2**  (tại file `/var/lib/nova/.ssh/authorized_keys`). Có thể dùng `scp` hoặc copy thủ công .
    - Thực hiện copy public key của **compute2** (file `/var/lib/nova/.ssh/id_rsa.pub`) sang **compute1**  (tại file `/var/lib/nova/.ssh/authorized_keys`). Có thể dùng `scp` hoặc copy thủ công .
- **B3 :** Kiểm tra SSH giữa 2 node :
    - Trên **compute1** :
        ```
        # su nova
        $ ssh compute2
        Warning: Permanently added 'compute2,10.10.230.12' (ECDSA) to the list of known hosts.
        Last login: Wed Aug 26 15:10:59 2020
        $ 
        ```
    - Trên **compute2** :
        ```
        # su nova
        $ ssh compute1
        Warning: Permanently added 'compute1,10.10.230.11' (ECDSA) to the list of known hosts.
        Last login: Wed Aug 26 15:10:09 2020
        $
        ```
- **B4 :** Khởi động lại dịch vụ trên cả 2 node **compute** :
    ```
    # systemctl restart libvirtd openstack-nova-compute
    ```
- **B5 :** Thực hiện **cold migrate** trên node **controller** :
    - Tắt instance nếu nó đang chạy :
        ```
        # openstack server stop VM03
        ```
    - Thực hiện migrate :
        ```
        # openstack server migrate VM03
        ```
    - Kiểm tra lại trạng thái của instance (lúc này sẽ là `RESIZE`) :
        ```
        # openstack server list | grep VM03
        ```
        <img src=https://i.imgur.com/B8Gi4Nj.png>
    - Xác thực việc resize (hay migrate) :
        ```
        # openstack server resize confirm VM03
        ```
    - Kiểm tra lại kết quả :
        ```
        # openstack server show VM03
        ```
        <img src=https://i.imgur.com/Ch7h3Pa.png>
        
        > `VM03` đã được migrate thành công từ **compute1** sang **compute2**
### **4.2) Live migrate**
- **OpenStack** hỗ trợ 2 loại live migrate:
    - ***True live migration*** (shared storage or volume-based) : Trong trường hợp này, máy ảo sẽ được di chuyển sửa dụng storage mà cả hai máy computes đều có thể truy cập tới. Nó yêu cầu máy ảo sử dụng block storage hoặc shared storage.
    - ***Block live migration*** (local storage): Mất một khoảng thời gian lâu hơn để hoàn tất quá trình migrate bởi máy ảo được chuyển từ host này sang host khác. Tuy nhiên nó lại không yêu cầu máy ảo sử dụng hệ thống lưu trữ tập trung.
- Các yêu cầu chung:
    - Cả hai node nguồn và đích đều phải được đặt trên cùng subnet và có cùng loại CPU.
    - Cả controller và compute đều phải phân giải được tên miền của nhau.
    - Compute node buộc phải sử dụng KVM với libvirt.
- Live-migration làm việc với 1 số loại VM và storage :
    - Shared storage: Cả 2 hypervisor có quyền truy cập shared storage chung.
    - Block storage: VM sử dụng root disk, không tương thích với các loại read-only devices.
    - Volume storage: VM sử dụng iSCSI volumes.
#### **4.2.1) Thực hiện ***Block live migrate*** với local storage**
- **B1 :** Thực hiện trên các node **compute** :
    - Chỉnh sửa cấu hình `libvirt` :
        ```
        # sed -i 's/#listen_tls = 0/listen_tls = 0/g' /etc/libvirt/libvirtd.conf
        # sed -i 's/#listen_tcp = 1/listen_tcp = 1/g' /etc/libvirt/libvirtd.conf
        # sed -i 's/#auth_tcp = "sasl"/auth_tcp = "none"/g' /etc/libvirt/libvirtd.conf
        # sed -i 's/#LIBVIRTD_ARGS="--listen"/LIBVIRTD_ARGS="--listen"/g' /etc/sysconfig/libvirtd
        ```
    - Sửa file `nova.conf` :
        ```
        # crudini --set /etc/nova/nova.conf libvirt block_migration_flag VIR_MIGRATE_UNDEFINE_SOURCE, VIR_MIGRATE_PEER2PEER, VIR_MIGRATE_LIVE, VIR_MIGRATE_NON_SHARED_INC
        ```
    - Khởi động lại dịch vụ :
        ```
        # systemctl restart libvirtd openstack-nova-compute
        ```
- **B2 :** Thực hiện migrate trên node controller :
    - Kiểm tra các node còn lại :
        ```
        # openstack compute service list --service nova-compute
        ```
        <img src=https://i.imgur.com/an9Nz9o.png>

    - Trên instance muốn **live-migrate**, đặt thử một lệnh `ping` :
        ```
        # ping 192.168.1.123
        ```
    - Thực hiện migrate `VM01` từ **compute1** qua **compute2** :
        - Đối với `shared storage` :
            ```
            # openstack server migrate 905d3d6a-8e5b-4a83-add8-88e6a24d13a3 --live-migration --host compute2
            ```
        - Đối với `local storage` :
            ```
            # nova live-migration --block-migrate VM01 compute2
            ```
    - Quá trình ping trên `VM01` vẫn bình thường (tuy nhiên có thể sẽ bị mất kết nối SSH hoặc VNC đến instance một chút xíu):
        
        <img src=https://i.imgur.com/SxgUO9J.png>

    - Kiểm tra lại kết quả :
        ```
        # openstack server show VM01
        ```
        <img src=https://i.imgur.com/N1PX6hw.png>

        > `VM01` đã được migrate thành công từ **compute1** sang **compute2**

-------------------------------
Tham khảo thêm :
- https://docs.openstack.org/nova/train/admin/configuring-migrations.html#securing-live-migration-streams
- https://docs.openstack.org/nova/train/admin/secure-live-migration-with-qemu-native-tls.html
- https://docs.openstack.org/nova/train/admin/migration.html
- https://docs.openstack.org/nova/train/admin/live-migration-usage.html