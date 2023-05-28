---
title: JS如何实现图片压缩
date: 2023/05/20/ 21:59
tag: 
 - JavaScript
category:
 - 前端
---


## 前言

<img src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a82edb207c814cd09c2c6b0627829478~tplv-k3u1fbpfcp-watermark.image?" alt="image.png" width="100%">
关于图片压缩这一般都是由后端来完成~，但是前端掌握这项技能也是必不可少的。

## 图片压缩思路

我们读取源图片之后，利用`canvas`画板画出源图片，然后利用`toDataURL`这个API转换成`base64`的格式

需要FileReade这个对象去reader图片，并且利用reader.onload这个监听事件完成图片压缩。

## 源码

### index.html文件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        .hide{
            display: none;
        }
        .show{
            display: initial;
        }
        .img-preview{
            width: 300px;
        }
    </style>
</head>
<body>
    <p>
        <input type="file"  id="imgFileSelector" value="请选择图片">  
    </p> 
    <p>
        <img id="originImgPreview" class="img-preview  hide">
    </p>
    <p>
        <img id="compressedImgPreview" class="img-preview hide">
    </p>

    <script src="./index.js" type="module" ></script>
</body>
</html>
```

因为img元素既不是`块级元素`也不是`行内级元素`，所以添加类`.show`时需要设置为`display:initial`。

### index.js文件

```js
//获取HTML元素
const oImageFileSelector=document.querySelector("#imgFileSelector")
const oOriginImgPreview=document.querySelector("#originImgPreview")
const oCompressedImgPreview=document.querySelector("#compressedImgPreview")
//创建reader对象
const reader=new FileReader()

let imgFile=null
let quality=90
let compressedImgSrc=""

//构建IMG_TYPES表
const IMG_TYPES={
    "image/jpeg":"image/jepg",
    "image/png":"image/png",
    "image/gif":"image/gif",
    "image/jpg":"image/jpg"
}

function init (){
    bindEvent()
}
//构建绑定事件函数
function bindEvent(){
    oImageFileSelector.addEventListener("change",handleFileSelectorChange,false)
}

function handleFileSelectorChange(e){
    console.log(1)
    imgFile=e.target.files[0]
    console.log(imgFile)
    //判断imgFile是否存在并且imgFile的类型是否为IMG_TYPES表中的类型
    if(!imgFile || !IMG_TYPES[imgFile.type] ){
        alert("请选择正确格式的图片")
        setImgFileEmpty()
        return
    }
    setImgPreview(imgFile)

}

//初始化imgFile
function setImgFileEmpty(){
    oImageFileSelector.value=""
    imgFile=null
    setPreviewVisible(oOriginImgPreview,false)
    setPreviewVisible(oCompressedImgPreview,false)
}

function setImgPreview(imgFile){
    if( imgFile instanceof File){
        reader.onload=async ()=>{
            const originImgSrc=reader.result
            
            const compressedImgSrc=await createCompressedImage({
                imgSrc:originImgSrc,
                type:imgFile.type
            })
            console.log(compressedImgSrc)
            
            console.log("未压缩",originImgSrc.length)
            console.log("压缩后",compressedImgSrc.length)
            oOriginImgPreview.src=originImgSrc
            setPreviewVisible(oOriginImgPreview,true)
            oCompressedImgPreview.src=compressedImgSrc
            setPreviewVisible(oCompressedImgPreview,true)
        }

        reader.readAsDataURL(imgFile)
    }
}

function createCompressedImage ({
    imgSrc,
    type
}){
    const oCan=document.createElement("canvas")
    const oImg=document.createElement("img")
    const context=oCan.getContext("2d")

    oImg.src=imgSrc
    //因为通过onload事件触发回调函数，所以需要Promise的resolve回调传回值并接收
    return new Promise((resolve)=>{
        oImg.onload=()=>{
            const imageWidth=oImg.width
            const imageHeight=oImg.height
    
            oCan.width=imageWidth
            oCan.height=imageHeight
            
            //利用canvas的drawImage函数去画出源图像
            context.drawImage(oImg,0,0,imageWidth,imageHeight)
            
            const compressedImgSrc = doCompress(oCan, type, imgSrc)
            resolve(compressedImgSrc)
        }
    })
}

//创建递归函数，如果压缩的文件大小大于源文件就继续压缩
function doCompress (canvas,type,imgSrc){
    compressedImgSrc=canvas.toDataURL(type,quality/100)

    if(compressedImgSrc.length >=imgSrc.length && quality>10){
        quality-=10
        doCompress(canvas,type,imgSrc)
    }
    return compressedImgSrc
}

//是否显示img元素
function setPreviewVisible (node,visible){
    switch(visible){
        case true:
            node.classList.remove("hide")
            node.classList.add("show")
            break
        case false:
            node.classList.remove("show")
            node.classList.add("hide")
            break
        default:
            break
    }
}

init()

```

需要注意的是，我们是通过oImg.onload来执行压缩代码的，所以我们需要用Promise的resolve回调来传出值，通过async、await来接收值。
