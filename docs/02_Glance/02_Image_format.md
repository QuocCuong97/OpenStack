# Các định dạng image được Glance hỗ trợ
## **1) Các định dạng image**

| Định dạng | Mô tả |
|-----------|-------|
| **raw** | định dạng image phi cấu trúc |
| **vhd** | định dạng chung hỗ trợ bởi nhiều công nghệ ảo hóa trong OPS, trừ KVM |
| **vhdx** | là định dạng cải tiến của **vhd** | 
| **vmdk** | định dạng hỗ trợ bởi VMware |
| **vdi** | định dạng hỗ trợ bởi Virtual Box và QEMU |
| **ploop** | định dạng được Virtuozzo hỗ trợ để chạy các OS Container |
| **qcow2** | định dạng đĩa QEMU, định dạng mặc định hỗ trợ bởi KVM và QEMU, hỗ trợ các chức năng nâng cao |
| **iso** | định dạng lưu trữ đĩa CD/DVD |
| **ami, aki, ari** | các định dạng Amazon machine, kernel, ramdisk |

## **2) Các định dạng container**

| Định dạng | Mô tả |
|-----------|-------|
| **bare** | Định dạng xác định không có container hoặc metadata đóng gói cho image |
| **ovf** | Định dạng container OVF |
| **ova** | Xác định lưu trữ trong Glance là file lưu trữ OVA
| **ami, .aki, .ari** | Xác định lưu trữ trong Glance là Amazon machine, kernel, ramdisk image |
| **docker** | Xác định lưu trữ trong Glance và file lưu trữ Docker |