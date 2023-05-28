---
title: React中使用Hooks的正确姿势
date: 2023/05/17 22-59
tag: React
category:
 - 前端
---

## 前言

<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5678f6490cc347fa9f71c0272caf27eb~tplv-k3u1fbpfcp-watermark.image?" alt="image.png" width="100%" />
Hooks是React为函数式组件提供的工具箱，可以由开发者自由搭配，这也是React的函数式组件轻量的原因。

Class组件固然好，但是用于解决小规模问题，犹如大炮打蚊子，太过于繁重，函数式组件的出现，就解决了这一问题。

所以就Hooks而言对函数式组件的重要是不言而喻的。

现如今React官方也推崇函数式组件，React的理念就是吃进数据吐出视图（数据驱动视图），函数式组件正好印证了这一点。

下面就由我来总结一下在React函数式组件中使用Hooks的正确姿势。

## useState
**为函数式组件解决了无法实现管理状态的缺点**

1. 不可以在**循环/判断**语句中使用，因为useState的底层实现是以链表的形式实现，更新是以批量更新的方式去更新，如果在**循环和/判断语句中去使用**会破坏链表的更新顺序，使得更新混淆等问题。
2. useState是异步更新（只要state还在React的掌控之中就是异步更新的，SetTimeout会帮助state逃脱React的掌控），如果需要同步更新请使用**flushSync**这个官方推出的API（非必要不建议同步更新），因为每一次的state更新就去改变视图，则会频繁更新，浪费性能，React在每一次的state更新时，暂存这个state，当state改变完时，则会去完成批量更新。

## useEffect和useLayoutEffect
**为函数式组件解决了没有生命周期的缺点（可以模拟生命周期）**
1. 主要模拟三个生命周期，componentDidMount（组件挂载后，第二个参数为[]）、componentDidUpdate（组件更新后，第二个参数为[需要监听的属性值]）、componentDidUnmount（组件销毁后，通过return返回函数）
2. 第二个参数为浅层比较，不要传递对象进行比较。
3. useLayoutEffect的出现顾名思义就是为了解决布局而出现的，它的触发时机在DOM更新之前获取DOM元素并且执行，所以它是同步渲染的。
4. useEffect，它的触发时机是DOM更新之后获取DOM元素并且执行，所以它是异步渲染的，React官方推荐我们在useEffect解决不了的场景下再去使用useLayoutEffect，但是我们需要明白，两个Hooks的执行时机不同，并且时间也不同，一个是异步渲染，一个是同步渲染。
5. 一般向后端发送请求时，常常使用useEffect模拟componentDidMount这个生命周期去发送请求。

## useCallback
**为函数式组件提供缓存函数的功能**
1. 当我们使用React.memo缓存组件时，如果我们用父组件向这个子组件传递了一个函数，因为函数是引用数据类型，当父组件每一次更新时，这个函数的引用都会改变，所以这个时候我们就需要去用useCallBack缓存这个函数搭配React.memo避免子组件重复渲染。
2. 同样，第二个参数为浅层比较，不要传递对象进行比较。

## useMemo
**为函数式组件提供缓存数据的功能**
1. 当我们通过属性计算出一个属性时，我们可以使用useMemo这个Hooks缓存这个计算出的属性，避免组件刷新时，重复计算，达到性能优化的作用。
2. 同样，第二个参数为浅层比较，不要传递对象进行比较。

## 函数式组件需要避免的问题
因为函数式组件每一次渲染都会重新执行，所以需要将常量放到函数外部，避免重复定义，浪费性能，如果定义的是一个常量函数，且需要用到函数内部的变量做计算，那么一定要用到useCallback缓存这个函数。

### React.memo VS React.useMemo
React.memo是一个高阶组件，它的作用类似于React.pureComponent，但在Hooks的场景下，更推荐使用React.useMemo。

可以通过分拆组件的方式阻断重渲染，但使用React.useMemo可以实现更精细化的控制。

考虑到更宽广的使用场景和维护性，推荐使用React.useMemo。
