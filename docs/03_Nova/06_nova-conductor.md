# `nova-conductor`
- `nova-conductor` như một nơi điều phối các task. Rebuilt, resize/migrate và building một instance đều được quản lý ở đây. Điều này làm cho việc phân chia trách nhiệm tốt hơn giữa những gì compute nodes nên xử lý và những gì scheduler nên được xử lý, để dọn dẹp các path của execution.
- Ví dụ: một old process để building một instance là:
    - API nhận request để build một instance.
    - API send một RPC cast để scheduler pick một compute
    - Scheduler sends một RPC cast để compute build một instance,   scheduler có thể sẽ cần giao tiếp với tất cả các compute
    - Nếu build thành công thì dừng ở đây
    - Nếu thất bại thì compute sẽ quyết định nếu max number của scheduler retries là hit. và dừng lại ở đó
    - Nếu việc build được lên lịch lại thì compute sẽ send một RPC cast tới scheduler để pick một compute khác.
