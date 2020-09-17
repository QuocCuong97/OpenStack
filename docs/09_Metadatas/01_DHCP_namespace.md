# Metadata service trong DHCP namespace
## **1) Cách làm việc mặc định của metadata service (trong router namespace)**
- Trong OpenStack, mặc định, cần **L3 agent** để giúp metadata service hoạt động, có nghĩa cần attach tenant network vào một neutron router. Trong router namespace, một tiến trình metadata proxy sẽ chạy để xử lý các metadata request, các rule của iptables sẽ phụ trách redirect metadata request về cho metadata proxy .
- Tiến trình `neutron-ns-metadata process` trong namespace listen ở port `9697` :
    ```
    # ip netns exec qrouter-045c5928-2a51-4b3e-a5fb-9a1e4386b13b netstat -ntlp | grep 9697
    ```
    <img src=https://i.imgur.com/AU3O1Qe.png>

- Rule iptables giúp redirect metadata request gửi đến `http://169.254.169.254:80` về port `9697` :
    ```
    # ip netns exec qrouter-045c5928-2a51-4b3e-a5fb-9a1e4386b13b iptables-save | grep REDIRECT
    ```
    <img src=https://i.imgur.com/Gw5dHtQ.png>

## **2) Metadata service trong DHCP namespace**
- **OpenStack** hiện đã có tùy chọn để kích hoạt dịch vụ metadata proxy trong DHCP namespace, do đó metadata service có thể hoạt động trọng một mạng cô lập, không cần có neutron router trong mạng nữa .
- Để bật tùy chọn này, cần cấu hình file `/etc/neutron/dhcp_agent.ini` :
    ```ini
    [DEFAULT]
    ...
    enable_isolated_metadata = True
    ...
    ```
    - Sau khi thay đổi xong thì cần khởi động lại dịch vụ :
        ```
        # systemctl restart neutron-dhcp-agent
        ```
- Mỗi mạng sau này được tạo ra sẽ tự động tạo ra một DHCP namespace, trong đó IP `169.254.169.254` của metadata service đã được cấu hình sẵn :

    <img src=https://i.imgur.com/7qk0ESU.png>

- Kiểm tra ta sẽ thấy có một tiến trình `metadata-proxy` cũng đang chạy sẵn :
    ```
    # ip netns exec qdhcp-4a9229fc-9d72-449d-b908-b8ca5c5be852 ps -aux | grep meta | grep 4a9229fc-9d72-449d-b908-b8ca5c5be852
    ```
    <img src=https://i.imgur.com/tpvc7zz.png>

    - Và tiến trình trên vẫn lắng nghe `169.254.169.254:80` :

        <img src=https://i.imgur.com/DKj8j3x.png>

> ## **Chú ý :**
- Cả router namespace và DHCP namespace đều có `metadata-proxy` chạy để xử lý metadata request, tuy nhiên  `169.254.169.254` được định tuyến đến DHCP server trước khi restart `neutron-dhcp-agent`. Sau khi restart, `169.254.169.254` sẽ được định tuyến đến router gateway IP thông qua default route .
