# VXLAN
## **1) Giới thiệu**
- **VXLAN** - ***Virtual eXtensible Local Area Network*** là một giao thức tunneling cho phép đóng gói traffic Ethernet (layer2) di chuyển qua mạng IP (Layer3)
- Mạng layer 2 truyền thống phát sinh một số vấn đề vì 3 nguyên nhân chính sau :
    - Giao thức **STP** (*Spanning-Tree*) :
        - Giao thức **STP** sẽ chặn bất cứ đường link thừa nào để tránh gây ra loop. Các link bị chặn sẽ góp phần tạo thành 1 topology không bị loop, và khiến network hoạt động tốt, nhưng cũng có nghĩa là chúng ta phải trả thêm phí cho đường mạng mà ta không sử dụng được. Ta hoàn toàn có thể thực hiện chuyển mạch (switch) trên mạng layer 3, tuy nhiên một số công nghệ vẫn yêu cầu mạng layer 2 .
    - Giới hạn số lương VLAN :
        - `VLAN-ID` có độ dài `12bit`, có nghĩa ta chỉ có thể tạo tối đa `4094` VLAN (VLAN `0` và `4095` đã được sử dụng). Chỉ có `4094` VLAN sẽ là một vấn đề cho 1 Data Center. **VD :** DC có 500 khách hàng - với `4094` VLAN, ta chỉ có thể đưa mỗi khách hàng `8` VLAN
    - Bảng **MAC** lớn :
        - Do sự phát triển của ảo hóa, số lượng các địa chỉ trong bảng MAC của **switch** sẽ nhanh chóng nở ra. Trước khi ảo hóa, **switch** chỉ phải học 1 địa chỉ MAC trên mỗi **switchport**. Sau khi ảo hóa, rất nhiều VM hoặc container chạy trên một server vậy lý. Mỗi VM có một NIC ảo và một địa chỉ MAC ảo. **Switch** sẽ phải học rất nhiều địa chỉ MAC đẩy vào 1 **switchport** .
        - Một **Top-of-Rack** (**ToR**) **switch** trong Data Center có thể kết nối tới `24` hoặc `48` server vật lý . Một Data Center sẽ có rất nhiều **rack**, vì vậy một **switch** phải lưu trữ rất nhiều địa chỉ **MAC** của tất cả các VM giao tiếp với VM khác. Bảng MAC sẽ rất lớn so với các mạng không có server ảo hóa .
- **VLAN** sử dụng một công nghệ đóng gói (encapsulation) giống như VLAN để đóng gói các Ethernet frame của layer 2 trong mô hình OSI bên trong datagram UDP của layer 4, sử dụng `4789` như destination UDP port được chỉ định bởi **IANA** .
- **VXLAN** sử dụng IP (cả unicast và multicast) để truyền gói tin đi trong mạng cho phép khả năng mở rộng vượt trội so với VLAN đang sử dụng `802.1q` hiện nay .
## **2) Một số khái niệm trong VXLAN**
### **2.1) Overlay và Underlay**
<p align=center><img src=https://i.imgur.com/taT8cLN.png width=40%></p>

- Một mạng **overlay** là một mảng ảo chạy trên mạng **underlay** vật lý .
- Với **VXLAN**, mạng overlay này là mạng Ethernet layer 2, mạng **underlay** mà mạng IP layer 3. **Underlay network** còn có tên gọi khác là **transport network** .
- Mạng **overlay** và **underlay** độc lập với nhau . **Overlay** là mạng ảo và yêu cầu mạng **underlay**, tuy nhiên bất cứ thay đổi gì dưới mạng **overlay** đều không ảnh hưởng đến mạng **underlay**
- Có thể thêm và xóa các link trong mạng **underlay**, và chỉ cần các giao thức định tuyến vẫn hoạt động bình thường, mạng **overlay** cũng sẽ không bị ảnh hưởng .
### **2.2) VNI**
- **VXLAN Network Identifier (VNI)** định danh VXLAN và có chức năng tương tự như VLAN ID trong VLAN truyền thống. **VNI** có độ dài `24bit`, có nghĩa có thể có tối đa `16,777.215` (~`16 triệu`) VXLAN. Đây là khá nhiều so với con số `4094` trong VLAN với `12bit` VLAN ID. Có thể tạo ra số lượng lớn VXLAN, có nghĩa một nhà cung cấp dịch vụ lớn có thê có hàng ngìn khách hàng và cung cấp bao nhiêu VLAN cũng được cho khách hàng nếu họ muốn .
### **2.3) VTEP**
- **VXLAN tunnel endpoint (VTEP)** là thiết bị chịu trách nhiện đóng gói và mở gói tin layer2 . Thiết bị này kết nối giữa mạng **overlay** và **underlay** .
- **VTEP** tồn tại ở 2 dạng :
    - Software (host-based) :
        - Sử dụng trong các hypervisor ESXi, Hyper-V hay QEMU-KVM. Những hypervisor sẽ sử dụng các switch ảo để hỗ trợ VXLAN .
            <p align=center><img src=https://i.imgur.com/rE27YoD.png width=50%></p>

            > Hình trên cho thấy sự kết nối giữa các switch ảo của các hypervisor . Mạng underlay sẽ không nhận ra sự tồn tại của **VXLAN**
    - Hardware (gateway) :
        - Hardware VTEP là một router, switch hoặc firewall hỗ trợ **VXLAN**. Cũng có thể gọi nó là một **VXLAN gateway** bởi nó gộp các segment của **VLAN** thông thường và **VXLAN** vào chung một domain layer 2. Một số switch hỗ trợ **VXLAN** với **ASICs**, đưa đến hiệu năng tốt hơn cho **VXLAN** hơn là VTEP software.
            <p align=center><img src=https://i.imgur.com/QhAx34r.png width=50%></p>

        > Hình trên cho thấy VXLAN tunnel giữa 2 switch vật lý . Các thiết bị kết nối đến switch vật lý sẽ không nhận ra sự tồn tại của **VXLAN** .
### **2.4) Interface**
- Mỗi **VTEP** có 2 kiểu interface :
    - **VTEP IP interface** : kết nối **VTEP** tới mạng underlay với IP duy nhất . Interface này sẽ đóng gói và mở gói Ethernet frame .
    - **VNI interface** : một interface ảo giữ cho các network traffic phân cách với các interface vậy lý, giống như một SVI interface .
- Một **VTEP** có thể có nhiều **VNI interface**, nhưng chúng luôn đi cùng với **VTEP IP interface** giống nhau.

    <p align=center><img src=https://i.imgur.com/ytXHQ8w.png></p>

## **3) Cấu trúc gói tin trong VXLAN**
<img src=https://i.imgur.com/5km0e7U.png>

- Cấu trúc gói tin trong **VXLAN** :
    - **Outer MAC header** : chứa địa chỉ MAC của source VTEP và địa chỉ MAC của next-hop router. Mỗi router trên đường đi của gói tin sẽ tự động ghi đè header để địa chỉ MAC nguồn là của chính nó và địa chỉ đích là địa chỉ MAC của hop tiếp theo .
    - **Outer IP header** – Chứa địa chỉ IP source và IP đích của VTEP
    - **Outer UDP header** – Chứa port UDP nguồn và đích
    - **VXLAN header** – Chứa `24bit` VNI
    - **Original L2 Frame** – Chứa frame L2 gốc
## **4) Cách thức hoạt động của VXLAN**
- **VXLAN** hoạt động dựa trên việc gửi các frame thông qua giao thức IP Multicast.
- Trong quá trình cấu hình VXLAN, cần cấp phát địa chỉ IP multicast để gán với **VXLAN** sẽ tạo. Mỗi IP multicast sẽ đại diện cho một **VXLAN**.
- Dưới đây là hoạt động chi tiết các frame đi qua VTEP và đi qua mạng vật lý trong mạng. **VXLAN** triển khai trên một mạng logic với mô hình như sau:
    <p align=center><img src=https://i.imgur.com/HyaKWZ4.png width=70%></p>
### **4.1) VM gửi request tham gia vào group multicast**
- Giả sử một mạng logic trên 4 host như hình. Topo mạng vật lý cung cấp một VLAN `2000` để vận chuyển các lưu lượng **VXLAN**. Trong trường hợp này, chỉ IGMP snooping và IGMP querier được cấu hình trên mạng vật lý. Một vài bước sẽ được thực hiện trước khi các thiết bị trên mạng vật lý có thể xử lý các gói tin multicast .
- **IGMP Packet Flow**
    <p align=center><img src=https://i.imgur.com/rOL9Uvo.png width=70%></p>

    - Máy ảo VM (MAC1) trên Host 1 được kết nối tới một mạng logical layer 2 mà có VXLAN 5001 ở đó.
    - VTEP trên Host 1 gửi bản tin IGMP để join vào mạng và join vào nhóm multicast 239.1.1.100 để kết nối tới VXLAN 5001.
    - Tương tự, máy ảo VM (MAC2) trên Host 4 được kết nối tới mạng mà có VXLAN 5001.
    - VTEP trên Host 4 gửi bản tin IGMP join vào mạng và join vào nhóm multicast 239.1.1.100 để kết nối tới VXLAN 5001.
    - Host 2 và Host 3 VTEP không join nhóm multicast bởi vì chúng không có máy ảo chạy trên nó và cần kết nối tới VXLAN 5001. Chỉ VTEP nào cần tham gia vào nhóm multicast mới gửi request join vào nhóm.
- **Multicast Packet Flow**
    <p align=center><img src=https://i.imgur.com/Rzb6Jfx.png width=70%></p>

    - Máy ảo VM (MAC1) trên Host 1 sinh ra một frame broadcast.
    - VTEP trên Host 1 đóng gói frame broadcast này vào một UDP header với IP đích là địa chỉ IP multicast 239.1.1.100
    - Mạng vật lý sẽ chuyển các gói tin này tới Host 4 VTEP, vì nó đã join vào nhóm multicast 239.1.1.100. Host 2 và 3 VTEP sẽ không nhận được frame broadcast này.
    - VTEP trên Host 4 đầu tiên đối chiếu header được đóng gói, nếu 24 bit VNI trùng với ID của VXLAN. Nó sẽ decapsulated lớp gói được VTEP host 1 đóng vào và chuyển tới máy ảo VM đích (MAC2).
### **4.2) VTEP học và tạo bảng forwarding**
- Ban đầu, mỗi VTEP sau khi join vào nhóm IP multicast đều có 1 bảng forwarding table như dưới đây:
    <p align=center><img src=https://i.imgur.com/ShH7YBd.png width=70%></p>
- Các bước sau sẽ được thực hiện để VTEP học và ghi vào bảng forwarding table:
    - Đầu tiên, 1 bản tin ARP request được gửi từ VM MAC1 để tìm địa chỉ MAC của máy ảo đích nó cần gửi tin đến VM MAC2 trên Host2. ARP request là bản tin broadcast.
        <p align=center><img src=https://i.imgur.com/4YY5Xu3.png width=70%></p>
    - Host 2 VTEP – Forwarding table entry:
        - VM trên Host 1 gửi bản tin ARP request với địa chỉ MAC đích là“FFFFFFFFFFF”
        - VTEP trên Host 1 đóng gói vào frame Ethernet broadcast vào một UDP header với địa chỉ IP đích multicast và địa chỉ IP nguồn 10.20.10.10 của VTEP.
        - Mạng vật lý sẽ chuyển gói tin multicast tới các host join vào nhóm IP multicast “239.1.1.10”.
        - VTEP trên Host 2 nhận được gói tin đã đóng gói. Dựa vào outer và inner header, nó sẽ tạo một entry trong bảng forwarding chỉ ra mapping giữa MAC của máy VM MAC1 ứng với VTEP nguồn và địa chỉ IP của nó. VTEP cũng kiểm tra VNI của gói tin để quyết định sẽ chuyển tiếp gói tin vào trong cho máy ảo VM bên trong nó hay không.
        -Gói tin được de-encapsulated và chuyển vào tới VM mà được kết nối tới VXLAN 5001.
    - Hình sau minh họa cách mà VTEP tìm kiếm thông tin trong forwarding table để gửi unicast trả lời lại từ VM từ VTEP 2:
        <p align=center><img src=https://i.imgur.com/pOaOwWp.png width=70%></p>
        
        - Máy ảo VM MAC2 trên Host 2 đáp trả lại bản tin ARP request bằng cách gửi unicast lại gói tin với địa chỉ MAC đích là địa chỉ MAC1
        - Sau khi nhận được gói tin unicast đó, VTEP trên Host 2 thực hiện tìm kiếm thông tin trong bảng forwarding table và lấy được thông tin ứng với MAC đích là MAC 1. VTEP sẽ biết rằng phải chuyển gói tin tới máy ảo VM MAC 1 bằng cách gửi gói tin tới VTEP có địa chỉ “10.20.10.10”.
        - VTEP tạo bản tin unicast với địa chỉ đích là “10.20.10.10” và gửi nó đi.
    - Trên Host 1, VTEP sẽ nhận được gói tin unicast và cũng học được vị trí của VM MAC2 như hình sau:
        <p align=center><img src=https://i.imgur.com/u5GvFZZ.png width=70%></p>

    - Host 1 VTEP – Forwarding table entry
        - Gói tin được chuyển tới Host 1
        - VTEP trên Host 1 nhận được gói tin. Dựa trên outer và inner header, nó tạo một entry trong bảng forwarding ánh xạ địa chỉ MAC 2 và VTEP trên Host 2. VTEP cũng check lại VNI và quyết định gửi frame vào các VM bên trong.
        - Gói tin được de-encapsulated và chuyển tới chính xác VM có MAC đích trùng và nằm trong VXLAN 5001.
## **5) Cấu hình Neutron sử dụng VXLAN**
- **B1 :** Sao lưu file cấu hình `ml2` :
    ```
    # cp /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugins/ml2/ml2_conf.ini.bak
    ```
- **B2 :** Cấu hình ML2 driver :
    ```ini
    [ml2]
    ...
    type_drivers = vxlan
    tenant_network_types = vxlan
    mechanism_drivers = linuxbridge
    ```
    > `mechanism_drivers = openvswitch` nếu sử dụng **OpenvSwitch**
- **B3 :** Cấu hình dải VNI range cho VXLAN :
    ```ini
    [ml2_type_vxlan]
    ...
    vni_ranges = 1:1000
    ```
    > Có thể sử dụng nhiều khoảng giá trị phân cách nhau bằng dấu "`,`" : `vni_ranges = 1001:2000,3001:4000,5001:6000`
- **B4 :** Trong file `/etc/neutron/plugins/ml2/linuxbridge_agent.ini` (đối với **OvS** là `/etc/neutron/plugins/ml2/openvswitch_agent.ini`), cấu hình `local_ip` là địa chỉ IP VTEP (hay chính là IP dải data VM) :
    ```ini
    [vxlan]
    ...
    local_ip = 10.10.240.10
    ```
    > Bước này thực hiện trên tất cả các node
### **Kiểm tra lại các VNI đã được gán cho các dải mạng**
- Hiển thị các dải mạng đang có :

    <img src=https://i.imgur.com/wfoknnq.png>

- Hiển thị chi tiết các dải mạng :

    <img src=https://i.imgur.com/2xYEnWW.png>
