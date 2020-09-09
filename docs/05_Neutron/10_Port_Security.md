# Port Security
## **1) Giới thiệu**
- Mặc định, **neutron** sẽ kích hoạt **port security** đi kèm mỗi port tạo ra trong **OpenStack** :




By default Neutron enforces the following port security i.e. security on a per-port basis.

Security Groups - All incoming and outgoing traffic is blocked for ports connected to virtual machine instances (unless a ‘Security Group’ has been applied).[1]

Anti-Spoofing - As part of Neutron’s security group implementation, anti-spoofing rules are included, preventing a VM from sending or receiving traffic with a MAC or IP address which does not belong to its Neutron port.[2] However this presents issues for NFV based instances where packets are passed through the VM, meaning the packets are not addressed to or from it.

https://superuser.openstack.org/articles/managing-port-level-security-openstack/

https://www.packetflow.co.uk/openstack-neutron-port-security-explained/#fn1