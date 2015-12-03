---
layout: post
title: "Sử Lỗi Uuid Khi Build Trong Mac Port"
date: 2015-12-02T16:10:28+07:00
tag: [personal, log]
comments: true
---

Hôm nay đẹp trời nâng cấp mớ lùng bùng trong mac port nhưng lại gặp lỗi
dở hơi, cũng mất tí thời gian nên note lại ai chẳng may gặp thì fix.

Nâng cấp toàn bộ packages

```
sudo port upgrade outdated
```

Gặp lỗi khi build

```
error: unknown type name 'uuid_string_t'; did you mean 'io_string_t mac
port
```

Tìm loanh quanh thì phát hiện ra là nó đã sử dụng file header sai của
thư viện uuid `/usr/local/include/uuid/uuid.h`. Bình thường mac port
không khuyến khích anh em cài các thư viện hoặc tools vào `/usr/local/`
vì có thể gặp conflict. Cơ mà chẳng may rồi thì tạm thời bỏ đi vậy.

```
sudo mv /usr/local/include/uuid/uuid.h{,.bak}
```

Chờ nâng cấp xong lại move về như cũ hoặc cứ để đấy, chạy cái gì thấy
lỗi thì move sau.

```
sudo mv /usr/local/include/uuid/uuid.h{.bak,}
```

Các trường hợp conflict khác cũng có thể xử lý tương tự, việc đầu tiên
là cần tìm header mà nó sử dụng. Sau đó quét toàn bộ hệ thống xem có
những file nào. Thử đổi tên lần lượt và check lại.

```
find / -name "uuid.h"
```
