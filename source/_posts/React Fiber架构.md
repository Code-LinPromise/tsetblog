---
title: React Fiber架构
date: 2023/04/26 21-24
tag: React
sticky: 100
category:
 - 前端
---


## 前言

<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9449275c49dd47609943a39b8c2f087a~tplv-k3u1fbpfcp-watermark.image?" alt="image.png" width="100%" />
在了解React Fiber架构之前，我们得从React 15中的栈调和（stack Reconciler）入手。

在React15中，因为使用的是不可打断stack Reconciler的同步渲染，所以当调和时间很长时间，就会导致Javascript线程长时间霸占主线程，进而导致渲染卡顿、卡死、交互时间长时间无响应等问题，所以React团队在React16做出了改变-Fiber。

## React 调和过程是怎样的
**调和（Reconciliation），又译为 协调。**
### React调和与diff算法
调和指的是将虚拟 DOM 映射到真实 DOM 的过程，Diff 过程只是其中一个环节。

React 源码结构佐证了这一点：React 从大的板块上将源码划分为 Core、Renderer、Reconciler 三部分。

其中 Reconciler 调和器所做的工作是一系列的，包括组件的挂载、卸载、更新等过程，其中更新过程涉及对 Diff 算法的调用。

所以，`调和 !== Diff`。但 Diff 确实是调和过程中最具有代表性的一环。

根据 Diff 实现形式的不同，调和过程被划分为 以 React 15 为主的"栈调和" 以及 以 React 16 为主的"Fiber 调和"。

### React 15
当我们通过`render`和`setState`进行组件渲染和更新的时候，`React`主要有两个阶段：

-   **协调阶段Reconciler**：通过`diff`算法递归比较新旧两棵虚拟`dom`树，计算出需要改变的部分`patch`。
-   **渲染阶段Renderer**：负责将`patch`批量更新到真实`dom`。

 
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c39f3f10123460a80cbdacafd742dc8~tplv-k3u1fbpfcp-watermark.image?)

### React 16
`react 16`新增**Scheduler**阶段；并且重构了**Reconciler**，即 **`Fiber` Reconciler**。

-   **调度器Scheduler**：调度任务的执行，优先级高的任务会先进入协调阶段。
-   **协调器Reconciler**：负责找出变化的组件，可以中断更新过程。
-   **渲染器Renderer**：负责将变化的组件渲染到页面上。


![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/408e614fc1584ac4ac480835cc5f5111~tplv-k3u1fbpfcp-watermark.image?)

## Fiber架构是如何解决问题的
Fiber架构的应用目的是实现“增量渲染”

实现增量渲染的目的，是为了实现任务的可中断，可恢复，并给不同的任务赋予不同的优先级，最终达成更加顺滑的用户体验。

Fiber架构的重要特征就是可以被打断的异步渲染模式，Fiber架构的核心就是：可中断、可恢复与优先级，每个更新任务都会被赋予一个优先级，若发现B的优先级高于当前任务A的优先级，那么当前处于Reconciler层的A任务就会被打断，将B任务推进Reconciler层，当B任务调和完成时，根据优先级排序，A任务将会重新推入Reconciler层，继续它的渲染之旅，这便是可恢复。

 每一个工作单元都接受Scheduler（调度器）的优先级调度。
 
 ## Fiber架构中的diff算法
 ### fiber node
 在React中，每一个React元素都会有对应的一个fiber node，与React元素不同的是，fiber node不会在每一次渲染时重新创建这些fiber node。
 
 ### fiber tree
 fiber tree是一个链表结构，它通过`fiber`节点的`return、child、sibling`属性进行连接。
 
 ### current Tree、workInProgress Tree
在 React 中最多存在两颗 Fiber 树，当前屏幕上 DOM 结构对相应的 Fiber 树称为 current Fiber 树(首次渲染时会得到第一个Fiber树)，在内存中构建的 Fiber 树称为 workInProgress Fiber 树。其中，Diff 算法的计算过程就是生成 workInProgress Fiber 树的过程，每次页面状态更新都会产生新的 workInProgress Fiber 树，当 workInProgress Fiber 树构建完成后交给 Renderer 渲染在页面上，之后在 React 中使用根节点的 current 指针完成由 current Fieber 树到 workInProgress Fiber 树的切换，此时

workInProgress Fiber 树就成为了 current Fiber 树，完成 DOM 更新。

### 如何构建workInProgress Tree
Diff 算法的本质是对比 JSX 对象和 current Fiber 节点，然后根据对比结果生成 workInProgress Fiber 节点，进而生成

workInProgress Fiber 树。其中需要执行相关操作的 Fiber 节点将会被打上 flags 标记，之后 Renderer 渲染器基于 Diff 过程中打上 flags 标记的 Fiber 节点**链接**成的**链表**进行相关的 DOM 操作。

所以在构建workInProgress Fiber时我们也会复用current Fiber未发生改变的fiber node，如果对比current Fiber和JSX对象，得到有差异的fiber node则会重新创建（不会复用）。

### 提示

-   **在构建`Fiber`树的过程中，`Fiber Reconciler`会将需要更新的节点信息保存在`Effect List`当中，在`commit`阶段时进行批量更新。**
-   构建`workInProgress tree`的过程就是`diff`的过程，通过`requestIdleCallback`来调度执行一组任务。

## Renderer（渲染）

### commit阶段
处理`Effect List`：包含更新`dom`、调用组件生命周期函数、更新`ref`等内部状态。

### 注意！！！

**commit**阶段是不可中断的，是一次执行完成的，此阶段不能暂停，否则会出现UI更新不连续的现象。此阶段需要根据effect list，将所有更新都 commit 到DOM树上。（因为Reconciler是在用户察觉不到的情况下运行，所以可中断、可恢复影响不到用户体验）。
## 总结
1. 相较于 React15，React16 利用 Scheduler 和基于 Fiber 节点的链表结构的虚拟 DOM 实现了可中断异步 DOM 更新，改善了页面 DOM 层级过深时造成的页面卡顿现象。React16 中的虚拟 DOM 树是由 Fiber 节点链接成的 Fiber 树，其中的每一个 Fiber 节点都有与之相对应的真实 DOM 节点。
2. 在 React 中最多存在两颗 Fiber 树，current Fiber 树和 workInProgress Fiber 树，Diff 算法的本质就是对比 current Fiber 节点和 JSX 对象，然后生成 workInProgress Fiber 树。根据同级的节点数量将 Diff 算法分为两类，单节点 Diff 和多节点 Diff。
3. `调和 !== Diff`。但 Diff 确实是调和过程中最具有代表性的一环。
4. **commit**阶段是不可中断的，是一次执行完成的，此阶段不能暂停，否则会出现UI更新不连续的现象。此阶段需要根据effect list，将所有更新都 commit 到DOM树上。（因为Reconciler是在用户察觉不到的情况下运行，所以可中断、可恢复影响不到用户体验）。



  


