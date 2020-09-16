# Các command thường sử dụng trong Cinder
## **1) Volume**
#### **Tạo volume no-source**
- Cú pháp :
    ```
    # openstack volume create --size <volume_size_GB> <volume_name>
    ```
- **VD :** Tạo 1 volume `10GB` :
    ```
    # openstack volume create --size 10 volume_1
    ```
    <img src=https://i.imgur.com/hvNQLLn.png>

#### **Tạo volume từ image**
- Cú pháp :
    ```
    # openstack volume create --size <volume_size_GB> --image <image_name|image_ID> <volume_name>
    ```
- **VD :**
    ```
    # openstack volume create --size 10 --image centos7 centos7-volume
    ```
    <img src=https://i.imgur.com/yF2sCwS.png>

#### **Tạo volume từ một volume khác**
- Cú pháp :
    ```
    # openstack volume create --source <source_volume> --size <volume_size> <volume_name>
    ```
    > **Lưu ý:** Volume mới tạo ra phải có dung lượng &ge; volume gốc
- **VD :**
    ```
    # openstack volume create --source centos7-volume --size 12 centos7-volume2
    ```
    <img src=https://i.imgur.com/UaALxS1.png>
#### **Tạo volume từ một bản snapshot**
- Cú pháp :
    ```
    # openstack volume create --snapshot <source_snapshot> --size <volume_size> <volume_name>
    ```
#### **List các volume**
- Cú pháp :
    ```
    # openstack volume list
    ```
    <img src=https://i.imgur.com/5OtUd97.png>
#### **Show thông tin volume**
- Cú pháp :
    ```
    # openstack volume show <volume_name|volume_ID>
    ```
- **VD :**
    ```
    # openstack volume show centos7-volume --fit-width
    ```
    > Option `--fit-width` giúp hiển thị kết quả vừa chiều rộng với terminal

    <img src=https://i.imgur.com/ENag9Qm.png>

#### **Xóa volume**
- Cú pháp :
    ```
    # openstack volume delete <volume_name|volume_ID>
    ```
#### **Attach volume**
- Cú pháp :
    ```
    # openstack server add volume <instance_name|instance_ID> <volume_name|volume_ID> --device <device_name_on_instance>
    ```
    > `device_name_on_instance` : tên phân vùng trên instance. **VD :** `/dev/vdb`, `dev/vdc`,... Nếu không thêm option này, OS sẽ tự quyết định device cho volume
- **VD :**
    ```
    # openstack server add volume VM03 centos7-volume --device /dev/vdb
    ```
    <img src=https://i.imgur.com/qynMsq2.png>

#### **Detach volume**
- Cú pháp :
    ```
    # openstack server remove volume <instance_name|instance_ID> <volume_name|volume_ID>
    ```
- **VD :**
    ```
    # openstack server remove volume VM03 centos7-volume
    ```
#### **Resize volume**
- Volume muốn resize cần được ở trạng thái `available` (detach khỏi instance)
- Kích cỡ resize phải lớn hơn kích gỡ gốc của volume
- Cú pháp :
    ```
    # openstack volume set <volume_name|volume_ID> --size <volume_size_GB>
    ```
- Resize volume đang attach (`in-use`):
    - Với volume bootable (không detach được): Không thực hiện được, nhưng khi chuyển trạng thái về `available` thì có thể resize (thêm dung lượng), thực hiện với volume LVM, sau khi resize, volume sẽ tự động chuyển về trạng thái `In-use`. Khi kiểm tra bằng lệnh `lsblk` thì disk đã thay đổi kích thước, nhưng trên filesystem thực chất là chưa. Do thay đổi kích thước volume chứ không phải partition, vậy nên sau khi extent thì cần vào server để mount lại.
    - Với volume non bootable: Tương tự với volume bootable nhưng có thể detach khỏi instance.
- Sau khi resize volume cần reboot lại instance .
- **VD :**
    ```
    # openstack volume set centos7-volume --size 12
    ```
## **2) Volume type**
#### **List các volume type**
- Cú pháp :
    ```
    # openstack volume type list
    ```
    <img src=https://i.imgur.com/qeUugCx.png>

#### **Tạo mới volume type**
- Cú pháp :
    ```
    # openstack volume type create <volume_type_name>
    # openstack volume type set <volume_type_name> --property volume_backend_name=<volume_backend_name>
    ```
    > Trong đó `<volume_backend_name>` là option `volume_backend_name` khi cấu hình backend trong file `/etc/cinder/cinder.conf`
- **VD :** Tạo volume type `lvm1` :
    - Cấu hình `volume_backend_name` trong file `cinder.conf` :
        <img src=https://i.imgur.com/JHfAS9A.png>

    - Tạo volume type `lvm1` :
        ```
        # openstack volume type create lvm1
        ```
        <img src=https://i.imgur.com/EGVmp0q.png>

    - Gán backend cho volume type vừa tạo :
        ```
        # openstack volume type set lvm1 --property volume_backend_name=lvm1
        ```
        > `volume_type_name` và `volume_backend_name` không cần thiết phải đặt giống nhau 
    - Kiểm tra lại list các volume type :
        ```
        # openstack volume type list
        ```
        <img src=https://i.imgur.com/qeUugCx.png>

    - Tạo mới volume với volume type `lvm1` :
        ```
        # openstack volume create --size 10 --type lvm1 test
        ```
        <img src=https://i.imgur.com/ZAwCUYg.png>

#### **Xóa volume type**
- Cú pháp :
    ```
    # openstack volume type delete <volume_type_name>
    ```
> ### **Chú ý :**
- Mặc định khi tạo volume backend đầu tiên mà không cấu hình volume type thì **cinder** sẽ set volume type của nó là `__DEFAULT__`. Các volume được tạo ra nếu không chỉ định `--type` thì sẽ sử dụng backend này .
- Để set chính xác volume type mặc định cho **cinder**, chỉnh sửa `default_volume_type` trong file cấu hình `/etc/cinder/cinder.conf` :
    <img src=https://i.imgur.com/nE4UPTD.png>
    - Sau khi thay đổi cần restart `cinder-api` :
        ```
        # systemctl restart openstack-cinder-api
        ```
    - Volume tiếp theo khi tạo ra nếu không set `--type` thì sẽ sử dụng mặc định là `nfs`
## **3) Snapshot volume**
#### **Tạo volume snapshot**
- Cú pháp :
    ```
    # openstack volume snapshot create --volume <volume_name|volume_ID> <snapshot_name> --force
    ```
    > Option `--force` cho phép tạo snapshot trong khi volume đang attach vào instance. Nếu không dùng, volume bắt buộc phải ở trạng thái `available`
- **VD :**
    ```
    # openstack volume snapshot create --volume centos7-volume2 snap_cent7
    ```
    <img src=https://i.imgur.com/tNZ19K5.png>

#### **List các snapshot volume**
- Cú pháp :
    ```
    # openstack volume snapshot list
    ```
    <img src=https://i.imgur.com/7p0RcuU.png>

#### **Xóa snapshot volume**
- Cú pháp :
    ```
    # openstack volume snapshot delete <snapshot_name>
    ```
---------------------------
Tham khảo thêm
- https://docs.openstack.org/cinder/train/cli/cli-manage-volumes.html
- https://docs.openstack.org/python-openstackclient/latest/cli/command-objects/volume-snapshot.html