# Luồng làm việc của Cinder
## **1) Khởi tạo volume mới**
<p align=center><img src=https://i.imgur.com/AIpSOVZ.png></p>

- Quá trình tạo một volume mới trên cinder
    - Client yêu cầu tạo ra một Volume thông qua việc gọi REST API (Client cũng có thể sử dụng tiện ích CLI của python-client)
    - Cinder-api thực hiện xác nhận hợp lệ yêu cầu thông tin người dùng mỗi khi xác nhận được một mesasge được gửi lên từ hang chờ AMQP để xử lý.
    - Cinder-volume đưa message ra khỏi hàng đợi, gửi thông báo tới cinder-scheduler để báo cáo xác định backend cung cấp volume.
    - Cinder-scheduler thực hiện quá trình báo cáo sẽ đưa thông báo ra khỏi hàng đợi, tạo danh sách các ứng viên dựa trên trạng thái hiện tại và yêu cầu tạo volume theo tiêu chí như kích thước, vùng sẵn có, loại volume (bao gồm cả thông số kỹ thuật bổ sung).
    - Cinder-volume thực hiện quá trình đọc message phản hồi từ cinder-scheduler từ hàng đợi. Lặp lại qua các danh sách ứng viên bằng các gọi backend driver cho đến khi thành công.
    - Storage Cinder tạo ra volume được yêu cầu thông qua tương tác với hệ thống lưu trữ con (phụ thuộc vào cấu hình và giao thức).
    - Cinder-volume thực hiện quá trình thu thập dữ liệu và metadata volume và thông tin kết nối để trả lại thông báo đến AMQP.
    - Cinder-api thực hiện quá trình đọc message phản hồi từ hàng đợi và đáp ứng tới client.
    - Client nhận được thông tin bao gồm trạng thái của yêu cầu tạo, Volume UUID,...
## **2) Attach volume**
<p align=center><img src=https://i.imgur.com/Q1vGk36.png></p>

- Quá trình attach volume :
    - Client yêu cầu attach volume thông qua Nova REST API (Client có thể sử dụng tiện ích CLI của python-novaclient)
    - Nova-api thực hiện quá trình xác nhận yêu cầu và thông tin người dùng. Một khi đã được xác thực, gọi API Cinder để có được thông tin kết nối cho volume được xác định.
    - Cinder-api thực hiện quá trình xác nhận yêu cầu hợp lệ và thông tin người dùng hợp lệ . Một khi được xác nhận , một message sẽ được gửi đến người quản lý volume thông qua AMQP.
    - Cinder-volume tiến hành đọc message từ hàng đợi , gọi Cinder driver tương ứng với volume được gắn vào.
    - Cinder driver chuẩn bị Cinder Volume chuẩn bị cho việc attach (các bước cụ thể phụ thuộc vào giao thức lưu trữ được sử dụng).
    - Cinder-volume thưc hiện gửi thông tin phản hồi đến cinder-api thông qua hàng đợi AMQP.
    - Cinder-api thực hiện quá trình đọc message phản hồi từ cinder-volume từ hàng đợi; Truyền thông tin kết nối đến RESTful phản hồi gọi tới NOVA.
    - Nova tạo ra kết nối với bộ lưu trữ thông tin được trả về Cinder.
    - Nova truyền volume device/file tới hypervisor , sau đó gắn volume device/file vào máy ảo client như một block device thực thế hoặc ảo hóa (phụ thuộc vào giao thức lưu trữ).
## **3) Backup volume**
<p align=center><img src=https://i.imgur.com/4bm3mod.png></p>

- Quá trình backup volume :
    - Client yêu cầu backup volume qua API
    - Cinder-api xác nhận yêu cầu, thông tin người dùng. Sau khi xác thực, gửi message qua AMQP
    - Cinder-backup đọc message từ hàng đợi, tạo 1 bản ghi trong DB để backup và tìm thông tin của volume cần được backup
    - Cinder-backup gọi phương thức backup_volume của Cinder volume driver tương ứng với volume thực hiện backup.
    - Cinder-volume driver thích hợp sẽ được gán vào volume cần backups
    - Volume driver gọi phương thức backup cho backup service cấu hình, xử lý các volume gán vào
    - Backup service chuyển dữ liệu và metadata của volume sang Backup repository (Nơi lưu trữ các bản backups)
    - Backup service cập nhật DB với bản ghi đã hoàn thành cho bản backup và gửi message phản hồi lên Cinder-api qua hàng đợi AMQP
    - Cinder-api đọc message từ hàng đợi và chuyển kết quả cho Client
## **4) Restore volume**
<p align=center><img src=https://i.imgur.com/0X5xF3T.png></p>

- Quá trình restore volume :
    - Client gửi yêu cầu restore volume thông qua REST API
    - Cinder-api xác nhận yêu cầu, thông tin user. Sau khi xác nhận, nó sẽ gửi message vào AMQP
    - Cinder-backup đọc message từ hàng đợi, tìm bản ghi trong DB của bản backup
    - Cinder-backup gọi phương thức backup_restore của Cinder volume driver tương ứng với volume được sao lưu. Kết nối volume với backup service được sử dụng (NFS, ...)
    - Cinder-volume driver thích hợp sẽ được gán vào volume đích
    - Volume driver gọi phương thức restore để cấu hình backup service, xử lý các volume đính kèm
    - Backup service tìm vị trí và metadata của volume trong backup repository và sử dụng chúng để restore volume đến trạng thái khớp với bản backup
    - Sau khi restore xong, backup service gửi lại phản hồi về cho cinder-api qua AMQP
    - Cinder-api đọc thông điệp từ cinder-backup và chuyển kết quả cho client