---
title: React15的三个Will生命周期为什么会在Fiber出现后被废弃
data: 2023/04/11 21-50
tag: React
category:
 - 前端
---


## 前言


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/07b352c9afa74a6680f92c4f396ea7a4~tplv-k3u1fbpfcp-watermark.image?)
React16出现Fiber之后就废弃了React15中的三个Will生命周期钩子，并新推出两个生命周期钩子，具体原因我们得从生命周期和Fiber架构具体了解。

###  React15生命周期

在React15中，组件的生命周期分为三大类，组件挂载、组件更新、组件卸载。
<img src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a656a71c94af420881f275ee912c7613~tplv-k3u1fbpfcp-watermark.image?" alt="image.png" width="100%" />

#### 组件挂载
在组件挂载时，触发两个生命周期的钩子，分别是componentWillMount（组件挂载前）、componentDidMount（组件挂载后），一般的请求都放在componentDidMount中。

#### 组件更新 
在组件更新时，首先会调用componentWillReceiveProps这个生命周期钩子，那么这个生命周期钩子的执行时机是什么时候呢？如果我们从这个钩子的名字来看，当组件接收的props值发生改变时，这个生命周期钩子会执行，这样的答案对也不对，正确的执行时机应该是父组件重新渲染时，会调用这个钩子。

其次，会根据shouldComponentUpdate这个钩子的返回值来判断是否更新，开发者也可以通过这个钩子来决定组件是否更新，随后就是componentWillUpdate和componentDidUpdate，分别是组件更新前和更新后的生命周期钩子

#### 组件卸载
在组件卸载时，会调用componentWillUnmount这个生命周期钩子

### React16生命周期
在React16中，组件的生命周期也分为三大类，组件挂载，组件更新，组件卸载，与React15不同的是因为Fiber的引入，删除了三个Will生命周期的钩子，引入了两个getDerivedStateFromProps和getSnapshotBeforeUpdate。

<img src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75d3c7a1b9e24f18b0d184dd22fa5d13~tplv-k3u1fbpfcp-watermark.image?" alt="image.png" width="100%" />

getDerivedStateFromProps 会在调用 render 方法之前调用，即在渲染 DOM 元素之前会调用，并且在初始挂载及后续更新时都会被调用。

####  getDerivedStateFromProps
static getDerivedStateFromProps(props, state)

state 的值在任何时候都取决于 props。

getDerivedStateFromProps 的存在只有一个目的：让组件在 props 变化时更新 state。

该方法返回一个对象用于更新 state，如果返回 null 则不更新任何内容。

getDerivedStateFromProps是一个静态方法

注意！！！

getDerivedStateFromProps不是componentWillMount的替代品，componentWillMount的存在不仅“鸡肋”而且危险，因此它不值得被“代替”，它就应该被废弃，getDerivedStateFromProps有且仅有一个用途：使用props来派生/更新state

React团队直接从命名层面约束了它的用途，getDerivedStateFromProps在更新和挂载两个阶段都会“出境”

getDerivedStateFromProps方法对state的更新动作并非“覆盖”式的更新，而是针对某个属性的定向更新

getDerivedStateFromProps是作为一个试图代替componentWillReceiveProps的API而出现的

getDerivedStateFromProps不能完全和componentWillReceiveProps划等号


#### getSnapshotBeforeUpdate

getSnapshotBeforeUpdate() 方法在最近一次渲染输出（提交到 DOM 节点）之前调用。

在 getSnapshotBeforeUpdate() 方法中，我们可以访问更新前的 props 和 state。

getSnapshotBeforeUpdate() 方法需要与 componentDidUpdate() 方法一起使用，否则会出现错误。

## 为什么出现Fiber
在React15中，因为使用的是不可打断stack Reconciler的同步渲染，所以当调和时间很长时间，就会导致Javascript线程长时间霸占主线程，进而导致渲染卡顿、卡死、交互时间长时间无响应等问题，所以React团队在React16做出了改变-Fiber

### Fiber是如何解决问题的
因为stack Reconciler是一个宏大的任务且是深度遍历，避免不了长时间霸占主线程，所以Fiber就将一个宏大的任务差分成许多工作单元，每个工作单元都有自己的优先级，每个工作单元都是可中断、可恢复的

所以Fiber架构由原先的Reconciler、Renderer两层变为了Scheduler、Reconciler、Renderer三层

![95a3b93cd26c70b77f0f6735020451c.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4ba2f5c0f73743b4b7425dd0c3723a3b~tplv-k3u1fbpfcp-watermark.image?)

![bd53e295df3871f70b3d51fb1ff7aa9.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e108b30b7fe74335ba6dd15e1e1010ab~tplv-k3u1fbpfcp-watermark.image?)

Fiber架构的应用目的是实现“增量渲染”

实现增量渲染的目的，是为了实现任务的可中断，可恢复，并给不同的任务赋予不同的优先级，最终达成更加顺滑的用户体验

Fiber架构的重要特征就是可以被打断的异步渲染模式，Fiber架构的核心就是：可中断、可恢复与优先级，每个更新任务都会被赋予一个优先级，若发现B的优先级高于当前任务A的优先级，那么当前处于Reconciler层的A任务就会被打断，将B任务推进Reconciler层，当B任务调和完成时，根据优先级排序，A任务将会重新推入Reconciler层，继续它的渲染之旅，这便是可恢复。

## 废弃三个Will钩子的原因
componentWillMount、componentWillUpdate、componentWillReceiveProps这三个钩子常年被开发者滥用，是产生副作用的重灾区，在Fiber中，因为所有的工作单元都是可中断，可继续的，所以会导致这三个钩子重复调用，导致不可想象的局面，所以这三个钩子随着Fiber的出现废弃是必然的。

