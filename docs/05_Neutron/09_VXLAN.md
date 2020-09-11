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
    § Outer IP header – Contains the IP addresses of the source and destination VTEPs.
§ Outer UDP header – Contains source and destination UDP ports:

 - Source UDP port – The VXLAN protocol repurposes this standard field in a UDP
packet header. Instead of using this field for the source UDP port, the protocol uses
it as a numeric identifier for the particular flow between VTEPs. The VXLAN
standard does not define how this number is derived, but the source VTEP usually
calculates it from a hash of some combination of fields from the inner Layer 2
packet and the Layer 3 or Layer 4 headers.
§ Destination UDP port – The VXLAN UDP port. The Internet Assigned Numbers
Authority (IANA) allocates port 4789 to VXLAN.
§ VXLAN header – Contains the 24-bit VNI.
§ Original Ethernet Frame – Contains the original Layer 2 Ethernet frame.
In total, VXLAN encapsulation adds between 50 and 54 bytes of additional header
information to the original Ethernet frame. Because this can result in Ethernet frames
that exceed the default 1514 byte MTU, best practice is to implement jumbo frames
throughout the network.