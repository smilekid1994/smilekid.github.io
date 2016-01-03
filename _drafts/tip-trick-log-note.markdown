---
layout: post
title: "Tip, Trick, Log, Note,..."
---

Ghi lại mấy cái note ngắn, search google tí thì cũng ra nhưng thích ghi
lại :P. Biết đâu có ai cũng search ra từ trang này.

## export mysql ra csv

Trong mysql

```sql
SELECT * FROM table INTO OUTFILE '/tmp/table.csv' FIELDS TERMINATED BY ',' LINES TERMINATED BY '\n';
```

Nếu muốn có thêm header ở dòng đầu tiên (cho tiện import) thì dùng shell
có vẻ ngắn hơn.

```bash
echo 'SELECT * FROM table' | mysql -u user -pMysqlPassword database > /tmp/table.csv
```
