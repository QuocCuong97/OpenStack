# Evacuate VM
## **1) Giới thiệu**
- **Evacuation** là một kĩ thuật được dùng để chuyển máy ảo từ một node compute đã chết hoặc bị tắt sang một node compute khác ở trong cùng một môi trường. Vì thế nó chỉ có tác dụng khi máy ảo sử dụng shared storage hoặc block storage bởi nếu không thì ổ cứng của máy ảo sẽ không thể được truy cập từ bên ngoài trong trường hợp host bị chết. Trong trường hợp rebuild máy ảo được boot từ local sử dụng ephemeral disk thì một máy mới sẽ được tạo mang cùng thông số của máy ảo cũ (IP, ID, flavor...) nhưng ổ đĩa lúc này đã mất đồng nghĩa với việc dữ liệu cũng không còn nữa.
- **Evacuation** cho phép người dùng lựa chọn host mới, nếu không thì compute node sẽ được lựa chọn bởi scheduler.
    > Chỉ có thể evacuate máy ảo khi compute node đã bị tắt.
- Một số kĩ thuật khác được dùng để vận chuyển máy ảo
    - Tạo ra một bản copy của máy ảo cho mục đích backup hoặc copy nó tới một môi trường/hệ thống mới, sử dụng snapshot (nova image-create)
    - Di chuyển máy ảo ở trạng thái static tới host trên cùng 1 môi trường/hệ thống, sử dụng cold migrate (nova migrate)
    - Di chuyển máy ảo ở trạng thái đang chạy tới host mới trên cùng 1 môi trường/hệ thống, sử dụng live migrate (nova live-migration)
## **2) Luồng hoạt động evacuate**
<img src=https://i.imgur.com/JVKIGWX.png>

- User gửi yêu cầu evacuate máy ảo tới `nova-api`
- `nova-api` tiếp nhận yêu cầu, kiểm tra các tùy chọn
- `nova-api` gửi yêu cầu lấy instance object tới `nova-compute`, sau khi lấy được, nó sẽ gọi tới phương thức evacuate.
- `nova-compute` đặt trạng thái cho máy ảo là `REBUILDING` và record lại action của máy ảo là `REBUILD` vào database.
- `nova-compute` yêu cầu nova compute manager tạo lại máy ảo.
- `nova-compute` check lại tài nguyên
- `nova-compute` đặt trạng thái instance là `REBUILDING` trong database thông qua `nova-conductor`
- Nếu capacity ok thì nó sẽ bắt đầu yêu cầu `neutron` set up network cho máy ảo. Trường hợp capacity không ok thì những thông tin về network và bdm (block device mapping) sẽ bị xóa bởi `nova-compute` driver.
- `nova-compute` lấy network info từ network service
- `nova-compute` lấy tất cả các thông tin của bdm thông qua `nova-conductor`
- Sau khi có được thông tin, nó sẽ yêu cầu volume service gỡ volume ra khỏi máy ảo cũ.
- `nova-compute` sau đó thiết lập trạng thái `REBUILD_BLOCK_DEVICE_MAPPING` cho máy ảo thông qua `nova-conductor`
- `nova-compute` yêu cầu thiết lập bdm cho máy ảo mới.
- Trạng thái của máy ảo bắt đầu được chuyển thành `REBUILD_SPAWNING`.
Cùng lúc đó, nova compute yêu cầu `nova-compute` driver spawn máy ảo với những thông tin đã có sẵn
- Trạng thái của máy ảo chuyển thành `ACTIVE`
## **3) Sự khác biệt giữa evacuate, migrate và live-migrate**

| Evacuate | Migrate | Live-migrate |
|----------|---------|--------------|
| Rebuild máy ảo đang ở trên một compute node (đã down) sang một compute node khác | Rebuild máy ảo đang ở trên một compute node (đang chạy) sang một compute node khác | Di chuyển máy ảo tới một node khác mà không có downtime (hoặc downtime không đáng kể) |

## **4) Cú pháp lệnh evacuate**
- Cú pháp :
    ```
    # nova evacuate [--password pass] [--on-shared-storage] instance_name [target_host]
    ```
    - Trong đó :
        - `--password pass` : Admin password cho instance sau khi evacuate (không thể sử dụng nếu đi kèm với tùy chọn `--on-shared-storage`). Nếu password không được chỉ định, một random password sẽ được generate sau khi quá trình evacuate kết thúc.
        - `--on-shared-storage` : Tất cả mọi file của máy ảo đều ở trên shared storage
        - `instance_name` : Tên máy ảo cần evacuate
        - `target_host` : Host chứa máy ảo sau khi rebuild, nếu không lựa chọn thì scheduler sẽ làm nhiệm vụ này.
- **VD :** Có 2 node `compute1` và `compute2`. `VM04` chạy trên `compute2` và `compute2` bị sập. `VM04` sử dụng **Block Storage** . Ta cần khôi phục lại `VM04` trên node `compute1` :
    ```
    # nova evacuate VM04 compute1
    ```
    => Kết quả :

    <img src=https://i.imgur.com/P8ERr5d.png>

    <img src=https://i.imgur.com/FQmuye8.png>

    > Theo kết quả ta thấy instance đã được khôi phục lại thành công trên `compute1`