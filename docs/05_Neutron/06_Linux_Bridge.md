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
## **2) Provider network**
### **2.1) Kiến trúc và thành phần**
- 
### **2.2) Network traffic**
#### **2.2.1) Luồng Bắc-Nam (North - South)** : instance có địa chỉ IP cố định
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
#### **2.2.2) Luồng Đông-Tây 1 (East-West 1)** : instance trên cùng một node
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
#### **2.2.3) Luồng Đông-Tây 2 (East-West 2)** : instance trên 2 mạng khác nhau
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
## **3) Self-service network**
### **3.1) Kiến trúc và thành phần**
### **3.2) Network traffic** 
