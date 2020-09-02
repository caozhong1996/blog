# vuex设计理念兼谈flux与(CQRS,DDD)

## 前文

当你使用vuex的时候，可能有如下疑惑
- 为什么修改store需要使用`$store.commit('updateXXX', value)`，而不是直接`$store.XXX = value`？
- 使用vuex和将变量储存在window对象下有什么区别？
- vuex和eventBus的区别在哪里？

## 本文将讨论以下信息

1. vuex与Flux架构的关系
2. 描述 CQRS 模式
3. Flux 如何应用来自 CQRS 的概念
4. 讨论 Flux 何时适用于 JavaScript 应用

本文假设你已经了解vuex的基本使用，并不会具体简绍vuex使用方式。

## vuex是什么

> Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。

### 背景问题

>当我们的vue应用遇到多个组件共享状态时，单向数据流的简洁性很容易被破坏：

>- 多个视图依赖于同一状态。
>- 来自不同视图的行为需要变更同一状态。

>对于问题一，传参的方法对于多层嵌套的组件将会非常繁琐，并且对于兄弟组件间的状态传递无能为力。对于问题二，我们经常会采用父子组件直接引用或者通过事件来变更和同步状态的多份拷贝。以上的这些模式非常脆弱，通常会导致无法维护的代码。

如果我们暂时不去讨论vue中的响应式（mvvm），而是简单的把层级分为MVC的话，大概数据的流动可能如图所示

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3dcc368021204e5f8509aaa6bdb38073~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e7880c3839840a7b13ca9d106a6319d~tplv-k3u1fbpfcp-zoom-1.image)

甚至夸张一点

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a81aebdaddc44615a53c926eb396e0f6~tplv-k3u1fbpfcp-zoom-1.image)

很容易理解，复杂的数据流动必定会带来维护上的困难

>因此，我们为什么不把组件的共享状态抽取出来，以一个全局单例模式管理呢？在这种模式下，我们的组件树构成了一个巨大的“视图”，不管在树的哪个位置，任何组件都能获取状态或者触发行为！

>通过定义和隔离状态管理中的各种概念并通过强制规则维持视图和状态间的独立性，我们的代码将会变得更结构化且易维护。

>这就是 Vuex 背后的基本思想，借鉴了 Flux、Redux 和 The Elm Architecture。与其他模式不同的是，Vuex 是专门为 Vue.js 设计的状态管理库，以利用 Vue.js 的细粒度数据响应机制来进行高效的状态更新。

注意，此时提到了vuex与flux关系，现在我们来讲讲flux架构的基本理念

## flux架构
在前文我们提到了因为多向数据流带来状态管理上的问题，Flux 试图通过强制单向数据流来解决这个复杂度。

在这种架构当中，Views 查询 Stores（而不是 Models），并且用户交互将会触发 Actions，Actions 则会被提交到一个集中的 Dispatcher 当中。当 Actions 被派发之后，Stores 将会随之更新自己并且通知 Views 进行修改。这些 Store 当中的修改会进一步促使 Views 查询新的数据。

一个简单的flux架构模型如下

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c72b159254646edb0baae469219fb63~tplv-k3u1fbpfcp-zoom-1.image)

当有多个Store和View被添加后，复杂的flux流程图如下图所示
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf2414de96b145c48e08f562bc7a2179~tplv-k3u1fbpfcp-zoom-1.image)
如果还是让你感觉到复杂的话，只留下flux最精简的流程：
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d1b9a5d00164e6aa264fe60d571d765~tplv-k3u1fbpfcp-zoom-1.image)

由此可见即使是复杂的flux应用，它的数据流和程序的运作过程仍然是清晰可辨的。

MVC 和 Flux 最大的不同就是查询和更新的分离。在 MVC 中，Model 同时可以被 Controller 更新并且被 View 所查询。在 Flux 里，View 从 Store 获取的数据是只读的。而 Stores 只能通过 Actions 被更新，这就会影响 Store 本身而不是那些只读的数据。

以上所描述的模式非常接近于由 Greg Young 第一次所提出的 CQRS。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3642ff8a91a043cb88224d033b5b7ddc~tplv-k3u1fbpfcp-zoom-1.image)