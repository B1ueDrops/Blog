---
title: 前端语言使用日记
categories: 编程语言
---



## `HTML`

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

  





## `Javascript`



### `ES6`语法糖

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
    function (a, b) {x
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



## `Vue3`



### 创建项目

* 安装Vue命令行客户端:

  ```bash
  npm i -g @vue/cli
  ```


* 要创建Vue项目: 

  ```bash
  vue create <项目名字>
  ```


  * 插件: 

    * `vue-router`
    * `vuex`
    * `vuetify`
    * `electron-builder`: 如果不需要GUI的话不需要选.

    ```bash
    vue add router
    vue add vuex
    vue add vuetify
    vue add electron-builder
    ```
    
* 启动项目:

  ```bash
  # 调试环境
  vue serve
  
  # 生产环境
  vue build
  
  # electron gui
  npm run electron:serve
  ```

* 路由文件: `src/router/index.js`


  * 将`createWebHashHistory`改成`createWebHistory`, 浏览器地址栏就不会出现`#`.

* 主页面: `src/public/index.html`

### 组件定义

* 需要这样写

* ```js
  export default {
    name: '你当前组件的名字',
    // 你当前组件中用到的所有子组件
    components: {
      NavBar,
    }
  }
  ```

### 父组件调用子组件

* `slot`插槽相当于子组件的参数

* 父组件中调用子组件时:

  * ```vue
    <!--父组件-->
    <template>
    	<Child>
      	hahaha
      </Child>
    </template>
    ```
  
  * 子组件会把`hahaha`渲染到`slot`中:

    ```vue
    <template>
    	<!-hahaha会被填充到这里-->
    	<slot></slot>
    </template>
    ```
  
  
  
  > 多个参数的情况
  
  * 子组件叫`MainFrame.vue`, 有三个参数
  
    ```vue
    <!-- MainFrame.vue -->
    <template>
    	<slot name="header"></slot>
    	<slot name="main"></slot>
    	<slot name="footer"></slot>
    </template>
    ```
  
  * 如果在其他组件中想要调用这个组件, 并且向`slot`中填充内容:
  
    ```vue
    <MainFrame>
      
    	<template #header>
      	<!-- 这里放header内容 -->
      </template>
      
      <template #main>
      	<!-- 这里放main内容 -->
      </template>
      
      <template #footer>
      	<!-- 这里放footer内容 -->
      </template>
      
    </MainFrame>
    ```
  



### 父组件给子组件传递数据

* 首先在父组件中定义数据:

  ```js
  import { ref, reactive } from 'vue'
  
  export default {
  	setup() {
      // 定义的数据
      // reactive只可以定义Object
    	const user = reactive({
        name: 'haha',
        age: 18
      })
      // ref可以定义其他类型变量
      const myname = ref('haha')
      /* ref定义的值需要用myname.value进行访问/修改 */
      // 最后return出去
      return {
        user, myname
      }
  	} 
  }
  ```

* 给子组件传递数据:

  ```html
  <Child :user=”user“>
  ```

  * 这个`:`是`v-bind`的缩写, 当使用`:`的时候, 后面的`""`里面就不是字符串, 而是一个表达式, 其他标签也可以使用:

    ```html
    <img :src="photo.address">
    ```

* 子组件接收数据:

  ```js
  export default {
    
    props: {
      // 接收的参数
      user: {
        type: Object,
        required: true
      }
    }
  }
  ```

  * 然后, 在`template`中, 直接用`{{ user.username }}`展示即可.

* 如果子组件需要一些通过基本数据组合而成的数据:

  ```js
  import { computed } from 'vue'
  
  export default {
    
    setup(props) {
      let fullName = computed(() => {
        props.user.userName + props.user.lastName
      })
      
      return {
        fullName,
      }
    }
  }
  ```

  * 也就是说`setup`中既可以定义数据, 也可以用父组件传的参数.



### 控制标签显示

* 如果要用一个变量控制标签是否显示, 可以用`v-if`

  ```html
  <button v-if="!user.is_followed">
    +关注
  </button>
  ```

* `v-if`对应`v-else`, 表示`v-if`条件不满足就显示`v-else`的东西.



### 绑定事件

#### 组件内部事件

* 在组件的`setup`中定义所有用到的函数:

  ```js
  export default {
    
    setup(props) {
      
      let handler = (a) => {
        //...
      }
      
      // 别忘记return出去
      return {
        handler
      }
    }
  }
  ```

* 然后用`@` 绑定:

  ```html
  <!--click是点击事件-->
  <button @click="handler(a)">
    
  </button>
  ```

  



#### 子组件更新父组件信息

* 首先, 子组件中事件的处理函数, 使用`context.emit`, 调用父组件中的处理函数:

  ```js
  export default {
    
    setup(props, context) {
      // follow是父组件的事件
      context.emit('follow', 参数1, 参数2)
    }
  }
  ```

* 然后在父组件调用子组件时, 绑定一个`follow`事件:

  ```html
  <!--第一个follow是事件, 第二个是函数-->
  <Child @follow="follow"></Child>
  ```

* 然后在父组件的`setup`中更新即可.



### 循环标签

* 用`v-for`, 但是需要保证`key`是唯一的(不推荐使用下标)

  ```html
  <div v-for="item in array" :key="item.id">
    <!--会把里面的东西循环-->
  </div>
  ```



### 组件内容绑定到变量

* 首先需要在`setup`中定义一个变量:

  ```js
  export default {
    
    setup(props, context) {
      
      let myname = ref('haha')
      
      return {
        myname
      }
    }
  }
  ```

* 然后, 在标签中, 用`v-model`进行绑定:

  ```html
  <textarea v-model='content'>
  </textarea>
  ```

* 然后, 在输入框内输入什么, `content`就会显示什么, 在`template`中也可以用`{{ content }}`实时渲染.



### 配置路由

* 在`src/router/index.js`中, 导入所有组件配置.
  * 这样可以通过地址栏切换页面.

* 每一项中:

  ```js
  const routes = [
    {
      // 这里, 最好最后面一个字符是/
      path: '/userprofile/',
      name: 'userprofile',
      component: UserProfileView
    }
  ]
  ```

  

#### 重定向

  如果地址栏输入不存在的路由, 需要重定向到一个404页面, 可以在`routes`上加上:

  ```javascript
  const routes = [
    {
      path: '/:catchAll(.*)',
      redirect: "/404/"
    }
  ]
  ```



#### 路由参数给Vue组件

* 浏览器路由中传参数给Vue组件:

  * 首先, `routes`里面:

    ```javascript
    const routes = [
      {
        path: '/userprofile/:userId/',
        name: 'userprofile',
        component: UserProfileView
      }
    ]
    ```

  * 然后, 在组件`UserProfileView`中:

    ```javascript
    import { useRoute } from 'vue-router'
    
    export default {
      
      setup() {
        const route = userRoute();
        // 访问路由中的变量
        console.log(route.params.userId);
      }
    }
    ```

    

#### Vue组件点击切换路由

* 首先导入:

```javascript
// @表示src目录
import router from '@/router/index'
```

* 然后, 在绑定的事件函数中:

  ```js
  router.push({
    path: "/userprofile",
  	params: {
      userId: userId,
    }
  })
  ```




### `vuex`维护全局状态

* `vuex`中的所有文件在`store/`中.

* 全局状态可以分模块存储, 每一个模块对应一个`.js`文件, 每一个`js`文件的格式是:

  ```javascript
  const ModuleUser = {
    state: {
      
    },
    getters: {
      
    },
    mutations: {
      update(state, ...) {
       
      }
    },
    actions: {
      
    },
    modules: {
      
    }
  };
  
  export default ModuleUser;
  ```

  * `state`: 定义全局要维护的状态.
  * `getters`: 通过状态变量计算得到的值.
  * `mutations`: 更新状态变量的操作, 不是异步操作.
  * `actions`: 对`state`变量的一些操作, 是异步操作.
  * `modules`: 需要用到的子模块.



#### actions中调用mutations

* 在`actions`中定义的函数需要加上一个参数叫`context`.
  * 然后用`context.commit('函数名字', {'函数参数'})`



#### 组件中使用全局变量

* 在组件中:

  ```html
  <script>
  	
    import { useStore } from 'vuex';
    
    setup() {
      const store = useStore();
      
      // 获取变量
      
      // 使用actions中的函数
      store.dispatch('函数名称', {'函数参数'});
      // 调用mutations中的函数用store.commit();
      // 用法和dispatch一样
    }
  </script>
  ```

