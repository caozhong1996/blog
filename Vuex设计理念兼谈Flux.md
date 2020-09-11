# Vuex设计理念兼谈Flux

## 前文

当你刚开始使用Vuex的时候，相信你都感到过一些别扭，或者有类似的疑惑：

- 组件间状态共享为什么不推荐使用eventBus？
- 为什么修改Vuex的store需要使用`$store.commit('updateXXX', value)`，而不是直接`$store.XXX = value`？

这些疑问甚至会让你怀疑Vuex存在的意义，在了解Vuex的理念之前，这些确实是灵魂之问，让我们带着疑问来看看为什么吧。

## 本文将讨论以下信息

1. Vuex的使用背景
2. Flux架构的介绍
3. Vuex如何践行Flux理念

本文假设你已经了解Vuex的基本使用，也不会具体介绍Vuex实现方式。

## Vuex是什么

> Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式。它采用`集中式`存储管理应用的所有组件的状态，并以相应的规则保证状态以一种`可预测`的方式发生变化。

注意这两个关键词：`集中式`，`可预测`

### 背景问题

>当我们的vue应用遇到多个组件共享状态时，单向数据流的简洁性很容易被破坏：
>
>- 多个视图依赖于同一状态。
>- 来自不同视图的行为需要变更同一状态。
>
>对于问题一，传参的方法对于多层嵌套的组件将会非常繁琐，并且对于兄弟组件间的状态传递无能为力。对于问题二，我们经常会采用父子组件直接引用或者通过事件来变更和同步状态的多份拷贝。以上的这些模式非常脆弱，通常会导致无法维护的代码。

为了减少干扰，我们暂时简单的用MVC来代替vue的MVVM，，上面描述的数据的流动问题可能如图所示：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3dcc368021204e5f8509aaa6bdb38073~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e7880c3839840a7b13ca9d106a6319d~tplv-k3u1fbpfcp-zoom-1.image)

甚至夸张一点

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a81aebdaddc44615a53c926eb396e0f6~tplv-k3u1fbpfcp-zoom-1.image)

很容易理解，复杂的数据流动必定会带来维护上的困难。

那么现在我们就可以回答第一个问题了-- 组件间传值为什么不使用eventBus？

答：eventBus虽然是事件总线，但是却缺少了`数据中心仓库`，没有把组件的共享状态抽取出来，是`去中心化`的，分散地传递数据，需要手动维护数据。

以下可以看做是eventBus的数据流动

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75e5bdce65154688b6eb31c4b310e0e2~tplv-k3u1fbpfcp-zoom-1.image)

举个例子。有一个display组件，他的作用是展现App根组件上increment的当前值。你新招了两个新员工，给他们分配了两个任务。

1. 你让小A开发一个新的计数器在一个新的组件(childDisplay)，实现点击一次增加increment一次，并在组件内展现increment当前的数字。这个计数器订阅了increment，点击计数器成功让display和childDisplay显示increment增加1后的数字。小A完成这个任务便push代码了
2. 让小B开发按钮组件，这个按钮组件向App实例emit了一个reset事件，将重置App上increment为0，小B也完成这个任务便push代码了
3. 这个时候你会发现当你点击A组件时，在display和childDiplay上都成功显示了increment增加1以后的数字，但点击B组件触发reset事件时，根组件上的increment重置为了0，但由于小B并不知道A组件也订阅了increment这个数据，导致A组件状态没有更新。

即缺少`集中式`状态中心，需要手动维护数据导致的。

### 问题结论

>因此，我们为什么不把组件的共享状态抽取出来，以一个全局单例模式管理呢？在这种模式下，我们的组件树构成了一个巨大的“视图”，不管在树的哪个位置，任何组件都能获取状态或者触发行为！--`集中式`
>
>通过定义和隔离状态管理中的各种概念并通过强制规则维持视图和状态间的独立性，我们的代码将会变得更结构化且易维护。--`可预测`
>
>这就是 Vuex 背后的基本思想，借鉴了 Flux、Redux 和 The Elm Architecture。与其他模式不同的是，Vuex 是专门为 Vue.js 设计的状态管理库，以利用 Vue.js 的细粒度数据响应机制来进行高效的状态更新。

注意，此时提到了Vuex与Flux关系，现在我们来讲讲Flux架构的基本理念

## Flux架构

在前文我们提到了因为多向数据流带来状态管理上的问题，Flux 试图通过强制`单向数据流`来解决这个复杂度。

在这种架构当中，Views 查询 Stores（而不是 Models），并且用户交互将会触发 Actions，Actions 则会被提交到一个集中的 Dispatcher 当中。当 Actions 被派发之后，Stores 将会随之更新自己并且通知 Views 进行修改，这些 Store 当中的修改会进一步促使 Views 查询新的数据。

一个简单的Flux架构模型如下

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c72b159254646edb0baae469219fb63~tplv-k3u1fbpfcp-zoom-1.image)

当有多个Store和View被添加后，复杂的Flux流程图如下图所示

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf2414de96b145c48e08f562bc7a2179~tplv-k3u1fbpfcp-zoom-1.image)

如果还是让你感觉到复杂的话，只留下Flux最精简的流程：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d1b9a5d00164e6aa264fe60d571d765~tplv-k3u1fbpfcp-zoom-1.image)

Flux 最大的特点就是查询和更新的分离，View 从 Store 获取的数据是只读的。而 Stores 只能通过 Actions 被更新，这就会影响 Store 本身而不是那些只读的数据。

由此可见即使是复杂的Flux应用，它的数据流和程序的运作过程仍然是清晰可辨的，所以这样做的好处就是`可预测`。

## Vuex中对于Flux架构的实践

Vuex与Flux的理念一拍即合，都是期望用单向数据流动来代替多向数据流动，从而减少维护成本。那么，Vuex是怎么实践Flux呢？

先来看看Vuex官网的一张图
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3642ff8a91a043cb88224d033b5b7ddc~tplv-k3u1fbpfcp-zoom-1.image)

Vuex中保留了action与store的概念，并且引入了新的mutation。action和mutation广义上来说都是提交对store修改，不同的是action可以是异步的，并且大多数情况是在event handler中提交，通过$store.dispatch方法；唯一修改 Store 的地方只能通过mutation，而且mutation必须是同步的，直接对store进行修改，所以便可以保证：`集中式`，`可预测`。

此外，Vuex也获得官方调试工具vue-devtools的支持，提供了零配置的 time-travel 调试、状态快照导入导出等高级调试功能。

现在就可以回答第二个问题：为什么修改store需要使用`$store.commit('updateXXX', value)`，而不是直接`$store.XXX = value`？

使用`$store.commit('updateXXX', value)`，每一条 mutation 被记录，devtools 都需要捕捉到前一状态和后一状态的快照（action的意义也在于此），使用`$store.XXX = value`则无法获得开发工具的追踪功能，也不能对这个修改动作进行统一的管理。这么做的好处就是为了做到`单向数据流`, 保证`可预测`。

## 小结

本文从多个vue组件状态传递这个难题出发，到介绍提出单向数据流的Flux架构，再到Vuex对Flux的实践，围绕`集中式`，`可预测`进行分析，相信你多多少少会有自己的收获，本文如有错误，欢迎大家斧正！
