# JWT Token trong Keystone
## **1) Giới thiệu**
- **JSON Web Signature (JWS)** token được giới thiệu lần đầu trong bản `Stein` .
- So với **fernet**, **JWS** mang đến một lợi ích tiềm năng để vận hành bằng cách giới hạn số host cần chia sẻ ***symmetric key***. Điều này giúp phòng tránh các tác nhân độc hại đang ở một node nào đó tấn công sang các node khác .


Compared to fernet, JWS offer a potential benefit to operators by limiting the number of hosts that need to share a symmetric encryption key. This helps prevent malicious actors who might already have a foothold in your deployment from spreading into additional nodes.