# Tổng quan về Keystone
## **1) Giới thiệu**
- Môi trường cloud theo mô hình dịch vụ **Infrastructure-as-a-Service** layer (**IaaS**) cung cấp cho người dùng khả năng truy cập vào các tài nguyên quan trọng ví dụ như máy ảo, kho lưu trữ và kết nối mạng. Bảo mật vẫn luôn là vấn đề quan trọng đối với mọi môi trường cloud và trong **OpenStack** thì **Keystone** chính là dịch vụ đảm nhiệm công việc ấy. Nó sẽ chịu trách nhiệm cung cấp các kết nối mang tính bảo mật cao tới các nguồn tài nguyên cloud.
## **2) Các chức năng chính của Keystone**
### **2.1) Identity *(nhận diện)***
- Nhận diện những người đang cố truy cập vào các tài nguyên trên Cloud
- Trong **Keystone**, **identity** thường được hiểu là User
Tại những mô hình OpenStack nhỏ, identity của user thường được lưu trữ trong database của keystone. Đối với những mô hình lớn cho doanh nghiệp thì 1 external Identity Provider thường được sử dụng.
### **2.2) Authentication *(xác thực)***
- Là quá trình xác thực những thông tin dùng để nhận định user (***user's identity***)
- **Keystone** có tính pluggable tức là nó có thể liên kết với những dịch vụ xác thực người dùng khác như **LDAP** hoặc **Active Directory**
- Thường thì **Keystone** sử dụng Password cho việc xác thực người dùng. Đối với những phần còn lại, **keystone** sử dụng tokens.
- **OpenStack** dựa rất nhiều vào tokens để xác thực và **Keystone** chính là dịch vụ duy nhất có thể tạo ra tokens
- Token có giới hạn về thời gian được phép sử dụng. Khi token hết hạn thì user sẽ được cấp một token mới. Cơ chế này làm giảm nguy cơ user bị đánh cắp token.
- Hiện tại, **Keystone** đang sử dụng cơ chế ***bearer token***. Có nghĩa là bất cứ ai có token thì sẽ có khả năng truy cập vào tài nguyên của cloud. Vì vậy việc giữ bí mật token rất quan trọng.
    > ***Bearer token*** là một giá trị nằm trong phần Authorization header của mỗi HTTP request. Nó mặc định không tự được lưu ở bất cứ đâu (không như **cookie**), bạn phải quyết định nơi lưu nó.
### **2.3) Access Management - Authorization *(Quản lý truy cập, quyền hạn)***
- **Access Management** hay còn được gọi là **Authorization** là quá trình xác định những tài nguyên mà user được phép truy cập tới.
- Trong **OpenStack**, **Keystone** kết nối users với những Projects hoặc Domains bằng cách gán role cho user vào những project hoặc domain ấy.
- Các projects trong **OpenStack** như **Nova**, **Cinder**...sẽ kiểm tra mối quan hệ giữa role và các user's project và xác định giá trị của những thông tin này theo cơ chế các quy định (**policy engine**). **Policy engine** sẽ tự động kiểm tra các thông tin (thường là **role**) và xác định xem user được phép thực hiện những gì.
## **3) Lợi ích khi sử dụng Keystone**
- Cung cấp giao diện xác thực và quản lí truy cập cho các services của **OpenStack**. Nó cũng đồng thời lo toàn bộ việc giao tiếp và làm việc với các hệ thống xác thực bên ngoài.
- Cung cấp danh sách đăng kí các containers (Projects) mà nhờ vậy các services khác của **OpenStack** có thể dùng nó để "tách" tài nguyên (servers, images,...)
- Cung cấp danh sách đăng kí các Domains được dùng để định nghĩa các khu vực riêng biệt cho users, groups, và projects khiến các khách hàng trở nên "tách biệt" với nhau
- Danh sách đăng kí các roles được **Keystone** dùng để ủy quyền cho các services của **OpenStack** thông qua file `policy`.
- **Assignment store** cho phép users và groups được gán các roles trong projects và domains.
- **Catalog** lưu trữ các services của **OpenStack**, **endpoints** và **regions**, cho phép người dùng tìm kiếm các endpoints của services mà họ cần.
## **4) Các khái niệm lớn trong Keystone**
### **4.1) Project**
- Trong **Keystone**, **Project** được dùng bởi các service của **OPS** để nhóm và cô lập các nguồn tài nguyên. Nó có thể hiểu là 1 nhóm các tài nguyên mà chỉ có một số các user mới có thể truy cập và hoàn toàn tách biệt với các nhóm khác.
- Ban đầu nó được gọi là **tenants** sau đó được đổi tên thành **projects**.
- Mục đích cơ bản nhất của **Keystone** chính là nơi để đăng ký cho các **projects** và xác định ai được phép truy cập project đó.
- Bản thân **projects** không sở hữu **users** hay **groups** mà **users** và **groups** được cấp quyền truy cập tới **project** sử dụng cơ chế gán **role**.
- Trong một vài tài liệu của **OpenStack** thì việc gán **role** cho **user** còn được gọi là "`grant`".

    <img src=https://i.imgur.com/wufMuvX.png>

### **4.2) Domains**
- Trong thời kì đầu, không có bất cứ cơ chế nào để hạn chế sự xuất hiện của các **project** tới những nhóm user khác nhau. Điều này có thể gây ra những sự nhầm lẫn hay xung đột không đáng có giữa các tên của **project** của các tổ chức khác nhau.
- Tên **user** cũng vậy và nó hoàn toàn cũng có thể dẫn tới sự nhầm lẫn nếu hai tổ chức có **user** có tên giống nhau.
- Vì vậy mà khái niệm **Domain** ra đời, nó được dùng để cô lập danh sách các **Projects** và **Users**.
- **Domain** được định nghĩa là một tập hợp các **users**, **groups**, và **projects**. Nó cho phép người dùng chia nguồn tài nguyên cho từng tổ chức sử dụng mà không phải lo xung đột hay nhầm lẫn.

    <img src=https://i.imgur.com/nAhlHpT.png>