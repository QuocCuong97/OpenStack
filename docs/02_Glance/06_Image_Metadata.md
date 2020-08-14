# Image metadata trong Glance
## **1) Metadata trong Glance**
- Image metadata xác định bản chất của image và được sử dụng bởi các thành phần và drivers của **OpenStack** có thể tương tác với Image service.
- Có thể thêm các metadata tới các image của **Glance** bằng cách sử dụng tham số `--property key=value` trong câu lệnh `openstack image create` hoặc `openstack image set` :
    ```
    # openstack image set --property architecture=arm \
    --property hypervisor_type=qemu <image_id|image_name>
    ```
- Các thuộc tính của image được chỉ ra trong file `/etc/glance/schema-image.json` :
    ```
    {
        "kernel_id": {
            "type": ["null", "string"],
            "pattern": "^([0-9a-fA-F]){8}-([0-9a-fA-F]){4}-([0-9a-fA-F]){4}-([0-9a-fA-F]){4}-([0-9a-fA-F]){12}$",
            "description": "ID of image stored in Glance that should be used as the kernel when booting an AMI-style image."
        },
        "ramdisk_id": {
            "type": ["null", "string"],
            "pattern": "^([0-9a-fA-F]){8}-([0-9a-fA-F]){4}-([0-9a-fA-F]){4}-([0-9a-fA-F]){4}-([0-9a-fA-F]){12}$",
            "description": "ID of image stored in Glance that should be used as the ramdisk when booting an AMI-style image."
        },
        "instance_uuid": {
            "type": "string",
            "description": "Metadata which can be used to record which instance this image is associated with. (Informational only, does not create an instance snapshot.)"
        },
        "architecture": {
            "description": "Operating system architecture as specified in https://docs.openstack.org/python-glanceclient/latest/cli/property-keys.html",
            "type": "string"
        },
        "os_distro": {
            "description": "Common name of operating system distribution as specified in https://docs.openstack.org/python-glanceclient/latest/cli/property-keys.html",
            "type": "string"
        },
        "os_version": {
            "description": "Operating system version as specified by the distributor.",
            "type": "string"
        },
        "description": {
            "description": "A human-readable string describing this image.",
            "type": "string"
        },
        "cinder_encryption_key_id": {
            "description": "Identifier in the OpenStack Key Management Service for the encryption key for the Block Storage Service to use when mounting a volume created from this image",
            "type": "string"
        },
        "cinder_encryption_key_deletion_policy": {
            "description": "States the condition under which the Image Service will delete the object associated with the 'cinder_encryption_key_id' image property.  If this property is missing, the Image Service will take no action",
            "type": "string",
            "enum": [
                "on_image_deletion",
                "do_not_delete"
            ]
        }
    }
    ```
- Để xem các thuộc tính của một image sử dụng lệnh :
    ```
    # openstack image show <image_id|image_name>
    ```
    - **VD :**
        ```
        # openstack image show cirros-2
        ```
        <img src=https://i.imgur.com/MtO5NvN.png>

## **2) Một số thuộc tính hữu ích của image**
- Có thể set các thuộc tính của image cái mà có thể được sử dụng bởi các dịch vụ khác ảnh hưởng tới các hành động của service đó tới image :

    |**Specific to**| **Key**| **Description**| **Supported values**|
    |---|---|---|---|
    |All| **architecture**| CPU phải được hỗ trợ bởi hypervisor. Ví dụ: x86_64, arm, ppc64.<br> Sử dụng: `uname -m` để xem thông tin|alpha, armv7l, cris, i686, ia64, lm32 , m68k, microblaze, microblazeel, mips, mipsel, mips64, openrisc,...|
    |All| **hypervisor_type** | Hypervisor type. <br>Chú ý: **qemu** được sử dụng cho cả QEMU và KVM hypervisor types.|hyperv, ironic, lxc, qemu, uml, vmware, or xen|
    |ALL| **instance_type_rxtx_factor**| |Float (mặc định giá trị là 1.0)|
    |ALL| **instance_uuid** | Dành cho snapshot image, đây là UUID của server được sử dụng để tạo image này| Valid server UUID|
    |All| **img_config_drive**| Chỉ định nếu image cần cấu hình drive|mandatory or optional (mặc định nó sẽ không được sử dụng)|
    |All|**kernel_id**| ID của image được lưu trữ trên Glance, thường được sử dụng làm kernel khi boot một image kiể AMI| Valid image ID|
    |ALL| **os_distro**| Tên chung của operating system distribution|arch (Arch Linux.), centos, ubuntu, windows, debian, fedora, freebsd, gentoo, mandrake, mandriva, mes, msdos,...|
    |All| **os_version**| Phiên bản của distro| Valid version number (ví dụ: 16.04)|
    |All| **os_secure_boot**| Secure Boot là một tiêu chuẩn bảo mật. Khi máy ảo hoạt động, Secure Boot trước tiên sẽ kiểm tra các phần mềm như firmware và OS bằng chữ ký của chúng và chỉ cho phép chúng chạy khi chữ ký hợp lệ.| **required** - Enable the Secure Boot feature.<br>
    **disabled** or **optional** - (default) Disable the Secure Boot feature.|
    |libvirt API driver, XenAPI driver| **os_type**| Hệ điều hành được cài đặt trên image. <br>Libvirt API driver and XenAPI driver thực hiện các hành động khác nhau tùy thuộc vào giá trị của **os_type**.<br>Ví dụ image có **os_type=windows**, nó sẽ tạo một phân vùng FAT32-based swap thay vì Linux swap partion, và nó sẽ giới hạn tên server dưới 16 ký tự   | linux or windows.|

---------------------------
[Tham khảo thêm về các thuộc tính của image](https://docs.openstack.org/glance/latest/admin/useful-image-properties.html)