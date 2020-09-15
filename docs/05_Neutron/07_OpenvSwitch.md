# OpenvSwitch
## **1) Giới thiệu về SDN và Openflow**
### **1.1) SDN - Sofware Define Networking**
- **SDN** hay ***mạng điều khiển bằng phần mềm*** (**Software Defined Networking**) được dựa trên cơ chế tách riêng việc kiểm soát một luồng mạng với luồng dữ liệu (**control plane** và **data plane**). 
- **SDN** dựa trên giao thức luồng mở (**OpenFlow**) và là kết quả nghiên cứu của **Đại học Stanford** và **California Berkeley**. SDN tách định tuyến và chuyển các luồng dữ liệu riêng rẽ và chuyển kiểm soát luồng sang thành phần mạng riêng có tên gọi là thiết bị kiểm soát luồng (**Flow Controller**). Điều này cho phép luồng các gói dữ liệu đi qua mạng được kiểm soát theo lập trình. - Trong **SDN**, **control plane** được tách ra từ các thiết bị vật lý và chuyển đến các bộ điều khiển. Bộ điều khiển này có thể nhìn thấy toàn bộ mạng lưới, và do đó cho phép các kỹ sư mạng làm cho chính sách chuyển tiếp tối ưu dựa trên toàn bộ mạng. Các bộ điều khiển tương tác với các thiết bị mạng vật lý thông qua một giao thức chuẩn **OpenFlow**. 
- Kiến trúc của **SDN** gồm 3 lớp riêng biệt: lớp ứng dụng, lớp điều khiển, và lớp cơ sở hạ tầng (lớp chuyển tiếp) :

    <p align=center><img src=https://i.imgur.com/JDGdNvn.png width=70%></p>

    - **Lớp ứng dụng:** Là các ứng dụng kinh doanh được triển khai trên mạng, được kết nối tới lớp điều khiển thông qua các API, cung cấp khả năng cho phép lớp ứng dụng lập trình lại (cấu hình lại) mạng (điều chỉnh các tham số trễ, băng thông, định tuyến, …) thông qua lớp điều khiển.
    - **Lớp điều khiển:** Là nơi tập trung các bộ điều khiển thực hiện việc điều khiển cấu hình mạng theo các yêu cầu từ lớp ứng dụng và khả năng của mạng. Các bộ điều khiển này có thể là các phần mềm được lập trình.
    - **Lớp cơ sở hạ tầng:** Là các thiết bị mạng thực tế (vật lý hay ảo hóa) thực hiện việc chuyển tiếp gói tin theo sự điều khiển của lớp điểu khiển. Một thiết bị mạng có thể hoạt động theo sự điều khiển của nhiều bộ điều khiển khác nhau, điều này giúp tăng cường khả năng ảo hóa của mạng.
### **1.2) OpenFlow**
- **OpenFlow** là tiêu chuẩn đầu tiên, cung cấp khả năng truyền thông giữa các giao diện của lớp điều khiển và lớp chuyển tiếp trong kiến trúc **SDN**. **OpenFlow** cho phép truy cập trực tiếp và điều khiển mặt phẳng chuyển tiếp của các thiết bị mạng như switch và router, cả thiết bị vật lý và thiết bị ảo, do đó giúp di chuyển phần điều khiển mạng ra khỏi các thiết bị chuyển mạch thực tế tới phần mềm điều khiển trung tâm. Các quyết định về các luồng traffic sẽ được quyết định tập trung tại OpenFlow Controller giúp đơn giản trong việc quản trị cấu hình trong toàn hệ thống. Một thiết bị OpenFlow bao gồm ít nhất 3 thành phần:
    - Secure Channel: kênh kết nối thiết bị tới bộ điều khiển (controller), cho phép các lệnh và các gói tin được gửi giữa bộ điều khiển và thiết bị.
    - OpenFlow Protocol: giao thức cung cấp phương thức tiêu chuẩn và mở cho một bộ điều khiển truyền thông với thiết bị.
    - Flow Table: một liên kết hành động với mỗi luồng, giúp thiết bị xử lý các luồng thế nào.

    <p align=center><img src=https://i.imgur.com/olqoDcw.png width=70%></p>

- Một số lợi ích khi sử dụng OpenFlow:
    - Công nghệ SDN trên cơ sở OpenFlow cho phép nhân viên IT giải quyết các ứng dụng băng thông cao và biến đổi động hiện nay, khiến cho mạng thích ứng với các nhu cầu kinh doanh thay đổi, và làm giảm đáng kể các hoạt động và quản lý phức tạp. Những lợi ishc mà các doanh nghiệp cà nhà khai thác mạng có thể đạt được thông qua kiến trúc SDN trên cơ sở OpenFlow bao gồm:
    - Tập trung hóa điều khiển trong môi trường nhiều nhà cung cấp thiết bị: phần mềm điều khiển SDN có thể điều khiển bất kỳ thiết bị mạng nào cho phép OpenFlow từ bất kỳ nhà cung cấp thiết bị nào, bao gồm switch, router, và các switch ảo.
    - Giảm sự phức tạp thông qua việc tự động hóa: kiến trúc SDN trên cơ sở OpenFlow cung cấp một framework quản lý mạng tự động và linh hoạt. Từ framework này có thể phát triển các công cụ tự động hóa các nhiệm vụ hiện đang được thực hiện bằng tay.
    - Tốc độ đổi mới cao hơn: việc áp dụng OpenFlow cho phép các nhà khai thác mạng lập trình lại mạng trong thời gian thực để đạt được các nhu cầu kinh doanh và yêu cầu người dùng cụ thể khi có sự thay đổi.
    - Gia tăng độ tin cậy và khả năng an ninh của mạng: các nhân viên IT có thể định nghĩa các trạng thái cấu hình và chính sách ở mức cao, và áp dụng tới cơ sở hạ tầng thông qua OpenFlow. Kiến trúc SDN trên cơ sở OpenFlow cung cấp điều khiển và tầm nhìn hoàn chỉnh trên mạng, nên có thể đảm bảo điều khiển truy nhập, định hình lưu lượng, QoS, an ninh, và các chính sách khác được thực thi nhất quán trên toàn bộ cơ sở hạ tầng mạng không dây và có dây, bao gồm cả các văn phòng chi nhánh, các cơ sở chính và DC.
    - Điều khiển mạng chi tiết hơn: mô hình điều khiển trên cơ sở flow của OpenFlow cho phép nhân viên IT áp dụng các chính sách tại mức chi tiết, bao gồm phiên, người dùng, thiết bị, và các mức ứng dụng, trong một sự trừu tượng hóa cao, tự động điều chỉnh thích hợp.
    - Tốt hơn với trải nghiệm người dùng: bằng việc tập trung hóa điều khiển mạng và tạo ra trạng thái thông tin có sẵn cho các ứng dụng mức cao hơn, kiến trúc SDN trên cơ sở OpenFlow có thể đáp ứng tốt hơn cho các nhu cầu thay đổi của người dùng
## **2) OpenvSwitch**
### **2.1) Giới thiệu**
- **OpenvSwitch (OVS)** là một dự án về chuyển mạch ảo đa lớp (multilayer). Mục đích chính của OpenvSwitch là cung cấp lớp chuyển mạch cho môi trường ảo hóa phần cứng, trong khi hỗ trợ nhiều giao thức và tiêu chuẩn được sử dụng trong hệ thống chuyển mạch thông thường. OpenvSwitch hỗ trợ nhiều công nghệ ảo hóa dựa trên nền tảng Linux như Xen/XenServer, KVM, và VirtualBox.

    <p align=center><img src=https://i.imgur.com/eyqJNtq.png width=70%></p>

- OpenvSwitch hỗ trợ các tính năng sau: 
    - VLAN tagging & 802.1q trunking 
    - Standard Spanning Tree Protocol (802.1D) 
    - LACP 
    - Port Mirroring (SPAN/RSPAN) 
    - Tunneling Protocols 
    - QoS
- Các thành phần chính của OpenvSwitch:
    - `ovs-vswitchd`: thực hiện chuyển đổi các luồng chuyển mạch.
    - `ovsdb-server`: là một lightweight database server, cho phép ovs-vswitchd thực hiện các truy vấn đến cấu hình.
    - `ovs-dpctl`: công cụ để cấu hình các switch kernel module.
    - `ovs-vsctl`: tiện ích để truy vấn và cập nhật cấu hình ovs-vswitchd.
    - `ovs-appctl`: tiện ích gửi command để chạy OpenvSwitch.
### **2.2) So sánh LinuxBridge và OpenvSwitch**
- **OpenvSwitch**
    - Ưu điểm :
        - Dễ quản lý mạng hơn : với **OpenvSwitch**, sẽ thuận tiện hơn cho người quản trị quản lý và giám sát trạng thái mạng và luồng dữ liệu trong môi trường cloud .
        - Hỗ trợ nhiều giao thức tunnel : **OVS** hỗ trợ **GRE**, **VXLAN**, **IPSec**,...
        - Được tích hợp trong **SDN** : **OVS** được tích hợp trong **SDN** vì vật nó có thể được cài bằng cách sử dụng một **OpenStack plug-in** hoặc trực tiếp từ **SDN controller**, như **OpenDaylight**
    - Nhược điểm :
        - Thiếu ổn định : **OpenvSwitch** có một số vấn đề về tính ổn định như Kernetl panics, ovs-switched segfaults, and data corruption.
        - Vận hành phức tạp : **OpenvSwitch** là một giải pháp phức tạp, nhiều chức năng, khó học, cài đặt và vận hành
- **LinuxBridge**
    - Ưu điểm :
        - Ổn định và tin cậy : **LinuxBridge** đã được sử dụng nhiều năm, sự ổn định và tin tưởng đã được chứng minh .
        - Dễ cài đặt : **LinuxBridge** là một phần của việc cài đặt **Linux** và không cần cài thêm gói hỗ trợ nào khác .
        - Thuận tiện cho việc troubleshoot : **LinuxBridge** là một giải pháp đơn giản và dễ vận hành hơn **OpenvSwitch**
    - Nhược điểm :
        - Ít chức năng : **LinuxBridge** không hỗ trợ **Neutron DVR**,VXLan và nhiều chức năng khác .
        - Ít được support : **LinuxBridge** sẽ ít cộng đồng hỗ trợ hơn **OpenvSwitch**
### **2.3) Các lệnh thường dùng trong OpenvSwitch**
### **2.4) Triển khai OpenvSwitch**
#### **2.4.1) Trên node controller**
- **B1 :**