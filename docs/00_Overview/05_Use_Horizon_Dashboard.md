# Sử dụng Horizon Dashboard để làm việc với OpenStack
## **1) Đăng nhập**
- Đăng nhập vào đường dẫn `http://IP_CONTROLLER/dashboard` :

    <img src=https://i.imgur.com/QzblonM.png>

    - Domain: `default`
    - Username: `admin` (đã tạo từ trước)
    - Password: `P@ssw0rd` (đã tạo từ trước)

- Dashboard sau khi đăng nhập :

    <img src=https://i.imgur.com/x5nl7EA.png>

## **2) Khai báo các dải network**
### **2.1) Tạo provider network**
- Chọn Tab ***Admin*** -> ***Network*** -> ***Networks*** -> ***Create Network*** :
    
    <img src=https://i.imgur.com/QXIttg6.png>

- Tại tab **Network** :

    <img src=https://i.imgur.com/f0ByP6e.png>
    
    - `1` : Nhập tên của public network, tên này có thể là bất kỳ
    - `2` : Chọn project để tạo network, ở đây chọn `admin`
    - `3` : Chọn chế độ mạng là `flat`
    - `4` : Nhập tên của **Physical Network**, tên này phải giống với tên trong bước khai báo của **`neutron`** ở cả `controller` và `compute`. Để là `provider`
    - `6` : Tích chọn các mục này để đảm bảo đây là provider network và được dùng chung cho các người dùng khác nhau trên hệ thống **OpenStack**.
    - Chọn ***Next*** để sang tab tiếp theo
- Tại tab **Subnet**, tạo range IP cho dải provider :

    <img src=https://i.imgur.com/ki5ahnX.png>

    - `1` : Nhập tên subnet provider network
    - `2` : Nhập range sẽ làm provider network. Trong mô hình mạng này ta sẽ sử dụng provider network là `10.5.8.0/242`. Lưu ý range này cần lựa chọn đúng với mô hình mạng đang sử dụng.
    - `3` : Mục nay để trống thì hệ thống sẽ tự động lấy IP đầu tiên của range. Chủ động điền IP Gateway là `10.5.8.1`
    - Chọn ***Next*** để sang tab tiếp theo 
- Tại tab **Subnet Details**, khai báo range cấp DHCP cho VM và DNS

    <img src=https://i.imgur.com/p4NjlXM.png>

    - `1` : Tùy chọn bật/tắt DHCP
    - `2` : Khai báo IP bắt đầu và IP kết thúc của DHCP Pool theo cú pháp `start_IP,end_IP`.
    - `3` : IP của DNS Server
    - Chọn ***Create*** để kết thúc việc khai báo provider network
    > **Lưu ý:** 
- Sau khi dải mạng `public` được thêm thành công, chọn vào nó để xem chi tiết :

    <img src=https://i.imgur.com/41j4oTa.png>

- Tại tab **Port**, ta thấy IP: `10.5.11.101` đã được sử dụng để cấp IP DHCP :

    <img src=https://i.imgur.com/ZDa4oq2.png>

### **2.2) Tạo private network**
- Chọn Tab ***Project*** -> ***Network*** -> ***Networks*** -> ***Create Network*** :

    <img src=https://i.imgur.com/BSqKSBs.png>

- sss

    <img src=https://i.imgur.com/nl2Qg57.png>

- sss

    <img src=https://i.imgur.com/C25XTfd.png>

- sss

    <img src=https://i.imgur.com/uWnMd4V.png>

- sss

    <img src=https://i.imgur.com/bEHIKby.png>