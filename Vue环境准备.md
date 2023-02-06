# Vue开发准备工作

## 1.准备Nodejs

- 进入Node官网 https://nodejs.org/zh-cn/ 下载并安装Node并加入到path，npm会随着Node被安装，通过以下命令查看Node和npm的版本。

  ```shell
  node -v
  npm -v
  ```

- 设置npm缓存和全局模块安装路径：在安装目录下创建node_cache和node_global目录，使用以下命令进行配置

  ```shell
  npm config set prefix "xxx/node_global"
  npm config set cache "xxx/node_cache"
  ```

  用户目录下：.npmrc文件记录了以上路径配置。.node_repl_history文件记录node命令交互历史。

## 2.准备VueCli

- 安装VueCli

  ```shell
  npm install -g @vue/cli
  ```

- VueCli安装好之后在Node全局安装模块路径中可以找到启动命令，将该路径加入path

- 创建vue项目

  ```shell
  vue create demo
  ```

## 3.Vue项目开发

### vue.config.js配置

```javascript
module.exports = {
  publicPath: './',
  devServer: {
    port: 8082,
    // 代理配置(表)，在这里可以配置特定的请求代理到对应的API接口
    // 例如将'localhost:8080/api/xxx'代理到'http://xxxxxx.com/api/xxx'，xxxxxx.com为target
    // 在请求接口的时候，vue自动拼接本地ip + axios baseurl + 接口路径
    proxy: {
      '/api': { // 规定请求地址api作为开头，如登录接口localhost/api/login，然后将localhost的地址转换为代理的地址
        target: 'http://127.0.0.1',
        ws: true,
        changeOrigin: true, // 是否跨域
        pathRewrite: {
          '^/api': '' // 正则表达式字符串替换，将开头的api去掉，地址最后变为http://localhost:8080/login
        }
      }
    }
  },
  lintOnSave: false
}
```

### main.js 常用配置

```javascript
import Vue from 'vue'
import App from './App.vue'
import ElementUI from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css'
import router from './router/router'
import axios from 'axios'
import notification from './js/notification.js'
import em from './js/event-manager.js'
import store from './store/store'
Vue.config.productionTip = false
Vue.use(ElementUI);
notification.register(Vue)

// 如果是开发环境，则加上api,触发代理
Vue.prototype.axios = axios.create({
  baseURL: process.env.NODE_ENV !== 'production' ? '/api/' : '',
  crossDomain: true,
});
Vue.prototype.em = em
new Vue({
  store,
  router,
  render: h => h(App),
}).$mount('#app');

```

### 安装其他依赖

- 安装ElementUI

  ```bash
  npm i element-ui --save
  ```

  引入并使用ElementUI

  ```bash
  import ElementUI from 'element-ui';
  import 'element-ui/lib/theme-chalk/index.css';
  
  Vue.use(ElementUI);
  ```

- 安装stylus

  ```bash
  npm install stylus stylus-loader --save-dev
  ```

  使用时只需在 .vue 文件的style标签中设置 lang="stylus" 即可。

### 创建Vue实例（this指向实例）

```javascript
let myVue = new Vue({
	el: '#app', //通过Css选择器或者元素名选择挂载的元素，只能挂载一个元素
	data: {
		return() {
            // 实例属性数据
        }
	},
	methods: {
		// 存放方法
		// 方法没有缓存，每次渲染时方法都会执行，
		// 计算属性有缓存，只有在相关依赖发生变化时才会重新取值，
	},
	filters: {
		// 存放过滤器（其实就是有返回值的方法，管道中调用），2.0版本后取消了内置过滤器
	},
	computed: {
		// 计算属性，与方法类似，只是有缓存功能和返回值
		// 通常是一个有返回值的方法（get方法，方法也是对象），通过插值法直接调用
		// 也可以是一个对象，对象中设置set(与get相反操作)和get方法，以及cache属性
	},
	watch: {
		// 监听属性，存放方法。可以监听data中某属性的值，如果值有变化，方法会执行
        // 方法名需与监听的属性名相同，参数可选oldValue、newValue
	},
	directives:{
		// 自定义指令（局部），不需要前缀v-；可通过Vue.directive(id, function)设置全局指令
    }，
});

```

### Vue生命周期

```javascript
let myVue = new Vue({
    beforeCreate: function(){
        // 在实例初始化时同步调用，数据观测、事件等未初始化
    },
    created: function(){
        // 实例初始化完成后调用，已经完成数据绑定、事件方法。但还未挂载到document中
    },
    beforeMount: function(){
        // 在挂载之前执行
    },
    mouted: function(){
        // DOM编译，挂载完成
    },
    beforeUpdate: function(){
        // 实例挂载之后，再次更新实例（如data中的变量）时会调用，但此时还未渲染DOM
    },
    updated: function(){
        // vue实例数据更新之后，DOM的更新也渲染好了
    },
    beforeDestroy: function(){
        // 实例在被销毁之前调用，此实例还有效
    },
    destroyed: function(){
        // 实例被销毁之后调用。之前所有绑定和实例指令都已经解绑，子实例也被销毁
    },  
})
```

## 4.Vuex（单一状态树）

- 简介

  vuex：解决组件之间传值的问题，创建一个所有组件都能访问到的状态树，可以进行修改。

  vuex有五大组成部分：state、mutations、getters、actions，modules。辅助函数可以提高开发效率。

  store：vuex对象，内部含有state等

  state：单一状态树，存放数据

  mutations：对state修改的唯一方式。（使用commit提交mutatsion。这么做是为了数据修改不那么随意而且方便溯源）。只能使用同步函数，因为异步函数不知道为什么时候才会改变结果，使得我们不太清楚数据当前的状态。

  getters：类似于vue中的计算属性，用于获取state的值

  actions：对state进行修改，参数是context，相当于当前store，所以可以提交mutation，区别是action可以使用异步函数。dispatch()返回的是promise，所以多个action组合使用的时候可以使用async和await进行同步操作。

  modules：为防止项目数据过于复杂臃肿，modules分担store的数据。


- Vuex安装

  ```bash
  npm install --save vuex@3 # 不指定版本使用最新版本，可能出现问题
  ```

- 注册

  在src创建一个store文件夹，下面创建一个index.js里面写上以下代码。同时需要在main.js中引入该文件并使用。

  ````js
  import Vuex from 'vuex'
  import Vue from 'vue'
  Vue.use(Vuex)
  const store = new Vuex.Store({
      state: {num: 2}, // 存放数据
      getters: {}, // 计算属性
      mutations: {}, // 修改state中数据的一些方法
      actions: {}, // 异步方法
      modules: {} // store模块
  })
  // 暴露实例
  export default store 
  ````

  

- 使用实例代码

  ```javascript
  // store.js代码
  import Vue from 'vue'
  import Vuex from 'vuex'
  import localState from '@/store/modules/local-state.js'
  import sessionState from '@/store/modules/session-state.js'
  
  Vue.use(Vuex)
  // 分module的方式
  export default new Vuex.Store({
      modules: {
          localState,
          sessionState
      }
  })
  // 不分module的方式
  export default new Vuex.store({
      // state可以被放在modules下
      state: {
          count： 0
      },
      mutations: {
  		increment(state, payload) {
              state.count += payload.count2
          }    
  	},
      getters: {
  		getCount(state) {
              return state.count
          }    
  	},
      actions: {
  		increment(context) {
              // 这里可以异步提交，比如使用timeout之后再commit，或者需要网络请求
              context.commit('increment')
          }    
  	}
  })
  
  // ----调用----
  // mutations 1 以类型和载荷方式
  store.commit('increment', {
      count: 10
  })
  // mutations 2 以对象方式
  store.commit({
      type: 'increment', 
      count: 10
  })
  // getters  属性，不是方法
  store.getters.getCount
  // actions 也有两种方式，与mutation调用类似
  store.dispatch('increment', {
      count2: 10
  })
  ```

  

## 5.Axios（封装原生ajax）

- 请求示例代码

  ```javascript
  // post请求1
  this.axios({
          url: '127.0.0.1://hi',
          method: 'post',
          data: obj
      }).then(data => {
          // do something
      })
  // post请求2
  this.axios.post('127.0.0.1://hi', obj).then(data => {
          // do something
      }
  // get请求1
  this.axios.get('device/shutDownDevice', {
          params: {
              userId: userId
          }
  	}).then(data => {
          // do something
      })
  // get请求2
  this.axios.get('device/shutDownDevice', {
          params: {
              userId: userId
          }
      }).then(data => {
          // do something
      })
  ```

  ## 5. 使用路由（vue-router）
  
  安装（如果不指定版本，则下载最新版本，可能会有兼容问题）
  
  ```bash
  npm install --legacy-peer-deps vue-router@3.5.2 --save
  ```
  
  定义路由与组件关系
  
  ```javascript
  import Router from 'vue-router'
  import Vue from 'vue'
  import PageOne from '../components/PageOne'
  import PageTwo from '../components/PageTwo'
  Vue.use(Router)
  
  export default new Router({
      routes:[
          {
              path: '/page-one',
              component: PageOne
          },
          {
              path: '/page-two',
              component: PageTwo
          }
      ]
  })
  ```
  
  在Vue对象注册路由
  
  ```javascript
  import Vue from 'vue'
  import App from './App.vue'
  import router from './router/index'
  Vue.config.productionTip = false
  
  new Vue({
    router,
    render: h => h(App),
  }).$mount('#app')
  ```
  
  路由的使用与跳转
  
  ```vue
  <template>
    <div id="app">
      <!-- 显示路由中的页面 -->
      <router-view></router-view>
      <!-- 标签跳转 -->
      <router-link to="/page-one">页面1</router-link>
      <router-link to="/page-two">页面2</router-link>
      <!-- js 调用 router 对象方法跳转 -->
      <button @click="goto">跳转</button>
    </div>
  </template>
  <script>
  
  export default {
    name: 'App',
    methods: {
      goto() {
        this.$router.push('/page-one')
      }
    }
  }
  </script>
  ```
  
  
  
  







 



