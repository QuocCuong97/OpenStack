# Các loại Token trong Keystone
## **1) Lịch sử Token trong OpenStack**
- **UUID Token**
    - Vào những ngày đầu, **Keystone** hỗ trợ **UUID token**. Đây là loại **token** gồm `32` kí tự được dùng để xác thực và ủy quyền. Lợi ích mà loại token này mạng lại đó là nó nhỏ gọn và dễ sử dụng, có thể thêm trực tiếp vào câu lệnh `curl`. Tuy nhiên nó lại không thể chứa đủ thông tin thực hiện việc ủy quyền. Các service của **OpenStack** sẽ luôn phải gửi lại token này cho **Keystone** để xác thực xem hành động có hợp lệ không. Điều này khiến cho **Keystone** trở thành trung tâm cho mọi hoạt động của **OpenStack**
- **PKI Token**
    - Trong nỗ lực tìm kiếm một loại **token** mới để khắc phục những nhược điểm của **UUID Token**, nhóm phát triển đã tạo ra **PKI Token**. **Token** này chứa đủ thông tin để xác thực và ủy quyền và đồng thời nó cũng chứa cả danh mục dịch vụ. Bên cạnh đó, **token** này được gán vào và các dịch vụ có thể cache lại nó để sử dụng cho đến khi nó hết hiệu lực hoặc bị hủy. Loại **token** này vì thế cũng khiến lưu lượng traffic tới **Keystone server** ít hơn. Tuy nhiên kích cỡ của nó có thể lên tới `8k` và điều này làm việc gán vào HTTP header trở nên khó khăn hơn. Rất nhiều các web server mặc định sẽ không cho phép điều này nếu chưa được config lại. Thêm vào đó, loại token này cũng rất khó được sử dụng trong câu lệnh `curl` .
- **PKIz Token** 
    - Vì những hạn chế của **PKI**, các nhà phát triển đã cố gắng sửa đổi và ra mắt một phiên bản khác đó là **PKIz**, tuy nhiên theo đánh giá của cộng đồng thì loại **token** này vẫn rất lớn về kích thước.
- **Fernet Token**
    - Với tất cả các hạn chế trên, buộc **Keystone** team phải đưa ra một loại mới là **Fernet Token**. **Fernet Token** khá nhỏ (`255` ký tự) tuy nhiên nó lại chưa đủ thông tin để ủy quyền. Bên cạnh đó, việc nó chứa đủ thông tin cũng không khiến **token database** phải lưu dữ liệu **token** nữa. Các nhà vận hành thường phải dọn dẹp **Keystone** **token database** để hệ thống của họ hoạt động ổn định. Mặc dù vậy, **Fernet token** có nhược điểm đó là symmetric keys được dùng để tạo ra **token** cần được phân phối và xoay vòng. Các nhà vận hành cần phải giải quyết vấn đề này, tuy nhiên họ có vẻ thích thú với việc này hơn là sử dụng những loại **token** khác.
## **2) Các loại token**
### **2.1) UUID Token**
- **UUID - Universally Unique Identifier**, hiểu nôm na là một thông tin định danh duy nhất 
- **UUID** được đại diện bởi `32` chữ số thập lục phân gạch nối, với dạng `8-4-4-4-12`. Có tổng cộng `36` ký tự, trong đó có `32` ký tự và `4` dấu gạch ngang
- **UUID** có 5 phiên bản, trong đó **Keystone** sử dụng **UUIDv4**
- Đặc điểm của **UUID** trong **Keystone** :
    - Có độ dài `32 byte`, nhỏ, dễ sử dụng, không nén
    - Không mang theo đủ thông tin, chỉ đơn giản là định danh khóa chiếu đến 1 bảng trong database **Keystone**
    - Được lưu vào database
    - Sử dụng thời gian dài làm giảm hiệu suất hoạt động, CPU tăng và thời gian đáp ứng lâu .
    - Sử dụng câu lệnh `keystone-manager token flush` để làm tăng cường hiệu suất hoạt động 
- Ưu điểm :
    - Phương pháp tạo **token** đơn giản
    - Khuyến nghị sử dụng cho môi trường phát triển
- Nhược điểm :
    - **Token** bị cố định
    - Không linh hoạt khi triển khai **Multi OpenStack**
- Method dùng để sinh **UUID Token** (Python) :
    ```py
    def _get_token_id(self, token_data):
        return uuid.uuid4().hex
    ```
- **Quá trình tạo Token** (***Token Generation Workflow***) :

    <img src=https://i.imgur.com/2EgDMz5.png>

    - **B1 :** User request tới **Keystone** tạo **token** với các thông tin: `username`, `password`, `project-name`
    - **B2 :** Chứng thực user, lấy `User ID` từ **LDAP Backend** (dịch vụ **Identity**)
    - **B3 :** Chứng thực **project**, thu thập thông tin `Project ID` và `Domain ID` từ **SQL backend** (dịch vụ **Resources**)
    - **B4 :** Lấy ra **role** từ backend trên **project** hoặc **domain** tương ứng trả về cho **user**, nếu **user** không có bất kỳ **role** nào thì trả về Failure (dịch vụ **Assignment**)
    - **B5 :** Thu thập các Service và các Endpoint của các service đó (dịch vụ **Catalog**)
    - **B6 :** Tổng hợp các thông tin về **Identity**, **Resources**, **Assignment** và **Catalog** ở trên đưa vào token payload, tạo ra **token** sử dụng hàm `uuid.uuid4().hex`
    - **B7 :** Lưu thông tin của **Token** vào **SQL/KVS backend** với các thông tin : `Token ID`, `Expiration`, `Valid`, `UserID`, `Extra`

- **Quá trình xác thực Token** (***Token Validation Workflow***) :

    <img src=https://i.imgur.com/3FtVsnO.png>

    - **B1 :** Gửi yêu cầu chứng thực **token** sử dụng API: `GET v3/auth/tokens` và **token** (**X-Subject-Token**, **X-Auth-Token**)
    - **B2 :** Thu thập token payloads từ token backend **KVS/SQL** kiểm tra trường `valid`
    - **B3 :** Phân tích **token** và thu thập metadata: `User ID`, `Project ID`, `Audit ID`, `Token Expire`
    - **B4 :** Kiểm tra **token** đã expire chưa. Nếu thời điểm hiện tại < "`Token Expire`" theo `UTC` thì **token** chưa expire,  chuyển sang bước tiếp theo
    - **B5 :** Kiểm tra xem **token** đã bị thu hồi chưa (kiểm tra trong bảng `revocation_event` của database **Keystone**)
    - **B6 :** Nếu **token** đã bị thu hồi (tương ứng với 1 event trong bảng `revocation_event`), trả về thông báo `Token Not Found`. Nếu chưa bị thu hồi, trả về token (truy vấn HTTP thành công `HTTP/1.1 200 OK`)

- **Quá trình thu hồi Token** (***Token Revocation Workflow***) :

    <img src=https://i.imgur.com/8fKCvAi.png>

    - **B1 :** Gửi yêu cầu thu hồi **token** với API request `DELETE v3/auth/tokens`. Trước khi thực hiện sự kiện thu hồi **token** thì phải chứng thực **token** nhờ vào tiến trình **Token Validation Workflow** đã trình bày ở trên.
    - **B2 :** Kiểm tra trường `Audit ID`. Nếu có, tạo sự kiện thu hồi với `audit id`. Nếu không có `audit id`, tạo sự kiện thu hồi với `token expire`
    - **B3 :** Nếu tạo sự kiện thu hồi **token** với `audit ID`, các thông tin cần cập nhật vào bảng `revocation_event` của database **Keystone** gồm `audit_id`, `revoke_at`, `issued_before` . Nếu tạo sự kiện thu hồi **token** với **token expired**, các thông tin cần thiết cập nhật vào bảng `revocation_event` của database **Keystone** bao gồm `user_id`, `project_id`, `revoke_at`,  `issued_before`, `token_expired`
    - **B4 :** Loại bỏ các sự kiện của các **token** đã expired từ bảng `revocation_event` của database **Keystone**. Cập nhật vào token database, thiết lập lại trường "`valid`" thành `false` (`0`)

- **Multiple DataCenters**

    <img src=https://i.imgur.com/pCZBTXc.png>

    - **UUID Token** không hỗ trợ xác thực và ủy quyền trong trường hợp **multiple data centers** bởi **token** được lưu dưới dạng ***persistent*** (cố định và không thể thay đổi). Như ví dụ mô tả ở hình trên, một hệ thống cloud triển khai trên hai datacenter ở hai nơi khác nhau. Khi xác thực với **keystone** trên datacenter W và sử dụng **token** trả về để request tạo một máy ảo với **Nova**, yêu cầu hoàn toàn hợp lệ và khởi tạo máy ảo thành công. Trong khi nếu mang **token** đó sang datacenter E yêu cầu tạo máy ảo thì sẽ không được xác nhận do **token** trong backend database W không có bản sao bên E.
### **2.2) PKI Token**
- **Token** này chứa một lượng khá lớn thông tin ví dụ như: *thời gian nó được tạo*, *thời gian nó hết hiệu lực*, *thông tin nhận diện người dùng*, *project*, *domain*, *thông tin về role cho user*, *danh mục dịch vụ*,... Tất cả các thông tin này được lưu ở trong phần payload của file định dạng `JSON`. Phần payload được "signed" theo chuẩn `X509` và đóng gói dưới dạng **cryptographic message syntax (CMS)**. Với **PKIz** thì phần payload được nén sử dụng `zlib`.
- **PKI Token** dựa trên chữ ký điện tử. **Keystone** sẽ dùng private key cho việc ký nhận và các **Openstack API** sẽ có public key để xác thực thông tin đó.
- **PKI** và **PKIz** đã không được hỗ trợ và không được dùng từ bản **Ocata**.