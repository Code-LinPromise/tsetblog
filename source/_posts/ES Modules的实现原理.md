---
title: 迭代器（iterator）、可迭代对象、生成器（generator）到底是什么
date: 2023/04/06 22-39
tag: JavaScript
category:
 - 前端
---


## 前言
在JS初期，JS没有模块化时，团队分工合作问题以及变量作用域问题很是问题，随后在JS社区衍生出AMD、CMD、CommonJS等模块化规范，[关于CommonJS的实现原理可以参考我这篇](https://juejin.cn/post/7205508171524096060)，现如今，AMD、CMD的使用逐渐变少，应用广泛的就是CommonJS和ES modules，ES modules是ECMA官方定义的模块化语法。

## 使用陷阱
在没有脚手架的帮助下，我们使用ES modules时，在HTML文件中引入script标签时，需要加上type="module"属性，表示script使用方法为模块化。

### HTML文件中引入方法
```js
<script src="./foo.js" type="module"></script>
```
### JS文件中使用方法
```js
//bar.js
export const name="1in"
export const age=20
```
```js
//foo.js
import {name,age} from "./bar"

console.log(name)
console.log(age)
```
### 结果
打开浏览器，看一下console.log出我们想要的结果出来没

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/82bcda331af44df4b0a5d31d4ef1d765~tplv-k3u1fbpfcp-watermark.image?)

得到了我们想要的结果。但是！！！ 注意，因为我使用的编译器是Webstrom，所以我运行html文件时，Webstrom会帮我自动开启一个服务端口运行此文件，但是我们都知道，本地双击html文件也是可以用浏览器打开的，这个时候，就会报错，所以，我们在VScode中就会遇到这种情况。

### 解决VScode报错问题
我们只需要安装一个插件，安装之后，在html文件中右键用这个插件打开即可。

在VScode中搜索 **Live Server**

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/19e8b68deefa421d9d8363f331f08379~tplv-k3u1fbpfcp-watermark.image?)

### 为什么报错
MDN中是这样解释的。

MDN： 如果你尝试用本地文件加载 HTML 文件 (i.e. with a `file://` URL)，由于 JavaScript 模块的安全性要求，你会遇到 CORS 错误。你需要通过服务器来做你的测试。


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/876cad088cb048e9bbab7042dd1163e0~tplv-k3u1fbpfcp-watermark.image?)

## 实现原理
ES Module是如何被浏览器解析并且让模块之间可以互相引用的呢？

ES Module的解析过程可以划分为三个阶段。
1. 构建（Construction），根据地址查找js文件，并且下载，将其解析成模块记录（Module Record）；
2. 实例化（Instantiation），对模块记录进行实例化，并且分配内存空间，解析模块的导入和导出语句，把模块指向对应的内存地址；
3. 运行（Evaluation），运行代码，计算值，并且将值填充到内存地址中；

在构建时，浏览器会静态扫描每个js文件的import语句（不会执行代码），所以我们的import语句不能写在类似于if语句这种需要动态判断的代码之中，import函数除外，扫描到import语句后，开始下载每一个js文件，下载好的js文件会parse成一个Module Record，同时下载过程中，还会生成一个Module Map的表，会记录已经下载的js文件和正在下载的js文件，避免重复引入。

当所有的js文件构建成Module Record后，就会开始实例化阶段和运行求值阶段，每一个Module Record会实例化一个Module Environment Record，将自己导出的值绑定（Bindings）到Module Environment Record（内存地址）中，同时，需要导入js文件的Module Record也会将自己需要导入的值绑定（Bindings）到Module Environment Record（内存地址）中，在这个地址内存中，导入、导出自己对应的值，当然这个值，是需要求值运算过得，也就是阶段三完成后，对应的js文件就可以导入对应的值。如果导出的是数值，导入的js文件就会拿到对应的数值，如果导出的是函数或者对象，导入的js文件就会拿到对应的地址

### 注意！！！
导出的模块是可以修改导出的值（value），导入模块是不可以修改导入的值（value）否则会报错！！！

## 总结
ES Module的解析过程可以划分为三个阶段。

1.1 构建（Construction），根据地址查找js文件，并且下载，将其解析成模块记录（Module Record）；

1.2  实例化（Instantiation），对模块记录进行实例化，并且分配内存空间，解析模块的导入和导出语句，把模块指向对应的内存地址；
  
1.3  运行（Evaluation），运行代码，计算值，并且将值填充到内存地址中；

2. 导出的模块是可以修改导出的值（value），导入模块是不可以修改导入的值（value）否则会报错！！！ 
3. 在没有脚手架的帮助下，我们使用ES modules时，在HTML文件中引入script标签时，需要加上type="module"属性，表示script使用方法为模块化。
4. 在本地使用（没有脚手架的情况下）ES modules，需要使用VScode的一个插件**Live Server**，而Webstrom编辑器打开时会自动创建一个服务端口。



