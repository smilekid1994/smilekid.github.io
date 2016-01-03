---
layout: post
title: "Rtmpdump - Lưu Lại Các Chương Trình Trên Xonefm"
date: 2016-01-03T17:33:11+07:00
comments: true
tag: [personal, log]
---

Đợt rồi có nhu cầu ghi lại các chương trình trên xonefm nên thử xài qua
rtmpdump để download và ffmpeg để convert thấy rất ổn.

## Lấy các thông tin cần thiết

Muốn làm gì thì làm, quan trọng nhất vẫn là cái link rtmp, cơ mà cần
thêm cả cái flashplayer của nó nữa (cũng có thể dùng player riêng, nhưng
lấy của nó luôn thì sẽ tiện và dễ hơn).

Trên xonefm thì mấy thông tin này hiển thị khá rõ ràng ở mã HTML (các
trang khác thì có thể khó hơn nhưng nói chung là đều có thể lấy được)
![xonfm_player_info](https://cdn.rawgit.com/smilekid1994/smilekid1994.github.io/source/images/xonefm_player_info.png "XoneFM player infomation")

Giả sử mình lấy được các thông tin sau:

* Stream url
`rtmp://42.117.5.91:1935/xonefm/_definst_/mp3/234571/bfxone-mon-100815-07-08am-mc-cao-ft-van.mp3`

* Player
`http://www.xonefm.com//swf/audio-player.swf`

Như trong hình trên thì ở phần stream URL có đoạn `mp3:podcast`, mình sẽ
thay `:` bằng `/`. Để nguyên `:` thì éo được nên mình thử thay thành `/`
cho giống URL bình thường thấy OK.

## Ghi lại

```bash
/usr/local/bin/rtmpdump \
-r "rtmp://42.117.5.91:1935/xonefm/_definst_/mp3/234571/bfxone-mon-100815-07-08am-mc-cao-ft-van.mp3" \
-W http://www.xonefm.com//swf/audio-player.swf -o ~/xone.flv -A start_at_seconds -B end_at_seconds
```

Ghi lại thành file flv (mặc định, convert sau nên không cần quan tâm
lắm). `-A` và `-B` để ghi một phần của chương trình. Vì đây là ghi lại
nên thời gian ghi sẽ phụ thuộc vào thời gian của chương trình chứ không
phụ thuộc vào tốc độ của mạng. Chờ thôi ^^, cần thì chạy trên server.

## Convert

FLV nghe vẻ không phổ biến lắm. Chuyển sang mấy định dạng khác cho quen
mắt.

* Convert to mp4
```bash
ffmpeg -i ~/Downloads/bfxone.flv -c copy -copyts bfxone.mp4
```

OK upload lên Facebook.

* Convert to mp3
```bash
ffmpeg -i ~/Downloads/bfxone.flv \
-acodec libmp3lame -ac 2 -ab 128k -vn -y bfxone.mp3
```
Ok upload lên Soundcloud :D

## Tham khảo

* https://rtmpdump.mplayerhq.hu/
* http://ffmpeg.org/
* http://getfirebug.com/
