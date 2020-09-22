# Metadata service trong DHCP namespace
## **1) Cách hoạt động mặc định của metadata service (trong router namespace)**
- Trong **OpenStack**, mặc định cần **L3 agent** để giúp metadata service hoạt động, có nghĩa cần gắn tenant network cho một **neutron router**. Trong router namesoace, một tiến trình metadata proxy sẽ chạy để xử các request metadata, iptable rule sẽ redirect metadata request về cho metadata proxy .
- Tiến trình `neutron-ns-metadata-proxy` trong router namespace listen port `9697` :
    ```
    # ip netns exec qrouter-045c5928-2a51-4b3e-a5fb-9a1e4386b13b netstat -ntlp | grep 9697
    ```
    <img src=https://i.imgur.com/AU3O1Qe.png>

- Rule `iptables` redirect metadata request về cho metadata proxy :
    ```
    # ip netns exec qrouter-045c5928-2a51-4b3e-a5fb-9a1e4386b13b iptables-save | grep REDIRECT
    ```
    <img src=https://i.imgur.com/Gw5dHtQ.png>

## **2) Metadata service trong DHCP namespace**
- **OpenStack** có tùy chọn cho phép kích hoạt dịch vụ **metadata proxy** trong DHCP namespace, theo đó metadata service có thể làm việc trong môi trường tenant network cô lập, không cần kết nối đến **neutron router** .
- Để kích hoạt dịch vụ, cần cấu hình file `/etc/neutron/dhcp_agent.ini` :
    ```ini
    [DEFAULT]
    ...
    enable_isolated_metadata = True
    ...
    ```
    - Sau khi thay đổi cấu hình cần restart lại dịch vụ :
        ```
        # systemctl restart neutron-dhcp-agent
        ```
- Khi tạo 1 network, IP metadata service `169.254.169.254` sẽ được cấu hình :
    ```
    # ip netns exec qdhcp-4a9229fc-9d72-449d-b908-b8ca5c5be852 ip a
    ```
    <img src=https://i.imgur.com/7qk0ESU.png>

- Sẽ có 1 tiến trình metadata proxy chạy trong namespace :
    ```
    # ip netns exec qdhcp-4a9229fc-9d72-449d-b908-b8ca5c5be852 ps -aux | grep meta | grep 4a9229fc-9d72-449d-b908-b8ca5c5be852
    ```
    <img src=https://i.imgur.com/tpvc7zz.png>

- Metadata proxy sẽ lắng nghe port `80` trong namespace :
    ```
    # ip netns exec qdhcp-4a9229fc-9d72-449d-b908-b8ca5c5be852 netstat -lutpn | grep 14275
    ```
    <img src=https://i.imgur.com/DKj8j3x.png>

> ## **Chú ý :**
- Cả router namespace và DHCP namespace đều có `metadata-proxy` chạy để xử lý metadata request, tuy nhiên  `169.254.169.254` được định tuyến đến DHCP server trước khi restart `neutron-dhcp-agent`. Sau khi restart, `169.254.169.254` sẽ được định tuyến đến router gateway IP thông qua default route .