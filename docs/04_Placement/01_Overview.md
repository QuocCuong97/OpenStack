# Tổng quan về Placement
## **1) Giới thiệu**
- **Placement** được phát hành lần đầu trong bản `Newton`, dần dần tách thành một project riêng từ bản `Stein`
- **Placement** cho phép các project kiểm soát tài nguyên của chúng. Các project có thể đăng ký/xóa thông tin tài nguyên của chúng thông qua HTTP API
- 2 process tương tác phần lớn với **placement** là `nova-compute` và `nova-scheduler` .
- Trình theo dõi tài nguyên trong `nova-compute` chịu trách nhiệm tạo bản ghi ***resource provider*** tương ứng với các compute node, thiết lập các bản ghi ***inventory*** mô tả các tài nguyên định sẵn có thể sử dụng (VD : VCPU, MEMORY_MB, DISK_GB) và thiết lập các khía cạnh định tính của tài nguyên (VD: STORAGE_DISK_SSD)
- `nova-scheduler` sẽ chịu trách nhiệm lựa chọn các host phù hợp mỗi khi tạo instance. Nó sẽ tính toán dựa trên các thông tin có trên **placement**
