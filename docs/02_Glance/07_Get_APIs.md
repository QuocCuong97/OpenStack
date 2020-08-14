# APIs trong Glance
## **Sử dụng `cURL` để gọi APIs**
- **Method** sử dụng : `POST`
- **Options :** 
    - `-i` : output sẽ chứa cả HTTP-Header (không sử dụng nếu muốn hiển thị kết quả theo định dạng `json`)
    - `-H` : kết hợp với `Content-Type` để xác định kiểu dữ liệu truyền vào header. Ở đây sẽ là dùng `json`
    - `-d` : dữ liệu truyền vào request body
    - `-s` : không hiển thị trình processing ra output
### **1) Lấy token**
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
    X-Subject-Token: gAAAAABfNgp_NYBVvO-A9GAuATxr_JIQiAyUBGOEq4LEz76HDBMwEAP8aS1OQuVakLwXjmc_wuVvUt5VFdic81WFrD5-Pnc6eesWk-k0RaRzgDS2izVbv-u_VovgJX2WrkZRfWRCjgfnlJ5ZeQa9ybPPGizfCUFp5Wlvh_7jNLaZI3ZH4c8pvMs
    ```
- Sau khi lấy token, có thể import token vào biến môi trường để sử dụng cho tiện :
    ```
    # export OS_TOKEN=gAAAAABfNgp_NYBVvO-A9GAuATxr_JIQiAyUBGOEq4LEz76HDBMwEAP8aS1OQuVakLwXjmc_wuVvUt5VFdic81WFrD5-Pnc6eesWk-k0RaRzgDS2izVbv-u_VovgJX2WrkZRfWRCjgfnlJ5ZeQa9ybPPGizfCUFp5Wlvh_7jNLaZI3ZH4c8pvMs
    ```
    > Chú ý : Token chỉ có thời hạn `3600s`
- Sau khi import token vào biến môi trường, cú pháp curl API sẽ là :
    ```
    # curl -X GET -s -H "X-Auth-Token: $OS_TOKEN" <endpoint> | json_pp
    ```
    - Trong đó :
        - `endpoint` : URL tới API
        - `json_pp` : lệnh pipeline giúp hiển thị dữ liệu dạng json. Có thể thay bằng `python3 -m json.tool` cũng cho kết quả tương tự
### **2) Lấy danh sách image**
- Command :
    ```
    # openstack image list
    ```
    ```
    +--------------------------------------+---------------+--------+
    | ID                                   | Name          | Status |
    +--------------------------------------+---------------+--------+
    | 98c404c7-f349-4f31-9b0a-b2335f234379 | cirros        | active |
    | 88988016-9074-4615-8d37-45838e5dfe7d | cirros        | active |
    | 3cc5e037-83c4-46e9-8e2e-36900ddbbb38 | cirros-2      | active |
    | 2f6a034a-3f8c-464a-aba6-39ef24fc37d6 | cirros-empty  | queued |
    | 3e6ee696-06f2-4751-b183-a65b86b06736 | cirros-test   | active |
    | cab13784-5485-4f57-9456-61fe1e449274 | cirros-upload | active |
    +--------------------------------------+---------------+--------+
    ```
- Curl API :
    ```
    # curl -X GET -s -H "X-Auth-Token: $OS_TOKEN" http://10.5.11.210:9292/v2/images | json_pp
    ```
    => Output :
    ```json
    {
        "first" : "/v2/images",
        "images" : [
            {
                "checksum" : null,
                "container_format" : "bare",
                "created_at" : "2020-08-13T18:28:27Z",
                "disk_format" : "qcow2",
                "file" : "/v2/images/2f6a034a-3f8c-464a-aba6-39ef24fc37d6/file",
                "id" : "2f6a034a-3f8c-464a-aba6-39ef24fc37d6",
                "min_disk" : 0,
                "min_ram" : 0,
                "name" : "cirros-empty",
                "os_hash_algo" : null,
                "os_hash_value" : null,
                "os_hidden" : false,
                "owner" : "3137268b602d4d0793c2a997db9dd8d7",
                "protected" : false,
                "schema" : "/v2/schemas/image",
                "self" : "/v2/images/2f6a034a-3f8c-464a-aba6-39ef24fc37d6",
                "size" : null,
                "status" : "queued",
                "tags" : [],
                "updated_at" : "2020-08-13T18:28:27Z",
                "virtual_size" : null,
                "visibility" : "shared"
            },
            {
                "checksum" : "1d3062cd89af34e419f7100277f38b2b",
                "container_format" : "bare",
                "created_at" : "2020-08-13T15:37:39Z",
                "disk_format" : "qcow2",
                "file" : "/v2/images/3cc5e037-83c4-46e9-8e2e-36900ddbbb38/file",
                "id" : "3cc5e037-83c4-46e9-8e2e-36900ddbbb38",
                "min_disk" : 0,
                "min_ram" : 0,
                "name" : "cirros-2",
                "os_hash_algo" : "sha512",
                "os_hash_value" : "553d220ed58cfee7dafe003c446a9f197ab5edf8ffc09396c74187cf83873c877e7ae041cb80f3b91489acf687183adcd689b53b38e3ddd22e627e7f98a09c46",
                "os_hidden" : false,
                "owner" : "3137268b602d4d0793c2a997db9dd8d7",
                "protected" : false,
                "schema" : "/v2/schemas/image",
                "self" : "/v2/images/3cc5e037-83c4-46e9-8e2e-36900ddbbb38",
                "size" : 16338944,
                "status" : "active",
                "tags" : [],
                "updated_at" : "2020-08-13T15:55:21Z",
                "virtual_size" : null,
                "visibility" : "shared"
            },
            {
                "checksum" : "1d3062cd89af34e419f7100277f38b2b",
                "container_format" : "bare",
                "created_at" : "2020-08-13T14:52:33Z",
                "disk_format" : "qcow2",
                "file" : "/v2/images/cab13784-5485-4f57-9456-61fe1e449274/file",
                "id" : "cab13784-5485-4f57-9456-61fe1e449274",
                "min_disk" : 0,
                "min_ram" : 0,
                "name" : "cirros-upload",
                "os_hash_algo" : "sha512",
                "os_hash_value" : "553d220ed58cfee7dafe003c446a9f197ab5edf8ffc09396c74187cf83873c877e7ae041cb80f3b91489acf687183adcd689b53b38e3ddd22e627e7f98a09c46",
                "os_hidden" : false,
                "owner" : "3137268b602d4d0793c2a997db9dd8d7",
                "protected" : false,
                "schema" : "/v2/schemas/image",
                "self" : "/v2/images/cab13784-5485-4f57-9456-61fe1e449274",
                "size" : 16338944,
                "status" : "active",
                "tags" : [],
                "updated_at" : "2020-08-13T15:17:46Z",
                "virtual_size" : null,
                "visibility" : "shared"
            },
            {
                "checksum" : "1d3062cd89af34e419f7100277f38b2b",
                "container_format" : "bare",
                "created_at" : "2020-08-13T04:20:54Z",
                "disk_format" : "qcow2",
                "file" : "/v2/images/3e6ee696-06f2-4751-b183-a65b86b06736/file",
                "id" : "3e6ee696-06f2-4751-b183-a65b86b06736",
                "min_disk" : 0,
                "min_ram" : 0,
                "name" : "cirros-test",
                "os_hash_algo" : "sha512",
                "os_hash_value" : "553d220ed58cfee7dafe003c446a9f197ab5edf8ffc09396c74187cf83873c877e7ae041cb80f3b91489acf687183adcd689b53b38e3ddd22e627e7f98a09c46",
                "os_hidden" : false,
                "owner" : "3137268b602d4d0793c2a997db9dd8d7",
                "protected" : false,
                "schema" : "/v2/schemas/image",
                "self" : "/v2/images/3e6ee696-06f2-4751-b183-a65b86b06736",
                "size" : 16338944,
                "status" : "active",
                "tags" : [],
                "updated_at" : "2020-08-13T04:20:54Z",
                "virtual_size" : null,
                "visibility" : "shared"
            },
            {
                "checksum" : "443b7623e27ecf03dc9e01ee93f67afe",
                "container_format" : "bare",
                "created_at" : "2020-08-13T01:38:07Z",
                "disk_format" : "qcow2",
                "file" : "/v2/images/98c404c7-f349-4f31-9b0a-b2335f234379/file",
                "id" : "98c404c7-f349-4f31-9b0a-b2335f234379",
                "min_disk" : 0,
                "min_ram" : 0,
                "name" : "cirros",
                "os_hash_algo" : "sha512",
                "os_hash_value" : "6513f21e44aa3da349f248188a44bc304a3653a04122d8fb4535423c8e1d14cd6a153f735bb0982e2161b5b5186106570c17a9e58b64dd39390617cd5a350f78",
                "os_hidden" : false,
                "owner" : "3137268b602d4d0793c2a997db9dd8d7",
                "protected" : false,
                "schema" : "/v2/schemas/image",
                "self" : "/v2/images/98c404c7-f349-4f31-9b0a-b2335f234379",
                "size" : 12716032,
                "status" : "active",
                "tags" : [],
                "updated_at" : "2020-08-13T01:38:07Z",
                "virtual_size" : null,
                "visibility" : "public"
            },
            {
                "checksum" : "1d3062cd89af34e419f7100277f38b2b",
                "container_format" : "bare",
                "created_at" : "2020-08-12T04:38:12Z",
                "disk_format" : "qcow2",
                "file" : "/v2/images/88988016-9074-4615-8d37-45838e5dfe7d/file",
                "id" : "88988016-9074-4615-8d37-45838e5dfe7d",
                "min_disk" : 0,
                "min_ram" : 0,
                "name" : "cirros",
                "os_hash_algo" : "sha512",
                "os_hash_value" : "553d220ed58cfee7dafe003c446a9f197ab5edf8ffc09396c74187cf83873c877e7ae041cb80f3b91489acf687183adcd689b53b38e3ddd22e627e7f98a09c46",
                "os_hidden" : false,
                "owner" : "3137268b602d4d0793c2a997db9dd8d7",
                "protected" : false,
                "schema" : "/v2/schemas/image",
                "self" : "/v2/images/88988016-9074-4615-8d37-45838e5dfe7d",
                "size" : 16338944,
                "status" : "active",
                "tags" : [],
                "updated_at" : "2020-08-12T04:38:13Z",
                "virtual_size" : null,
                "visibility" : "public"
            }
        ],
        "schema" : "/v2/schemas/images"
    }
    ```
### **3) Xem thông tin chi tiết về image**
- Command :
    ```
    # openstack image show <image_id>
    ```
- Curl API :
    ```
    # curl -X GET -s -H "X-Auth-Token: $OS_TOKEN" http://10.5.11.210:9292/v2/images/<image_id> | json_pp
    ```
### **4) Tạo mới image (chưa upload dữ liệu)**
- Curl API :
    ```
    # curl -s -X POST \
    -H "X-Auth-Token: $OS_TOKEN" \
    -H "Content-Type: application/json" \
    -d \
    '{
        "name": "cirros-test",
        "container_format": "bare",
        "disk_format": "qcow2"
    }' \
    http://10.5.11.210:9292/v2/images | json_pp
    ```
    => Output :
    ```json
    {
        "checksum" : null,
        "container_format" : "bare",
        "created_at" : "2020-08-14T04:20:11Z",
        "disk_format" : "qcow2",
        "file" : "/v2/images/d7837bd9-6c0b-4c61-8cce-826964782b64/file",
        "id" : "d7837bd9-6c0b-4c61-8cce-826964782b64",
        "min_disk" : 0,
        "min_ram" : 0,
        "name" : "cirros-test",
        "os_hash_algo" : null,
        "os_hash_value" : null,
        "os_hidden" : false,
        "owner" : "3137268b602d4d0793c2a997db9dd8d7",
        "protected" : false,
        "schema" : "/v2/schemas/image",
        "self" : "/v2/images/d7837bd9-6c0b-4c61-8cce-826964782b64",
        "size" : null,
        "status" : "queued",
        "tags" : [],
        "updated_at" : "2020-08-14T04:20:11Z",
        "virtual_size" : null,
        "visibility" : "shared"
    }
    ```
### **5) Upload image**
- Để upload dữ liệu cho image, sử dụng phương thức `PUT` tới API của từng image.
- Curl API :
    ```
    # curl -s -X PUT \
    -H "X-Auth-Token: $OS_TOKEN" \
    -H "Content-Type: application/octet-stream" \
    -d /root/cirros-0.5.1-x86_64-disk.img \
    http://10.5.11.210:9292/v2/images/d7837bd9-6c0b-4c61-8cce-826964782b64/file
    ```
### **6) Xóa image**
- Để xóa image vừa tạo, sử dụng phương thức `DELETE` gửi request tới API của image đó .
- Curl API :
    ```
    # curl -s -X DELETE \
    -H "X-Auth-Token: $OS_TOKEN" \
    -H "Content-Type: application/octet-stream" \
    http://10.5.11.210:9292/v2/images/d7837bd9-6c0b-4c61-8cce-826964782b64
    ```
-----------------
[Tham khảo thêm về APIs Glance](https://docs.openstack.org/glance/latest/user/glanceapi.html)