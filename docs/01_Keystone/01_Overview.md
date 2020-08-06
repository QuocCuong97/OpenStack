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
OpenStack dựa rất nhiều vào tokens để xác thực và keystone chính là dịch vụ duy nhất có thể tạo ra tokens
Token có giới hạn về thời gian được phép sử dụng. Khi token hết hạn thì user sẽ được cấp một token mới. Cơ chế này làm giảm nguy cơ user bị đánh cắp token.
Hiện tại, keystone đang sử dụng cơ chế bearer token. Có nghĩa là bất cứ ai có token thì sẽ có khả năng truy cập vào tài nguyên của cloud. Vì vậy việc giữ bí mật token rất quan trọng.