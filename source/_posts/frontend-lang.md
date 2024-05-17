---
title: 前端语言使用日记
categories: 编程语言
---



## HTML

> MDN官方文档: https://developer.mozilla.org/zh-CN/

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
  
  * 如果都没办法播放, 就会显示`无法播放`.


  ```html
  <audio controls>
      <source src="/audios/sound1" type="audio/mpeg"/>
      <source src="/audios/sound2" type="audio/mpeg"/>
    	无法播放
  </audio>
  ```

* 视频标签: `<video>`

  ```html
  <video controls width="300">
  	<source src="..." type="video/mp4"/>
    <source src="..." type="video/mp4"/>
  </video>
  ```

* 超链接标签: `<a>`

  ```html
  <a href="你要跳到的超链接地址">哈哈哈</a>
  ```

  



## Javascript



### ES6语法糖

* `bind()`函数:

  * 在Javascript中, 函数里的`this`指的是执行时的调用者, 而不是定义时所在的对象, 例如:

    ```javascript
    const person = {
      name: "ligaog",
      talk: function() {
        console.log(this);
      }
    };
    
    // 这里的this是person
    person.talk();
    
    const talk = person.talk;
    // 这里的this是window (window托管全局)
    talk();
    ```

  * 但是, 如果不想`this`改变的话, 可以用`bind`函数:

    * `const talk = person.talk.bind(person)`.
    * `bind()`里面传入`this`到底是谁, 就可以用谁去调这个函数.

* 箭头函数:

  * 箭头函数是对普通匿名函数的简化:

    ```javascript
    function (a, b) {
      return a + b;
    }
    
    (a, b) => a + b;
    ```

  * 在箭头函数中使用`this`, this始终指向父级作用域, 不用`this`绑定.

    ```javascript
    const person = {
      talk: function() {
        setTimeout(function() {
          console.log(this);
        }, 1000);
      }
    };
    
    person.talk();  // 输出Window
    
    // 箭头函数
    const person = {
      talk: function() {
        setTimeout(() => {
          console.log(this);
        }, 1000);
      }
    };
    
    person.talk();  // 输出 {talk: f}
    ```

* 对象解构:

  ```javascript
  const person = {
    name: "ligaog",
    age: 18
  };
  
  const name = person.name;
  const age = person.age;
  
  // 语法糖
  const { name: my_name, age: my_age } = person;
  ```

* 数组展开: `...a`表示把`a`中的元素铺开

  ```javascript
  let a = [1, 2, 3];
  let b = [4, 5, 6];
  // [1, 2, 3, 4, 5, 6]
  let c = [...a, ...b];
  ```

* Export Default和Export区别:

  * `export`: 可以`export`多个, `import`时需要加上`{}`, 名称需要匹配 (用别名用`as`).
  * `export default`: 只能`export default`一个, `import`时不用`{}`, 名称不需要匹配.

## React

### 配环境

* 配置环境:

  ```bash
  npm i -g create-react-app
  ```

### 创建项目

* 创建项目:

```bash
create-react-app <项目名字>
```

* 启动项目:

  ```bash
  cd <项目名字>
  npm start
  ```

* 在项目文件中装`bootstrap`:

  ```bash
  npm i bootstrap
  ```

### JSX

* JSX是一种Javascript的扩展, 可以在Javascript中写XML语言.
* JSX会被Babel编译成正常的Javascript.



### Components

