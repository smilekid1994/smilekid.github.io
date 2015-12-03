---
layout: post
title: "Ansible Getting Started"
date: 2015-10-20T10:34:28+07:00
comments: True
tags: [ansible, log]
---

Ở đây là mình dịch lại và comment [Ansible
Document](http://docs.ansible.com/ansible/intro_getting_started.html#id4) và coi như cài cài đặt các kiểu ok rồi (cài qua pip ý mà :D). Bây giờ đi vào sâu hơn và bắt đầu với những câu lệnh. Do tính chất kĩ thuật nên Anh Việt lẫn lộn nha.

## Remote Connection Information

Trước khi bắt đầu, anh em phải hiểu cách mà Ansible giao tiếp với server
qua SSH.

Mặc định, Ansible >=1.3 sẽ sử dụng native OpenSSH để kết nối nếu được.
Nó cho phép ControlPersist, Kerberos, và các tùy chọn trong
`~/.ssh/config` như cấu hình Jump Host. Tuy nhiên, khi sử dụng RHEL 6
hoặc CentOS 6, phiên bản của OpenSSh có thể quá cũ để hỗ trợ
ControlPersist. Trên những hệ điều hành này, Ansible sẽ sử dụng
`paramiko`, một bản OpenSSL viết bằng Python (thấy bảo là chất lượng
cao). Nếu muốn sử dụng các tính năng của native OpenSSH như Kerberized
SSH hay những tùy chọn khác, chắc nên dùng Fedora, OS X hoặc Ubuntu làm
server điều khiển. Cũng có thể cài đặt manual các phiên bản mới của
OpenSSH hoặc dùng `accelerated mode` trong Ansible.

> Đại khái là nếu dùng CentOS 6 với OpenSSH cũ thì speed có vẻ chậm, nên
> dùng mấy cái OS khác. Cơ mà chắc cài OpenSSH từ source là được.

Những phiên bản cũ tới Ansible 1.2, mặc định sử dụng paramiko. Native
SSH phải được chỉ định qua option -c hoặc đặt trong file cấu hình.

Thỉnh thoảng bạn có thể bắt gặp một thiết bị mà không hỗ trợ SFTP.
Trường hợp này khá hiếm nhưng vẫn có thẻ xảy ra, bạn có thể chuyển sang
chế độ SCP trong file cấu hình.

Khi giao tiếp với máy chủ, Ansible mặc định giả sử là bạn đang sử dụng
SSH keys. SSH keys được khuyên dùng nhưng xác thực bằng password cũng có
thể được sử dụng với tùy chọn `--ask-pass`. Nếu sử dụng sudo và cần nhập
password cho sudo, thêm tùy chọn `--ask-sudo-pass`.

Một chia sẻ khá phổ biến và hữu ích: Nên để máy quản trị gần với những
máy chủ mà nó điều khiển (gần về mặt network). Nếu bạn đang chạy Ansible
trên cloud, tốt nhất chạy nó từ một máy ở bên trong cụm cloud đó. Nó sẽ
tốt hơn điều khiển qua Internet.

> Đại khái là nên đặt máy quản lý trong mạng LAN cùng với các máy mà nó
> sẽ triển khai nếu được. Hoặc cùng Datacenter nếu được.

Nâng cao hơn, Ansible không chỉ kết nối, điều khiển qua SSH. Nó có thể
quản lý trực tiếp ở local (chắc kiểu copy script vào rồi chạy), và có
tùy chọn cho việc quản lý local, cũng như quản lý chroot, lxc, và jail
containers. Chế độ `ansible-pull` ngược lại với chế độ mặc định (push
command qua SSH), có thể chạy trực tiếp các script bằng việc sử dụng git
lấy về các lệnh cấu hình.

## Những dòng lệnh đầu tiên

> Kiểu ngày đầu tiên đi học A B C à???

Ok, sau khi cài đặt Ansible, bây giờ chúng ta chơi với vài thứ cơ bản.

Sửa hoặc tạo `/etc/ansible/hosts` để thêm các remote servers. Sau đó
thêm public key lên những server này, thường là thêm vào
`~/.ssh/authorized_keys`. Ví dụ về file hosts:

```
192.168.1.50
aserver.example.org
bserver.example.org
```

Đây được gọi là một inventory file, để hiểu sâu hơn về inventory có thể
đọc thêm tại [Inventory](http://docs.ansible.com/ansible/intro_inventory.html).

> Trước mắt các link reference sẽ dẫn đến link gốc, sẽ cập nhật nếu dịch
> đến, hoặc không bao giờ nếu bận quá :D

Chúng ta sẽ giả sử là đang sử dụng SSH keys để xác thực. Cấu hình SSH
agent để đỡ phải nhập password liên tục bằng cách:

``` bash
$ ssh-agent bash
$ ssh-add ~/.ssh/id_rsa
```

(Phụ thuộc vào cài đặt của bạn, có thể sử dụng tùy chọn `--private-key` để
sử dụng pem file)

Bây giờ thử ping tất cả các nodes:

``` bash
$ ansible all -m ping
```

Ansible sẽ thử kết nối tới các remote servers sử dụng user name hiện
tại, tương tự với SSH. Để kết nối với user name khác, sử dụng tham số
`-u`.

Nếu bạn muốn truy cập chế độ sudo, có vài tùy chọn để thao tác với việc
này:

```bash
# as bruce
$ ansible all -m ping -u bruce
# as bruce sudo to root
$ ansible all -m ping -u bruce --sudo
# as bruce, sudoing to batman
$ ansible all -m ping -u bruce --sudo --sudo-user batman

# Từ phiên bản mới nhất, `sudo` được thay thế bằng `become`
# as bruce, sudoing to root
$ ansible all -m ping -u bruce -b
# as bruce, sudoing to batman
$ ansible all -m ping -u bruce -b --become-user batman
```

(Cách thao tác với sudo có thể thay đổi được trong file cấu hình của
Ansible.  Những tham số đi kèm sudo (ví dụ -H) cũng có thể điều chỉnh.)

Bây giờ thử chạy một lệnh trên tất cả server:

``` bash
$ ansible all -a "/bin/echo hello"
```

Ó sầm. Bạn vừa liên lạc với các nodes bằng Ansible. Trong thời gian ngắn
tới, chúng ta sẽ: đọc về một vài trường hợp thực tế trong [Giới thiệu
Ad-Hoc Commands](http://docs.ansible.com/ansible/intro_adhoc.html), tìm
hiểu những gì có thể làm với những modules khác nhau, học ngôn ngữ
Ansible Playbooks. Ansible không chỉ là chạy những tập lệnh, nó còn là
công cụ quản lý cấu hình và triển khai hệ thống khá mạnh. Còn nhiều thứ
để khám phá, but you already have a fully working infrashtructure!

## Kiểm tra Host Key

Từ bản Ansible 1.2.1, tính năng kiểm tra host key được bật mặc định.

Nếu một host sau khi cài đặt lại có một key khác trong `known_hosts`,
một thông báo lỗi sẽ được trả về đến khi được sửa lại. Nếu một host chưa
được thêm vào `known_hosts`, sẽ có prompt để xác nhận key này. Việc này
yêu cầu người dùng thao tác và trả lời trực tiếp nên không thích hợp với
cron hoặc at.

Nếu bạn đã biết và muốn bỏ qua hành vi này, bạn có thể làm bằng cách
chỉnh sửa trong `/etc/ansible/ansible.cfg` hoặc `~/.ansible.cfg`:

```
[defauts]
host_key_checking = False
```

Hoặc có thể điều chỉnh qua biến môi trường:

```bash
$ export ANSIBLE_HOST_KEY_CHECKING=False
```

Lưu ý rằng kiểm tra host key ở chế độ paramiko khá chậm, vị vậy nếu sử
dụng tính năng này, nên chuyển sang dùng ssh.

Ansible sẽ log vài thông tin về những tham số của module trên máy remote
trong remote syslog, nếu không một task hoặc play sẽ được đánh dấu với
một thuộc tính `no_log: True`. Cái này sẽ được làm rõ sau (vì mình cũng
chưa thử).

> Hết chap đầu, còn đoạn giới thiệu với quảng cáo éo đọc nữa :D. Thực
> hành sẽ được update sau.
