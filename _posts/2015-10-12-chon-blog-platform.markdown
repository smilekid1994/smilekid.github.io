---
layout: post
comments: true
tag: [personal, log]
title: "Chọn Blog Platform"
date: 2015-10-12T15:13:12+07:00
---

Định kiếm một cái blog mà ngó qua thấy cả mớ. Dạo này đang thích
markdown nên đành tìm một cái theo tiêu chí:

* Đơn giản
* Có thể viết (quản lý thì tốt) từ terminal
* Chạy được trên một số hosting miễn phí
* Hỗ trợ markdown

Tìm qua thì thấy có vài cái:

* octopress
* logdown
* calepin
* wordpress (thằng này thì cái éo gì cũng hỗ trợ :|)
* blogger

Lúc đầu bỏ qua octopress vì thấy nó chỉ là platform (ko có host), bỏ qua
luôn wordpress, blogger vì thấy nó phức tạp vl, chưa kể là thỉnh thoảng
còn bị chặn..

## [calepin](http://calepin.co/)

Khá hứng thú với thằng này vì lấy nội dung từ markdown files, chưa kể
còn hỗ trợ lấy từ dropbox.
Ngắm nghía, tạo tài khoản một hồi, sau vào twitter:
https://twitter.com/calepinapp thì thấy chẳng active gì mấy. Chán lại đi
tìm tiếp, chưa cả kịp test ^^.

## [logdown](http://logdown.com)

Trang chủ chuyên nghiệp, tính năng ngon. Back-end ngon, hỗ trợ markdown,
cho download backup về. Mỗi tội thấy chữ price hơi nản.
Tuy nhiên vẫn đăng ký 1 tài khoàn free. Tài khoản này chỉ cho phép 1
static post (chắc là 1 post ở homepage), không cho custom domain. Tìm
hiểu thêm thì thấy bạn này lại là hàng Tàu. Bản thân anh không cầu kì gì
nhưng không thích gò bó, tạm bỏ qua đã.

## [octopress](https://github.com/octopress/octopress) & [jekyll](https://github.com/jekyll/jekyll)

Quay lại đọc kĩ bạn octopress này thì lại thấy phù hợp với các yêu cầu của mình.
Hóa ra nó là framework nhưng là viết cho jekyll, chính là cái mình đang
tìm và nó được dùng cho các trang của github.io.

* Đơn giản, không có database
* Nội dung được viết bằng markup language (hỗ trợ markdown)
* Lưu trữ trên github (mình thích cái này), s3, rsync (chưa test chắc
đẩy lên web server bình thường)
* Hỗ trợ các tính năng của blog: categories, pages, comment, theme các kiểu

Kiểu hoạt động của nó là thế này: Thằng jekyll là thằng base, thằng
octopress là thằng làm cho jekyll dễ sử dụng hơn. Nội dung của blog sẽ
được viết bằng markdown, textile hoặc html, sau đó jekyll sẽ build toàn
bộ thành html để tạo thành website.

Kiểu build thành html này load trang thì nhanh vđ.

Trang web sau khi build sẽ được push lên master branch của github.io,
còn nội dung chưa build sẽ được push lên source branch.

OK. Giờ tất cả những gì chúng ta cần là:

* vim
* git client
* github account
* octopress and jekyll gems
* thêm cái browser để ngắm nữa

### Triển thôi

Cài đặt

``` bash
sudo -H gem install octopress
octopress new /path/to/blog_folder
cd blog_folder
```

Tạo mới 1 post

``` bash
octopress new post "title" --slug "day-la-slug"
```

Lệnh trên sẽ tạo 1 file `_posts/yyyy-mm-dd-day-la-slug.markdown`. Bây
giờ chỉ việc mở file này và viết 1 post bằng markdown.
Quá đơn giản.

Preview blog in localhost

``` bash
jekyll build
jekyll server
```

Một web server đơn giản sẽ được tạo ra tại <http://127.0.0.1:4000> để view blog. Nếu không muốn
publish thì có thể dừng ở đây. Làm một cái local diary :D

**publish**

Việc đầu tiên là tạo một chỗ để chứa nó trên internet cái đã. Như đã nói
ở trên thì thằng này hỗ trợ github, s3, rsync (vcđ). Chọn github cho nó
đơn giản.

Tạo một project với tên là: username.github.io. Sẽ được một cái git repo
kiểu: <https://github.com/smilekid1994/smilekid1994.github.io>. Repo này
sẽ chứa cả source (markdown post) và website luôn (sau khi build bằng
jekyll)

Giờ tới việc link local blog với github repo đã tạo ở trên.

``` bash
cd blog-folder
git init
octopress deploy init git https://github.com/smilekid1994/smilekid1994.github.io
git remote add origin https://github.com/smilekid1994/smilekid1994.github.io
```

file `_deploy.yml` sẽ được tạo ra, giữ luôn giá trị mặc định cho nó đẹp
:D. Add file này vào `.gitignore` cho nó lành.

``` bash
echo "_deploy.yml" >> .gitignore
```

Thử deploy local blog lên github

``` bash
# build lại phát cho chắc ăn
jekyll build
git add .
# check lại các thư mục vì lệnh trên sẽ đưa tất cả vào repo
git status
git commit -m "init blog"

# deploy
octopress deploy

# nếu chưa có branch source thì sẽ được cảnh báo và tạo manual
git checkout -b source
git push origin source

# thử deploy lại
octopress deploy
```

Nếu mọi thứ bình thường thì sẽ view được qua url
<http://username.github.io>

Giờ ngồi chỉnh lại mấy cái `about.md`, `_config.yml` sửa mấy cái mặc định
là thành một cái blog rồi.

Một việc cần lưu ý là khi chạy `deploy` qua bằng octopress, sẽ tạo ra
một thư mục deploy riêng (mặc định là .deploy) và khởi tạo một git
repository ở đó. Vì vậy nếu muốn thao tác hoặc chỉnh sửa git config thì
nên lưu ý cả 2 thư mục là `.git` và `.deploy/.git`.

## Kết

Cái này dùng liquid template nên có thể tùy chỉnh rất nhiều thứ. Chắc là
sẽ có một bài sử dụng chi tiết đi sâu vào việc viết blog và template,
cái này chỉ là cài đặt. Mà đm, nói wordpress phức tạp chắc nhiều người
thấy cái này cũng phức tạp vl =)).

Cơ mà mình thấy rất đơn giản và happy ^^
