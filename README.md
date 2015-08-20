Install WEBVIRTMGR-KVM simple
======

Mục Lục

[1. Giới thiệu](#gioithieu)

[2. Mô hình cài đặt](#mohinh)
				
[3. Thực hiện Lab](#thuchien)

- [a. WebvirtMgr](#web)
	
- [b. Hypervisor-KVM](#kvm)

[4. Thao tác trên Webvirt](#thaotac)

=====================
<a name="gioithieu"></a>

#### 1. Giới thiệu

WebVirtMgr là một giao diện libvirt dựa trên internet dùng cho việc quản lý các máy ảo. Nó cho phép bạn tạo 
và cấu hình các domains mới, và điều chỉnh phân bổ tài nguyên của một domain . Một viewer VNC 
qua một tunnel SSH đưa ra một giao diện điều khiển đồ họa đầy đủ tính năng đến các guest domain. Hiện Webvirt chỉ hỗ trợ KVM.

<a name="mohinh"></a>

#### 2. Mô hình cài đặt

![img](http://i.imgur.com/7oKIMQe.png "img")

Theo như mô hình này, tôi sẽ dựng 2 node, 1 là WebvirtMgr: node sẽ cài đặt giao diện quản lí web, 2 là Hypervisor-KVM: node này cài KVM, chịu trách nhiệm lưu máy ảo và các image.

<a name="thuchien"></a>

#### 3. Thực hiện Lab

<a name="web"></a>

###### a. WebvirtMgr (Frontend)

```
apt-get update && apt-get upgrade && apt-get dist-upgrade
apt-get install -yq python-pip gcc python-dev git
git clone https://github.com/retspen/webvirtmgr.git
cd webvirtmgr/deploy/fabric
pip install -r fab_requirements.txt
fab -H 127.0.0.1 -u root deploy_webvirt
```

Trong khi cài đặt hệ thống muốn bạn tạo 1 user mà sẽ dùng để đăng nhập trên giao diện Web.

Sau khi cài đặt xong, vào trình duyệt kiểm tra, đăng nhập bằng user bạn tạo ở trên.

![img](http://i.imgur.com/M47Jr9W.png "img")

<a name="kvm"></a>

###### b. Hypervisor-KVM

```
apt-get update && apt-get upgrade && apt-get dist-upgrade
apt-get install -yq qemu-kvm libvirt-bin
```

- Thực hiện cấu hình

/etc/libvirt/libvirtd.conf

```
listen_tls = 0
listen_tcp = 1
listen_addr = "0.0.0.0"
auth_tcp = "none"
```

/etc/default/libvirt-bin

```
libvirtd_opts="-l -d"
```

- Kiểm tra cài đặt

```
# sudo service libvirt-bin restart
# ps ax | grep [l]ibvirtd
9207 ?        Sl     0:01 /usr/sbin/libvirtd -l -d
# sudo netstat -pantu | grep libvirtd
tcp        0      0 0.0.0.0:16509           0.0.0.0:*               LISTEN      9207/libvirtd
# virsh -c qemu+tcp://127.0.0.1/system
Welcome to virsh, the virtualization interactive terminal.

Type:  'help' for help with commands
       'quit' to quit

virsh # exit
#
```

- Cài đặt OpenVswitch để thiết lập thêm chế độ card Bridge trong KVM (mặc định KVM chỉ nó NAT)

```
apt-get install -y openvswitch-switch openvswitch-datapath-dkms
```

- Cấu hình card br-ex

```
auto br0
iface br0 inet static
address 172.16.69.125
netmask 255.255.255.0
gateway 172.16.69.1
dns-nameservers 8.8.8.8
bridge_ports eth0
bridge_fd 9
bridge_hello 2
bridge_maxage 12
bridge_stp off

auto eth0
iface eth0 inet manual
#bridge_ports eth0
up ip link set dev $IFACE up
down ip link set dev $IFACE down
```

- Add card br-ex vào eth0

```
ovs-vsctl add-br br-ex
ovs-vsctl add-port br-ex eth0
```
- Tạo thư mục chứa file ISO

/var/www/webvirtmgr/images

- Đẩy file ISO lên hệ thống (có thể tải về từ internet)

![img](http://i.imgur.com/zPRQTyw.png "img")

- Cách add user vào hệ thống (lưu ý lúc này chuyển sang node cài web để thao tác)
```
cd /var/www/webvirtmgr
./manage.py createsuperuser
```

<a name="thaotac"></a>

*Chú ý: Bạn nên khởi động lại toàn bộ hệ thống trước khi thao tác*
#### 4. Thao tác trên Webvirt

- Login

![img](http://i.imgur.com/a5UxMe0.png "img")

- Add host

![img](http://i.imgur.com/5RJ8Lbw.png "img")

![img](http://i.imgur.com/0KZcTzU.png "img")

![img](http://i.imgur.com/4PDsY0K.png "img")

- Tạo thư mục chứa các file .img

![img](http://i.imgur.com/rADwDOR.png "img")

- Tạo thư mục chứa các file iso

![img](http://i.imgur.com/PFXKs80.png "img")

- Tạo card Bridge

![img](http://i.imgur.com/tdvSMzc.png "img")

- Tạo máy ảo

![img](http://i.imgur.com/Xgw4Iit.png "img")

![img](http://i.imgur.com/LAuz9xi.png "img")

![img](http://i.imgur.com/wWreh2u.png "img")

![img](http://i.imgur.com/VEiaaRO.png "img")

![img](http://i.imgur.com/U0N0T3g.png "img")

![img](http://i.imgur.com/MjFxEFy.png "img")

![img](http://i.imgur.com/g7sthPM.png "img")

![img](http://i.imgur.com/RVBgz3i.png "img")

![img](http://i.imgur.com/AP8oNhu.png "img")

![img](http://i.imgur.com/AP8oNhu.png "img")

- Bắt đầu cài máy ảo

![img](http://i.imgur.com/SAMfg8q.png "img")

======

Tài Liệu Tham Khảo


[Link trang chủ](https://www.webvirtmgr.net/)

[Link 1](http://www.ainoniwa.net/pelican/2014/0520a.html)
