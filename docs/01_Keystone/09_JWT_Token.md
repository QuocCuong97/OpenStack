# JWS Token trong Keystone
## **1) Giới thiệu**
- **JSON Web Signature (JWS)** token được giới thiệu lần đầu trong bản `Stein` .
- **JWS** được định nghĩa trong RFC 7519
- **JWS** hỗ trợ mã hóa bất đối xứng (***asymmetric***), signed và mã hóa payload
- Lý do không sử dụng UUID Token: token cố định
    ```
    779810523fb24886b67a23f4f823b685
    ```
- Lý do không sử dụng PKI Token : token quá lớn
    ```
    MIIE-gYJKoZIhvcNAQcCoIIE7zCCBOsCAQExDTALBglghkgBZQMEAgEwggNMBgkqhkiG9w0BBwGgggM9BIIDO
    XsidG9rZW4iOnsibWV0aG9kcyI6WyJwYXNzd29yZCJdLCJyb2xlcyI6W3siaWQiOiIzNjBiMTc3ZDhjMjM0
    N2ZmOTVlMGFjMTYxNWJhOGZiNiIsIm5hbWUiOiJhZG1pbiJ9XSwiZXhwaXJlc19hdCI6IjIwMTUtMDItMjZ
    UMDU6NDg6MjYuMDk0MDk4WiIsInByb2plY3QiOnsiZG9tYWluIjp7ImlkIjoiZGVmYXVsdCIsIm5hbWUiOi
    JEZWZhdWx0In0sImlkIjoiNTkwMDJjZTczOWYxNDNiYjhiMmNjMzNjYWY5OGZjZjkiLCJuYW1lIjoiYWRta
    W4ifSwiY2F0YWxvZyI6W3siZW5kcG9pbnRzIjpbeyJyZWdpb25faWQiOm51bGwsInVybCI6Imh0dHA6Ly8x
    MDQuMjM5LjE2My4yMTU6MzUzNTcvdjMiLCJyZWdpb24iOm51bGwsImludGVyZmFjZSI6ImFkbWluIiwiaWQ
    iOiI5YTI5ZWFmMjBmNzk0MmI2YjljOTZjZmIwYWEwMmEzZSJ9LHsicmVnaW9uX2lkIjpudWxsLCJ1cmwiOi
    JodHRwOi8vMTA0LjIzOS4xNjMuMjE1OjM1MzU3L3YzIiwicmVnaW9uIjpudWxsLCJpbnRlcmZhY2UiOiJwd
    WJsaWMiLCJpZCI6ImQzMjMzYWZkMmI2MDQxZDRhMzlmOGFjMTIzMzc1N2ZkIn1dLCJ0eXBlIjoiaWRlbnRp
    dHkiLCJpZCI6IjFiNzk2ZTIxNGY4MTQwMTE4MTA4YTdlNGU0Y2E2ZTE2IiwibmFtZSI6IktleXN0b25lIn1
    dLCJleHRyYXMiOnt9LCJ1c2VyIjp7ImRvbWFpbiI6eyJpZCI6ImRlZmF1bHQiLCJuYW1lIjoiRGVmYXVsdC
    J9LCJpZCI6Ijg1YTlhZjE0NWRkYjRkMTlhOTU0NGRmYmVhYzVkMWYwIiwibmFtZSI6ImFkbWluIn0sImF1Z
    Gl0X2lkcyI6WyJZeW9iU2FIY1ROQ3U3c2V1c2RUdHBRIl0sImlzc3VlZF9hdCI6IjIwMTUtMDItMjZUMDU6
    MzM6MjYuMDk0MTI3WiJ9fTGCAYUwggGBAgEBMFwwVzELMAkGA1UEBhMCVVMxDjAMBgNVBAgMBVVuc2V0MQ4
    wDAYDVQQHDAVVbnNldDEOMAwGA1UECgwFVW5zZXQxGDAWBgNVBAMMD3d3dy5leGFtcGxlLmNvbQIBATALBg
    lghkgBZQMEAgEwDQYJKoZIhvcNAQEBBQAEggEAYJR+ETbjA4RpgToeRm0qh-zxRWyBL4RdN99hLHV6foIpc
    r6uXMN-DaUJvGygPDi1wi-HAbpErJAe9iRHk4+8BUnX--jQRTaYhkg237eyjpYHU8Hgt8Ydn7Wdnn0hriXK
    t+RZBG-ZEnnP-MZ9V9GGJz-BoAMHx42uF5j6mlfVvUxtJGSaZ2wPROkLIHAjrX-8zEo8YhtGQHi-rFvXOoP
    +w8TVb907R2WNsGs3LbFKRmDv-yev6pMnz+gQu8uImf2idd18hyEYdw8M9bgZc2YsGBiPSeIm-VhzH9qTX0
    e7fK-chhAE+saIEbl5Mw0PzybhTyKHRzqtsW4HWFOlbE0yOA==
    ```
- Lý do không sử dụng PKIz Token : token quá lớn
    ```
    PKIZ_eJxtVcmSozgUvPMVc6_oKMBgm0Mf2IzBCIpVlm4sNiAEtssLy9eP7K6Jqo4YboCUysyX7-nXL_ZopmV7
    _-gger784oBtm-8VcnYnbNePwlODQj-xb6tZ1zX_qquBORqx6moVreq20nAATLUyh6rygFa1F65uG0sZeE0
    brKqqgKLZtuHvr01pKZ8YSo3fX5scpnxmKW0x2Us4OQPae3MpKhPWnZJzdWfKxZG-fi6uTQaDxm9s2TPAgE
    gwe10i-9DkPWLOfkwpIJWMYq32LId4c7LgfN2-2p1c5zBhG50aW8I5bxxlHw0N3tdDtndoISh1qdtLm9gDi
    JMbMOwbIDgBBlpyIEZLQII7mNuJnTrDhgH2GmN1pmgRvCRgS7khSO82Oa_sjrY2ObFvaYf26ZUr_2ZgYojr
    Eo683fPX78WmhOaw82MgITHtPCvhgWjzvpW2HLBwh4nX-kYgYENtmCd3BAX63IhgeMuYkUcmB4kbHsHxgb-
    8wlBuC0s5c3kfzoxafpicCcPynIvy8WVkJwu5NTA56ZQ_9Xc1X27VpTutR2AwyQTILjFFDkzSxIxZgjmZvb
    h4lAQ8WXyBSd9AHb2XVjrhbkNw9ATctDnzhbOb4at0Tu2RkIC4HX3DHDFBPIYhRXG1AHNKEUEy6hAPIJhw5
    Cju9toUXdpzGVTue_Fp1vnOzLuy04WiG56Ap3IbDn6zfoBY5V1iz34kjR4BjL4p-AQI4JkDd4HmJ4sn2hPs
    B9CZ-UOLDtdIfFVoKKFzzeBL4hm_fAELDhgVQy07TwwpjkMmg9a-0cqsTIJnPdPXDqBDC7sXSraRP-y1V4U
    yJo8dcObKbfuNSBIex7YErISFqlpgI-CxUdYotmcQOy0mxeiJKYuwR5-s825z416Otjd62Hs8KyH9Ooketu
    GE9oAl8aa8fBHT6U8Sw0cONyzu9pKV_sz90cLodxsh3wZ_BSn8imupO8o3S6_GsSkxhjyaW55jNAVECtm37
    AUmlQQgK6eFJCAC-T-aP-v-J-IbAVuUf1aP--rxNklGMekrIRM290g8NxnFt6yjJOmd3qavvpiLRUrx5u_O
    5H62JjDMH52JJMja-hhbuooSNoEsjU0iDWyGIZ1NF6itpQqJyWk10NMUjAZR2YjyUrYKaGl6Z6bxIJAGQ0V
    GGgRbQ03TvPdoaZg-UIfXZr0aNlwK5Rnvg9EyVPgHAABjUS7KSaYHa3MrrJG6nffIA1tT_2c2ckbwc6Camh
    aoZlWZ6s5fHiM7FSN_F4LPwIZ62eK-Ck7bCCpG5gpWk55VZuJb-wZ30-Uwfh6c4_0Srgp12Ak0si9usTwdm
    uUcuHlIuqUjXarRXcN-_THIn6tdAN-nPSg57PGwD4Wt2Avm6qpmghnW1w0ZrGUX7cQ3MprKmr7nWFmkufam
    ysNiZfWSqNPDabMl54Q7ykPw2Gzxx1G8gzcNvGvRvTCjTLAqtQ1dZ7xM-zxbbam8Vha3SgGNhxL8-bESItc
    8SiF3PhHSXD4Mfztp16N2Em_F8CYqviBlaj917zPUwf2h-1nsiVSIpWGKeu-Gdtc6rtfD2eRWEbn5VNhNUwivHb8i14U1yo6RNH7qf0Y4ValpVTG9nR4NMHv39zrQjM94_ty-xc2_Erg
    ```
- Lý do không sử dụng Fernet Token : chúng sử dụng mã hóa đối xứng (***symmetric key***) :
    ```
    gAAAAABU7roWGiCuOvgFcckec-
    0ytpGnMZDBLG9hA7Hr9qfvdZDHjsak39YN98HXxoYLIqVm19Egku5YR3wyI7heVrOmPNEtmrfIM1rtahudEdEAPM4HCiMrBmiA1Lw6SU8jc2rPLC7FK7nBCia_BGhG17NVHuQu0S7waA306jyKNhHwUnp
    sBQ=
    ```
- Lý do sử dụng JWT Token : tương đối nhỏ, sử dụng mã hóa bất đối xứng, token không cố định :
    ```
    eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCJ9.eyJvcGVuc3RhY2tfcHJvamVjdF9pZCI6IjMzMzNhMDQ0ZW
    MyYzQxMzNhMWQ0NGI1ZmRhYjBjMjg2Iiwic3ViIjoiM2ZlMTUxMTNjZjc5NGU4ZjljNWRhZDlmMTA3M2I
    wODkiLCJleHAiOjE1NTQxMzMzMzEsIm9wZW5zdGFja19hdWRpdF9pZHMiOlsiZW1BUVRCZWVSVmFidzI4
    QW9FRURqdyJdLCJpYXQiOjE1NTQxMjk3MzEsIm9wZW5zdGFja19tZXRob2RzIjpbInBhc3N3b3JkIl19.
    tHcVIaW43RwREduckh2itJ_RrZ5DctFElox1SsORO3Q7DsDLlWDQbuhCRuRd6_QgB0Brm1x_q7aB2lZcHy_fw=
    ```
## **2) Cấu trúc một JWS Token**
- Công thức mã hóa :
    ```
    ECDSASHA256(baseUrlEncode(header) + "." + baseUrlEncode(payload), publicKey, privateKey)
    ```
- **VD :**
    ```
    eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCJ9.eyJvcGVuc3RhY2tfcHJvamVjdF9pZCI6IjMzMzNhMDQ0ZW
    MyYzQxMzNhMWQ0NGI1ZmRhYjBjMjg2Iiwic3ViIjoiM2ZlMTUxMTNjZjc5NGU4ZjljNWRhZDlmMTA3M2I
    wODkiLCJleHAiOjE1NTQxMzMzMzEsIm9wZW5zdGFja19hdWRpdF9pZHMiOlsiZW1BUVRCZWVSVmFidzI4
    QW9FRURqdyJdLCJpYXQiOjE1NTQxMjk3MzEsIm9wZW5zdGFja19tZXRob2RzIjpbInBhc3N3b3JkIl19.
    tHcVIaW43RwREduckh2itJ_RrZ5DctFElox1SsORO3Q7DsDLlWDQbuhCRuRd6_QgB0Brm1x_q7aB2lZcHy_fw
    ```
- Trong đó :
    - Phần 1: **JWT header**
        ```
        eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCJ9
        ```
        - Nội dung :
            ```json
            {"alg": "ES256", "typ": "JWT"}
            ```
    - Phần 2: **JWT Payload**
        ```
        eyJvcGVuc3RhY2tfcHJvamVjdF9pZCI6IjMzMzNhMDQ0ZW
        MyYzQxMzNhMWQ0NGI1ZmRhYjBjMjg2Iiwic3ViIjoiM2ZlMTUxMTNjZjc5NGU4ZjljNWRhZDlmMTA3M2I
        wODkiLCJleHAiOjE1NTQxMzMzMzEsIm9wZW5zdGFja19hdWRpdF9pZHMiOlsiZW1BUVRCZWVSVmFidzI4
        QW9FRURqdyJdLCJpYXQiOjE1NTQxMjk3MzEsIm9wZW5zdGFja19tZXRob2RzIjpbInBhc3N3b3JkIl19
        ```
        - Nội dung :
            ```json
            {"openstack_project_id": "3333a044ec2c4133a1d44b5fdab0c286","sub":
            "3fe15113cf794e8f9c5dad9f1073b089","exp": 1554133331,"openstack_audit_ids":
            ["emAQTBeeRVabw28AoEEDjw"],"iat": 1554129731,"openstack_methods": ["password"]}
            ```
    - Phần 3: **JWT Signature**
        ```
        tHcVIaW43RwREduckh2itJ_RrZ5DctFElox1SsORO3Q7DsDLlWDQbuhCRuRd6_QgB0Brm1x_q7aB2lZcHy_fw=
        ```
## **3) So sánh Fernet Token và JWT Token**
- Phương thức mã hóa và sử dụng chữ ký số :
    - **Fernet** sử dụng `128bit AES-CBC` encryption key + `128bit SHA256 HMAC` signing key
    - **JWS** sử dụng `ES256` JWA signing với `ECDSA` sử dụng `P-256 curve` và `SHA256 HMAC`
- Tốc độ mã hóa và giải mã : Fernet sẽ nhanh hơn
- Kiểu mã hóa :
    - Fernet : Symmetric
    - JWS : Asymmetric
- Hỗ trợ :
    - Fernet đang là dạng token mặc định của Keystone
    - JWS mới được đưa vào từ bản Stein, chưa hỗ trợ nhiều, cấu hình khó hơn
## **4) Cấu hình JWS**
- **B1 :** Cài đặt **Keystone** (bỏ qua các cấu hình liên quan đến fernet token)
- **B2 :** Chỉnh sửa file cấu hình `keystone.conf`
    ```
    # crudini --set /etc/keystone/keystone.conf token provider jws
    # crudini --set /etc/keystone/keystone.conf jwt_tokens jws_public_key_repository /etc/keystone/jws-keys/public
    # crudini --set /etc/keystone/keystone.conf jwt_tokens jws_private_key_repository /etc/keystone/jws-keys/private
    ```
- **B3 :** Khởi tạo key pair :
    ```
    # keystone-manage create_jws_keypair --keystone-user keystone --keystone-group keystone
    ```
    > Lệnh chạy sẽ sinh ra 2 file `public.pem` và `private.pem`
- **B4 :** Sử dụng lệnh sinh token để kiểm tra :
    ```
    # openstack token issue
    ```
    <img src=https://i.imgur.com/xDvWy2q.png>
    
## **5) Quá trình rotation và distribution**
- Mỗi API server cần một cặp `public-private` key pair riêng
- `keystone-manage` sẽ không xử lý phần rotation
- Có 3 phase trong 1 quá trình rotation và distribution :
    - `create key pair`
    - `distribute public key`
    - `configure private key`
- **VD :** Quá trình rotation giữa 3 node Keystone :
    - **B1 :** Có 3 node **node1**, **node2**, **node3** :
        ```
        node1               node2               node3
        ```
    - **B2 :** **node1** `create key pair` :
        ```
        node1               node2               node3

        pub1.pem
        pri1.pem
        ```
    - **B3 :** **node1** `distribute public key` :
        ```
        node1               node2               node3

        pub1.pem            pub1.pem            pub1.pem
        pri1.pem
        ```
    - Lặp lại tương tự các bước với các **node2**, **node3** . Sau cùng sẽ là bước `configure private key` :
        ```
        node1               node2               node3

        pub1.pem            pub1.pem            pub1.pem
        pub2.pem            pub2.pem            pub2.pem
        pub3.pem            pub3.pem            pub3.pem
        pri1.pem            pri2.pem            pri3.pem
        ```
## **6) So sánh các loại token**

|  | UUID | PKI | PKIz | Fernet | JWS |
|------------|------|-----|------|--------|-----|
| Kích cỡ | 32 Byte | cỡ KBs | cỡ KBs | Khoảng 255 Byte | |
| Lưu trong database | có | có | có | không | không |
| Thông tin mang theo | không có | user, catalog,.. | user, catalog,... | user,... | user,... |
| Kiểu mã hóa | không | asymmetric | asymmetric | symmetric | asymmetric |
| Nén | Không | Không | Có | Không | Không |
