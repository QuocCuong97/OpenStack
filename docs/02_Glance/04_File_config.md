# File cấu hình của Glance
## **1) Giới thiệu**
- **Glance** có một số tùy chọn mà bạn có thể sử dụng để cấu hình **Glance API server**, **Glance Registry server** và nhiều storage backends lưu trữ image cho **Glance**.
- Chủ yếu các cấu hình đều được để trong file, **Glance API server** và **Glance Registry server** sử dụng hai file cấu hình khác nhau.
- Khi **Glance server** khởi động, có thể chỉ định file cấu hình được sử dụng. Nếu bạn không chỉ định cụ thể, **Glance** sẽ xem xét các thư mục sau để lấy file cấu hình:
    - `~/.glance`
    - `~/`
    - `/etc/glance`
    - `/etc`
- Mặc định, thư mục chứa các file cấu hình của **Glance** là `/etc/glance`
- File cấu hình được tổ chức theo dạng ***Section***, các tùy chọn theo dạng `key = value`
- **Glance** gồm các file cấu hình cơ bản sau đây:
    - `glance-api.conf` : file cấu hình cho API của image service.
    - `glance-registry.conf` : file cấu hình cho glance image registry - nơi lưu trữ metadata về các images.
    - `glance-scrubber.conf` : được dùng để dọn dẹp các image đã được xóa
    - `policy.json` : bổ sung policy áp dụng cho các image service. Trong này, chúng tra có thể xác định vai trò, chính sách, làm tăng tính bảo mật trong **Glance OpenStack**.
## **2) File `glance-api.conf`**
- Đường dẫn file : `/etc/glance/glance-api.conf`

    <img src=https://i.imgur.com/IgRNETk.png>

- Các Section trong file :
    - `[database]` : cấu hình kết nối tới database
        ```
        [database]
        connection = mysql+pymysql://glance:Password123@10.10.230.10/glance
        ```
        - Trong đó:
            - `glance` : user truy cập database của **glance**
            - `Password123` : mật khẩu của user `glance`
            - `10.10.230.10` : IP node controller
    - `[keystone_authtoken]` và `[paste_deploy]` : cấu hình kết nối tới identity service tại section với các thông tin cần thiết
        ```
        [keystone_authtoken]
        www_authenticate_uri = http://10.10.230.10:5000
        auth_url = http://10.10.230.10:5000
        memcached_servers = 10.10.230.10:11211
        auth_type = password
        project_domain_name = Default
        user_domain_name = Default
        project_name = service
        username = glance
        password = Password123

        [paste_deploy]
        flavor = keystone
        ```
    - `[glance_store]` : cấu hình kiểu lưu trữ image
        ```
        [glance_store]
        stores = file,http
        default_store = file
        filesystem_store_datadir = /var/lib/glance/images/
        ```
        - Ở đây có hai kiểu lưu trữ được sử dụng và mặc định sẽ sử dụng file system. Có thể cấu hình để lưu trữ trên nhiều nơi khác nhau theo cú pháp như sau:
            ```
            filesystem_store_datadirs = PATH:PRIORITY
            ```
            - Trong đó: `PATH` là đường dẫn tới thư mục chứa image, `PRIORITY` là mức độ ưu tiên.
            - **VD :**
                ```
                filesystem_store_datadirs = /var/glance/store
                filesystem_store_datadirs = /var/glance/store1:100
                filesystem_store_datadirs = /var/glance/store2:200
                ```
## **3) Thư mục lưu trữ image mặc định**
- Kiểm tra thư mục lưu các image:
    ```
    # cat /etc/glance/glance-api.conf | grep "filesystem_store_datadir" | egrep -v "^#|^$"
    filesystem_store_datadir = /var/lib/glance/images/
    ```
- Kiểm tra các image đang có :
    ```
    # ls -lh /var/lib/glance/images/
    ```
    <img src=https://i.imgur.com/c6vgiVH.png>
- Kiểm tra các image đang có qua OpenStack CLI :
    ```
    # openstack image list
    ```
    <img src=https://i.imgur.com/h5sIbAB.png>
