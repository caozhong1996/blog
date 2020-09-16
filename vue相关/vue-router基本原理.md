## 前置知识

前端路由主要有两种实现方法：
1. Hash 路由
2. History 路由

### Hash路由

url 的 hash 是以 # 开头，原本是用来作为锚点，从而定位到页面的特定区域。当 hash 改变时，页面不会因此刷新，浏览器也不会向服务器发送请求。

>http://www.test.com/#/home

hash 改变时，触发 hashchange 事件。所以，hash 很适合被用来做前端路由。当 hash 路由发生了跳转，便会触发 hashchange 回调，回调里可以实现页面更新的操作，从而达到跳转页面的效果。

````javascript
window.addEventListener('hashchange', function () {
  console.log('render')
})
````

### History路由

HTML5 规范中提供了 [history.pushState](https://developer.mozilla.org/zh-CN/docs/Web/API/History/pushState) 和 [history.replaceState](https://developer.mozilla.org/zh-CN/docs/Web/API/History/replaceState) 来进行路由控制。通过这两个方法，可以改变 url 且不向服务器发送请求。同时不会像 hash 有一个 #，更加的美观。但是 History 路由需要服务器的支持，并且需将所有的路由重定向到根页面。
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

function handleHref () {
  console.log('render')
}
````

第二个方式，浏览器的前进与后退会触发 popstate 事件。

````js
window.addEventListener('popstate', handleHref);
````

## VueRouter本质

vue是一个渐进式的框架，当需要路由管理时候：

1. 安装VueRouter，再通过import VueRouter from 'vue-router'引入;
2. 创建一个router实例 const router = new VueRouter({...}),再把router作为参数的一个属性值挂载到根组件上`new Vue({router})`;
3. 通过Vue.use(VueRouter)注册插件

那么由以上几点可以推测

然后再通过传进来的Vue创建两个组件router-link和router-view

````js
//myVueRouter.js
let Vue = null;
class VueRouter{

}
VueRouter.install = function (v) {
    Vue = v;
    //新增代码
    Vue.component('router-link',{
        render(h){
            return h('a',{},'首页')
        }
    })
    Vue.component('router-view',{
        render(h){
            return h('h1',{},'首页视图')
        }
    })
};

export default VueRouter
````