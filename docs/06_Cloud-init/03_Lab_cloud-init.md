# Lab sử dụng Cloud-init khi tạo Instance
- Cú pháp tạo instance khi truyền thêm cloud-init :
    ```
    # openstack server create --flavor <flavor> \
    --image <image> \
    --nic net-id=<net-id> \
    --security-group <security-group-id> \
    --user-data <file_cloud-config> \
    <instance_name>
    ```
- **VD1 :** Thay đổi password user mặc định khi đăng nhập :
    - **B1 :** Tạo file `cloud_config` :
        ```yaml
        #cloud-config
        password: Password123
        chpasswd: {expire: False}
        ssh_pwauth: True
        ```
    - **B2 :** Tạo instance với file `cloud_config` vừa tạo :
        ```
        # openstack server create --flavor Flavor A --image cirros --user-data cloud_config
        ```