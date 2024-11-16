---
title: 使用Vue快速搭建网站原型
categories: 编程语言
---



## 环境配置

* 安装客户端:

  ```shell
  npm i -g @vue/cli
  ```

* 创建项目, 并且带上插件`vue-router`和`vuex`:

  ```shell
  vue create <project-name>
  vue add router
  vue add vuex
  vue add vuetify
  ```

* 之后启动调试:

  ```shell
  vue serve
  ```

* 之后, 在`src/router/index.js`中, 将所有`createWebHashHistory`替换为`createWebHistory`.
  * 用于去除浏览器地址栏中的`#`



## 组件定义

假设一个组件文件叫`MyComponent.vue`.

* 它的起始文件结构是:

```vue
<template></template>
<style scoped></style>
<script>
// @是src/
import NavBar from '@/components/NavBar.vue'
export default {
  name: 'MyComponent', // 你当前组件的名字
  // 所有用到的子组件
  components: {
    NavBar,
  }
}
</script>
```





### 插槽(slot)

* 组件参数用插槽`slot`定义, 假设组件有三个参数叫做`header, main, footer`, 那么:

  ```vue
  <template>
  <!- 三个参数 ->
  	<slot name="header"></slot>
  	<slot name="main"></slot>
  	<slot name="footer"></slot>
  </template>
  
  <style scoped></style>
  
  <script>
  // @是src/
  import NavBar from '@/components/NavBar.vue'
  export default {
    name: 'MyComponent', // 你当前组件的名字
    // 所有用到的子组件
    components: {
      NavBar,
    }
  }
  </script>
  ```

* 父组件使用子组件时:

  ```vue
  <MyComponent>
      <template #header>
        <!-- 这里放header内容 -->
      </template>
      <template #main>
        <!-- 这里放main内容 -->
      </template>
      <template #footer>
        <!-- 这里放footer内容 -->
      </template>
  </MyComponent>
  ```



### 成员变量

* 在一个组件中定义成员变量的步骤如下:

  * 第一步, 在`<script>`中, 导入:

    ```javascript
    import { ref, reactive } from 'vue'
    ```

  * 第二步, 在`export default`中, 定义`setup()`函数, 定义数据, 并且`return`出去:

    ```javascript
    import { ref, reactive } from 'vue'
    
    export default {
    	setup() {
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




### 成员函数

* 成员函数需要在`setup(props)`中定义, 并且需要使用箭头函数:

  ```javascript
  export default {
    
    setup(props) {
      
      let handler = (param) => {
        //...
      }
      
      // 别忘记return出去
      return {
        handler
      }
    }
  }
  ```

  



### 接口数据

* 组件可以定义一些数据, 这些数据让父组件传进来:

  * 需要在`export default`中定义一个对象`props`.

  ```javascript
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

* 如果组件内部需要使用这些数据, 只要在`setup`函数中加上`props`参数, 然后用`props.user.username`访问即可.

* 如果父组件例化子组件, 那么只需要用`:`后面加上子组件`props`中定义的数据即可.

  ```vue
  <Child :user="..."></Child>
  ```




### 路由数据

* 当前组件中的数据有可能来自于URL中的参数.

* 如果当前组件需要定义路由数据, 首先, 需要在`src/router.js`中定义好接口:

  ```javascript
  const routes = [
    {
      path: '/userprofile/:userId/',
      name: 'userprofile',
      component: UserProfileView
    }
  ]
  ```

* 之后, 可以用`vue-router`在组件内部访问:

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



### 全局数据

* 如果要在一个组件中访问全局数据, 首先要导入`vuex`

  ```javascript
  import { useStore } from 'vuex'
  ```

* 然后在`setup`函数中, 用`store.state.user`进行访问.

  ```javascript
  export default {
    setup() {
      const store = useStore();
      console.log(store.state.user);
    }
  }
  ```

  



## 数据绑定



### 动态计算绑定

* 组件在`setup(props)`中定义了数据, 如果要由这些数据通过计算生成新的数据, 并且数据改变后, 计算结果也要动态改变, 就需要用到`compute`:

  ```javascript
  import { compute } from 'vue'
  
  export default {
    
    setup(props) {
      let fullName = computed(() => {
        props.user.userName + props.user.lastName
      })
      
      return {fullName}
    }
  }
  ```



### 单向绑定

* 我需要将定义在组件中的数据, 渲染到标签中, 并且数据一变, 标签中渲染的内容也变.

* 这时候需要用到`v-bind`:

  ```vue
  <Component :prop1="user">
  </Component>
  ```

  * 这个时候, 组件`Component`中的参数`prop1`会绑定到当前组件数据`user`.



### 双向绑定

* 双向绑定不仅需要将组件中的数据绑定到标签, 并且用户修改数据后, 要同步到组件数据.

* 这时候需要用到`v-model`属性, 但是不是所有组件都支持`v-model`, 一般是表单等组件支持.

  ```vue
  <Form :items="showed_items" v-model="selected_item"></Form>
  ```



## 事件模型



### 内部事件

* 用`@` 在组件内部绑定事件:

  ```html
  <!--click是点击事件, handler是组件的成员函数-->
  <button @click="handler(a)">
    
  </button>
  ```



### 接口事件

* 当前组件有可能需要处理子组件传递上来的事件.

* 首先, 在当前组件中, 例化子组件时, 需要让子组件知道自己有哪些处理函数:

  ```vue
  <!--func1是告诉子组件, 我当前组件有一个函数叫func1, 但是当前组件内部的处理函数叫follow-->
  <Child @func1="follow"></Child>
  ```

* 在子组件中, 使用`context.emit`调用父组件的预留的处理函数:

  ```javascript
  export default {
    setup(props, context) {
      // follow是父组件的事件
      context.emit('func1', 参数1, 参数2)
    }
  }
  ```

  



## 路由模型

路由的配置文件在`src/router/index.js`中.

如果某个页面需要根据不同路由, 显示不同的组件, 那么只需要用`<router-view />`标签即可.



### 路由绑定组件

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



### 重定向

  如果地址栏输入不存在的路由, 需要重定向到一个404页面, 可以在`routes`上加上:

  ```javascript
  const routes = [
    {
      path: '/:catchAll(.*)',
      redirect: "/404/"
    }
  ]
  ```



### 切换路由

* 首先在当前组件中导入:

  ```javascript
  // @表示src目录
  import router from '@/router/index'
  ```

* 然后, 在绑定的事件函数进行路由切换:

  ```javascript
  router.push({
    path: "/userprofile",
  	params: {
      userId: userId,
    }
  })
  ```





## 全局数据模型

全局数据在`src/store/index.js`中, 在这个文件中, 需要定义全局数据, 以及操作全局数据的接口.

这个文件的大概长这个样子:

```javascript
import { createStore } from 'vuex'

export default createStore({
  state: {
    use_dataset: false
  },
  getters: {
  },
  mutations: {
    set_use_dataset(state, set_value) {
      state.use_dataset = set_value
    }
  },
  actions: {
  },
  modules: {
  }
})

```



### 定义全局数据

在`state`中可以定义全局数据:

```javascript
import { createStore } from 'vuex'

export default createStore({
  state: {
    use_dataset: false,
    user: {
      username: 'hello',
      password: '123'
    }
  },
})
```



### 定义全局数据更新方法

* 第一步, 在`mutations`中定义同步更新的操作:

  * 这里, 同步更新的操作可以理解为`setter`方法, 里面不能有异步操作 (例如从云端获取数据等).

  ```javascript
  export default createStore({
    mutations: {
      // 第一个参数必须是state
      updateUser(state, user) {
        state.user = user;
      }
    }
  })
  ```

* 第二步, 在`actions`中定义复杂的更新操作, 这一步可以有异步, 比如用`ajax`访问某个链接获取数据等.

  * `actions`中不能直接修改`state`, 需要借助`mutations`完成修改.

  ```javascript
  export default createStore({
    actions: {
      // 第一个参数必须是context
      updateUserAction(context, user) {
        // 调用mutations中的方法
        context.commit("updateUser", {user: user})
      }
    }
  })
  ```

* 第三步, 如果在某个组件中需要更新全局数据:

  * 可以用`store.commit`调用`mutations`中的方法:

    ```javascript
    import { useStore } from 'vuex'
    
    export default {
      setup() {
        const store = useStore();
        // store.dispatch调用
        store.commit("updateUserAction", {
          user: user
        })
      }
    }
    ```
  
  * 可以用`store.dispatch`调用`actions`中的方法:
  
    ```javascript
    import { useStore } from 'vuex'
    
    export default {
      setup() {
        const store = useStore();
        // store.dispatch调用
        store.dispatch("updateUserAction", {
          user: user
        })
      }
    }
    ```
  
  
