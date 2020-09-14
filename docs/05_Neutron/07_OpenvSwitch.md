# OpenvSwitch
## **1) Giới thiệu về SDN và Openflow**
### **1.1) SDN - Sofware Define Networking**
- **SDN** (*Software Defined Networking*) hay ***mạng điều khiển bằng phần mềm*** là một kiến trúc đem tới sự tự động, dễ dàng quản lí, tiết kiệm chi phí và có tính tương thích cao, đặc biệt phù hợp với những ứng dụng yêu cầu tốc độ băng thông cũng như sự tự dộng ngày nay. 
- Kiến trúc này tách riêng hai chức năng là quản lí và truyền tải dữ liệu. **SDN** dựa trên giao thức luồng mở (**OpenFlow**) và là kết quả nghiên cứu của Đại học Stanford và California Berkeley. 
- **SDN** tách định tuyến và chuyển các luồng dữ liệu riêng rẽ và chuyển kiểm soát luồng sang thành phần mạng riêng có tên gọi là thiết bị kiểm soát luồng (Flow Controller).
- Tóm lại có 3 ý chính đối với **SDN** đó là:
    - Tách biệt phần quản lí (control plane) với phần truyền tải dữ liệu (data plane).
    - Các thành phần trong network có thể được quản lí bởi các phần mềm được lập trình chuyên biệt.
    - Tập trung vào kiểm soát và quản lí network.
    Cùng quay trở lại quá khứ, khi mà người ta vẫn sử dụng Ethernet Hub. Về bản chất, thiết bị này chỉ làm công việc lặp đi lặp lại đó là mỗi khi nhận dữ liệu, nó lại forward tới tất cả các port mà nó kết nối.

