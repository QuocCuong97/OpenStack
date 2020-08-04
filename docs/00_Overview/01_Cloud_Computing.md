# Tổng quan về Cloud Computing
### **Định nghĩa về Cloud Computing**
- Theo định nghĩa của [**NIST**](https://csrc.nist.gov/publications/detail/sp/800-145/final) : " **Cloud computing** ( *điện toán đám mây* ) là mô hình cho phép access qua network để lựa chọn và sử dụng ***computing resource*** (VD: networks, servers, storage, application, và service) theo nhu cầu một cách thuận tiện và nhanh chóng; đồng thời cho phép kết thúc sử dụng dịch vụ, giải phóng tài nguyên dễ dàng, giảm thiểu các giao tiếp với nhà cung cấp. Mô hình **cloud** có `5` đặc tính, `4` mô hình triển khai và `3` mô hình dịch vụ. "
### **5 đặc tính**
- ***On-demand self-service*** : Khả năng tự phục vụ của người dùng, chủ động khởi tạo, tạm dừng dịch vụ …
- ***Broad network access*** : Khả năng truy cập trên mọi nền tảng thiết bị, mọi loại hạ tầng về mạng, khu vực địa lý.
- ***Resource pooling*** : Khả năng gom – gộp tài nguyên vật lý – sau đó phân bổ một cách tự động cho người sử dụng – dựa vào nhu cầu.
- ***Rapid elasticity*** : Khả năng "co giãn và đàn hồi" tài nguyên một cách nhanh chóng và thuật tiện. Có thể cấp phát và thu hồi một cách nhanh chóng.
- ***Measured service*** : Khả năng đo lường dịch vụ để kiểm tra thời gian sử dụng – từ đó tính toán chi phí theo mức độ sử dụng dịch vụ.
### **4 mô hình triển khai**
- **Private Cloud** : Được cung cấp cho nội bộ tổ chức, ít nhu cầu bảo mật và tính pháp lý hơn so với Public Cloud. 
- **Public Cloud** : Là dịch vụ cung cấp cho khác hàng sử dụng thông qua internet, thường là thương mại hóa. **VD :** **AWS**, **Azure**,...
- **Hybird Cloud** : Sự kết hợp giữa **Public Cloud** và **Private Cloud**.
- **Community Cloud** : Sự kết hợp của nhiều **CSP – Cloud Service Provider**.
### **3 mô hình dịch vụ**
<p align=center><img src=https://i.imgur.com/80mV65A.png width=80%></p>

- **Iaas – Infrastructure as a Services** : 
    - Là mô hình dịch vụ ***pay-per-use*** (tức là trả tiền cho những gì sử dụng). Chi phí sử dụng dịch vụ này được tính dựa trên chức năng và lượng tài nguyên mà khách hàng dùng. Theo **Amazon** thì đây là mức độ cơ bản nhất của điện toán đám mây.
    - Nhà cung cấp dịch vụ **Iaas** sẽ bán cho khách hàng các server (máy chủ), thiết bị mạng, bộ nhớ, CPU, storage (không gian lưu trữ), máy tính (có thể máy thật hoặc máy ảo, tùy nhu cầu), trang thiết bị trung tâm dữ liệu và một số tính năng bảo vệ an ninh nâng cao.
    - Với hạ tầng mà **Iaas** tạo ra, bạn cần vào đó và thiết lập. Và cài thêm những phần mềm cần thiết khác như web server, database,… **Iaas** không được tạo ra để phục vụ cho người dùng cuối. Mà nó để cho các công ty, đơn vị phát hành web sử dụng với mục đích triển khai phần mềm.
    - **VD :** DigitalOcean, Linode, Rackspace, Amazon Web Services (AWS), Cisco Metapod, Microsoft Azure, Google Compute Engine (GCE),...
- **PaaS – Platform as a Services** : 
    - **Paas** là mô hình dịch vụ giúp các developer có thể phát triển. Nó cho phép triển khai các ứng dụng, website trên đám mây. **Paas** về cơ bản cũng khá giống với Iaas nhưng cấp độ cao hơn một chút. **Paas** được trang bị thêm các công cụ phát triển doanh nghiệp thông minh (BI), middleware và nhiều tool khác. Với **Paas**, bạn sẽ có một nền tảng (Platform) được cài đặt sẵn để phù hợp cho việc phát triển ứng dụng.
    - Nhà cung cấp sẽ lo cho bạn từ OS (Windows hoặc  Linux) cho tới Runtime (Docker, NodeJS, C#, Java), chỉ cần bỏ code vào mà chạy là được.
    - **VD :** AWS Elastic Beanstalk, Windows Azure, Heroku, Force.com, Google App Engine, Apache Stratos, OpenShift,...
- **SaaS – Software as a Services** : 
    - **Saas** là một mô hình dịch vụ điện toán đám mây cao nhất hiện nay. Cho phép người dùng sử dụng được các ứng dụng dễ dàng trên nền tảng đám mây thông qua internet. Đơn giản hơn, **Saas** sẽ cung cấp phần mềm/ứng dụng chạy trên internet. Từ đó người dùng cuối (end-user) có thể sử dụng ngay. Nhà cung cấp dịch vụ **Saas** có thể lưu trữ trên server của họ. Hoặc cho phép người dùng tải xuống và vô hiệu hóa nó khi hết hạn.
    - Ví dụ điển hình cho mô hình dịch vụ này là **Microsoft Office 365**. Đôi khi các web email (**Gmail**, **Outlook**, **Yahoo Mail**,..) cũng dùng dịch vụ này. Đây đều là các sản phẩm hoàn chỉnh. Người dùng có thể sử dụng ngay lập tức mà không cần phải thiết lập server để quản lý.
    - Tương tự, **OneDrive**, **Dropbox** cũng là mô hình điện toán đám mây kiểu **Saas**. Các trang web (phần mềm) này cung cấp không gian lữu trữ cần thiết để bạn có thể upload/download dữ liệu thông qua internet.
    - **VD :** 	Google Apps, Dropbox, Salesforce, Cisco WebEx, Concur, GoToMeeting,...
### **Lợi ích từ Cloud Computing**
- ***Tiết kiệm chi phí*** : Giúp giảm thiểu chi phí. Bạn sẽ không tốn tiền đầu tư cơ sở hạ tầng ban đầu. Ví dụ như: mua phần cứng, phần mềm, lắp đặt hệ thống,…
- ***Tiện lợi*** : Người dùng có thể nhanh chóng truy cập, sử dụng tài nguyên thông qua internet mà không cần cài đặt phức tạp
- ***An toàn và liên tục*** : Mọi dữ liệu được đồng bộ hóa trên đám mây. Giúp đảm bảo độ an toàn cao hơn, tránh trường hợp mất dữ liệu do hư hỏng ổ cứng. Ngoài ra, nhà cung cấp sẽ sao lưu định kỳ và có các phương thức bảo mật để bảo vệ dữ liệu tốt hơn.
- ***Triển khai nhanh chóng ở bất kỳ đâu*** : Chỉ với một vài thao tác đơn giản để triển khai chúng mọi nơi. Điều này đồng nghĩa với việc người dùng sẽ có được trải nghiệm tốt hơn với độ trễ thấp hơn.
### **Thực trạng tại Việt Nam**
- Ở Việt Nam, đa phần các công ty lớn dùng dạng **On-Premise**, tự xây dựng hệ thống hạ tầng vì họ có team IT riêng. Ở nước ngoài, các công ty startup, công ty vừa và nhỏ thường dùng **IaaS** hoặc **PaaS** do **Google**, **Amazon**, **Azure** cung cấp để đỡ tốn chi phí cho IT.
- Phần lớn các công ty đều sử dụng SaaS cho các hoạt động thường ngày (thay vì tự phát triển):
    - **Slack** để giao tiếp giữa các thành viên
    - **Jira** để quản lý task
    - **Confluence** hoặc **Google Docs** để quản lý document
    - **Skype** hoặc **Jitsi** để họp online 
