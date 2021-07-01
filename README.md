# Vuex 源码解析

## 概述

Vuex 的本质是全局状态管理。

全局状态就是一个对象，保存着整个应用的一些公共数据。

和普通全局对象不一样的是：

1、基于 vue 的响应式机制，这个全局状态能够在自己更新时通知订阅它的视图重新渲染。

2、不直接修改这个对象，而是通过提交 mutation 的方式同步修改，异步可以使用 action，有结果后再提交 mutation。这样的机制可以让操作这个对象变得可预测，同时也具备了可调式能力（logger）。

3、避免一个对象过于复杂，将引入 module 的机制，让开发者可以自行拆分。当然，vuex 会帮我们合并。


## 组成

Vuex 由 5 部分构成：

1、State 全局状态数据源

2、Getter 基于 State 派生出一些状态

3、Mutation 同步更改 State

4、Action 异步操作，最后提交 Mutation

5、Module 拆分 State，每一个模块都拥有 State、Getter、Mutation、Action


## 实现原理

1、基于 vue 的插件机制，全局混入一个 `beforeCreate` 钩子，将所有组件的 `$store` 都指向根组件注入的 Store 实例。

2、如何注册 module？

> 有两个类：ModuleCollection 和 Module，可以把他们的关系看作是 Tree 和 Node。
> Module 保存着开发者定义的源 state 和它的子 Module。
> ModuleCollection 用于递归注册 Module 到它的 root 属性上，这样就拥有了一棵 Module 实例树。
> 当然，此时整个 Module 实例树都是各个 module 保存自己的 state，并且 state 也不是响应的。

3、如何合并 state 并让其响应？

> store 实例挂载了一个 _vm 属性，这个属性是一个 Vue 实例，会把开发者定义的 state 作为 `data.$$state` ，getter 作为 `computed`，从而实现了 state 的响应。初始化、动态注册 module 或者热重载时都会重新生成一个 _vm。
> 对于子 module 的 state，会通过 Vue.set 的方式往父 module 的 state 上响应式注册 moduleName 到 module.state 的映射，这样 vuex 就把开发者拆分的 module.state 合并到一个 state 中。这也是为什么可以直接 `store.state[moduleName]` 直接访问到这个 module 的 state 的原因。

4、如何更新 state？

> 这个分为两步：
> step 1. 首先需要注册 mutation 来描述如何更改 state。
> vuex 会把不同 module 注册的 mutation 按照 type 归类到一起，开启 `namespaced` 可以将这个 type 唯一化。
> 开发者在使用 mutation 时，第一个参数其实是 local.state（当前 module 的 state），并不是整个 state。
> local 代理它的 state 和 getter 到具体的 store.state.xxx.yyy
> step 2. 使用 commit 提交 mutation 修改 state。
> vuex 对整个 state 进行了监听，当直接修改 state 时，会发出警告
> 而使用 mutation 的方式修改 state，会事先把标志位 commiting 打开，修改完之后关闭


## TODO

namesapce plugin helper
