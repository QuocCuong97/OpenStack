# APIs trong Keystone
## **1) Khái niệm API**
- **API - *Application Programming Interface*** : là một giao tiếp phần mềm được dùng bởi các ứng dụng khác nhau. Cũng giống như bàn phím là thiết bị giao tiếp giữa người dùng và máy tính, thì **API** là giao tiếp phần mềm, ví dụ như giữa chương trình và hệ điều hành (OS).
- **API** cung cấp khả năng truy xuất đến một tập các hàm hay dùng. Và từ đó có thể trao đổi dữ liệu giữa các ứng dụng.
## **2) Sử dụng `cURL` để gọi APIs**
- **Method** sử dụng : `POST`
- **Options :** 
    - `-i` : output sẽ chứa cả HTTP-Header (không sử dụng nếu muốn hiển thị kết quả theo định dạng `json`)
    - `-H` : kết hợp với `Content-Type` để xác định kiểu dữ liệu truyền vào header. Ở đây sẽ là dùng `json`
    - `-d` : dữ liệu truyền vào request body
    - `-s` : không hiển thị trình processing ra output
### **2.1) Lấy token**
- **VD :**
    ```
    # curl -s -i -X POST \
    -H "Content-Type: application/json" \
    -d \
    '{ "auth": {
        "identity": {
            "methods": ["password"],
            "password": {
                  "user": {
                  "name": "admin",
                  "domain": { "name": "Default" },
                  "password": "Password123"
                }
              }
            },
            "scope": {
            "project": {
                "name": "admin",
                "domain": { "name": "Default" }
            }
        }
      }
    }' \
    http://10.5.11.210:5000/v3/auth/tokens | grep 'X-Subject-Token'
    ```
    > Trong đó: `10.5.11.210` là IP Provider của host
    => Output :
        ```
        X-Subject-Token: gAAAAABfMh0FxvbC_EFiXVZI8qQKxwOWLgoueZPiBqdwf9-v6BdcvP_7hCfRD7umSRc7Ee_7_sfkiSFp_fKz0emi9g_1ZiESGhO58OoB3xAImsmvhq2vECTfS0LVZqAnzxsqNOIWT2tyxge3ySY5Ko1V_TZQmlfWqXD9Nqm_KTt8CSfaLgFf6Jk
        ```
- Sau khi lấy token, có thể import token vào biến môi trường để sử dụng cho tiện :
    ```
    # export OS_TOKEN=gAAAAABfMh0FxvbC_EFiXVZI8qQKxwOWLgoueZPiBqdwf9-v6BdcvP_7hCfRD7umSRc7Ee_7_sfkiSFp_fKz0emi9g_1ZiESGhO58OoB3xAImsmvhq2vECTfS0LVZqAnzxsqNOIWT2tyxge3ySY5Ko1V_TZQmlfWqXD9Nqm_KTt8CSfaLgFf6Jk
    ```
    > Chú ý : Token chỉ có thời hạn `3600s`
- Sau khi import token vào biến môi trường, cú pháp curl API sẽ là :
    ```
    # curl -X GET -s -H "X-Auth-Token: $OS_TOKEN" <endpoint> | json_pp
    ```
    - Trong đó :
        - `endpoint` : URL tới API
        - `json_pp` : lệnh pipeline giúp hiển thị dữ liệu dạng json. Có thể thay bằng `python3 -m json.tool` cũng cho kết quả tương tự
### **2.2) Lấy thông tin user**
- Command :
    ```
    # openstack user list
    ```
    => Output :
    ```
    +----------------------------------+-----------+
    | ID                               | Name      |
    +----------------------------------+-----------+
    | 30b0df02569640debd4b02049f21199d | admin     |
    | e5cded5945194fc18dffb9b3d00f4b71 | demo      |
    | dacd6f2bcd2842789c8833470f9705bb | glance    |
    | 7b3c017d23554c8d8c64759066bf6eb3 | placement |
    | a2b126e46f124f318bcc984394c97721 | nova      |
    | ee160a4fce5e4e4eb7a88eb713f003b2 | neutron   |
    | a6ad9a8fb445474ab9a8332ffd4ba2c1 | cuongnqc  |
    +----------------------------------+-----------+
    ```
- Curl API :
    ```
    # curl -X GET -s -H "X-Auth-Token: $OS_TOKEN" http://10.5.11.210:5000/v3/users | json_pp
    ```
    => Output :
    ```
    {
        "links" : {
            "next" : null,
            "previous" : null,
            "self" : "http://10.5.11.210:5000/v3/users"
        },
        "users" : [
            {
                "domain_id" : "default",
                "enabled" : true,
                "id" : "30b0df02569640debd4b02049f21199d",
                "links" : {
                    "self" : "http://10.5.11.210:5000/v3/users/30b0df02569640debd4b02049f21199d"
                },
                "name" : "admin",
                "options" : {},
                "password_expires_at" : null
            },
            {
                "domain_id" : "default",
                "enabled" : true,
                "id" : "e5cded5945194fc18dffb9b3d00f4b71",
                "links" : {
                    "self" : "http://10.5.11.210:5000/v3/users/e5cded5945194fc18dffb9b3d00f4b71"
                },
                "name" : "demo",
                "options" : {},
                "password_expires_at" : null
            },
            {
                "domain_id" : "default",
                "enabled" : true,
                "id" : "dacd6f2bcd2842789c8833470f9705bb",
                "links" : {
                    "self" : "http://10.5.11.210:5000/v3/users/dacd6f2bcd2842789c8833470f9705bb"
                },
                "name" : "glance",
                "options" : {},
                "password_expires_at" : null
            },
            {
                "domain_id" : "default",
                "enabled" : true,
                "id" : "7b3c017d23554c8d8c64759066bf6eb3",
                "links" : {
                    "self" : "http://10.5.11.210:5000/v3/users/7b3c017d23554c8d8c64759066bf6eb3"
                },
                "name" : "placement",
                "options" : {},
                "password_expires_at" : null
            },
            {
                "domain_id" : "default",
                "enabled" : true,
                "id" : "a2b126e46f124f318bcc984394c97721",
                "links" : {
                    "self" : "http://10.5.11.210:5000/v3/users/a2b126e46f124f318bcc984394c97721"
                },
                "name" : "nova",
                "options" : {},
                "password_expires_at" : null
            },
            {
                "domain_id" : "default",
                "enabled" : true,
                "id" : "ee160a4fce5e4e4eb7a88eb713f003b2",
                "links" : {
                    "self" : "http://10.5.11.210:5000/v3/users/ee160a4fce5e4e4eb7a88eb713f003b2"
                },
                "name" : "neutron",
                "options" : {},
                "password_expires_at" : null
            },
            {
                "domain_id" : "default",
                "enabled" : true,
                "id" : "a6ad9a8fb445474ab9a8332ffd4ba2c1",
                "links" : {
                    "self" : "http://10.5.11.210:5000/v3/users/a6ad9a8fb445474ab9a8332ffd4ba2c1"
                },
                "name" : "cuongnqc",
                "options" : {},
                "password_expires_at" : null
            }
        ]
    }
    ```
### **2.3) Lấy list domain**
- Curl command
    ```
    # curl -X GET -s -H "X-Auth-Token: $OS_TOKEN" http://10.5.11.210:5000/v3/users | json_pp
    ```
### **2.4) Lấy list project**
- Curl command
    ```
    # curl -X GET -s -H "X-Auth-Token: $OS_TOKEN" http://10.5.11.210:5000/v3/projects | json_pp
    ```
### **2.5) Lấy list role***
- Curl command
    ```
    # curl -X GET -s -H "X-Auth-Token: $OS_TOKEN" http://10.5.11.210:5000/v3/roles | json_pp
    ```
### **2.6) Lấy list group**
- Curl command
    ```
    # curl -X GET -s -H "X-Auth-Token: $OS_TOKEN" http://10.5.11.210:5000/v3/groups | json_pp
    ```
### **2.7) Tạo user mới**
- Curl command :
    ```
    # curl -s -X POST \
    -H "X-Auth-Token: $OS_TOKEN" \
    -H "Content-Type: application/json" \
    -d \
    '{
        "user": {
            "default_project_id": "admin",
            "domain_id": "default",
            "enabled": true,
            "name": "trangptt",
            "password": "Password123",
            "description": "Phung Thi Thuy Trang",
            "email": "trangptt@gmail.com",
            "options": {
                "ignore_password_expiry": true
            }
        }
    }' \
    http://10.5.11.210:5000/v3/users | json_pp
    ```
    => Output :
    ```
    {
        "user" : {
            "default_project_id" : "admin",
            "description" : "Phung Thi Thuy Trang",
            "domain_id" : "default",
            "email" : "trangptt@gmail.com",
            "enabled" : true,
            "id" : "336c9e5f199b4919a788cec1f47893aa",
            "links" : {
                "self" : "http://10.5.11.210:5000/v3/users/336c9e5f199b4919a788cec1f47893aa"
            },
            "name" : "trangptt",
            "options" : {
                "ignore_password_expiry" : true
            },
            "password_expires_at" : null
        }
    }
    ```
    => Kiểm tra lại list user :
    ```
    # openstack user list
    +----------------------------------+-----------+
    | ID                               | Name      |
    +----------------------------------+-----------+
    | 30b0df02569640debd4b02049f21199d | admin     |
    | e5cded5945194fc18dffb9b3d00f4b71 | demo      |
    | dacd6f2bcd2842789c8833470f9705bb | glance    |
    | 7b3c017d23554c8d8c64759066bf6eb3 | placement |
    | a2b126e46f124f318bcc984394c97721 | nova      |
    | ee160a4fce5e4e4eb7a88eb713f003b2 | neutron   |
    | a6ad9a8fb445474ab9a8332ffd4ba2c1 | cuongnqc  |
    | 336c9e5f199b4919a788cec1f47893aa | trangptt  |
    +----------------------------------+-----------+
    ```
    => User được tạo thành công!

------------------------------
[Tham khảo thêm về APIs Keystone](https://docs.openstack.org/api-ref/identity/v3/index.html)




