# Làm việc với Volume trên Horizon
## **1) Tạo instance (tạo mới volume)**
- **B1 :** Tại màn hình Dashboard, chọn tab ***Project*** -> ***Compute*** -> ***Instance*** :

    <img src=https://i.imgur.com/q6zkN3e.png>

- **B2 :** Chọn **Launch Instance** để thêm instance (VM) mới :

    <img src=https://i.imgur.com/AV0V6tv.png>

- **B3 :** Tại tab **Details**, nhập tên VM => ***Next*** để qua tab tiếp theo :

    <img src=https://i.imgur.com/sbCSKms.png>

- **B4 :** Tại tab **Source**, chọn kích cỡ volume, image muốn boot cho VM => Next để qua tab tiếp theo :

    <img src=https://i.imgur.com/huSLpuC.png>

    > Tùy chọn ***Delete Volume on Instance Delete*** cho phép volume cũng bị xóa khi instance bị xóa
- **B5 :** Tại tab **Flavor**, chọn gói cấu hình cho VM => Next để qua tab tiếp theo :

    <img src=https://i.imgur.com/FQJqrEg.png>

    > **Flavor** được chọn chỉ ảnh hưởng đển RAM và CPU của instance, Disk của instance sẽ là kích cỡ của volume
- **B6 :** Tại tab **Networks**, chọn các dải network muốn cấp cho VM => **Launch Instance** để tạo VM:

    <img src=https://i.imgur.com/lbYoTDu.png>

    > Đến đây đã đủ thông tin cần thiết để tạo VM, nếu muốn cấu hình thêm, có thể chọn Next để lướt qua các tab cấu hình khác
    > Với các image lớn có thể mất nhiều thời gian để tạo VM
- **B7 :** Instance sau khi tạo :

    <img src=https://i.imgur.com/2gPY5Jk.png>

- **B8 :** Kiểm tra lại cấu hình instance :

    <img src=https://i.imgur.com/duUJTHz.png>

    > Theo kết quả ta thấy thông số RAM, CPU của instance là của **Flavor**, thông số disk là của **volume**
## **2) Tạo instance (có sẵn volume)**
### **2.1) Tạo volume**
- **B1 :** Tại màn hình Dashboard, chọn tab ***Project*** -> ***Volumes*** -> ***Volumes*** :

    <img src=https://i.imgur.com/8qNQKm3.png>

- **B2 :** Chọn **Create Volume** để thêm instance (VM) mới :

    <img src=https://i.imgur.com/cCmMmiM.png>

- **B3 :** Tại cửa sổ **Create Volume** :
    - `(1)` : **Volume Name** : Tại đây có thể nhập tên volume nhằm mục đích quản lý, hoặc để trống thì tên volume sẽ tự động được đặt là ID của volume .
    - `(2)` : **Volume Source** : có 2 option :
        - ***Image*** : nếu chọn option này thì sẽ cần chọn thêm image ở bước `(3)` : **Use Image as a source***
        - ***No source, empty volume*** : nếu chọn option này thì lúc tạo VM cần chọn thêm image để boot
    - `(4)` : **Size (GiB)*** Kích cỡ volume
    - `(5)` : Chọn **Create Volume** để tạo volume

    <img src=https://i.imgur.com/A78R9sn.png>

- **B4 :** Volume sau khi tạo xong :

    <img src=https://i.imgur.com/KgVkfU9.png>

### **2.2) Tạo instance**
- **B5 :** Tạo instance tương tự như [**phần 1**](#1). Chú ý bước chọn Volume :
    - `(1)` : **Select Boot Source** : tại đây chọn volume thay vì chọn image
    - `(2)` : **Delete Volume on Instance Delete** : tùy chọn cho phép tự động xóa volume khi xóa instance .
    - `(3)` : **Allocated** : chọn volume đã tạo trước đó 

    <img src=https://i.imgur.com/cbd85bK.png>

- **B6 :** Instance sau khi tạo :

    <img src=https://i.imgur.com/2WZ8LLK.png>

- **B7 :** Console vào instance để kiểm tra lại cấu hình :

    <img src=https://i.imgur.com/8hMxcZl.png>

    > Theo kết quả ta thấy thông số RAM, CPU của instance là của **Flavor**, thông số disk là của **volume**
## **3) Snapshot**
> **Phép thử :** Tạo file `test_snapshot` trên instance muốn snapshot, sau đó thử xóa đi và revert lại khi instance vẫn còn file `test_snapshot` 
### **3.1) Tạo snapshot**
- **B1 :** Tạo file `test_snapshot` :

    <img src=https://i.imgur.com/KKtiUMV.png>

- **B2 :** Tại màn chứa danh sách các instance, chọn **Create Snapshot** tại instance muốn tạo **snapshot** :

    <img src=https://i.imgur.com/yW1PwXY.png>

- **B3 :** Tại cửa sổ **Create Snapshot**, nhập tên **snapshot**, sau đó chọn ***Create Snapshot*** để tạo :

    <img src=https://i.imgur.com/AHURwIS.png>

- **B4 :** Kiểm tra bản **snapshot** vừa tạo tại tab ***Project*** -> ***Volumes*** -> ***Snapshots*** :

    <img src=https://i.imgur.com/OhycEiG.png>

- **B5 :** Thử xóa file `test_snapshot` vừa tạo trên instance :
    
    <img src=https://i.imgur.com/8LzA1mX.png>

#### **3.2) Tạo instance mới gắn với volume vừa tạo**
- **B6 :** Các bước tạo **instance** tương tự [**phần 1**](#1). Chú ý bước chọn **volume** :
    
    <img src=https://i.imgur.com/jhmQeRR.png>

- **B7 :** Instance sau khi tạo xong :s

    <img src=https://i.imgur.com/r2TiA0h.png>

- **B8 :** Console vào instance và kiểm tra :

    <img src=https://i.imgur.com/aknjul9.png>