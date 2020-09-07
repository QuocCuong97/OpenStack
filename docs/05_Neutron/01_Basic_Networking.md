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
- **Load balancer** có thể là phần mềm hoặc thiết bị phần cứng cho phép traffic được phân phối đều trên các server. Bằng việc phân phối đều traffic trên các server, nó sẽ tránh được sự quá tải trên 1 server, do đó tránh được việc 1 điểm bị sập trong mạng .
## **3) Các giao thức overlay (tunnel)**
- **Tunneling** là một cơ chế cho phép trung chuyển các payload qua các mạng phức tạp, không tương thích với nhau. Nó cho phép người dùng mạng truy cập được tới những mạng được bảo mật hoặc đến những mạng kém an toàn. Dữ liệu được mã hóa sẽ được đưa vào payload. Các user trong tunnel sẽ là public với nhau mặc dù họ đang ở trong các mạng private .
### **3.1) Generic routing encapsulation (GRE)**
- **GRE** là một 
### **3.2) Virtual extensible local area network (VXLAN)**
- Mục đích của **VXLAN** là cung cấp mạng cô lập có thể mở rộng. **VXLAN** là một giao thức layer 2 chạy trên mạng layer 3. Nó cho phép
### **3.3) Generic Network Virtualization Encapsulation (GENEVE)**
## **4) Network namespace**
- Một **namespace** là một cách xác định phạm vi một tập hợp các identifier cụ thể . Có thể sử dụng cùng một identifier nhiều lần trong các namespace khác nhau. Cũng có thể giới hạn một identifier cho các process riêng biệt .
- **VD :** Linux cung cấp namespace cho network và các proceess. Nếu một process chạy bên trong một process namespace, nó chỉ có thể nhìn thấy và giao tiếp với các process khác trong cùng namespace. Do đó, nếu một shell trong một process namespace cụ thể chạy lệnh `ps waux`, nó chỉ show được các process khác trong cùng namespace với nó .
### **4.1) Linux network namespaces**
- Trong một network namespace, các identifier chính là các network device; vì vậy các network device cho trước, như `eth0` chỉ tồn tại trong một namespace cụ thể .
- Linux khởi động với một network namespace mặc định, vì vậy nếu không có gì xen vào, đó chính là nơi chứa tất cả các network device . Tuy nhiên hoàn toàn có thể tạo ra các namespace khác (không phải mặc định), và tạo các network device bên trong các namespace nàu, hoặc di chuyển một device có sẵn từ một namespace này sang namespace khác.
- Mỗi network namespace cũng có bảng định tuyến riêng của nó (*routing table*) . Một bảng định tuyến sẽ chứa đường tới các IP đích, vì vậy network namespace là thứ cần thiết nếu muốn sử dụng cùng một địa chỉ IP đích trong từng môi trường khác nhau. Điều này hình thành nên tính năng quan trọng của **OpenStack Network** : overlap IP trong các network ảo khác nhau .
- Mỗi network namespace sẽ có một bộ `iptable` riêng (cho cả IPv4 và IPv6). Vì vậy có thể áp dụng các chính sách bảo mật khác nhau hay các routing khác nhau cho các IP giống nhau trên các namespace khác nhau
### **4.2) Virtual routing and forwarding (VRF)**
- **VRF - *Virtual routing and forwarding*** là một công nghệ IP cho phép nhiều instance của bản định tuyến tồn tại trên cùng 1 router tại cùng 1 thời điểm . Đây chỉ đơn giản là 1 tên khác của network namespace
## **5) Network Address Translation**
- **NAT - *Network Address Translation*** một quá trình thay đổi địa chỉ nguồn hoặc địa chỉ đích trong header của gói tin IP trong khi gói tin được vận chuyển. Cả ứng dụng gửi gói tin và ứng dụng nhận gói tin đều không nhận thức được quá trình này.
- **NAT** thường được tích hợp vào router. Tuy nhiên, trong **OpenStack**, server Linux sẽ thực hiện chức năng NAT, không phải router. Những server này sử dụng `iptables` để thực hiện chức năng NAT .
### **5.1) SNAT**
- Trong **SNAT** (*Source Network Address Translation*), NAT router sẽ thay đổi địa chỉ của sender trong gói tin IP.
- **SNAT** thường được sử dụng để cho phép host có IP Private giao tiếp với mạng Internet.
- **RFC 1918** quy định 3 subnet sau là dải địa chỉ private :
    - `10.0.0.0/8`
    - `172.16.0.0/12`
    - `192.168.0.0/16`
- Các IP này không được định tuyến public, có nghĩa là 1 host ở ngoài Internet không thể gửi gói tin IP tới các IP này. IP Private được sử dụng rộng rãi trong các tòa nhà, môi trường cô lập.
- Thông thường, 1 ứng dụng chạy trên host với IP Private sẽ kết nối tới server trên Internet. **VD :** user muốn truy cập tới trabg web `www.openstack.org`. Cho dù gói tin đi đến được trang web với IP Private thì trang web cũng không thể gửi lại gói tin cho sender.
- **SNAT** giải quyết vấn đề này bằng cách thay đổi IP source thành 1 địa chỉ có thể định tuyến trên đường Internet. Có nhiều kiểu **SNAT**. Kiểu mà **OpenStack** sử dụng là : một NAT router ở giữa sender và reiceiver thay thế source IP trong gói tin bằng IP Public của router, đồng thời cũng thay đổi các port TCP, UDP thành các giá trị khác. Router sẽ lưu trữ các record chứa các giá trị ban đầu và các giá trị sau khi thay đổi . Khi router nhận lại gói tin với đúng IP và port, nó sẽ translate ngược lại thành private IP và port, forward packet dựa theo đó .
- Bởi NAT router sử dụng port để thay thế cho 1 IP, vi vậy hình thức này đôi khi giống như **Port Address Translation (PAT)** - hay còn được gọi là **NAT Overload**
- OpenStack sử dụng SNAT để cho phép các ứng dụng chạy bên trong instance kết nốt ra ngoài đến mạng internet
### **5.2) DNAT**
- Trong **DNAT** (*Destination Network Address Translation*), NAT router thay địa chỉ của header )
- **OpenStack** sử dụng **DNAT** để định tuyến gói tin từ instance đến service metadata. Ứng dụng chạy bên trong các instance sẽ truy cập vào metadata service bằng cách thực hiện request `HTTP GET` đến IP `169.254.169.254` . Trong **OpenStack**, không có host nào có địa chỉ IP này, thay vào đó, **OpenStack** sẽ sử dụng **DNAT** để thay đổi IP đích của các packet này để chúng có thể tiếp cận card netword mà metadata service đang listen .
### **5.3) One-to-one NAT**
- Trong NAT one-to-one, router NAT sẽ duy trì các record mapping 1-1 giữa IP Private và IP Public. **OpenStack** sử dụng **NAT one-to-one** để thực hiện float IP .