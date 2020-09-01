# `cinder-scheduler`
## **1) Giới thiệu**
- Giống với `nova-scheduler`, **cinder** cũng có một daemon chịu trách nhiệm cho việc quyết định xem sẽ tạo cinder volume ở đâu khi mô hình có hơn một backend storage. Mặc định nếu người dùng không chỉ rõ host để tạo máy ảo thì `cinder-scheduler` sẽ thực hiện filter và weight theo những opions sau:

    <img src=https://i.imgur.com/fZrbYBu.png>

- Để cấu hình filter theo ý muốn, cần cấu hình thêm `filter_scheduler`.
## **2) Filters**
- `AvailabilityZoneFilter`: Filter bằng availability zone
- `CapabilitiesFilter`: Filter theo tài nguyên (máy ảo và volume)
- `CapacityFilter`: Filter dựa vào công suất sử dụng của volume backend
- `DifferentBackendFilter`: Lên kế hoạch đặt các volume ở các backend khác nhau khi có 1 danh sách các volume
- `DriverFilter`: Dựa vào filter function và metrics.
- `InstanceLocalityFilter`: lên kế hoạch cho các volume trên cùng 1 host. Để có thể dùng filter này thì Extended Server Attributes cần được bật bởi nova và user sử dụng phải được khai báo xác thực trên cả nova và cinder.
- `JsonFilter`: Dựa vào `JSON-based grammar` để chọn lựa backends
- `RetryFilter`: Lọc ra các node đã được thử trước đó
    - Host có thể bỏ qua filter này nếu nó chưa được cố gắng thử scheduling trước đó. Scheduler sẽ cần phải thêm các host đã được thử trước đó vào retry key của `filter_properties` để có thể làm việc một cách chính xác. Ví dụ:
        ```
        {
        'retry': {
                    'backends': ['backend1', 'backend2'],
                    'num_attempts': 3,
                }
        }
        ```
- `SameBackendFilter`: Lên kế hoạch đặt các volume có cùng backend như những volume khác.
## **3) Weights**
- `AllocatedCapacityWeigher` : `Allocated Capacity Weigher` sẽ tính trọng số của host bằng công suất được phân bổ. Nó sẽ đặt volume vào host được khai báo chiếm ít tài nguyên nhất.
- `CapacityWeigher` : Trạng thái công suất thực tế chưa được sử dụng.
- `ChanceWeigher` : Tính trọng số random, dùng để tạo các volume khi các host gần giống nhau
- `GoodnessWeigher` : Gán trọng số dựa vào goodness function. Goodness rating
    ```
    0 -- host is a poor choice
    .
    .
    50 -- host is a good choice
    .
    .
    100 -- host is a perfect choice
    ```
- `VolumeNumberWeigher`: Tính toán trọng số của các host bởi số lượng volume trong backends.
## **4) Affinity và Anti-affinity policy**
- Có thể sử dụng ***affinity*** hoặc ***anti-affinity policy*** để hỗ trợ việc scheduler giữa các backend :
    - ***Affinity*** : 2 instance sẽ đặt cùng backend
    - ***Anti-affinity*** : 2 instance sẽ được đặt khác backend
- **VD :**
    - Tạo 1 volume cùng backend với Volume_A :
        ```
        # openstack volume create --hint same_host=Volume_A-UUID --size SIZE VOLUME_NAME
        ```
    - Tạo 1 volume khác backend với Volume_A :
        ```
        # openstack volume create --hint different_host=Volume_A-UUID --size SIZE VOLUME_NAME
        ```
    - Tạo volume cùng backend với Volume_A và Volume_B :
        ```
        # openstack volume create --hint same_host=Volume_A-UUID --hint same_host=Volume_B-UUID --size SIZE VOLUME_NAME
        ```
        hoặc
        ```
        # openstack volume create --hint different_host=Volume_A-UUID --hint different_host=Volume_B-UUID --size SIZE VOLUME_NAME
        ```
    - Tạo volume khác backend với Volume_A và Volume_B :
        ```
        # openstack volume create --hint different_host=Volume_A-UUID --hint different_host=Volume_B-UUID --size SIZE VOLUME_NAME
        ```
        hoặc
        ```
        # openstack volume create --hint different_host="[Volume_A-UUID, Volume_B-UUID]" --size SIZE VOLUME_NAME
        ```