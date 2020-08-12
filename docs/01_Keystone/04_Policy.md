# Policy trong Keystone
## **1) Role-based Access Control (RBAC)**
- **Role-Based Access Control - RBAC** (*điều khiển truy cập trên cơ sở vai trò*) là phương pháp điều khiển và đảm bảo quyền sử dụng cho người dùng.
- Đây là phương pháp thay thế cho **Discretionary Access Control - DAC** (*điều khiển truy cập tùy quyền*) và **Mandatory Access Control - MAC** (*điều khiển truy cập bắt buộc*).
- Các vai trò (**roles**) được tạo để đảm nhận các chức năng công việc khác nhau. Mỗi **role** được gắn liền với một số quyền hạn cho phép nó thao tác một số hoạt động cụ thể ('permissions').
- Các user được gán một role riêng, và thông qua việc gán role này mà user tiếp nhận được một số những quyền hạn cho phép họ được thực thi những chức năng cụ thể trong hệ thống.
- Vì user không được cấp phép một cách trực tiếp mà được tiếp nhận các quyền hạn thông qua role của user, việc quản lý quyền của user trở thành một việc đơn giản, và người quản trị chỉ cần chỉ định những role thích hợp cho user.
- Việc chỉ định role đơn giản hóa những công việc thông thường như việc cho thêm một người dùng vào trong hệ thống, hay tập quyền của người dùng.
## **2) File `policy.json`**
- Mỗi service trong **OpenStack** như identity, networking, compute... đều có một cơ chế quản lý quyền **role-based access** policies riêng của chúng. Các **role-based** này được chúng sử dụng để xác định một người dùng nào có thể truy cập vào một object nào trong chúng, và các **policy** này được định nghĩa tại file `policy.json`
- Khi user gọi tới API, các **policy** trên serivice sẽ được sử dụng để kiểm tra và chấp nhận các request này. Mỗi khi cập nhật một rule trên file `policy.json` thì sẽ có tác dụng ngay lập tức trong khi service đang chạy .
- Một file `policy.json` được định dạng dưới dạng `JSON`. Mỗi **policy** sẽ được định nghĩa theo format sau :
    ```json
    "<target>" : "<rule>"
    ```
    - Trong đó :
        - `target` sẽ là một action, đại diện cho một API có thể là `os_compute_api:servers:create`. Các action name này sẽ tùy theo từng loại dịch vụ sẽ có các đặc tính đi kèm, và được đặt tại file "`/etc/{service_name}/policy.json`" để các service được hiểu đây là các action cần được kiểm tra.
        - `rule` sẽ thường được sử dụng để xác định một **role** hay user nào được làm việc hay không làm việc với API target