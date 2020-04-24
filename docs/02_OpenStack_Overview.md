# Tổng quan về OpenStack
## **1) Giới thiệu** <img src=https://i.imgur.com/kpZLTDp.png align=right width=30%>
- **OpenStack** là nền tảng mã nguồn mở, được sử dụng để xây dựng mô hình **private cloud** và **public cloud** .
- Lịch sử hình thành :
    - **Amazone Web Service (AWS)** là nguồn cảm hứng ra đời cho **OpenStack**.
    - **OpenStack** được sáng lập bởi **NASA** và **RACKSPACE** năm `2010`. **NASA** đóng góp **Nebula project**, sau có tên là **NOVA** như ngày nay, **RACKSPACE** đóng góp **SWIFT project** – project về lưu trữ file.
    - Ngày nay , project **OpenStack** đã có sự tham gia của nhiều “ông lớn” như : **AT&T**, **Ubuntu**, **IBM**, **RedHat**, **SUSE**, **Mirantis**,...
- **OpenStack** được thiết kế theo hướng module. **OpenStack** là một project lớn là sự kết hợp của các project thành phần: nova, swift, neutron, glance, etc.
- Mở về: Thiết kế/ Phát triển/ Cộng đồng/ Mã nguồn.
- Chu kì 6 tháng sẽ cho ra một phiên bản mới.
- `99.99%` mã nguồn được viết bằng **Python `2.x`**
- Tên các phiên bản được đánh theo `A`, `B`, `C` (**Austin**, **Bexar**, **Cactus**, etc.)…. Tối đa 10 kí tự là danh từ.
- Tên các project: Compute – **NOVA**, Network – **NEUTRON**,....
## **2) Kiến trúc OpenStack**
- Kiến trúc mức khái niệm :

    <img src=https://i.imgur.com/DVfcKxw.png>

- Kiến trúc mức logic :

    <img src=https://i.imgur.com/HelRYWA.png>

## **3) Các thành phần trong OpenStack**
- Có thể coi **OpenStack** như một hệ điều hành cloud có nhiệm vụ kiểm soát các tài nguyên tính toán(***compute***), lưu trữ(***storage***) và ***networking*** trong hệ thống lớn datacenter, tất cả đều có thể được kiểm soát qua giao diện dòng lệnh hoặc một dashboard( do project **Horizon** cung cấp). **OpenStack** có 6 core project. 6 core project của **OpenStack** bao gồm: **Nova**, **Neutron**, **Swift**, **Cinder**, **Keystone**, **Glance**. 
### **3.1) Keystone – Identity Service**
- Cung cấp dịch vụ xác thực và ủy quyền cho các dịch vụ khác của OpenStack, cung cấp danh mục của các endpoints cho tất các dịch vụ trong OpenStack. Cụ thể hơn:
    - Xác thực user và vấn đề token để truy cập vào các dịch vụ
    - Lưu trữ user và các tenant cho vai trò kiểm soát truy cập(cơ chế role-based access control - RBAC)
    - Cung cấp catalog của các dịch vụ (và các API enpoints của chúng) trên cloud
    - Tạo các policy giữa user và dịch vụ
Mỗi chức năng của Keystone có kiến trúc pluggable backend cho phép hỗ trợ kết hợp với LDAP, PAM, SQL