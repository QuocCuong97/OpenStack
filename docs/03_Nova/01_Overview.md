# Tổng quan về Nova
## **1) Giới thiệu**
- **Nova** là service chịu trách nhiệm chứa và quản lí các hệ thống cloud computing. **OpenStack Nova** là một project core trong **OpenStack**, nhằm mục đích cấp phép các tài nguyên và quản lý số lượng lớn máy ảo.
- **Nova - OpenStack Compute Service** chính là phần chính quan trọng nhất trong kiến trúc hệ thống **Infrastructure-as-a-Service (IaaS)**. 
- Phần lớn các modules của **Nova** được viết bằng **Python**.
- **OpenStack Compute** giao tiếp với các service khác của **OpenStack** :
    - **OpenStack Identity (Keystone)** để xác thực
    - **OpenStack Image (Glance)** để lấy images
    - **OpenStack Dashboard (Horizon)** để lấy giao diện cho người dùng và người quản trị.
    - Ngoài ra còn có thể tương tác với các service khác : block storage, disk, baremetal compute instance
- **Nova** cho phép điều khiển các máy ảo và cũng có thể quản lí các truy cập tới cloud từ users và projects. 
- **Nova** không chứa các phần mềm ảo hóa. Thay vào đó, nó sẽ định nghĩa các drivers để tương tác với các kĩ thuật ảo hóa khác chạy trên hệ điều hành của người dùng và cung cấp các chức năng thông qua một web-based API.
## **2) Các thành phần của Nova**
- **`nova-api`** : là service tiếp nhận và phản hồi các compute API request (các yêu cầu từ API) từ user. Hỗ trợ OpenStack Compute APIs. File cấu hình `nova.conf` sẽ được tạo ra khi **nova** được cài đặt .
- **`nova-compute`** : quản lý các máy ảo . Load các service object, thực hiện các phương thức trên ComputeManager thông qua Remote Procedure Call (RPC) .
- **`nova-conductor`** : là module chịu trách nhiệm về các tương tác giữa **`nova-compute`** và database. Nó sẽ loại bỏ tất cả các kết nối trực tiếp từ **`nova-compute`** tới database.
- **`nova-scheduler`** : lấy các yêu cầu máy ảo đặt vào queue và xác định xem chúng được chạy trên compute server host nào.
- **`nova-novncproxy`** : cung cấp VNC proxy thông qua browser, cho phép VNC console để truy cập máy ảo .
## **3) Kiến trúc của Nova**

<img src=https://i.imgur.com/VF044Af.png>

- Trong đó :
    - **DB** : SQK database để lưu trữ dữ liệu
    - **API** : thành phần nhận các HTTP request, chuyển đổi các lệnh và giao tiếp với các thành phần khác thông qua `oslo.messaging` queue hoặc HTTP
    - **Scheduler** : quyết định compute nào được sử dụng để chạy instance
    - **Compute** : quản lý giao tiếp với hypervisor và các máy ảo
    - **Conductor** : xử lý các yêu cầu mà cần sự phối hợp (build/resize), hoạt động như một database proxy, hoặc xử lý các đối tượng chuyển đôi .
    - **Placement** : theo dõi các tài nguyên còn lại và đã sử dụng

