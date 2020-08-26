# Cài đặt OpenStack Train trên Ubuntu 18.04
## **Mô hình**

<img src=https://i.imgur.com/XpxU968.png>

## **Phân hoạch IP**

<img src=https://i.imgur.com/SlZ1Rz5.png>

## **Các bước cài đặt**
### **1) Cài đặt ban đầu trên cả 3 node**
- **B1 :** Update các gói phần mềm và các package cơ bản:
    ```
    # apt update -y
    ```
- **B2 :** Thiết lập hostname :
    ```
    # hostnamectl set-hostname [controller|compute1|compute2]
    # bash
    ```
- **B3 :** Khai báo file `/etc/hosts` :
    ```
    # echo "127.0.0.1 localhost" > /etc/hosts
    # echo "10.10.230.10 controller" >> /etc/hosts
    # echo "10.10.230.11 compute1" >> /etc/hosts
    # echo "10.10.230.12 compute2" >> /etc/hosts
    ```
- **B4 :** Thiết lập phân hoạch IP cho node `controller`
- **B5 :** Disable firewalld và SELinux :
    ```
    # systemctl disable ufw
    # systemctl stop ufw
    # reboot
    ```
### **2) Cài đặt OpenStack**
#### **2.1) Cài đặt package OpenStack trên cả 3 node**
- Thực hiện các lệnh để cài đặt **OpenStack** :
    ```
    # apt-get install -y software-properties-common
    # add-apt-repository cloud-archive:train
    # apt -y upgrade -y
    # apt -y install crudini
    # apt -y install python-openstackclient python3-openstackclient
    ```
#### **2.2) Cài đặt NTP**
##### **2.2.1) Cài đặt NTP trên node `controller`**
- **B1 :** Cài đặt `chrony` sử dụng làm NTP :
    ```
    # apt -y install chrony
    ```
- **B2 :** Sao lưu file cấu hình của `chrony` :
    ```
    # cp /etc/chrony.conf /etc/chrony.conf.bak
    ```
- **B3 :** Set timezone :
    ```
    # timedatectl set-timezone Asia/Ho_Chi_Minh
    ```
- **B4 :** Sửa file cấu hình :
    ```
    # sed -i s'/0.centos.pool.ntp.org/128.138.140.44/'g /etc/chrony.conf
    # sed -i s'/server 1.centos.pool.ntp.org iburst/#server 1.centos.pool.ntp.org iburst/'g /etc/chrony.conf
    # sed -i s'/server 2.centos.pool.ntp.org iburst/#server 2.centos.pool.ntp.org iburst/'g /etc/chrony.conf
    # sed -i s'/server 3.centos.pool.ntp.org iburst/#server 3.centos.pool.ntp.org iburst/'g /etc/chrony.conf
    ```
    > Node `controller` sẽ cập nhật thời gian từ internet hoặc máy chủ NTP. Các máy compute còn lại sẽ đồng bộ thời gian từ `controller`. Trong bài lab này sẽ sử dụng địa chỉ NTP của **NIST** là `128.138.140.44` (có thể thay thế bằng IP của NTP Server trong mạng).
- **B5 :** Để cho phép các node `compute` kết nối với `chrony` trên node `controller`, ta sẽ sửa file cấu hình allow subnet chung của các node :
    ```
    # sed -i s'|#allow 192.168.0.0/16|allow 10.10.230.0/24|'g /etc/chrony.conf
    ```
- **B6 :** Khởi động lại chrony sau khi sửa file cấu hình :
    ```
    # service chrony restart
    ```
- **B7 :** Kiểm tra lại trạng thái của `chrony` :
    ```
    # service chrony status
    ```
    <img src=https://i.imgur.com/Cr3QPzj.png>

- **B8 :** Kiểm tra xem thời gian đã được đồng bộ chưa :
    ```
    # chronyc sources
    ```
    <img src=https://i.imgur.com/8aiOBtf.png>

    > Dấu "`*`" thể hiện việc đồng bộ thành công
#### **2.2.2) Cài đặt NTP trên các node `compute`**
- **B1 :** Cài đặt `chrony` sử dụng làm NTP :
    ```
    # apt -y install chrony
    ```
- **B2 :** Sao lưu file cấu hình của `chrony` :
    ```
    # cp /etc/chrony.conf /etc/chrony.conf.bak
    ```
- **B3 :** Set timezone :
    ```
    # timedatectl set-timezone Asia/Ho_Chi_Minh
    ```
- **B4 :** Sửa file cấu hình :
    ```
    # sed -i 's/server 0.centos.pool.ntp.org iburst/server controller iburst/g' /etc/chrony.conf
    # sed -i s'/server 1.centos.pool.ntp.org iburst/#server 1.centos.pool.ntp.org iburst/'g /etc/chrony.conf
    # sed -i s'/server 2.centos.pool.ntp.org iburst/#server 2.centos.pool.ntp.org iburst/'g /etc/chrony.conf
    # sed -i s'/server 3.centos.pool.ntp.org iburst/#server 3.centos.pool.ntp.org iburst/'g /etc/chrony.conf
    ```
    > 2 node `compute` sẽ đồng bộ thời gian về từ `controller`
- **B6 :** Khởi động lại chrony sau khi sửa file cấu hình :
    ```
    # service chrony restart
    ```
- **B7 :** Kiểm tra lại trạng thái của `chrony` :
    ```
    # service chrony status
    ```
    <img src=https://i.imgur.com/fjRIxSR.png>
    <img src=https://i.imgur.com/0L1YU6g.png>

- **B8 :** Kiểm tra xem thời gian đã được đồng bộ chưa :
    ```
    # chronyc sources
    ```
    <img src=https://i.imgur.com/jZkGLG3.png>
    <img src=https://i.imgur.com/06JIeIR.png>

    > Dấu "`*`" thể hiện việc đồng bộ thành công
### **2.3) Cài đặt và cấu hình `memcached` trên node `controller`**
- Cơ chế xác thực cho các dịch vụ sử dụng `memcached` để cache các token
- `Memcached` thường chạy trên node `controller`
- **B1 :** Cài đặt `memcached` :
    ```
    # apt -y install memcached python-memcache
    ```
- **B2 :** Backup file cấu hình của `memcached` :
    ```
    # cp /etc/sysconfig/memcached /etc/sysconfig/memcached.bak
    ```
- **B3 :** Cấu hình dịch vụ để sử dụng địa chỉ IP quản lý của node `controller`. Điều này là để cho phép truy cập từ các node khác thông qua dải VLAN `MGNT` (dải management). Chỉnh sửa file cấu hình `memcached` như sau :
    ```
    # sed -i "s/-l 127.0.0.1,::1/-l 127.0.0.1,::1,controller/g" /etc/sysconfig/memcached
    ```
- **B4 :** Khởi động lại `memcached` :
    ```
    # service memcached restart
    ```
- **B5 :** Kiểm tra trạng thái dịch vụ :
    ```
    # service memcached status
    ```
    <img src=https://i.imgur.com/PenIOOl.png>

### **2.4) Cài đặt và cấu hình `MariaDB` trên node `controller`**
- Hầu hết các dịch vụ của **OpenStack** sử dụng cơ sở dữ liệu **SQL** để lưu thông tin. DB thường sẽ chạy trên node `controller`. Các dịch vụ **OpenStack** cũng hỗ trợ các cơ sở dữ liệu SQL khác bao gồm **PostgreSQL**.
- **B1 :** Cài đặt **`MariaDB`** :
    ```
    # apt -y install mariadb-server python-pymysql
    ```
- **B2 :** Tạo và chỉnh sửa file cấu hình của **OpenStack** :
    ```
    # vi /etc/my.cnf.d/openstack.cnf
    ```
    - Thêm vào đoạn sau:
        ```
        [mysqld]
        bind-address = 10.10.230.10

        default-storage-engine = innodb
        innodb_file_per_table = on
        max_connections = 4096
        collation-server = utf8_general_ci
        character-set-server = utf8
        ```
- **B3 :** Khởi động **`MariaDB`** :
    ```
    # service mysql restart
    ```
- **B4 :** Kiểm tra trạng thái dịch vụ :
    ```
    # service mysql restart
    ```
    <img src=https://i.imgur.com/h041Agl.png>

- **B5 :** Bảo mật dịch vụ SQL bằng lệnh :
    ```
    # mysql_secure_installation
    ```
    > Trong bước này, set password cho user `root` (**VD :** '`Password123`)
- **B6 :** Gán quyền cho user `root` và xóa user mặc định :
    ```
    # mysql -u root -pPassword123
    > GRANT ALL PRIVILEGES ON *.* TO 'root'@'10.10.230.10' IDENTIFIED BY 'Password123' WITH GRANT OPTION;
    > FLUSH PRIVILEGES;
    > DROP USER 'root'@'::1';
    > exit;
    ```
### **2.5) Cài đặt và cấu hình `RabbitMQ` trên node `controller`**
- **OpenStack** sử dụng hàng đợi tin nhắn (***Message queue***) để phối hợp các hoạt động và thông tin trạng thái giữa các dịch vụ.
- ***Message queue*** thường chạy trên node `controller`. **OpenStack** hỗ trợ một số dịch vụ ***Message queue*** như là: **RabbitMQ**, **Qpid**, và **ZeroMQ**.
- Tuy nhiên, hầu hết các bản phân phối gói **OpenStack** đều hỗ trợ dịch vụ ***message queue*** cụ thể. Ta sẽ sử dụng **`RabbitMQ`** bởi vì hầu hết các phiên bản đều hỗ trợ nó.
- **B1 :** Cài đặt `rabbitmq` :
    ```
    # apt -y install rabbitmq-server
    ```
- **B2 :** Khởi động dịch vụ `rabbitmq` :
    ```
    # service rabbitmq-server start
    ```
- **B3 :** Khai báo plugin cho `rabbitmq` :
    ```
    # rabbitmq-plugins enable rabbitmq_management
    ```
- **B4 :** Cấu hình trang quản lý `rabbitmq` trên UI :
    ```
    # curl -O http://localhost:15672/cli/rabbitmqadmin
    # chmod a+x rabbitmqadmin
    # mv rabbitmqadmin /usr/sbin/
    ```
- **B5 :** Tạo user `openstack` với mật khẩu tùy ý (**VD :** `Password123`)
    ```
    # rabbitmqctl add_user openstack Password123
    ```
- **B6 :** Gán quyền cho user vừa tạo :
    ```
    # rabbitmqctl set_permissions openstack ".*" ".*" ".*"
    # rabbitmqctl set_user_tags openstack administrator
    ```
    - Hiển thị danh sách user `rabbitmq` :
        ```
        # rabbitmqadmin list users
        ```
        <img src=https://i.imgur.com/XGNurHs.png>
- **B7 :** Đăng nhập qua Web UI của **`RabbitMQ`** qua đường dẫn `http://IP_MANAGER_CONTROLLER:15672` với user vừa tạo để kiểm tra :

    <img src=https://i.imgur.com/27aoVF2.png>

    - Giao diện **`RabbitMQ`** khi đăng nhập :

        <img src=https://i.imgur.com/Nt8X0pv.png>

### **2.6) Cài đặt và cấu hình **`Etcd`** trên node `controller`**
- **`ETCD`** là một ứng dụng lưu trữ dữ liệu phân tán theo theo kiểu ***key-value***, nó được các services trong **OpenStack** sử dụng lưu trữ cấu hình, theo dõi các trạng thái dịch vụ và các tình huống khác.
- **B1 :** Cài đặt `etcd` :
    ```
    # apt -y install etcd
    ```
- **B2 :** Sao lưu file cấu hình của `etcd` :
    ```
    # cp /etc/etcd/etcd.conf /etc/etcd/etcd.conf.bak
    ```
- **B3 :** Chỉnh sửa file cấu hình của `etcd`. Lưu ý thay đúng IP MGNT (`10.10.230.10`) và hostname `controller` đã được thiết lập trước đó .
    ```
    # sed -i '/ETCD_DATA_DIR=/cETCD_DATA_DIR="/var/lib/etcd/default.etcd"' /etc/etcd/etcd.conf
    # sed -i '/ETCD_LISTEN_PEER_URLS=/cETCD_LISTEN_PEER_URLS="http://10.10.230.10:2380"' /etc/etcd/etcd.conf
    # sed -i '/ETCD_LISTEN_CLIENT_URLS=/cETCD_LISTEN_CLIENT_URLS="http://10.10.230.10:2379"' /etc/etcd/etcd.conf
    # sed -i '/ETCD_NAME=/cETCD_NAME="controller"' /etc/etcd/etcd.conf
    # sed -i '/ETCD_INITIAL_ADVERTISE_PEER_URLS=/cETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.10.230.10:2380"' /etc/etcd/etcd.conf
    # sed -i '/ETCD_ADVERTISE_CLIENT_URLS=/cETCD_ADVERTISE_CLIENT_URLS="http://10.10.230.10:2379"' /etc/etcd/etcd.conf
    # sed -i '/ETCD_INITIAL_CLUSTER=/cETCD_INITIAL_CLUSTER="controller=http://10.10.230.10:2380"' /etc/etcd/etcd.conf
    # sed -i '/ETCD_INITIAL_CLUSTER_TOKEN=/cETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"' /etc/etcd/etcd.conf
    # sed -i '/ETCD_INITIAL_CLUSTER_STATE=/cETCD_INITIAL_CLUSTER_STATE="new"' /etc/etcd/etcd.conf
    ```
- **B4 :** Khởi động dịch vụ `etcd` :
    ```
    # systemctl enable etcd
    # systemctl start etcd
    ```
- **B5 :** Kiểm tra trạng thái dịch vụ :
    ```
    # systemctl status etcd
    ```
    <img src=https://i.imgur.com/tZTq6dY.png>

### **2.7) Cài đặt và cấu hình `Keystone` trên node `controller`**
- **B1 :** Tạo Database, user và phân quyền cho `keystone` :
    - Tên database: `keystone`
    - Tên user trong database: `keystone`
    - Mật khẩu user: `Password123`
    ```
    # mysql -u root -pPassword123
    > CREATE DATABASE keystone;
    > GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'Password123';
    > GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'Password123';
    > FLUSH PRIVILEGES;
    > exit;
    ```
- **B2 :** Cài đặt **`Keystone`** :
    ```
    # apt install -y keystone
    ```
- **B3 :** Sao lưu file cấu hình của **`Keystone`** :
    ```
    # cp /etc/keystone/keystone.conf /etc/keystone/keystone.conf.bak
    ```
- **B4 :** Dùng lệnh `crudini` để sửa các dòng cần thiết trong file cấu hình của **`Keystone`** :
    ```
    # crudini --set /etc/keystone/keystone.conf database connection mysql+pymysql://keystone:Password123@controller/keystone
    # crudini --set /etc/keystone/keystone.conf token provider fernet
    ```
- **B5 :** Phân quyền lại cho file cấu hình của **`Keystone`** :
    ```
    # chown root:keystone /etc/keystone/keystone.conf
    ```
- **B6 :** Đồng bộ để sync database cho **`Keystone`** :
    ```
    # su -s /bin/sh -c "keystone-manage db_sync" keystone
    ```
- **B7 :** Sinh các file cho fernet :
    ```
    # keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
    # keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
    ```
    - Sau khi chạy 2 lệnh trên, thư mục `/etc/keystone/fernet-keys` sẽ được sinh ra và chứa các file key của `fernet`
- **B8 :** Thiết lập bootstrap cho **`Keystone`** :
    ```
    # keystone-manage bootstrap --bootstrap-password Password123 \
    --bootstrap-admin-url http://controller:5000/v3/ \
    --bootstrap-internal-url http://controller:5000/v3/ \
    --bootstrap-public-url http://controller:5000/v3/ \
    --bootstrap-region-id RegionOne
    ```
- **B9 :** **`Keystone`** sẽ sử dụng httpd để chạy service, các request vào **`keystone`** sẽ thông qua `apache2`. Do vậy cần cấu hình `apache2` để **`keystone`** sử dụng. Sửa dòng `95` trong file cấu hình `/etc/apache2/apache2.conf` của dịch vụ `apache2` :
    ```
    # vi /etc/apache2/apache2.conf
    :set nu
    ```
    <img src=https://i.imgur.com/y0dKTft.png>

- **B11 :** Khởi động dịch vụ `apache2` :
    ```
    # service apache2 restart
    ```
- **B12 :** Kiểm tra lại trạng thái dịch vụ :
    ```
    # systemctl status apache2
    ```
    <img src="https://i.imgur.com/sMjbj7L.png">

- **B13 :** Tạo file biến môi trường cho **`Keystone`** :
    ```
    # vi /root/admin-openrc
    ```
    - Thêm vào đoạn sau :
        ```
        export OS_USERNAME=admin
        export OS_PASSWORD=Password123
        export OS_PROJECT_NAME=admin
        export OS_USER_DOMAIN_NAME=default
        export OS_PROJECT_DOMAIN_NAME=default
        export OS_AUTH_URL=http://controller:5000/v3
        export OS_IDENTITY_API_VERSION=3
        export OS_IMAGE_API_VERSION=2
        ```
- **B14 :** Thực thi biến môi trường để sử dụng được CLI của OpenStack:
    ```
    # source /root/admin-openrc
    ```
- **B15 :** Kiểm tra lại hoạt động của **`Keystone`** :
    ```
    # openstack token issue
    ```
    <img src=https://i.imgur.com/ufF2BZN.png>

    > Kết quả trả về  như trên có nghĩa **`Keystone`** hoạt động bình thường
- **B16 :** Khai báo user `demo`, project `demo` :
    ```
    # openstack project create --domain default --description "Service Project" service
    # openstack project create --domain default --description "Demo Project" demo
    # openstack user create demo --domain default --password Password123
    # openstack role create user
    # openstack role add --project demo --user demo user
    ```
### **2.8) Cài đặt và cấu hình **`Glance`** trên node `controller`**
- **B1 :** Tạo Database, user và phân quyền cho `glance` :
    - Tên database: `glance`
    - Tên user trong database: `glance`
    - Mật khẩu user: `Password123`
    ```
    # mysql -u root -pPassword123
    > CREATE DATABASE glance;
    > GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'Password123';
    > GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'Password123';
    > FLUSH PRIVILEGES;
    > exit;
    ```
- **B2 :** Thực thi biến môi trường :
    ```
    # source /root/admin-openrc
    ```
- **B3 :** Tạo user, project cho **`glance`** :
    ```
    # openstack user create glance --domain default --password Password123
    # openstack role add --project service --user glance admin
    # openstack service create --name glance --description "OpenStack Image" image
    # openstack endpoint create --region RegionOne image public http://controller:9292
    # openstack endpoint create --region RegionOne image internal http://controller:9292
    # openstack endpoint create --region RegionOne image admin http://controller:9292
    ```
- **B4 :** Cài đặt `glance` và các package cần thiết :
    ```
    # apt install -y glance
    ```
- **B5 :** Sao lưu file cấu hình **`Glance`** :
    ```
    # cp /etc/glance/glance-api.conf /etc/glance/glance-api.conf.bak
    ```
- **B6 :** Cấu hình **`Glance`** :
    ```
    # crudini --set /etc/glance/glance-api.conf database connection mysql+pymysql://glance:Password123@controller/glance
    # crudini --set /etc/glance/glance-api.conf keystone_authtoken www_authenticate_uri http://controller:5000
    # crudini --set /etc/glance/glance-api.conf keystone_authtoken auth_url http://controller:5000
    # crudini --set /etc/glance/glance-api.conf keystone_authtoken memcached_servers controller:11211
    # crudini --set /etc/glance/glance-api.conf keystone_authtoken auth_type password
    # crudini --set /etc/glance/glance-api.conf keystone_authtoken project_domain_name default
    # crudini --set /etc/glance/glance-api.conf keystone_authtoken user_domain_name default
    # crudini --set /etc/glance/glance-api.conf keystone_authtoken project_name service
    # crudini --set /etc/glance/glance-api.conf keystone_authtoken username glance
    # crudini --set /etc/glance/glance-api.conf keystone_authtoken password Password123
    # crudini --set /etc/glance/glance-api.conf paste_deploy flavor keystone
    # crudini --set /etc/glance/glance-api.conf glance_store stores file,http
    # crudini --set /etc/glance/glance-api.conf glance_store default_store file
    # crudini --set /etc/glance/glance-api.conf glance_store filesystem_store_datadir /var/lib/glance/images/
    ```
- **B7 :** Đồng bộ để sync database cho **`Glance`** :
    ```
    # su -s /bin/sh -c "glance-manage db_sync" glance
    ```
- **B8 :** Khởi động dịch vụ **`glance`** :
    ```
    # systemctl glance-api restart
    ```
- **B9 :** Tải image và import vào **`glance`** :
    ```
    # wget http://download.cirros-cloud.net/0.5.1/cirros-0.5.1-x86_64-disk.img
    # openstack image create "cirros" --file cirros-0.5.1-x86_64-disk.img --disk-format qcow2 --container-format bare --public
    ```
    > Đây là một test image của **OpenStack**. Download các image khác tại [đây](https://docs.openstack.org/image-guide/obtain-images.html)
- **B10 :** Kiểm tra lại xem image đã được up hay chưa :
    ```
    # openstack image list
    ```
    <img src=https://i.imgur.com/ZzzPdea.png>

### **2.9) Cài đặt và cấu hình `Placement` trên node `controller`**
- **B1 :** Tạo Database, user và phân quyền cho `placement` :
    - Tên database: `placement`
    - Tên user trong database: `placement`
    - Mật khẩu user: `Password123`
    ```
    # mysql -u root -pPassword123
    > CREATE DATABASE placement;
    > GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'localhost' IDENTIFIED BY 'Password123';
    > GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'%' IDENTIFIED BY 'Password123';
    > FLUSH PRIVILEGES;
    > exit;
    ```
- **B2 :** Thực thi biến môi trường để sử dụng được CLI của OpenStack:
    ```
    # source /root/admin-openrc
    ```
- **B3 :** Tạo service, gán quyền, endpoint cho `placement` :
    ```
    # openstack user create placement --domain default --password Password123
    # openstack role add --project service --user placement admin
    # openstack service create --name placement --description "Placement API" placement
    # openstack endpoint create --region RegionOne placement public http://controller:8778
    # openstack endpoint create --region RegionOne placement internal http://controller:8778
    # openstack endpoint create --region RegionOne placement admin http://controller:8778
    ```
- **B4 :** Cài đặt `placement` :
    ```
    # apt install -y placement-api
    ```
- **B5 :** Sao lưu file cấu hình của `placement` :
    ```
    # cp /etc/placement/placement.conf /etc/placement/placement.conf.bak
    ```
- **B6 :** Cấu hình `placement` :
    ```
    # crudini --set /etc/placement/placement.conf placement_database connection mysql+pymysql://placement:Password123@controller/placement
    # crudini --set /etc/placement/placement.conf api auth_strategy keystone
    # crudini --set /etc/placement/placement.conf keystone_authtoken auth_url http://controller:5000/v3
    # crudini --set /etc/placement/placement.conf keystone_authtoken memcached_servers controller:11211
    # crudini --set /etc/placement/placement.conf keystone_authtoken auth_type password
    # crudini --set /etc/placement/placement.conf keystone_authtoken project_domain_name default
    # crudini --set /etc/placement/placement.conf keystone_authtoken user_domain_name default
    # crudini --set /etc/placement/placement.conf keystone_authtoken project_name service
    # crudini --set /etc/placement/placement.conf keystone_authtoken username placement
    # crudini --set /etc/placement/placement.conf keystone_authtoken password Password123
    ```
- **B8 :** Tạo các bảng, đồng bộ dữ liệu cho `placement` :
    ```
    # su -s /bin/sh -c "placement-manage db sync" placement
    ```
- **B9 :** Khởi động lại `httpd` :
    ```
    # service apache2 restart
    ```
### **2.10) Cài đặt `Nova`**
#### **2.10.1) Cài đặt `Nova` trên node `controller`**
- **B1 :** Tạo các database, user, mật khẩu cho service `nova` :
    ```
    # mysql -u root -pPassword123
    > CREATE DATABASE nova_api;
    > GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'Password123';
    > GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'Password123';
    > CREATE DATABASE nova;
    > GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'Password123';
    > GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'Password123';
    > CREATE DATABASE nova_cell0;
    > GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' IDENTIFIED BY 'Password123';
    > GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY 'Password123';
    > FLUSH PRIVILEGES;
    > exit
    ```
- **B2 :** Thực thi biến môi trường để sử dụng được CLI của **OpenStack** :
    ```
    # source /root/admin-openrc
    ```
- **B3 :** Tạo endpoint cho `nova` :
    ```
    # openstack user create nova --domain default --password Password123
    # openstack role add --project service --user nova admin
    # openstack service create --name nova --description "OpenStack Compute" compute
    # openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1
    # openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1
    # openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1
    ```
- **B4 :** Cài đặt `nova` và các package đi kèm :
    ```
    # apt install -y nova-api nova-conductor nova-novncproxy nova-scheduler
    ```
- **B5 :** Sao lưu file cấu hình của `nova` :
    ```
    # cp /etc/nova/nova.conf /etc/nova/nova.conf.bak
    ```
- **B6 :** Cấu hình **`Nova`** :
    ```
    # crudini --set /etc/nova/nova.conf DEFAULT my_ip 10.10.230.10
    # crudini --set /etc/nova/nova.conf DEFAULT use_neutron true
    # crudini --set /etc/nova/nova.conf DEFAULT firewall_driver nova.virt.firewall.NoopFirewallDriver
    # crudini --set /etc/nova/nova.conf DEFAULT enabled_apis osapi_compute,metadata
    # crudini --set /etc/nova/nova.conf DEFAULT transport_url rabbit://openstack:Password123@controller:5672/
    # crudini --set /etc/nova/nova.conf api_database connection mysql+pymysql://nova:Password123@controller/nova_api
    # crudini --set /etc/nova/nova.conf database connection mysql+pymysql://nova:Password123@controller/nova
    # crudini --set /etc/nova/nova.conf api connection mysql+pymysql://nova:Password123@controller/nova
    # crudini --set /etc/nova/nova.conf api auth_strategy keystone
    # crudini --set /etc/nova/nova.conf keystone_authtoken www_authenticate_uri http://controller:5000/
    # crudini --set /etc/nova/nova.conf keystone_authtoken auth_url http://controller:5000/
    # crudini --set /etc/nova/nova.conf keystone_authtoken memcached_servers controller:11211
    # crudini --set /etc/nova/nova.conf keystone_authtoken auth_type password
    # crudini --set /etc/nova/nova.conf keystone_authtoken project_domain_name default
    # crudini --set /etc/nova/nova.conf keystone_authtoken user_domain_name default
    # crudini --set /etc/nova/nova.conf keystone_authtoken project_name service
    # crudini --set /etc/nova/nova.conf keystone_authtoken username nova
    # crudini --set /etc/nova/nova.conf keystone_authtoken password Password123
    # crudini --set /etc/nova/nova.conf vnc enabled true
    # crudini --set /etc/nova/nova.conf vnc server_listen \$my_ip
    # crudini --set /etc/nova/nova.conf vnc server_proxyclient_address \$my_ip
    # crudini --set /etc/nova/nova.conf glance api_servers http://controller:9292
    # crudini --set /etc/nova/nova.conf oslo_concurrency lock_path /var/lib/nova/tmp
    # crudini --set /etc/nova/nova.conf placement region_name RegionOne
    # crudini --set /etc/nova/nova.conf placement project_domain_name default
    # crudini --set /etc/nova/nova.conf placement project_name service
    # crudini --set /etc/nova/nova.conf placement auth_type password
    # crudini --set /etc/nova/nova.conf placement user_domain_name default
    # crudini --set /etc/nova/nova.conf placement auth_url http://controller:5000/v3
    # crudini --set /etc/nova/nova.conf placement username placement
    # crudini --set /etc/nova/nova.conf placement password Password123
    # crudini --set /etc/nova/nova.conf scheduler discover_hosts_in_cells_interval 300
    # crudini --set /etc/nova/nova.conf neutron url http://controller:9696
    # crudini --set /etc/nova/nova.conf neutron auth_url http://controller:5000
    # crudini --set /etc/nova/nova.conf neutron region_name RegionOne
    # crudini --set /etc/nova/nova.conf neutron auth_type password
    # crudini --set /etc/nova/nova.conf neutron project_domain_name default
    # crudini --set /etc/nova/nova.conf neutron user_domain_name default
    # crudini --set /etc/nova/nova.conf neutron project_name service
    # crudini --set /etc/nova/nova.conf neutron username neutron
    # crudini --set /etc/nova/nova.conf neutron password Password123
    # crudini --set /etc/nova/nova.conf neutron service_metadata_proxy True
    # crudini --set /etc/nova/nova.conf neutron metadata_proxy_shared_secret Password123
    ```
- **B7 :** Thực hiện lệnh để sinh bảng cho **`Nova`** :
    ```
    # su -s /bin/sh -c "nova-manage api_db sync" nova
    # su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
    # su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
    # su -s /bin/sh -c "nova-manage db sync" nova
    ```
- **B8 :** Kiểm tra lại xem `cell0` đã được đăng ký chưa :
    ```
    # su -s /bin/sh -c "nova-manage cell_v2 list_cells" nova
    ```
    <img src=https://i.imgur.com/LvkPXdv.png>
- **B9 :** Kích hoạt và khởi động các dịch vụ của **`Nova`** :
    ```
    # service restart nova-api nova-scheduler nova-conductor nova-novncproxy
    ```
- **B10 :** Kiểm tra lại xem dịch vụ của `nova` đã hoạt động chưa :
    ```
    # openstack compute service list
    ```
    <img src=https://i.imgur.com/jhhwhru.png>

#### **2.10.2) Cài đặt `Nova` trên các node `compute`**
- **B1 :** Cài đặt các gói của `nova` :
    ```
    # apt install -y nova-compute
    ```
- **B2 :** Sao lưu file cấu hình của **`Nova`** :
    ```
    # cp /etc/nova/nova.conf /etc/nova/nova.conf.bak
    ```
- **B3 :** Cấu hình `nova` (trên `compute1`. Làm tương tự với `compute2` ):
    ```
    # crudini --set /etc/nova/nova.conf DEFAULT enabled_apis osapi_compute,metadata
    # crudini --set /etc/nova/nova.conf DEFAULT transport_url rabbit://openstack:Password123@controller
    # crudini --set /etc/nova/nova.conf DEFAULT my_ip 10.10.230.11
    # crudini --set /etc/nova/nova.conf DEFAULT use_neutron true
    # crudini --set /etc/nova/nova.conf DEFAULT firewall_driver nova.virt.firewall.NoopFirewallDriver
    # crudini --set /etc/nova/nova.conf api_database connection mysql+pymysql://nova:Password123@controller/nova_api
    # crudini --set /etc/nova/nova.conf database connection mysql+pymysql://nova:Password123@controller/nova
    # crudini --set /etc/nova/nova.conf api auth_strategy keystone
    # crudini --set /etc/nova/nova.conf keystone_authtoken www_authenticate_uri http://controller:5000/
    # crudini --set /etc/nova/nova.conf keystone_authtoken auth_url http://controller:5000/
    # crudini --set /etc/nova/nova.conf keystone_authtoken memcached_servers controller:11211
    # crudini --set /etc/nova/nova.conf keystone_authtoken auth_type password
    # crudini --set /etc/nova/nova.conf keystone_authtoken project_domain_name default
    # crudini --set /etc/nova/nova.conf keystone_authtoken user_domain_name default
    # crudini --set /etc/nova/nova.conf keystone_authtoken project_name service
    # crudini --set /etc/nova/nova.conf keystone_authtoken username nova
    # crudini --set /etc/nova/nova.conf keystone_authtoken password Password123
    # crudini --set /etc/nova/nova.conf vnc enabled true
    # crudini --set /etc/nova/nova.conf vnc server_listen 0.0.0.0
    # crudini --set /etc/nova/nova.conf vnc server_proxyclient_address \$my_ip
    # crudini --set /etc/nova/nova.conf vnc novncproxy_base_url http://controller:6080/vnc_auto.html
    # crudini --set /etc/nova/nova.conf glance api_servers http://controller:9292
    # crudini --set /etc/nova/nova.conf oslo_concurrency lock_path /var/lib/nova/tmp
    # crudini --set /etc/nova/nova.conf placement region_name RegionOne
    # crudini --set /etc/nova/nova.conf placement project_domain_name default
    # crudini --set /etc/nova/nova.conf placement project_name service
    # crudini --set /etc/nova/nova.conf placement auth_type password
    # crudini --set /etc/nova/nova.conf placement user_domain_name default
    # crudini --set /etc/nova/nova.conf placement auth_url http://controller:5000/v3
    # crudini --set /etc/nova/nova.conf placement username placement
    # crudini --set /etc/nova/nova.conf placement password Password123
    # crudini --set /etc/nova/nova-compute.conf libvirt virt_type qemu
    ```
- **B4 :** Khởi động **`Nova`** :
    ```
    # service start nova-compute
    ```
#### **2.10.3) Thêm các node `compute` vào hệ thống (trên node `controller`)**
- **B1 :** Kiểm tra các node `compute` đã up hay chưa :
    ```
    # source /root/admin-openrc
    # openstack compute service list --service nova-compute
    ```
    <img src=https://i.imgur.com/0oaLGYK.png>
- **B2 :** Add các note `compute` vào `cell` :
    ```
    # su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova
    ```
### **2.11) Cài đặt `Neutron`**
#### **2.11.1) Cài đặt `Neutron` trên node `controller`**
- **B1 :** Tạo database cho `Neutron` :
    ```
    # mysql -u root -pPassword123
    > CREATE DATABASE neutron;
    > GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'Password123';
    > GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'Password123';
    > FLUSH PRIVILEGES;
    > exit
    ```
- **B2 :** Tạo project, user, endpoint cho **`Neutron`** :
    ```
    # source /root/admin-openrc
    # openstack user create neutron --domain default --password Password123
    # openstack role add --project service --user neutron admin
    # openstack service create --name neutron --description "OpenStack Networking" network
    # openstack endpoint create --region RegionOne network public http://controller:9696
    # openstack endpoint create --region RegionOne network internal http://controller:9696
    # openstack endpoint create --region RegionOne network admin http://controller:9696
    ```
- **B3 :** Cài đặt **`Neutron`** :
    ```
    # apt install -y neutron-server neutron-plugin-ml2 neutron-linuxbridge-agent neutron-dhcp-agent neutron-metadata-agent
    ```
- **B4 :** Sao lưu các file cấu hình của **`Neutron`** :
    ```
    # cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.bak
    # cp /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugins/ml2/ml2_conf.ini.bak
    # cp /etc/neutron/plugins/ml2/linuxbridge_agent.ini /etc/neutron/plugins/ml2/linuxbridge_agent.ini.bak 
    # cp /etc/neutron/dhcp_agent.ini /etc/neutron/dhcp_agent.ini.bak
    # cp /etc/neutron/metadata_agent.ini /etc/neutron/metadata_agent.ini.bak
    ```
- **B5 :** Cấu hình file `/etc/neutron/neutron.conf` :
    ```
    # crudini --set /etc/neutron/neutron.conf DEFAULT core_plugin ml2
    # crudini --set /etc/neutron/neutron.conf DEFAULT service_plugins
    # crudini --set /etc/neutron/neutron.conf DEFAULT transport_url rabbit://openstack:Password123@controller
    # crudini --set /etc/neutron/neutron.conf DEFAULT auth_strategy keystone
    # crudini --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_status_changes True
    # crudini --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_data_changes True
    # crudini --set /etc/neutron/neutron.conf database connection mysql+pymysql://neutron:Password123@controller/neutron
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken www_authenticate_uri http://controller:5000
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken auth_url http://controller:5000
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken memcached_servers controller:11211
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken auth_type password
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken project_domain_name default
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken user_domain_name default
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken project_name service
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken username neutron
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken password Password123
    # crudini --set /etc/neutron/neutron.conf nova auth_url http://controller:5000
    # crudini --set /etc/neutron/neutron.conf nova auth_type password
    # crudini --set /etc/neutron/neutron.conf nova project_domain_name default
    # crudini --set /etc/neutron/neutron.conf nova user_domain_name default
    # crudini --set /etc/neutron/neutron.conf nova region_name RegionOne
    # crudini --set /etc/neutron/neutron.conf nova project_name service
    # crudini --set /etc/neutron/neutron.conf nova username nova
    # crudini --set /etc/neutron/neutron.conf nova password Password123
    # crudini --set /etc/neutron/neutron.conf oslo_concurrency lock_path /var/lib/neutron/tmp
    ```
- **B6 :** Sửa file cấu hình `/etc/neutron/plugins/ml2/ml2_conf.ini` :
    ```
    # crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 type_drivers flat,vlan,vxlan
    # crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 tenant_network_types vxlan
    # crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 mechanism_drivers linuxbridge
    # crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 extension_drivers port_security
    # crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2_type_flat flat_networks provider
    # crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2_type_vxlan vni_ranges 1:1000
    # crudini --set /etc/neutron/plugins/ml2/ml2_conf.ini securitygroup enable_ipset True
    ```
- **B7 :** Sửa file cấu hình `/etc/neutron/plugins/ml2/linuxbridge_agent.ini` :
    ```
    # crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini linux_bridge physical_interface_mappings provider:eth0
    # crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan enable_vxlan True
    # crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan local_ip $(ip addr show dev eth2 scope global | grep "inet " | sed -e 's#.*inet ##g' -e 's#/.*##g')
    # crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup enable_security_group True
    # crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup firewall_driver neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
    ```
- **B8 :** Khai báo `sysctl` :
    ```
    # echo 'net.bridge.bridge-nf-call-iptables = 1' >> /etc/sysctl.conf
    # echo 'net.bridge.bridge-nf-call-ip6tables = 1' >> /etc/sysctl.conf
    # modprobe br_netfilter
    # /sbin/sysctl -p
    ```
- **B10 :** Thiết lập database :
    ```
    # su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
    --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
    ```
- **B11 :** Khởi động dịch vụ **`neutron`** :
    ```
    # service start neutron-server neutron-linuxbridge-agent neutron-dhcp-agent neutron-metadata-agent
    # service nova-api restart
    ```
- **B12 :** Kiểm tra lại trạng thái dịch vụ :
    ```
    # openstack network agent list
    ```
    <img src=https://i.imgur.com/yMQePN0.png>

#### **2.11.2) Cài đặt `Neutron` trên các node `compute`**
- **B1 :** Khai báo bổ sung cho **`Nova`** :
    ```
    # crudini --set /etc/nova/nova.conf neutron url http://controller:9696
    # crudini --set /etc/nova/nova.conf neutron auth_url http://controller:5000
    # crudini --set /etc/nova/nova.conf neutron auth_type password
    # crudini --set /etc/nova/nova.conf neutron project_domain_name default
    # crudini --set /etc/nova/nova.conf neutron user_domain_name default
    # crudini --set /etc/nova/nova.conf neutron project_name service
    # crudini --set /etc/nova/nova.conf neutron username neutron
    # crudini --set /etc/nova/nova.conf neutron password Password123
    ```
- **B2 :** Cài đặt **`Neutron`** :
    ```
    # apt install -y neutron-linuxbridge-agent
    ```
- **B3 :** Sao lưu các file cấu hình của **`Neutron`** :
    ```
    # cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.bak
    # cp /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugins/ml2/ml2_conf.ini.bak
    # cp /etc/neutron/plugins/ml2/linuxbridge_agent.ini /etc/neutron/plugins/ml2/linuxbridge_agent.ini.bak
    # cp /etc/neutron/dhcp_agent.ini /etc/neutron/dhcp_agent.ini.bak
    # cp /etc/neutron/metadata_agent.ini /etc/neutron/metadata_agent.ini.bak
    ```
- **B4 :** Sửa file cấu hình `/etc/neutron/neutron.conf` :
    ```
    # crudini --set /etc/neutron/neutron.conf DEFAULT auth_strategy keystone
    # crudini --set /etc/neutron/neutron.conf DEFAULT core_plugin ml2
    # crudini --set /etc/neutron/neutron.conf DEFAULT transport_url rabbit://openstack:Password123@controller
    # crudini --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_status_changes true
    # crudini --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_data_changes true
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken www_authenticate_uri http://controller:5000
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken auth_url http://controller:5000
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken memcached_servers controller:11211
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken auth_type password
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken project_domain_name default
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken user_domain_name default
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken project_name service
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken username neutron
    # crudini --set /etc/neutron/neutron.conf keystone_authtoken password Password123
    # crudini --set /etc/neutron/neutron.conf oslo_concurrency lock_path /var/lib/neutron/tmp
    ```
- **B5 :** Khai báo `sysctl` :
    ```
    # echo 'net.bridge.bridge-nf-call-iptables = 1' >> /etc/sysctl.conf
    # echo 'net.bridge.bridge-nf-call-ip6tables = 1' >> /etc/sysctl.conf
    # modprobe br_netfilter
    # /sbin/sysctl -p
    ```
- **B6 :** Sửa file cấu hình `/etc/neutron/plugins/ml2/linuxbridge_agent.ini` :
    ```
    # crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini linux_bridge physical_interface_mappings provider:eth0
    # crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan enable_vxlan True
    # crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan local_ip $(ip addr show dev eth2 scope global | grep "inet " | sed -e 's#.*inet ##g' -e 's#/.*##g')
    # crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup enable_security_group True
    # crudini --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup firewall_driver neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
    ```
- **B7 :** Khai báo trong file `/etc/neutron/metadata_agent.ini`
    ```
    # crudini --set /etc/neutron/metadata_agent.ini DEFAULT nova_metadata_host 10.10.230.10
    # crudini --set /etc/neutron/metadata_agent.ini DEFAULT metadata_proxy_shared_secret Password123
    ```
- **B8 :** Khai báo cho file `/etc/neutron/dhcp_agent.ini` :
    ```
    # crudini --set /etc/neutron/dhcp_agent.ini DEFAULT interface_driver neutron.agent.linux.interface.BridgeInterfaceDriver
    # crudini --set /etc/neutron/dhcp_agent.ini DEFAULT enable_isolated_metadata True
    # crudini --set /etc/neutron/dhcp_agent.ini DEFAULT dhcp_driver neutron.agent.linux.dhcp.Dnsmasq
    # crudini --set /etc/neutron/dhcp_agent.ini DEFAULT force_metadata True
    ```
- **B9 :** Khởi động **`Neutron`** :
    ```
    # service neutron-linuxbridge-agent restart
    ```
- **B10 :** Khởi động lại dịch vụ `openstack-nova-compute` :
    ```
    # service nova-compute restart
    ```
### **2.12) Cài đặt và cấu hình `Horizon` trên node `controller`**
- **B1 :** Cài đặt `openstack-dashboard` :
    ```
    # apt install -y openstack-dashboard
    ```
- **B2 :** Sao lưu file `/etc/openstack-dashboard/local_settings` :
    ```
    # cp /etc/openstack-dashboard/local_settings /etc/openstack-dashboard/local_settings.bak
    ```
- **B3 :** Chỉnh sửa file `/etc/openstack-dashboard/local_settings` :
    - Chỉnh sửa 1 số dòng sau :
        ```py
        ALLOWED_HOSTS = ['*']
        ```
        ```py
        CACHES = {
            'default': {
                'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
                'LOCATION': 'controller:11211',
            }
        }
        ```
        ```py
        SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
        ```
        ```py
        OPENSTACK_HOST = "controller"
        ```
        ```py
        OPENSTACK_NEUTRON_NETWORK = {
            ...
            'enable_distributed_router': False,
            'enable_fip_topology_check': False,
            'enable_ha_router': False,
            'enable_quotas': False,
            'enable_router': False,
            ...
        }
        ```
        ```py
        TIME_ZONE = "Asia/Ho_Chi_Minh"
        ```
    - Thêm các dòng sau vào cuối file :
        ```py
        OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
        OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "default"
        OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
        OPENSTACK_API_VERSIONS = {
            "identity": 3,
            "image": 2,
            "volume": 3,
        }
        WEBROOT = "/dashboard/"
        ```
- **B4 :** Chỉnh sửa file `/etc/httpd/conf.d/openstack-dashboard.conf` :
    ```
    # vi /etc/httpd/conf.d/openstack-dashboard.conf
    ```
    - Thêm vào cuối file dòng sau :
        ```py
        WSGIApplicationGroup %{GLOBAL}
        ```
- **B5 :** Khởi động lại dịch vụ :
    ```
    # service httpd memcached restart
    ```
- **B6 :** Truy cập đường dẫn sau trên trình duyệt để vào dashboard. Đăng nhập bằng tài khoản `admin`/ `Passw0rd123` vừa tạo ở trên:
    ```
    http://IP_CONTROLLER/dashboard
    ```
    <img src=https://i.imgur.com/iSuHiam.png>

    <img src=https://i.imgur.com/7VO3zO4.png>

