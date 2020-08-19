# Quá trình khởi tạo Instance
## **1) Quá trình khởi tạo instance với 1 node controller và 1 node compute**
<img src=https://i.imgur.com/VeM1eXK.png>

- **Horizon Dashboard** hoặc **Openstack CLI** lấy thông tin đăng nhập và chứng thực của người dùng cùng với định danh service thông qua REST API để xác thực với **Keystone** sinh ra token.
- Sau khi xác thực thành công, client sẽ gửi request khởi chạy máy ảo tới **`nova-api`**.
- **`nova-api`** xác thực lại thông token với **Keystone** và nhận header với roles và permission
- **`nova-api`** gửi lệnh tới **`nova-conductor`** kiểm tra trong database conflicts hay không để khởi tạo một entry mới.
- **`nova-api`** gửi RPC tới **`nova-scheduler`** service để lập lịch tạo instance
- **`nova-scheduler`** lấy request từ message queue
- **`nova-scheduler`** thông qua filters và weights để tìm compute host phù hợp nhất chạy instance. Đồng thời trên database sẽ cập nhật lại entry của instance với host ID nhận được từ **`nova-scheduler`**. Sau đó **`nova-scheduler`** gửi RPC call tới **`nova-compute`** để khởi tạo máy ảo.
- **`nova-conductor`** lấy request từ message queue.
- **`nova-conductor`** lấy thông tin instance từ database sau đó gửi về cho **`nova-compute`**
- **`nova-compute`** lấy thông tin máy ảo từ queue. Tại thời điểm này, compute host đã biết được image nào sẽ được sử dụng để chạy instance. **`nova-compute`** sẽ hỏi tới **`glance-api`** để lấy url của image .
- **`glance-api`** sẽ xác thực token và gửi lại metadata của image trong đó bao gồm cả url của nó.
- **`nova-compute`** sẽ đưa token tới **`neutron-api`** và hỏi nó về network cho instance.
- Sau khi xác thực token, neutron sẽ tiến hành cấu hình network.
- **`nova-compute`** tương tác với **`cinder-api`** để gán volume vào instance.
- **`nova-compute`** sẽ generate dữ liệu cho Hypervisor và gửi thông tin thông qua libvirt.

## **2) Các trạng thái và quá trình chuyển đổi trạng thái của instance**

<img src=https://i.imgur.com/K89DzoB.png>

## **3) Các yêu cầu khi thực hiện các lệnh**

| Command | Req’d VM States | Req’d Task States | Target State |
|---------|-----------------|-------------------|------|-------|
| `pause` | Active, Shutoff, Rescued | Resize Verify, unset | Paused |
| `unpause` | Paused | N/A | Active |
| `suspend` | Active, Shutoff | N/A | Suspended |
| `resume` | Suspended | N/A | Active |
| `rescue` | Active, Shutoff | Resize Verify, unset | Rescued |
| `unrescue` | Rescued | N/A | Active |
| `set admin password` | Active | N/A | Active |
| `rebuild` | Active, Shutoff | Resize Verify, unset | Active, Shutoff |
| `force delete` | Soft Deleted | N/A | Deleted |
| `restore` | Soft Deleted | N/A | Active |
| `delete` | Active, Shutoff, Building, Rescued, Error | N/A | Deleted |
| `backup` | Active, Shutoff | N/A | Active, Shutoff |
| `snapshot` | Active, Shutoff | N/A | Active, Shutoff |
| `start` | Shutoff, Stopped | N/A | Active |
| `stop` | Active, Shutoff, Rescued | Resize Verify, unset | Stopped |
| `reboot` | Active, Shutoff, Rescued | Resize Verify, unset | Active |
| `resize` | Active, Shutoff | Resize Verify, unset | Resized |
| `revert resize` | Active, Shutoff | Resize Verify, unset | Active |
| `confirm resize` | Active, Shutoff | Resize Verify, unset | Active |

## **4) Các lệnh có thể thực hiện với các trạng thái của instance**

| Instance State | Commands |
|----------------|----------|
| Paused | `unpause` |
| Suspended | `resume` |
| Active | `set admin password`, `suspend`, `pause`, `rescue`, `rebuild`, `soft delete`, `delete`, `backup`, `snapshot`, `stop`, `reboot`, `resize`, `revert resize`, `confirm resize` |
| Shutoff | `suspend`, `pause`, `rescue`, `rebuild`, `soft delete`, `delete`, `backup`, `start`, `snapshot`, `stop`, `reboot`, `resize`, `revert resize`, `confirm resize` |
| Rescued | `unrescue`, `pause` |
| Stopped | `rescue`, `delete`, `start` |
| Soft Deleted | `force delete`, `restore` |
| Error | `soft delete`, `delete` |
| Building | `delete` |
| Rescued | `delete`, `stop`, `reboot` |

## **5) Quá trình thay đổi trạng thái khi khởi tạo Instance**
- Quá trình tuần tự thay đổi trạng thái VM, task state, power state khi tạo mới instance được thể hiện qua lược đồ sau :

    <img src=https://i.imgur.com/1pxO5qZ.png>

## **6) Các yếu tố tối thiểu để tạo một Instance**
- Chọn flavor :
    ```
    # openstack flavor list
    ```
- Chọn image :
    ```
    # openstack image list
    ```
- Chọn security group :
    ```
    # openstack security group list
    ```
- Chọn key pair (không bắt buộc) :
    ```
    # openstack keypair list
    ```
--------------------
[Tham khảo](https://docs.openstack.org/nova/latest/reference/vm-states.html#requirements-for-commands)