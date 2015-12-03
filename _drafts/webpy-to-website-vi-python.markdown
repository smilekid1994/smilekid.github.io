---
layout: post
title: "Webpy tạo web api Với Python"
---

Sau khi tìm loanh quanh để viết một web service đơn giản với python thì
thấy thằng này webpy này có vẻ ổn. Mình chỉ tìm hiểu để làm một web api nên chỉ quan tâm kết quả trả về, không để ý đến giao diện. Vì thế bài này chắc cũng sẽ không có liên quan tới template, just print :D

Hello World đây:

```python
import web
urls = (
    '/', 'hello'
)

app = web.application(urls, globals())

class hello:
    def GET(self):
        return 'Hello World'

if __name__ == "__main__":
    app.run()
```

Cài đặt

```bash
sudo -H pip install web.py
```

Chạy

```bash
python helloworld.py
curl http:/127.0.0.1:8080
```

Mặc định nó sẽ bind all network và sử dụng port 8080. Khi chạy thật thì nên thay đổi cho hợp lý, tốt nhất là đổi được cái gì thì đổi, không nên để mặc định. Ví dụ chỉ chạy localhost với port 8000

```bash
python helloworld.py 127.0.0.1:8000
```

## urls

```python
urls = (
  '/', 'hello'
)
```

Chỉ định các đường url của web. Như ở Hello World phía trên, thằng này chỉ phục vụ 1 url là `/` (http://127.0.0.1:8080/). Và khi truy cập vào url `/`, class được chỉ định để xử lý và hiển thị kết quả là `hello`. Cứ như thế url và class đi với nhau thành 1 cặp và tất cả những urls khác mẫu được khai báo đều 404. Ví dụ tạo trang about:

```python
urls = (
  '/', 'hello',
  '/about', 'about'
  '/about/me', 'about'
)

class about:
    def GET(self):
        return 'Smile'
```

truy cập vào `about` và `about/me` cho kết quả như nhau.

## GET

urls thì có vẻ dễ rồi, đến phần xử lý và trả về kết quả. Cũng dễ cmn nốt, tạo class và trả về thôi

```python
class about:
    def GET(self):
        return 'Smile'
```

hiển thị `Smile` khi vào các urls được khai báo tương ứng với class `about`

## GET/POST và tham số

Với những static web kiểu blog thì như trên là đủ, nhưng với web api thì còn một vấn đề chúng ta quan tâm là tham số của GET/POST. Dạng thế này `/about?name=smile`. Hoàn toàn có thể dùng cách truyền thống để lấy các tham số này (biến môi trường của web server) nhưng webpy cũng có cách của nó. GET trước cho nó dễ, đầu tiên khai báo ở url:

```python
urls = (
  '/', 'hello',
  '/about/(.*)', 'about',
  '/view/(.*)/(.*)', 'view'
)
```

`(.*)` thể hiện url `/about` có 1 tham số cho GET

`(.*)/(.*)` thể hiện url `/view` sẽ nhận 2 tham số cho GET

Giờ bắt các tham số này trong class để xử lý thôi

```python
class about:
    def GET(self, name):
        return 'It\'s ' + name

class view:
    def GET(self, page, cate):
        return 'It\'s ' + page + ' of ' + cate
```

Test

```bash
curl http://127.0.0.1:8080/about/smile
curl http://127.0.0.1:8080/view/page1/cate1
```

Theo cách truyền thống (dùng luôn cả POST và GET) thì url không được đẹp lắm với GET.

```python
urls = (
  '/', 'hello',
  '/about/(.*)', 'about',
  '/view/(.*)/(.*)', 'view',
  'api', 'api'
)

class api:
    def POST(self):
        try:
            data = web.input().name
        except:
            return "not found"
        return data

    def GET(self):
        try:
            data = web.input().name
        except:
            return "not found"
        return data
```

Hiển thị giá trị của `name` được truyền qua GET/POST.

```bash
curl http://127.0.0.1:8080/api?name=smile
curl http://127.0.0.1:8080/api -d "name=smile"
```

2 cái trên là như nhau chỉ khác phương thức.
