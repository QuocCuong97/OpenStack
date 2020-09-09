# Thêm Router trên Horizon
- **B1 :** Tại màn hình Dashboard, chọn tab ***Project*** -> ***Network*** -> ***Routers*** :

    <img src=https://i.imgur.com/Mi4tMvN.png>

- **B2 :** Chọn **Create Router** :

    <img src=https://i.imgur.com/EszLhPb.png>

- **B3 :** Tại cửa sổ **Create Router** :
    - **Router Name** : tên router muốn tạo
    - **External Network** : chọn dải mạng external
    - **Enable SNAT** : chọn để hỗ trợ NAT từ private ra external network

    <img src=https://i.imgur.com/Il0b5Zs.png>

- **B4 :** Sau khi tạo router, chọn vào router để cấu hình chi tiết :

    <img src=https://i.imgur.com/eOPSz3F.png>

- **B5 :** Chọn **Add Interface** :

    <img src=https://i.imgur.com/BS5Bzdp.png>

- **B6 :** Tại cửa sổ ***Add Interface***, chọn subnet muốn add vào interface => ***Submit*** :

    <img src=https://i.imgur.com/kNRVJlT.png>

- **B7 :** Sau khi add thành công interface, truy cập tab ***Network Topology*** để xem lại mô hình mạng :

    <img src=https://i.imgur.com/1yCv992.png>

- **B8 :** Ta thấy mô hình router vừa tạo :

    <img src=https://i.imgur.com/DzkwuzH.png>

- **B9 :** Kiểm tra lại các instance chỉ kết nối mạng private, đã có thể ra được Internet :

    <img src=https://i.imgur.com/MTF6K6d.png>