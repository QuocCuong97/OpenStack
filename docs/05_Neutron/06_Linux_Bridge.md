# Linux Bridge
## **1) Giới thiệu**
- **Linux bridge** là một phần mềm được tích hợp trong nhân Linux để giải quyết vấn đề ảo hóa phần Network trong các máy vật lý .
- Về mặt logic, **Linux bridge** tạo ra một con switch ảo để các VM kết nối vào và có thể nói chuyện với nhau cũng như sử dụng để ra ngoài mạng .
    <img src=https://i.imgur.com/6vhMD0d.png>
- Các tính năng do **Linux bridge** tạo ra :
    - **STP** : là tính năng chống loop gói tin trong switch
    - **VLAN** : là tính năng rất quan trọng trong một switch
    - **FDB** : là tính năng chuyển gói tin theo database được xây dựng giúp tăng tốc độ của switch .
- Linux bridge gồm 4 thành phần sau :
    - **Các ports** (hoặc interface) : được sử dụng để forward traffic giữa switch và các host khác, tương tự như một port vật lý thật
    - **Control plane** : được sử dụng để chạy giao thức **Spanning Tree (STP)** dùng để tính toán, chống loop
    - **Forwarding plane** : được sử dụng để xử lý các frame đi vào từ các port, forward chúng đến đúng port đích dựa vào bảng **MAC** học được .
    - **MAC database** : được sử dụng để nắm vị trí các host trong LAN
- Mô hình Linux Bridge trong Neutron :
    <p align=center><img src=https://i.imgur.com/8gNNKU6.png></p>

## **2) Các lệnh sử dụng trong LinuxBridge**
### **Hiển thị các bridge đang có**
- Cú pháp :
    ```
    # brctl show
    ```
    <img src=https://i.imgur.com/h8hjxI3.png>

### **Hiển thị riêng thông tin 1 bridge**
- Cú pháp :
    ```
    # brctl show <bridge_name>
    ```
    <img src=https://i.imgur.com/6EoMU5G.png>

### **Hiển thị chi tiết kết nối giữa tap interface với VM**
- Cú pháp :
    ```
    # ip -details tuntap
    ```
    <img src=https://i.imgur.com/okFZ6aT.png>
    
    > Output cho thấy PID của process chạy VM gắn với từng tap interface . Ta có thể sử dụng lệnh `ps` để lọc ra VM cụ thể :
    ```
    # ps -ef | grep 2544
    ```
    <img src=https://i.imgur.com/mqWaCd8.png>

    > ID của VM sẽ ở phần `uuid`. Sử dụng tiếp openstack CLI để biết đó là VM nào :
    ```
    # openstack server list | grep d0d16d65-01af-47ee-b2f2-41757fcd71d6
    ```
    <img src=https://i.imgur.com/7561eFB.png>

## **3) Provider network**
### **3.1) Kiến trúc và thành phần**
- 
### **3.2) Network traffic**
#### **3.2.1) Luồng Bắc-Nam (North - South)** : instance có địa chỉ IP cố định
- **TH** : 
    - Instance nằm trên node `compute1` và sử dụng `provider network 1`. 
    - Instance gửi packet đến một instance khác trong Internet :

    <p align=center><img src=https://i.imgur.com/AK3Pd3r.png></p>

- Đường đi gói tin :
    - Trên node compute 1:
        - Instance interface `(1)` forward packet tới instance port tương ứng trên provider bridge `(2)` thông qua `veth` pair
        - Các rules của Security group `(3)` trên provider bridge sẽ xử lý firewall và theo dõi kết nối của packet
        - Các sub-interface `(4)` trên provider bridge sẽ forward packet tới physical interface `(5)`
        - Physical interface `(5)` sẽ thêm gắn thêm tag `VLAN 101` (trong hình) vào packet và forward nó tới switch ngoài hạ tầng `(6)`
    - Trên hạ tầng vật lý (physical network infrastructure):
        - Switch sẽ bỏ tag `VLAN 101` khỏi packet và forward tới router `(7)`
        - Router `(7)` sẽ định tuyến cho packet từ provider network `(8)` tới external network - mạng ngoài `(9)` và forward packet tới switch - mạng ngoài`(10)`
        - Switch `(10)` forward packet ra mạng ngoài- external network `(11)`   
        - Mạng ngoài - external network `(12)` sẽ nhận packet để tiếp tục thực hiện gửi tới host đích.
#### **3.2.2) Luồng Đông-Tây 1 (East-West 1)** : instance trên cùng một node
- **TH :** Các instance trên cùng 1 mạng giao tiếp trực tiếp giữa các node compute chứa các instance đó:
    - Instance 1 : nằm trên node Compute1 và sử dụng provider network 1
    - Instance 2 : nằm trên node Compute1 và sử dụng provider network 2
    - Instance 1 gửi packet tới instance 2

    <p align=center><img src=https://i.imgur.com/c1GyJtq.png></p>

- Đường đi gói tin :
    - Trên node Compute1:
        - Instance interface `(1)` forward packet tới instance port tương ứng trên provider bridge `(2)` thông qua veth pair
        - Các rules của Security group `(3)` trên provider bridge sẽ xử lý firewall và theo dõi kết nối của packet
        - Các sub-interface `(4)` trên provider bridge sẽ forward packet tới physical interface `(5)`
        - Physical interface `(5)` sẽ thêm gắn thêm tag VLAN 101 (trong hình) vào packet và forward nó tới switch ngoài hạ tầng `(6)`
    - Trên cơ sở hạ tầng vật lý:
        - Switch forward packet từ compute1 tới compute2 `(7)`
    - Trên node Compute2:
        - Physical interface của node Compute2 `(8)` bỏ VLAN tag 101 khỏi packet và forward nó đến sub-interface `(9)` trên provider bridge
        - Các rules của Security group `(10)` trên provider bridge sẽ xử lý firewall và theo dõi kết nối của packet
        - Instance port trên provider bridge `(11)` forward packet đến interface của instance 2 `(12)` thông qua `veth` pair
#### **3.2.3) Luồng Đông-Tây 2 (East-West 2)** : instance trên 2 mạng khác nhau
- **TH :** Các instance khác mạng sẽ giao tiếp thông qua Router trên hạ tầng mạng vật lý.
    - Instance 1 nằm trên node Compute1 và sử dụng provider network 1
    - Instance 2 nằm trên node Compute1 và sử dụng provider network 2
    - Instance 1 gửi packet tới instance 2

    <p align=center><img src=https://i.imgur.com/IBGdV55.png></p>

- Đường đi gói tin :
    - Trên node Compute1 :
        - Instance interface `(1)` forward packet tới instance port tương ứng trên provider bridge `(2)` thông qua veth pair
        - Các rules của Security group `(3)` trên provider bridge sẽ xử lý firewall và theo dõi kết nối của packet
        - Các sub-interface `(4)` trên provider bridge sẽ forward packet tới physical interface `(5)`
        - Physical interface `(5)` sẽ thêm gắn thêm tag VLAN 101 (trong hình) vào packet và forward nó tới switch ngoài hạ tầng `(6)`
    - Trên cơ sở hạ tầng vật lý:
        - Switch `(6)` bỏ tag VLAN 101 trên packet và forward nó đến Router `(7)`
        - Router `(7)` định tuyến đường đi cho packet từ provider network 1 `(8)` tới provider network 2 `(9)`
        - Router forward packet tới switch `(10)`
        -Switch `(10)` thêm tag VLAN 102 vào packet và forward tới node Compute1 `(11)`
    - Trên node Compute2:
        - Physical interface `(12)` gỡ tag VLAN 102 khỏi packet và forward nó đến VLAN sub-interface (13) trên provider bridge
        - Các rules của Security group (14) trên provider bridge sẽ xử lý firewall và theo dõi kết nối của packet
        - Instance port tương ứngs trên provider bridge `(15)` sẽ forward packet tới interface của instance 2 `(16)` thông qua `veth` pair .
## **4) Self-service network**

### **4.1) Kiến trúc và thành phần**
### **4.2) Network traffic** 
- Trong self-service các instance sẽ sử dụng IPv4 Private . Để truy cập được interface , networking service sẽ làm nhiệm vụ SNAT ( Source Network Addresss Translation ) để truy cập ra mạng external . . Để từ các mạng có thể truy cập , các instance yêu cầu có một floating IP . Networking service thực hiện DNAT ( desnation network address translation ) từ IP Floating sang IP self-service
#### **4.2.1) Luồng Bắc-Nam 1 (North - South 1)** : instance có địa chỉ IP cố định
- Với các instance kèm IP v4 Floating, trên network node sẽ thực hiện SNAT để self-service có thể giao tiếp với mạng ngoài.
    - Instance ở node Compute1 và sử dụng mạng self-service 1
    - Instance gửi packet tới host ngoài internet

    <p align=center><img src=https://i.imgur.com/FCOZP3S.png></p>

- Đường đi gói tin :
    - Trên node Compute :
        - Instance interface (1) forward packet tới port tương ứng trên self-service bridge (2) thông qua veth pair
        - Các rules của Security group (3) trên self-service bridge sẽ xử lý firewall và theo dõi kết nối của packet
        - Self-service bridge forward packet tới VXLAN interface trên bridge (4) kèm VNI
        - Physical interface (5) cho phép mạng VXLAN interface forward packet tới node Network thông qua overlay network (6)
    - Trên node network :
        - Physical network (7) nhận packet từ Overlay network VXLAN interface sau đó forward tới self-service bridge port (8)
        - Self-service bridge router port (9) forward packet tới self-service network interface (10) trong rourter namespace
        - Đối với IPv4, router thực hiện SNAT trên packet thay đổi IP nguồn thành địa chỉ IP của router trên provider network và gửi nó đến địa chỉ IP gateway của provider network thông qua gateway interface trên provider network (11)
        - Router forward packet tới provider bridge router port (12)
        - VLAN sub-interface port (13) trên provider bridge sẽ forward packet tới physical network interface (14)
        - Provider physical network interface (14) gán tag VLAN 101 vào packet và forward nó ra internet thông quan physical network infrastructure (15).
#### **4.2.2) Luồng Bắc-Nam 2 (North - South 2)** : Instance with a floating IPv4 address
- Instance nằm trên node Compute1 và sử dụng self-service network 1
- Máy chủ trên internet gửi một packet tới instance
    <p align=center><img src=https://i.imgur.com/PNug8Et.png></p>
- Đường đi gói tin :
    - Trên node network :
        - Physical network infrastructure (1) forward packet tới provider physical network interface (2).
        - provider physical network interface(3) gỡ tag VLAN 101 và forward packet tới VLAN sub-interface (4) trên provider bridge
        - Provider bridge forward packet tới port gateway của self-service router trên provider network (5)
            - Đối với IPv4, router thực hiện DNAT trên packet thay đổi địa chỉ IP đích thành IP của instancetreen self-service network và gửi nó tới địa chỉ IP gateway trên self-service thông qua self-service interface (6).
        - Router forward packet tới self-service bridge router port (7)
        - self-service bridge forwards packet tới VXLAN interface (8) và kèm theo VNI 101
        - physical interface (9) forward pacekt tới node network thông qua overlay network (10).
    - Trên node Compute :
        - Physical interface (11) forward packet tới VXLAN interface (12) để mở packet
        - Các security group rules (13) trên self-service bridge xử lý firewall và theo dõi kết nối của packet
        - self-service bridge instance port (14) forwards packet tới interface của instance (15) thông qua veth
#### **4.2.3) Luồng Đông-Tây 1 (East-West 1)** : instance trên cùng một node
- Instance 1 nằm trên node Compute1 và sử dụng mạng self-service network 1
- Instance 2 nằm trên node Compute2 và sử dụng mạng self-service network 1
- Instance 1 gửi packet tới Instance 2
    <p align=center><img src=https://i.imgur.com/HZc17Qd.png></p>
- Đường đi gói tin :
    - Trên node Compute1 :
        - Instance interface (1) chuyển packet đến self-service port tương ứng (2)
        - Các security group rules (3) trên self-service bridge xử lý firewall và theo dõi kết nối của packet
        - self-service bridge forward packet tới VXLAN interface (4) kèm theo VNI 101
        - physical interface (5) forward packet tới node Compute2 thông qua overlay network (6)
    - Trên node Compute2 :
        - physical interface (7) forward packet tới VXLAN interface (8) để mở packet
        - Security group rules (9) của self-service bridge sẽ xử lý với firewall và theo dõi kết nối của packet
        - self-service bridge instance port (10) forward packet tới interface của instance (11) thông qua veth
#### **4.2.4) Luồng Đông-Tây 2 (East-West 2)** : instance trên 2 mạng khác nhau
- Instance1 nằm trên node Compute1 và sử dụng mạng self-service1
- Instance2 nằm trên node Compute1 và sử dụng mạng self-service2
- Instance1 gửi packet tới Instance2
    <p align=center><img src=https://i.imgur.com/XKfTjFz.png></p>
- Đường đi gói tin :
    - Trên node Compute :
        - Instance interface (1) chuyển packet đến self-service port tương ứng (2)
        - Các security group rules (3) trên self-service bridge xử lý firewall và theo dõi kết nối của packet
        - self-service bridge forward packet tới VXLAN interface (4) kèm theo VNI 101
        - Physical interface (5) cho phép mạng VXLAN interface forward packet tới node Network thông qua overlay network (6)
    - Trên node network :
        - physical interface (7) forward packet tới VXLAN interface (8) để mở packet
        - self-service bridge router port (9) forwards packet tới self-service network 1 interface (10) trên router namespace.
        - Router gửi packet tới IP tiếp theo, thường là IP gatewa của self-service network2 , thông qua self-service network2 interface (11)
        - Router forward pacekt tới self-service network 2 bridge router port (12).
        - self-service network 2 bridge forward packet tới VXLAN interface (13) để mở pacekt sử dụng VNI 102
        - physical network interface (14) của VXLAN interface gửi packet tới node compute thông qua overlay network (15)
    - Trên node Compute :
        - physical interface (16) gửi packet tới VXLAN interface (17) để mở packet
        - Security group rules (18) trên self-service bridge xử lý với firewall và theo dõi kết nối của packet
        - self-service bridge instance port (19) forward paceket tới interface của instance2 (20) thông qua veth pair.
------------
Tham khảo
- https://docs.openstack.org/neutron/train/admin/deploy-lb-provider.html
- https://docs.openstack.org/neutron/train/admin/deploy-lb-selfservice.html