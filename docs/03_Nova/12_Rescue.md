# Rescue VM
## **1) Giới thiệu**
- **OpenStack** cung cấp chức năng để rescue các instance trong các trường hợp lỗi hệ thống, mất mát filesystem, mất SSH key, cấu hình network bị sai hoặc dùng để khôi phục mật khẩu.
    > **Lưu ý:** Trong quá trình boot thì instance disk và recuse disk có thể trùng UUID , vì thế trong 1 một số trường hợp thì instance đã vào mode rescue nhưng vẫn boot từ local disk
- Một vài trường hợp sửa chữa một hệ thống bị lỗi bằng cách sử dụng rescue boot:
    - Mất ssh key và muốn kích hoạt tàm thời chế độ đăng nhập bằng password
    - Cấu hình mạng gặp lỗi
    - Cấu hình boot lỗi
## **2) Các bước thực hiện Rescue**
### **2.1) Rescue instance boot từ local disk**
#### **2.1.1) Thực hiện reset password VM sử dụng image SystemRescueCD**
- 
### **2.2) Rescue instance boot từ volume**