# Endpoint và Catalog
## **1) Endpoint**
- **Endpoint** trong **Keystone** là một URL có thể được sử dụng để truy cập dịch vụ trong **Openstack** .
- Một **endpoint** giống như một điểm liên lạc để người dùng sử dụng dịch vụ của **OpenStack**.
- Có 3 loại **endpoint** :
    - `adminurl` : dùng cho user quản trị
    - `internalurl`: là những gì các dịch vụ khác sử dụng để giao tiếp với nhau
    - `publicurl` : là nơi các user khác truy cập và sử dụng dịch vụ
- Hiển thị list các **endpoint** :
    ```
    # openstack endpoint list
    ```
    <img src=https://i.imgur.com/KtPUr4d.png>

- Hiển thị chi tiết một **endpoint** :
    ```
    # openstack show endpoint <endpoint_id>
    ```
    <img src=https://i.imgur.com/oyw3An2.png>

## **2) Catalog**
- **Catalog** trong **OpenStack** là tập hợp các danh mục sẵn sàng sử dụng mà user có thể sử dụng trong **OpenStack**.
- **Service catalog** cung cấp cho người dùng về các dịch vụ có sẵn trong **OpenStack**, cùng với thông tin bổ sung về các vùng, phiên bản API và các project có sẵn.
- **Catalog** làm cho việc tìm kiếm dịch vụ hiệu quả, chẳng hạn như cách định cấu hình liên lạc giữa các dịch vụ.
- Hiển thị list **catalog** :
    ```
    # openstack catalog list
    ```
    <img src=https://i.imgur.com/hRRG0bC.png>

- Hiển thị chi tiết một mục trong **catalog** :
    ```
    # openstack catalog show <catalog_name>
    ```
    <img src=https://i.imgur.com/IQJnSkm.png>