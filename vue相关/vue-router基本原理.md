# vue-router基本原理

## 前置知识

前端路由主要有两种实现方法：

1. Hash 路由
2. History 路由

### Hash路由

url 的 hash 是以 # 开头，原本是用来作为锚点，从而定位到页面的特定区域。当 hash 改变时，页面不会因此刷新，浏览器也不会向服务器发送请求。

> <http://www.test.com/#/home>

hash 改变时，触发 hashchange 事件。所以，hash 很适合被用来做前端路由。当 hash 路由发生了跳转，便会触发 hashchange 回调，回调里可以实现页面更新的操作，从而达到跳转页面的效果。

````javascript
window.addEventListener('hashchange', function () {
  console.log('render')
})
````

### History路由

HTML5 规范中提供了 [history.pushState](https://developer.mozilla.org/zh-CN/docs/Web/API/History/pushState) 和 [history.replaceState](https://developer.mozilla.org/zh-CN/docs/Web/API/History/replaceState) 来进行路由控制。通过这两个方法，可以改变 url 且不向服务器发送请求。不会像 hash 有一个 #，更加的美观。但是 History 路由需要服务器的支持，并且需将所有的路由重定向到根页面。
History 路由的改变不会去触发某个事件，需要考虑如何触发路由更新后的回调。
有以下两种方式会改变 url：

- 调用 [history.pushState](https://developer.mozilla.org/zh-CN/docs/Web/API/History/pushState) 或 [history.replaceState](https://developer.mozilla.org/zh-CN/docs/Web/API/History/replaceState)；
- 点击浏览器的前进与后退。

第一个方式可以封装一个方法，在调用 pushState（replaceState）后再调用回调。

````js
function push (url) {
  window.history.pushState({}, null, url)
  handleHref()
}

function replace (url) {
  window.history.replaceState({}, null, url)
  handleHref()
}

function handleHref () {
  console.log('render')
}
````

第二个方式，浏览器的前进与后退会触发 popstate 事件。

````js
window.addEventListener('popstate', handleHref);
````

我们来看看源码是如何使用这两个api的

````js
const eventType = supportsPushState ? 'popstate' : 'hashchange'
    window.addEventListener(
      eventType,
      handleRoutingEvent
    )
    this.listeners.push(() => {
      window.removeEventListener(eventType, handleRoutingEvent)
    })
  }
````

````js
export function pushState (url?: string, replace?: boolean) {
  saveScrollPosition()
  // try...catch the pushState call to get around Safari
  // DOM Exception 18 where it limits to 100 pushState calls
  const history = window.history
  try {
    if (replace) {
      // preserve existing history state as it could be overriden by the user
      const stateCopy = extend({}, history.state)
      stateCopy.key = getStateKey()
      history.replaceState(stateCopy, '', url)
    } else {
      history.pushState({ key: setStateKey(genStateKey()) }, '', url)
    }
  } catch (e) {
    window.location[replace ? 'replace' : 'assign'](url)
  }
}
````

## VueRouter本质

vue是一个渐进式的框架，并不自带路由管理功能，所以当我们当需要路由管理时候：

1. 安装VueRouter，再通过import VueRouter from 'vue-router'引入;
2. 创建一个router实例 const router = new VueRouter({...}),再把router作为参数的一个属性值挂载到根组件上`new Vue({router})`;
3. 通过Vue.use(VueRouter)注册插件

那么由以上几点可以得知。VueRouter本质上是一个插件，通过[插件机制](https://cn.vuejs.org/v2/guide/plugins.html)引入的

### $rouetr 和 $route

$router是路由插件的实例，包含了所有的路径对应相应组件的信息，即

````js
const home = () => import('@/views/home')

new Router({
  routes: [
    {
      path: '/',
      name: 'home',
      component: home
    }
    ....
  ]
})
````

也包含了push和replace等方法，例如`$router.push` `$router.replace`

$route是包含当前具体路由的信息

先来介绍下在vue组件中$router和$route的区别，方便我们之后理解代码

这里贴上vue-router的install方法

````js
import View from './components/view'
import Link from './components/link'

export let _Vue

export function install (Vue) {
  if (install.installed && _Vue === Vue) return

  install.installed = true
  _Vue = Vue
  const isDef = v => v !== undefined

  const registerInstance = (vm, callVal) => {
    let i = vm.$options._parentVnode
    if (isDef(i) && isDef(i = i.data) && isDef(i = i.registerRouteInstance)) {
      i(vm, callVal)
    }
  }

  Vue.mixin({
    beforeCreate () {
      // 如果是根组件
      if (isDef(this.$options.router)) {
        this._routerRoot = this
        this._router = this.$options.router
        this._router.init(this)
        // history.current保存的是当前路由信息
        Vue.util.defineReactive(this, '_route', this._router.history.current)
      } else {
        // 如果是子组件，从父组件去取
        this._routerRoot = (this.$parent && this.$parent._routerRoot) || this
      }
      registerInstance(this, this)
    },
    destroyed () {
      registerInstance(this)
    }
  })

  // 将`$router` `$route`挂载到组件实例上
  Object.defineProperty(Vue.prototype, '$router', {
    get () { return this._routerRoot._router }
  })

  Object.defineProperty(Vue.prototype, '$route', {
    get () { return this._routerRoot._route }
  })
  
  // 注册rouetr-view和rouetr-link全局组件
  Vue.component('RouterView', View)
  Vue.component('RouterLink', Link)

  const strats = Vue.config.optionMergeStrategies
  // use the same hook merging strategy for route hooks
  strats.beforeRouteEnter = strats.beforeRouteLeave = strats.beforeRouteUpdate = strats.created
}

````

