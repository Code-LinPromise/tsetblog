---
title: H5端中遇到的手势滑动问题——封装useSwipe Hooks
date: 2023/02/21 20:24
tags: 
  - Vue
  - Mobile
category: 前端
---


**在做H5端页面时，避免不了遇到手势滑动问题，就此问题，我封装了一个Hooks（useSwipe）来解决这个问题，下面，我将一步步讲解useSwipe的封装过程**

**下述源码引用了部分Vue的核心API，用Jsx语法编写**
**全部代码均为TypeScript编写**
## 完整源码
```tsx
import { computed, onMounted,onUnmounted,Ref,ref } from "vue";

type Point={
    x:number,
    y:number
}

interface Options{
    beforeStart?:(e:TouchEvent)=> void
    beforeMove?:(e:TouchEvent)=> void
    beforeEnd?:(e:TouchEvent)=> void
    afterStart?:(e:TouchEvent)=> void
    afterMove?:(e:TouchEvent)=> void
    afterEnd?:(e:TouchEvent)=> void
}
export const useSwipe=(element:Ref<HTMLElement | undefined>,options?:Options)=>{
    const start =ref<Point>()
    const end =ref<Point>()
    const swipeing=ref(false)

    const distance=computed(()=>{
        if(!start.value || !end.value){ return undefined}
        return{
            x:end.value.x-start.value.x,
            y:end.value.y-start.value.y
        }
    })

    const direction=computed(()=>{
        if(!swipeing.value){return ""}
        if(!distance.value){return ""}
        const { x , y } =distance.value
        if(Math.abs(x)>Math.abs(y)){
            return x>0? "right" : "left"
        }
        else{
            return y>0? "down" : "up"
        }
    })
    const onStart=(e:TouchEvent)=>{
        options?.beforeStart?.(e)
        start.value={
            x:e.touches[0].clientX,
            y:e.touches[0].clientY
        }
        options?.afterStart?.(e)
    }
    const onMove=(e:TouchEvent)=>{
        options?.beforeMove?.(e)
        end.value={
            x:e.touches[0].clientX,
            y:e.touches[0].clientY
        }
        swipeing.value=true
        options?.afterMove?.(e)
    }
    const onEnd=(e:TouchEvent)=>{
        options?.beforeEnd?.(e)
        swipeing.value=false
        options?.afterEnd?.(e)
    }
    onMounted(()=>{
        element.value?.addEventListener("touchstart",onStart)
        element.value?.addEventListener("touchmove",onMove)
        element.value?.addEventListener("touchend",onEnd)
    })
    onUnmounted(()=>{
        element.value?.removeEventListener("touchstart",onStart)
        element.value?.removeEventListener("touchmove",onMove)
        element.value?.removeEventListener("touchend",onEnd)
    })

    return {
        swipeing ,distance,direction 
    }
}
```
## 核心思路
**1. 当用户点击屏幕时，记录此时点击的屏幕坐标，当用户移动时，记录移动过程中的屏幕坐标，最后做差，即可判别用户的滑动行为**
**2. 同时需要return出相应的参数，例如distance（屏幕坐标的差值）、direction（用户的滑动行为）、swipeing（用户是否在滑动中）**
## 代码编写
**首先我们得声明记录屏幕坐标的类型Point，Point类型中有x坐标和y坐标，均为number类型，代码如下**
```tsx
type Point={x:number,y:number}
```
**接下来，useSwipe函数需要接受那些参数呢？这个时候，我们就要明白，useSwipe是指定用户在规定的DOM元素中的手势滑动行为，所以useSwipe需要接受一个HTML元素，我们也可以规定一个不定参数去接收对应的函数，去增加useSwipe的可扩展性。**
```tsx
const useSwipe=(element:Ref<HTMLElement | undefined>,options?:Options)=>{}
```
**注意！我们这里element接收的是一个ref类型，所以我们传入的DOM元素必须绑定ref并传入，此时element的类型也有可能undefined，因为你所传入的ref也有可能没有绑定DOM元素，这里的options是一个对象，里边可以传入六个不定函数，分别为beforeStart、beforeMove、beforeEnd、afterStart、afterMove、afterEnd，这六个参数分别在用户开始滑动、滑动中、滑动结束的前后调用，函数具体内容可自定义。所以，此时我们要声明一下options的类型。**
```tsx
interface Options{
    beforeStart?:(e:TouchEvent)=> void
    beforeMove?:(e:TouchEvent)=> void
    beforeEnd?:(e:TouchEvent)=> void
    afterStart?:(e:TouchEvent)=> void
    afterMove?:(e:TouchEvent)=> void
    afterEnd?:(e:TouchEvent)=> void
}
```
**接下来，我们可以开始具体实现useSwipe了，首先，我们得声明一下记录用户起始坐标和记录用户移动过程中坐标以及记录用户是否在移动的变量。**
```tsx
    const start =ref<Point>()
    const end =ref<Point>()
    const swipeing=ref(false)
```
**有了记录的变量之后，就可以给element添加事件，touchstart、touchmove、touchend，当用户touchstart时，记录用户的起始坐标，并将swipeing置为true，当用户touchmove时，记录用户的移动坐标，当用户touchend时，将swipeing置为false。并在执行这些代码前后分别执行可自定义的options中的函数**
```tsx
    const onStart=(e:TouchEvent)=>{
        options?.beforeStart?.(e)
        start.value={
            x:e.touches[0].clientX,
            y:e.touches[0].clientY
        }
        options?.afterStart?.(e)
    }
    const onMove=(e:TouchEvent)=>{
        options?.beforeMove?.(e)
        end.value={
            x:e.touches[0].clientX,
            y:e.touches[0].clientY
        }
        swipeing.value=true
        options?.afterMove?.(e)
    }
    const onEnd=(e:TouchEvent)=>{
        options?.beforeEnd?.(e)
        swipeing.value=false
        options?.afterEnd?.(e)
    }
```
**接下来就可以把对应的事件绑定到element DOM元素上,此时我们需要Vue中的onMounted钩子（当DOM元素全部挂载到页面上时，绑定监听事件）**
```tsx
onMounted(()=>{
        element.value?.addEventListener("touchstart",onStart)
        element.value?.addEventListener("touchmove",onMove)
        element.value?.addEventListener("touchend",onEnd)
    })
```
**有了start坐标以及end坐标后，我们可以继续封装distance以及direction函数。**

**distance和direction函数均用Vue中的computed方法去管理return出的值。**

**distance函数，根据end和start对应的x坐标和y坐标做差即可得出distance，distance的类型也为Point，内含x坐标和y坐标**

**direction函数，根据做差后的x值和y值作比较，分别判断出用户是左右滑动还是上下滑动，判断出上下滑动还是水平滑动后，通过正负值判断出方向。因为direction是在滑动过程中去判断出方向，所以我们通过swipeing变量防止没有滑动时direction去判断方向**
```tsx
    const distance=computed(()=>{
        if(!start.value || !end.value){ return undefined}
        return{
            x:end.value.x-start.value.x,
            y:end.value.y-start.value.y
        }
    })

    const direction=computed(()=>{
        if(!swipeing.value){return ""}
        if(!distance.value){return ""}
        const { x , y } =distance.value
        if(Math.abs(x)>Math.abs(y)){
            return x>0? "right" : "left"
        }
        else{
            return y>0? "down" : "up"
        }
    })
```
**最后，需要在DOM元素销毁后，移除绑定的监听事件**
```tsx
onUnmounted(()=>{
        element.value?.removeEventListener("touchstart",onStart)
        element.value?.removeEventListener("touchmove",onMove)
        element.value?.removeEventListener("touchend",onEnd)
    })
```
