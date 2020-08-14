# Image metadata trong Glance
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
        