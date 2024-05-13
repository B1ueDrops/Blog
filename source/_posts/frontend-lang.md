---
title: 前端语言使用日记
categories: 编程语言
---



## HTML

* HTML文档是一个树形结构(DOM):

```html
<html>
  
  <head>
  </head>
  
  <body>
  </body>
  
</html>
```

* `<head>`用来存储文档的配置信息.
* `<body>`用来存储文档内容.

* `<head>`中的标签:
  * `<title>`: 改变浏览器tab栏的内容/搜索引擎item的内容.
  * `<meta>`: 用来存储一些元数据, 例如`charset`.
    * `<meta name="description" content="哈哈哈哈">`
    * `<meta name="keywords" content="tag1,tag2">`, 这个也是用在搜索引擎中.
  * 网站Logo: `<link rel="icon" href="路径">`
* 单行/多行注释: `<!-- haha -->`

* 最根本的两个标签:

  * `<span>`: 行内元素, 默认不带回车.
  * `<div>`: 块级元素, 默认带回车.
  * 其他标签都是`<span>/<div>`扩展来的.

* **标题标签:**

  * `<h1> ~ <h6>`

* **段落标签:**

  * `<p>`
  * 本质上是对`<div>`做了一些限制.
  * 会把其中的回车和空格过滤, 如果不需要过滤, 就用`<pre>`

* HTML中的转义字符:

  * `<: &lt;`
  * `>: &gt;`
  * 空格: `&nbsp;`

* **回车标签:** `<br>`

* **样式标签:**

  * 背景黄色: `<mark>`
  * 斜体: `<i>`
  * 加粗: `<b>`

* **图片标签:** `<img>`

  * 行内标签

  ```html
  <img src="地址" alt="图片加载不出来显示什么" width="" height="">
  ```

* **音频标签: **`<audio>`: 

  * `<controls>`表示加上音频控制, 暂停快进等.

  * `<source>`表示多个音频源, 如果第一个加载不出来会尝试加载第二个.


  ```html
  <audio controls>
      <source src="/audios/sound1" type="audio/mpeg"/>
      <source src="/audios/sound2" type="audio/mpeg"/>
  </audio>
  ```

  
