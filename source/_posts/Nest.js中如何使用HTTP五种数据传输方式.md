---
title: Nest.js中如何使用HTTP五种数据传输方式
date: 2023/05/27/ 17:55
sticky: 100
tag: 
 - TepyScript
 - Nest.js
category:
 - 后端
---


## 前言

<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c1514657ca1e468dbd1d7b76dc951ef3~tplv-k3u1fbpfcp-watermark.image?" alt="image.png" width="100%">
Nest.js作为JS的后端框架，是JS开发者迈向全栈的不错选择，Nest.js也是企业中最流行的Node框架，它提供了IOC、AOP、微服务等架构特性。

接下来就让我们认识一下Nest.js在`HTTP五种数据`传输方式中的使用。

## Param

`param` 传输方式是通过url的参数传递，也是最常见的一种前端向后端传递参数的方式。

如果Nest后端接口这样设置

```ts
@Controller('api/parma')
export class ParmaController {
  @Get(':id')
  urlParm(@Param('id') id: string) {
    return `id=${id}`;
  }
}
```

前端请求这样请求

```js
axios.get("http://localhost:3000/api/parma/cdut")
```

那么，其中的`cdut`会被当做parma参数被Nest截取，Nest也为我们提供了便捷的`@Param`装饰器，使我们可以更便捷的提取param参数。

## Query

`query`传输方式也是通过url的参数传递，但是他与parma略有不同。query传输方式需要做url encode

如果Nest后端接口这样设置

```ts
@Controller('api/query')
export class QueryController { 
    @Get('find') query(@Query('name') name: string, @Query('age') age: number) 
    { return `name=${name},age=${age}`; } }
```

前端请求这样请求

```js
axios.get("http://localhost:3000/api/query/",{
    name:"1in",
    age:20
})
```

因为axios会自动帮我们url encode，所以我们不需要自己手动url encode，在Nest中，我们通过`@Query`这个装饰器来取到query参数。

## Form urlencoded

与query不同的是，from urlencoded是通过post请求中的body来传递参数，实际上，就是把query的参数放在body中。

需要注意的是我们需要在`请求头`中设置`'content-type': 'application/x-www-form-urlencoded'`。

如果Nest后端接口这样设置

```ts
@Controller('api/form-urlencoded') 
export class FormUrlencodedController { 
    @Post() body(@Body() body)
    { return `${JSON.stringify(body)}` } 
}
```

前端请求前，我们需要使用`qs`这个库来做一下url encode

前端请求这样请求

```js
axios.post('http://localhost:3000/api/form-urlencoded',
    Qs.stringify({ name: '1in', age: 20 }), 
    { headers: { 'content-type': 'application/x-www-form-urlencoded' } });
```

在Nest中，我们可以通过@Body这个装饰器来直接取到body中的内容。

## Json

与form urlencoded不同的是，json需要指定的`content-type`为`application/json`，也不需要url encode，同样的也是通过post请求中的Body传输数据。

如果Nest后端接口这样设置

```ts
@Controller('api/json')
export class JsonController {
    @Post()
    body(@Body() body) {
    return `received: ${JSON.stringify(createPersonDto)}`
    }
}
```

前端这样请求

```js
    axios.post("http://localhost:3000/api/json",{
        name:"1in",
        age:20
    })
```

因为`axios`会帮我们设置`content-type`为`application/json`，所以不需要我们自动设置，Nest同样也是通过@Body装饰器取到body中传输的数据。

## Form data

form data是通过---------作为boundary传输的内容，主要用于传输文件。

如果Nest后端接口这样设置

```ts
import { AnyFilesInterceptor } from '@nestjs/platform-express';

@Controller('api/form-data')
export class FormDataController {
  @Post('file')
  @UseInterceptors(AnyFilesInterceptor())
  body2(@Body() body, @UploadedFiles() files: Array<Express.Multer.File>) {
    console.log(files);
    return `received: ${JSON.stringify(body)}`
  }
}

```

前端代码使用 axios 发送 post 请求，指定 content type 为 `multipart/form-data`

前端请求是这样的

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <script src="https://unpkg.com/axios@0.24.0/dist/axios.min.js"></script>
</head>
<body>
    <input id="updateFile" type="file" multiple/>
    <script>
        const fileInput = document.querySelector('#updateFile');

        async function formData() {
            const data = new FormData();
            data.set('name','1in');
            data.set('age', 20);
            data.set('file1', fileInput.files[0]);
            data.set('file2', fileInput.files[1]);

            const res = await axios.post('http://localhost:3000/api/form-data/file', data, {
                headers: { 'content-type': 'multipart/form-data' }
            });
             
        }
        fileInput.onchange = formData;
    </script>
</body>
</html>
```

form data通过 ----- 作为 boundary 分隔的数据。主要用于传输文件，在Nest中使用 FilesInterceptor 来处理其中的 binary 字段，用 @UseInterceptors 装饰器来启用，其余字段用 @Body 装饰器来取。axios 发送请求时，需要设置请求头，指定 `content type`为 `multipart/form-data`，并且用 FormData 对象来封装传输的内容。

