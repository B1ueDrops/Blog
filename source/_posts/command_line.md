---
title: Shell脚本现用现学
categories: 编程语言
---



* 如果从网络上有一个文件夹的URL, 想用终端下载其中的所有内容, 可以使用:

```bash
wget -r -np -R "index.html*" <你的URL>
```

