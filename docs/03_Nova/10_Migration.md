# Migration
## **1) Giới thiệu**
- **Migration** là quá trình di chuyển máy ảo từ host vật lí này sang một host vật lí khác. **Migration** được sinh ra để hỗ trợ việc bảo trì nâng cấp hệ thống. Ngày nay tính năng này đã được phát triển để thực hiện nhiều tác vụ hơn:
    - Cân bằng tải: Di chuyển VMs tới các host khác khi phát hiện host đang chạy có dấu hiệu quá tải.
    - Bảo trì, nâng cấp hệ thống: Di chuyển các VMs ra khỏi host trước khi tắt nó đi.
    - Khôi phục lại máy ảo khi host gặp lỗi: Restart máy ảo trên một host khác.
- Trong **OpenStack**, việc **migrate** được thực hiện giữa các node compute với nhau hoặc giữa các project trên cùng 1 node compute.
- **Openstack** cung cấp 1 số phương pháp khác nhau đề **migrate** VM. Các phương pháp này có các hạn chế khác nhau:
    - Không thể live-migrate với shared storage.
    - Không thể live-migrate nếu đã bật enabled.
    - Không thể select target host nếu sử dụng nova migrate.
- **OpenStack** hỗ trợ hai kiểu **migration** đó là:
    - **Cold migration**: Non-live migration
    - **Live migration**:
        - True live migration (shared storage or volume-based)
        - Block live migration