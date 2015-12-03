---
layout: post
title: "Secure SSH Server"
date: 2015-11-24T23:15:49+07:00
tag: [log, secure, linux]
comments: true
---

Sau khi cài đặt server với một password đủ mạnh thì chúng ta đã khá yên
tâm với sự an toàn của server. Nhưng để chạy trong một thời gian dài thì
nên thay đổi và bổ sung một vài thứ thay vì giữ mọi thứ mặc định.

## Sử dụng phiên bản mới nhất

Nâng cấp phiên bản mới nhất đi theo OS hoặc có thể cài đặt từ mã nguồn.

<http://www.openssh.com/portable.html>

Chú ý backup các file config nếu cần thiết và build với tùy chọn
`--with-pam` có thể sau này sẽ cần dùng.

## Đổi port

```
Port 2222
```

Nhiều người cho rằng việc này không cần thiết nhưng sau một thời gian
dài làm việc với server, mình thấy nên đổi. Có một số trường hợp do bất
cẩn chưa setup firewall, dùng weak password nên đã bị chiếm quyền. Dù
khả năng xảy ra là nhỏ nhưng với một admin thì vẫn nên cẩn thận.

## Đổi host nếu cần thiết

```
ListenAddress 0.0.0.0
```

Mặc định thì sshd sẽ listen trên tất cả các IPs của server (wan, lan các
kiểu). Trong một số trường hợp chúng ta chỉ cho phép ssh qua LAN thì
cũng nên điều chỉnh cho chắc.

```
ListenAddress 192.168.2.22
```

## Vô hiệu hóa tài khoản root

```
PermitRootLogin no
```

Sau khi tạo một tài khoản có quyền sudo chúng ta có thể tắt đăng nhập
qua tài khoản root. Việc này cũng giống như việc đổi port và host, có
thể tránh được một số công cụ quét. Nếu cần sử dụng root có thể su từ
tài khoản thường sang.

## Chỉ định user có thể đăng nhập

Chỉ định các user có thể đăng nhập vào hệ thống. Nếu dùng cách này thì
không cẩn vô hiệu tài khoản root, chỉ cần không đưa root vào danh sách.

```
AllowUsers user1
AllowUsers user2
AllowUsers user3
```

## Sử dụng Public/Private keys để xác thực

Với cách xác thực này thì máy tính nào có private key tương ứng với
public key được lưu trên server thì mới có thể đăng nhập. Cách này rất
an toàn và được khuyến khích sử dụng mặc định. Khi tạo cặp keys thì lưu
ý nên để passphrase tránh trường hợp mất máy hoặc ai đó xin được key của
mình :D.

## Bỏ đăng nhập bằng mật khẩu

Sử dụng keys để đăng nhập thì chúng ta nên bỏ đăng nhập bằng mật khẩu,
chỉ những ai có key mới đăng nhập được.

```
# Disable password authentication forcing use of keys
PasswordAuthentication no
```

## Sử dụng firewall

Để thêm an toàn chúng ta có thể sử dụng iptables để limit ip và limit
rate.

```
iptables -A INPUT -p tcp --dport 2222 -m state --state NEW -m recent --set --name ssh --rsource
iptables -A INPUT -p tcp -m tcp --dport 2222 -m state --state NEW -m recent --update --seconds 60 --hitcount 6 --rttl --name SSH --rsource -j REJECT --reject-with icmp-port-unreachable
iptables -A INPUT -p tcp -s 192.168.2.2 --dport 2222 -j ACCEPT
iptables -A INPUT -j DROP
```

Từ chối nếu có quá 6 kết nối trong 1 phút và chỉ chấp nhận kết nối từ
192.168.2.2.

Nếu hệ thống cần mở cho nhiều user từ nhiều nơi truy cập thì nên chặn
một số IPs đen. Danh sách thì có nhiều nguồn, một trong số đó là
<http://www.openbl.org>. Theo mình thì cứ Tàu và Nga ngố là bem hết ^^

## Bật đăng nhập 2 lần

Sư dụng PAM để bật đăng nhập 2 lần với Google Authenticator. Sử dụng
phần này thì cần build bản openssh với `--with-pam` và dùng phiên bản
openssl từ 6.4 (hình như thế, cứ xài bản mới nhất cho chắc ăn).

Cài luôn từ repo của OS gói `google-authenticator` (tên gói có thể khác
ở các OSs khác nhau).
Hoặc build từ source

```
https://github.com/google/google-authenticator/tree/master/libpam
```

Thư viện sẽ được cài đặt tại `/lib64/security/pam_google_authenticator.so`

Cấu hình `sshd_config`

```
ChallengeResponseAuthentication yes
UsePAM yes
AuthenticationMethods publickey,keyboard-interactive
PasswordAuthentication no
```

Cấu hình PAM `/etc/pam.d/sshd`

```
auth required pam_google_authenticator.so
##
auth       required pam_sepermit.so
#auth       include      password-auth
```

Chú ý comment password-auth để bỏ qua yêu cầu nhập password.

Để lấy verification code chạy lệnh: `google-authenticator` và làm theo
hướng dẫn. Lưu lại mấy cái code backup để phòng khi mưa gió.

Kick hết các users đang đăng nhập vào hệ thống để test thử. Sau khi nhập
passphrase là yêu cầu nhập verification code.

```
sudo /etc/init.d/sshd restart
sudo skill -KILL -u user
```

## Check log thường xuyên

Thường xuyên theo dõi log để phát hiện sớm những trường hợp khả nghi.
Phát hiện sớm là bộ môn rất quan trọng trong việc quản trị hệ thống.

## Lưu ý

* Sau khi sửa cấu hình thì reload lại sshd
* Luôn giữ lại 1 shell backup, tránh tự chặt tay
* Cẩn thận khi thao tác với iptables, nên đặt lệnh clear nếu không chắc
chắn

## Tham khảo

1. <https://wiki.centos.org/HowTos/Network/SecuringSSH>
2. <http://www.openssh.com/>
3. <https://github.com/google/google-authenticator/wiki>
