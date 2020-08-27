# Quá trình khởi tạo và quản lý VNC trong Nova
- **VNC (Virtual Network Computing)** là một giao diện điều khiển đồ họa từ xa với sự hỗ trợ rộng rãi giữa nhiều hypervisor và client. **noVNC** cung cấp hỗ trợ **VNC** thông qua trình duyệt web.
## **1) Luồng hoạt động khi kết nối VNC**

<img src=https://i.imgur.com/bh6KMqM.png>

1. User kết nối API để lấy access_url, ví dụ: http://ip:port/?path=%3Ftoken%3Dxyz
2. User gán URL lấy được đó vào browser của họ hoặc sử dụng nó như một client parameter.
3. Browser hoặc client kết nối tới proxy
4. Proxy sẽ giao tiếp với `nova-consoleauth` để xác thực token của người dùng, và maps token với compute host và port của VNC server cho instance. Để cấu hình địa chỉ cho các host thì sửa tùy chọn `vnc.server_proxyclient_address` trong file cấu hình của nova compute là `nova.conf`
5. Proxy bắt đầu kết nối với VNC server và tiếp tục giữ kết nối đó cho tới khi kết thúc phiên.

## **2) Cấu hình noVNC**
- Chỉnh sửa file `/etc/nova/nova.conf` trên node `controller` :
    ```ini
    ....
    [vnc]
    enabled = true
    server_listen = $my_ip
    server_proxyclient_address = $my_ip
    ....
    ```
- Chỉnh sửa file `/etc/nova/nova.conf` trên các node compute :
    ```ini
    ....
    [vnc]
    enabled = true
    server_listen = $my_ip
    server_proxyclient_address = $my_ip
    novncproxy_base_url = http://IP_CONTROLLER:6080/vnc_auto.html
    ....

    ```
    - Lưu ý : `IP_CONTROLLER` phải là ip có thể ra ngoài và truy cập được trên trình duyệt. Nên cấu hình rõ IP, không ghi là `controller` .
## **3) Cấu hình thời gian hết hạn token**
- Mặc định khoảng thời gian này là `600s`
- Có thể sửa lại bằng cách chỉnh sửa file `/etc/nova/nova.conf`
    ```ini
    ...
    [consoleauth]
    token_ttl=600
    ...
    ```
## **4) Show đường dẫn console của instance**
- Có thể sử dụng lệnh sau trong terminal để lấy đường dẫn console `novnc` truy cập instance trên trình duyệt :
    ```
    # openstack console url show <instance_name|instance_id>
    ```
- **VD :**
    ```
    # openstack console url show VM06
    ```
    <img src=https://i.imgur.com/ikl4z5j.png>