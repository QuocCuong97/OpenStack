# Mạng cơ bản
## **1) Các khái niệm cơ bản**
### **1.1) Ethernet**
- **Ethernet** là một giao thức mạng, được định nghĩa trong chuẩn **IEEE 802.3** . Hầu hết các card mạng dây (**NICs**) giao tiếp thông qua **Ethernet**
- Trong mô hình **OSI** về các giao thức mạng, Ethernet nằm ở tầng thứ 2 - ***data-link layer***.
- Trong mạng Ethernet, các host kết nối tới mạng giao tiếp với nhau bằng các ***frame***. Tất cả các host trên mạng Ethernet được định danh duy nhất bởi 1 địa chỉ gọi là **Media Access Control (MAC)**. Trong thực tế, tất cả các instance trong **OpenStack** đều có 1 địa chỉ **MAC** duy nhất, khác với **MAC** của compute node. Một địa chỉ **MAC** gồm `48bits`, được biểu diễn dưới dạng thập lục phân. ( **VD :** `08:00:27:b9:88:74` ). Địa chỉ **MAC** được gắn liền với card mạng bởi nhà sản xuất, tuy nhiên, vẫn có cách để tự thay đổi địa chỉ **MAC**.
- Trong Linux, có thể xem địa chỉ **MAC** qua lệnh `ip` :
    ```
    # ip link show eth0
    8: eth0: <> mtu 1500 group default qlen 1
    link/ether 9c:b6:d0:e0:6f:4e
    ```
## **2) Một số thiết bị mạng**
### **2.1) Switch**
- **Switch** là một thiết bị **Multi-Input Multi-Output (MIMO)** cho phép các gói tin (packet) di chuyển từ node này qua node khác. **Switch** kết nối các host thuộc cùng mạng layer-2. **Switch** hoạt động ở Layer-2 trong mô hình **OSI**. Chúng forward các gói tin dựa vào địa chỉ Ethernet đích trong header của gói tin.
### **2.2) Router**
- **Router** là các thiết bị đặc biệt cho phép gói tin đi từ mạng layer-3 này đến mạng khác. **Router** cho phép việc giao tiếp giữa 2 node trên cùng mạng layer-3 mà không cần kết nối trực tiếp với nhau. **Router** hoạt động ở layer-3 trong mô hình **OSI**. Chúng định tuyến các gói tin dựa vào địa chỉ IP đích trong header của gói tin.
### **2.3) Firewall**
- **Firewall** được sử dụng để giới hạn lưu lượng tới và từ một host hoặc một mạng. Một **firewall** có thể hoặc là một thiết bị chuyên dụng kết nối giữa 2 mạng hoặc là một phần mềm dùng để lọc gói tin cài đặt trên hệ điều hành. **Firewall** được sử dụng để giới hạn traffic tới host dựa vào các rule được định nghĩa trên host. Chúng có thể lọc các gói tin dựa trên một số yếu tố như địa chỉ IP nguồn, IP đích, port, trạng thái kết nối,... Chúng thường được sử dụng để bảo vệ host khỏi các truy cập không an toàn hoặc các cuộc tấn công mạng. Các hệ điều hành Linux đều sử dụng **firewall** thông qua **Iptables** .
### **2.4) Load balancer**
- **Load balancer** có thể là phần mềm hoặc thiết bị phần cứng cho phép traffic được phân phối đều trên các server. Bằng việc phân phối đều traffic trên các server, nó sẽ tránh được sự quá tải trên 1 server, do đó ngăn chặn được việc 1 điểm bị sập trong mạng .
## **3) Các giao thức overlay (tunnel)**
### **3.1) 