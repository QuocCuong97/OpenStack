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

### **1.4) Upload image**
- Cú pháp :
    ```
    # glance image-upload [--file <FILE>] [--size <IMAGE_SIZE>] [--progress] <IMAGE_ID>
    ```
- **VD :**
    - Để upload image, trước hết cần tạo một image rỗng :
        ```
        # glance image-create --name cirros-upload --container-format bare --disk-format qcow2
        ```

        <img src=https://i.imgur.com/rAgmCHo.png width=75%>

        > Copy lại `image-id` vừa tạo ở trên
    - Upload image :
        ```
        # glance image-upload --file cirros-0.5.1-x86_64-disk.img --progress cab13784-5485-4f57-9456-61fe1e449274
        ```
        <img src=https://i.imgur.com/7voVhyy.png>

### **1.5) Thay đổi trạng thái image**
#### **1.5.1) Deactivate image**
- Để deactivate một image, trước hết cần biết được ID của image đó .
- Cú pháp :
    ```
    # glance image-deactivate <image_id>
    ```
- **VD :**
    ```
    # glance image-deactivate cab13784-5485-4f57-9456-61fe1e449274
    ```
    - Kiểm tra lại các image đang có :

        <img src=https://i.imgur.com/72x85q7.png>

#### **1.5.2) Activate image**
- Để activate một image, trước hết cần biết được ID của image đó .
- Cú pháp :
    ```
    # glance image-reactivate <image_id>
    ```
- **VD :**
    ```
    # glance image-reactivate cab13784-5485-4f57-9456-61fe1e449274
    ```
    - Kiểm tra lại các image đang có :

        <img src=https://i.imgur.com/TDx0ILR.png>

### **1.6) Xóa image**
- Để xóa một image, trước hết cần biết được ID của image đó .
- Cú pháp :
    ```
    # glance image-delete <image_id>|[<image_id_1> <image_id_2>...]
    ```
- **VD :**
    ```
    # glance image-delete cab13784-5485-4f57-9456-61fe1e449274
    ```
## **2) Sử dụng Openstack CLI**
### **2.1) List các image**
- Cú pháp :
    ```
    # openstack image list
    ```
    > Có thể dùng kết hợp với `grep` để lọc kết quả
- **VD :**
    ```
    # openstack image list | grep cirros
    ```
    <img src=https://i.imgur.com/iwCBB5U.png>

### **2.2) Show thông tin chi tiết của image**
- Để show thông tin chi tiết của một image, trước hết cần biết được ID hoặc tên của image đó .
- Cú pháp :
    ```
    # openstack image show [--human-readable] <ID | image_name>
    ```
    - `--human-readable` : định dạng kích thước của image dễ đọc
- **VD :**
    ```
    # openstack image show cirros-upload
    ```
    <img src=https://i.imgur.com/PxHpQvg.png>

### **2.3) Tạo image**
- Cú pháp :
    ```
    # openstack image create
        [--id <id>]
        [--container-format <container-format>]
        [--disk-format <disk-format>]
        [--min-disk <disk-gb>]
        [--min-ram <ram-mb>]
        [--file <file> | --volume <volume>]
        [--force]
        [--sign-key-path <sign-key-path>]
        [--sign-cert-id <sign-cert-id>]
        [--protected | --unprotected]
        [--public | --private | --community | --shared]
        [--property <key=value>]
        [--tag <tag>]
        [--project <project>]
        [--import]
        [--project-domain <project-domain>]
        <image-name>
    ```
- **VD :** 
    ```
    # openstack image create "cirros-2" --file cirros-0.5.1-x86_64-disk.img --disk-format qcow2 --container-format bare --public
    ```
    <img src=https://i.imgur.com/kNyFURE.png>

### 2.4) Xóa image**
- Để xóa một image, trước hết cần biết được ID hoặc tên của image đó .
- Cú pháp :
    ```
    # openstack image delete <image_id | image_name>
    ```
### **2.5) Update thông tin image**
- Để update một image, trước hết cần biết được ID hoặc tên của image đó .
- Cú pháp :
    ```
    # openstack image set
        [--name <name>]
        [--min-disk <disk-gb>]
        [--min-ram <ram-mb>]
        [--container-format <container-format>]
        [--disk-format <disk-format>]
        [--protected | --unprotected]
        [--public | --private | --community | --shared]
        [--property <key=value>]
        [--tag <tag>]
        [--architecture <architecture>]
        [--instance-id <instance-id>]
        [--kernel-id <kernel-id>]
        [--os-distro <os-distro>]
        [--os-version <os-version>]
        [--ramdisk-id <ramdisk-id>]
        [--deactivate | --activate]
        [--project <project>]
        [--project-domain <project-domain>]
        [--accept | --reject | --pending]
        <image>
    ```
### **2.6) Share image cho các project khác**
- Cú pháp :
    ```
    # openstack image add project [--project-domain <project-domain>] <image> <project>
    ```
    - Trong đó :
        - `--project-domain <project-domain>` : domain (tên hoặc ID)
        - `<image>` : image ở trạng thái shared (tên hoặc ID)
        - `<project>` : project (tên hoặc ID)
- Để thêm project cho image, thì image phải ở trạng thái `shared`
- **VD :**
    - Update trạng thái `shared` cho image :
        ```
        # openstack image set cirros-2 --shared
        ```
    - Add project `demo` cho image `cirros-2` :
        ```
        # openstack image add project cirros-2 demo
        ```
        <img src=https://i.imgur.com/AS2AXbO.png>

### **2.7) Xóa project gán cho image**
- Cú pháp :
    ```
    # openstack image remove project [--project-domain <project-domain>] <image> <project>
    ```
### **2.8) Kiểm tra các project đang gán cho image**
- Cú pháp :
    ```
    # openstack image member list [--sort-column SORT_COLUMN] [--project-domain <project-domain>] <image>
    ```
- **VD :**
    ```
    # openstack image member list cirros-2
    ```
    <img src=https://i.imgur.com/OLciiQb.png>
