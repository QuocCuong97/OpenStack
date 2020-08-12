# Policy trong Keystone
## **1) Role-based Access Control (RBAC)**
- **Role-Based Access Control - RBAC** (*điều khiển truy cập trên cơ sở vai trò*) là phương pháp điều khiển và đảm bảo quyền sử dụng cho người dùng.
- Đây là phương pháp thay thế cho **Discretionary Access Control - DAC** (*điều khiển truy cập tùy quyền*) và **Mandatory Access Control - MAC** (*điều khiển truy cập bắt buộc*).
- Các vai trò (**roles**) được tạo để đảm nhận các chức năng công việc khác nhau. Mỗi **role** được gắn liền với một số quyền hạn cho phép nó thao tác một số hoạt động cụ thể ('permissions').
- Các user được gán một role riêng, và thông qua việc gán role này mà user tiếp nhận được một số những quyền hạn cho phép họ được thực thi những chức năng cụ thể trong hệ thống.
- Vì user không được cấp phép một cách trực tiếp mà được tiếp nhận các quyền hạn thông qua role của user, việc quản lý quyền của user trở thành một việc đơn giản, và người quản trị chỉ cần chỉ định những role thích hợp cho user.
- Việc chỉ định role đơn giản hóa những công việc thông thường như việc cho thêm một người dùng vào trong hệ thống, hay tập quyền của người dùng.
## **2) File `policy.json`**
- Mỗi service trong **OpenStack** như identity, networking, compute... đều có một cơ chế quản lý quyền **role-based access** policies riêng của chúng. Các **role-based** này được chúng sử dụng để xác định một người dùng nào có thể truy cập vào một object nào trong chúng, và các **policy** này được định nghĩa tại file `policy.json`
- Khi user gọi tới API, các **policy** trên serivice sẽ được sử dụng để kiểm tra và chấp nhận các request này. Mỗi khi cập nhật một rule trên file `policy.json` thì sẽ có tác dụng ngay lập tức trong khi service đang chạy .
- Một file `policy.json` được định dạng dưới dạng `JSON`. Mỗi **policy** sẽ được định nghĩa theo format sau :
    ```json
    "<target>" : "<rule>"
    ```
    - Trong đó :
        - `target` sẽ là một action, đại diện cho một API có thể là `os_compute_api:servers:create`. Các action name này sẽ tùy theo từng loại dịch vụ sẽ có các đặc tính đi kèm, và được đặt tại file "`/etc/{service_name}/policy.json`" để các service được hiểu đây là các action cần được kiểm tra.
        - `rule` sẽ thường được sử dụng để xác định một **role** hay user nào được làm việc hay không làm việc với API target. `Rule` khi để là `""` sẽ có gía trị với tất cả các role
        > Danh sách các target có thể gán cho `rule` tại [đây](https://docs.openstack.org/keystone/train/getting-started/policy_mapping.html) hoặc tại [đây](https://docs.openstack.org/keystone/train/configuration/policy.html)
- Mặc định khi cài đặt **Keystone** sẽ có 3 rule `admin`, `member`, `reader`
- Từ bản **OpenStack `Train`**, các thiết lập mặc định ban đầu cho các role này sẽ không xuất hiện trong file `policy.json`
### **Cách sử dụng file `policy.json`**
- Để sử dụng file `policy.json`, cần sửa file cấu hình của **Keystone** :
    ```
    # vi /etc/keystone/keystone.conf
    ```
    - Bỏ dấu `#` ở đầu dòng `1889` :

        <img src=https://i.imgur.com/fTEPStK.png>

- Chỉnh sửa file `policy.json` :
    - Mặc định file `policy.json` sẽ là file trống với nội dung "`{}`"
    - Thêm vào bên trong cặp dấu "`{}`" các block sau :
        - Block định nghĩa các rule mới :
            ```json
            "<rule>": "define rule",
            ```
        - Block khai báo các policy :
            ```json
            "<target>": "<rule> or <role>"
            ```
- Thêm 1 policy mới :
    ```json
    { 
        "identity:list_users" : "role: user",
    }
    ```
    - Sau khi thêm policy này, API list_users hoặc CLI `openstack user list` chỉ sử dụng được khi user có role là `user`. Các role khác sẽ không sử dụng được chúng .
        ```
        [root@controller ~]# openstack user list
        You are not authorized to perform the requested action: identity:list_users. (HTTP 403) (Request-ID: req-f23af1b3-f547-4e5f-b970-063b235a0c12)
        ```
## **3) File `policy.yaml`**
- Là một file policy chứa các thông tin rule mặc định ban đầu
- Để gen ra file `policy.yaml`, thực hiện lệnh sau :
    ```
    # oslopolicy-policy-generator --namespace keystone --output-file /etc/keystone/policy.yaml
    ```
    > Trước khi thực hiện lệnh, chú ý xóa hết các role bên trong file `policy.json` (nếu có)
- Kiểm tra lại file `policy.yaml` :
    ```yaml
    "identity:delete_project": "(role:admin and system_scope:all) or (role:admin and domain_id:%(target.project.domain_id)s)"
    "identity:list_revoke_events": "rule:service_or_admin"
    "identity:revoke_system_grant_for_group": "role:admin and system_scope:all"
    "identity:get_region": ""
    "identity:list_system_grants_for_group": "role:reader and system_scope:all"
    "identity:create_implied_role": "role:admin and system_scope:all"
    "identity:list_endpoint_groups": "role:reader and system_scope:all"
    "identity:list_endpoints_associated_with_endpoint_group": "role:reader and system_scope:all"
    "identity:get_auth_catalog": ""
    "identity:list_access_tokens": "rule:admin_required"
    "identity:list_trusts": "role:reader and system_scope:all"
    "identity:update_registered_limit": "role:admin and system_scope:all"
    "identity:delete_implied_role": "role:admin and system_scope:all"
    "identity:get_domain_config_default": "role:reader and system_scope:all"
    "identity:update_service": "role:admin and system_scope:all"
    "identity:create_domain_config": "role:admin and system_scope:all"
    "identity:delete_service": "role:admin and system_scope:all"
    "identity:update_mapping": "role:admin and system_scope:all"
    "identity:update_limit": "role:admin and system_scope:all"
    "identity:list_domains": "role:reader and system_scope:all"
    "identity:list_policies": "role:reader and system_scope:all"
    "identity:update_endpoint_group": "role:admin and system_scope:all"
    "service_role": "role:service"
    ............
    ```
- Để sử dụng file `policy.yaml` như định dạng file chính để gán các policy, chỉnh sửa file `/etc/keystone/keystone.conf` :
    - Thêm đoạn sau vào cuối file :
        ```yaml
        [oslo_policy]
        policy_file = policy.yaml
        ```
    - Command lại dòng `1889` :
        ```
        #policy_file = policy.json
        ```
    - Restart dịch vụ `httpd` :
        ```
        # systemctl restart httpd
        ```
- **VD :** Thêm policy sau vào cuối file `policy.yaml` :
    ```
    "identity:create_user": "role:user"
    ```
    > Sau khi thêm policy này, API create_user hoặc CLI `openstack user create` chỉ sử dụng được khi user có role là `user`. Các role khác sẽ không sử dụng được chúng 

