# Các command thường sử dụng trong Glance
- Có thể thao tác với Glance CLI bằng 2 cách:
    - Glance CLI
    - Openstack Client CLI
## **1) Sử dụng Glance CLI**
### **1.1) List các image**
- Sử dụng lệnh sau :
    ```
    # glance image-list
    ```
    <img src="https://i.imgur.com/TSlY6oT.png">

### **1.2) Tạo image**
- Tạo mới một image theo cú pháp sau :
    ```
    # glance image-create [--architecture <ARCHITECTURE>]
                           [--protected [True|False]] [--name <NAME>]
                           [--instance-uuid <INSTANCE_UUID>]
                           [--min-disk <MIN_DISK>] [--visibility <VISIBILITY>]
                           [--kernel-id <KERNEL_ID>]
                           [--tags <TAGS> [<TAGS> ...]]
                           [--os-version <OS_VERSION>]
                           [--disk-format <DISK_FORMAT>]
                           [--os-distro <OS_DISTRO>] [--id <ID>]
                           [--owner <OWNER>] [--ramdisk-id <RAMDISK_ID>]
                           [--min-ram <MIN_RAM>]
                           [--container-format <CONTAINER_FORMAT>]
                           [--property <key=value>] [--file <FILE>]
                           [--progress]
    ```
- **VD :**
    ```
    # glance image-create --disk-format qcow2 --container-format bare --file cirros-0.5.1-x86_64-disk.img --name cirros-test
    ```
    <img src=https://i.imgur.com/45gyfph.png>

### **1.3) Show thông tin chi tiết của image**
- Để show thông tin chi tiết của một image, trước hết cần biết được ID của image đó .
- Cú pháp show thông tin image :
    ```
    # glance image-show --max-column-width <integer>] <IMAGE_ID>
    ```
- **VD :**
    ```
    # glance image-show 3e6ee696-06f2-4751-b183-a65b86b06736
    ```
    <img src=https://i.imgur.com/vYECCkF.png>



