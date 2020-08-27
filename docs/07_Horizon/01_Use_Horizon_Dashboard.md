# Sử dụng Horizon Dashboard để làm việc với OpenStack
## **1) Đăng nhập**
- Đăng nhập vào đường dẫn `http://IP_CONTROLLER/dashboard` :

    <img src=https://i.imgur.com/jPxBcVv.png>

    - Domain: `default`
    - Username: `admin` (đã tạo từ trước)
    - Password: `Password123` (đã tạo từ trước)

- Dashboard sau khi đăng nhập :

    <img src=https://i.imgur.com/flCKE1b.png>

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
- Sau khi dải mạng `public` được thêm thành công, chọn vào nó để xem chi tiết :

    <img src=https://i.imgur.com/41j4oTa.png>

- Tại tab **Port**, ta thấy IP: `10.5.11.101` đã được sử dụng để cấp IP DHCP :

    <img src=https://i.imgur.com/ZDa4oq2.png>

### **2.2) Tạo private network**
- Chọn Tab ***Project*** -> ***Network*** -> ***Networks*** -> ***Create Network*** :

    <img src=https://i.imgur.com/BSqKSBs.png>

- Tại tab **Network**, nhập tên dải mạng private (tên bất kỳ), chọn ***Next*** :

    <img src=https://i.imgur.com/nl2Qg57.png>

- Tại tab **Subnet** :

    <img src=https://i.imgur.com/C25XTfd.png>

    - `1` : Nhập tên subnet private network
    - `2` : Nhập range sẽ làm private network. (**VD :** `192.168.1.0/24`)
    - `3` : Mục nay để trống thì hệ thống sẽ tự động lấy IP đầu tiên của range. Chủ động điền IP Gateway là `192.168.1.1`
    - Chọn ***Next*** để sang tab tiếp theo 
- Tại tab **Subnet Details** :

    <img src=https://i.imgur.com/uWnMd4V.png>

    - `1` : Tùy chọn bật/tắt DHCP
    - `2` : Khai báo IP bắt đầu và IP kết thúc của DHCP Pool theo cú pháp `start_IP,end_IP`.
    - Chọn ***Create*** để kết thúc việc khai báo provider network
- Dải mạng **private** được tạo thành công :

    <img src=https://i.imgur.com/bEHIKby.png>

### **2.3) Cấu hình Security Group**
- Mặc định thì **OpenStack** chặn toàn bộ các kết nối từ ngoài vào trong các VM, do vậy ta cần thực hiện mở các kết nối này để tiện thao tác.
- Truy cập vào tab **Project** -> **Networks** -> **Security Group** -> **Manager Rules** :

    <img src=https://i.imgur.com/5EgAoTC.png>

- Chọn **Add Rule** :

    <img src=https://i.imgur.com/UL6HPQm.png>

- Tại cửa sổ **Add Rule**, chọn rule cần add, thực hiện lặp lại với các rule `All ICMP`, `All TCP`, `All UDP`, `SSH` => chọn ***Add*** để thêm :

    <img src=https://i.imgur.com/wCmTFYZ.png>

- Các rule sau khi được thêm vào :

    <img src=https://i.imgur.com/i42YP9j.png>

## **3) Tạo Flavor (gói cấu hình)**
- **Flavor** là các gói cấu hình được quy định trước về CPUs, RAM, Disk dùng để tạo VM
- Chọn Tab ***Admin*** -> ***Compute*** -> ***Flavors*** -> ***Create Flavor*** :

    <img src=https://i.imgur.com/Xg5GiRg.png>

- Chọn **Create Flavor** :

    <img src=https://i.imgur.com/aJlINfH.png>

- Tại của sổ **Create Flavor**, điền các thông số như **Tên gói (*Name*)**, **VCPUs**, **RAM**, **Root Disk** muốn cấp cho máy ảo, sau đó chọn tab **Flavor Access** :

    <img src=https://i.imgur.com/p0Sad7f.png>

- Tại tab **Flavor Access**, chọn **project** muốn sử dụng **flavor**, sau đó chọn **Create Flavor** để tạo :

    <img src=https://i.imgur.com/EiXOXq1.png>

- **Flavor** sau khi tạo xong :

    <img src=https://i.imgur.com/NJrFIuR.png>

## **4) Tạo VM (máy ảo)**
- Chọn tab ***Project*** -> ***Compute*** -> ***Instance*** :

    <img src=https://i.imgur.com/hupLsXK.png>

- Chọn **Launch Instance** để thêm instance (VM) mới :

    <img src=https://i.imgur.com/2922r6g.png>

- Tại tab **Details**, nhập tên VM => ***Next*** để qua tab tiếp theo :
    
    <img src=https://i.imgur.com/CnRl32M.png>

- Tại tab **Source**, chọn image muốn boot cho VM => ***Next*** để qua tab tiếp theo :
    
    <img src=https://i.imgur.com/rbKDMrH.png>

- Tại tab **Flavor**, chọn gói cấu hình cho VM => ***Next*** để qua tab tiếp theo :

    <img src=https://i.imgur.com/YXlHGOk.png>

- Tại tab **Networks**, chọn các dải network muốn cấp cho VM => ***Launch Instance*** để tạo VM:

    <img src=https://i.imgur.com/f41tuxM.png>

    > Đến đây đã đủ thông tin cần thiết để tạo VM, nếu muốn cấu hình thêm, có thể chọn ***Next*** để lướt qua các tab cấu hình khác

- **VM** sau khi tạo :

    <img src=https://i.imgur.com/QPRJnTG.png>

<!-- - Chọn ***Console*** để truy cập terminal của **VM** :

    <img src=https://i.imgur.com/51yDdWH.png> -->