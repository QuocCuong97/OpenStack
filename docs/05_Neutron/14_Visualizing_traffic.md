# Visualizing traffic in Neutron
## **1) LinuxBridge**
- Khi một Ethernet frame đi từ một instance đến một mạng vật lý từ xa, nó sẽ đi qua 3 hoặc 4 device sau :
    - Tap interface : `tapN`
    - Linux Bridge : `brgXXXX`
    - VXLAN interface : `vxlan-Z` (`z` là VNI)
    - VLAN interface : `ethX.Y` (`X` là tên interface và `Y` là VLAN ID)
    - Physical interface : `ethX` (`X` là interface)
### **VLAN**
- Hình sau cho thấy một cloud **OPS** cơ bản chứa một `vlan100` để sử dụng cho instance bên trong nó :

    <img src=https://i.imgur.com/3VqZTwt.png>

    - Trong hình, 3 instance kết nối vào một LinuxBridge tên `brqXXXX` thông qua các tap interface riêng biệt. Khi instance được tạo ra trong mạng `VLAN100`, một interface ảo tên là `eth1.100` sẽ tự đọng được tạo ra và kết nối tới **bridge**. Interface `eth1.100` sẽ link đến `eth1`. Khi traffic từ instance đi qua LinuxBridge và đi ra ngoài physical interface , interface `eth1.100` sẽ đánh tag cho traffic đo là VLAN 100 và gỡ nó trên `eth1`. Traffic đi vào ngược lại sẽ được `eth1.100` bỏ tag và forward đúng đến instance thông qua bridge
    - Khi sử dụng lệnh `brctl show` sẽ được kết quả như sau :

        <img src=https://i.imgur.com/0SMch6q.png>
        
        >Bridge ID sẽ được tự động gen ra dựa vào parent NIC của interface VLAN (ở đây là `eth1`)
- Trường hợp nhiều VLAN :

    <img src=https://i.imgur.com/HZMpUam.png>

### **Flat**
- Một mạng flat trong **Neutron** là một mạng không diễn ra quá trình gán tag VLAN. Không giống như mạng VLAN gắn tag, mạng **flat** yêu cầu 1 physical interface của host gắn thẳng vào linuxbridge. Có nghĩa là chỉ 1 mạng flat có thể tồn tại giữa 1 bridge và 1 interface .

    <img src=https://i.imgur.com/nmMimMy.png>

    - `eth1` kết nối tới bridge tên là `brqXXXX` cùng với 3 tap interface đại diện cho 3 instance. Linux kernel sẽ không thực hiện gán tag VLAN .

    - Khi thực hiện lệnh `brctl show` trên compute node :
    
        <img src=https://i.imgur.com/Glk2WLL.png>

- Khi nhiều mạng **flat** được tạo ra, một physical interface sẽ phải gắn vào nó

    <img src=https://i.imgur.com/kZRufwW.png>

    - Khi thực hiện lệnh `brctl show` trên compute node :

        <img src=https://i.imgur.com/SGCJCGV.png>

    - Với 2 mạng flat, host sẽ không thể thực hiện bất cứ hành động tab VLAN nào qua bridge. Các instance ở 2 bridge khác nhau sẽ cần một router để giao tiếp với nhau .
### **VXLAN**
- Khi một mạng **VXLAN** được tạo ra, LinuxBridge agent sẽ tạo ra một VXLAN interface sử dụng `iproute2` và kết nối nó với network bridge thay cho một physical interface hoặc tagged interface. VXLAN interface sẽ chứa thông tin như VNI và địa chỉ VTEP. Khi L2 driver được cấu hình, **Neutron** sẽ chuẩn bị trước một **forwarding database** chứa các địa chỉ MAC của instance và các địa chỉ VTEP tương ứng. Khi một packet đi qua bridge, host sẽ quyết định cách forward packet đi đâu dựa vào **forwarding database**. Nếu không tìm thấy entry nào, **Neutron** sẽ forward gói tin ra local interface tương ứng . Để xem **forwarding database** (hay **bridge table**) trên từng host, sử dụng lệnh :
    ```
    # bridge fdb show
    ```
    <img src=https://i.imgur.com/KzIbC6E.png>
### **Local**
- Khi tạo một local network trong **Neutron**, hoàn toàn không thể gán cho nó một VLAN ID hay 1 physical interface . **LinuxBridge** sẽ tạo ra một bridge và chỉ kết nối tap interface của instance với bridge. Instance ở trong cùng một mạng local sẽ kết nối với cùng một **bridge** và có thể giao tiếp thoải mái với nhau . Bởi vì không có physical hay VLAN interface kết nối đến bridge, traffic giữa các instance sẽ bị giới hạn trong host chứa chúng .

    <img src=https://i.imgur.com/gK2iqgp.png>

    - Trong hình, 2 mạng local được tạo ra sẽ 2 bridge riêng: `brqXXXX` và `brqYYYY`. Instance kết nối tới cùng 1 bridge có thể giao tiếp được với nhau nhưng không thể giao tiếp ở ngoài bridge. Không có cơ chế nào cho phép traffic giữa các instance nối đến các bridge khi sử dụng mạng local
## **2) OpenvSwitch**