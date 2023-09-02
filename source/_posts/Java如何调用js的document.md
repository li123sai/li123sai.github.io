---

title: Java如何调用js的document
date: 2023-08-24 09:55:52
categories:
- 前端
- JavaScript
tags:
- Node.js
---

## 一、前言

给这篇博文分类时很凌乱，内容是前端的JavaScript和Node.js，目的是为了让后端如何调用。为什么果断放到前端，完全是小私心，因为目前还没有前端分类，迫切需要一个，哈哈。

## 二、背景

项目中有一个需求是将一些前端样式进行转换，这个转换的代码由js完成在Html页面调用，目前需求是嵌入到后端代码中自动转换。

做的时候首先想到Kettle 中的 JavaScript脚本，于是将js代码放入Kettle执行，结果报错：

![kettle执行结果](https://raw.githubusercontent.com/li123sai/myPictures/main/img/js5.png)

然后尝试各种方法解决

```js
loadScript("path/to/your/script.js"); // 加载本地的JS文件
// 或者
loadScript("https://example.com/your/script.js"); // 加载远程的JS文件
// 或者
var file = new java.io.File("path/to/your/script.js");// 加载外部JS文件
var reader = new java.io.FileReader(file);
var bufferedReader = new java.io.BufferedReader(reader);
var line;
var scriptContent = "";
while ((line = bufferedReader.readLine()) != null) {
  scriptContent += line + "\n";
}
bufferedReader.close();
eval(scriptContent);
```

都报错。于是排除Kettle 中的 JavaScript的方式。

在项目中用Java调用js，结果也一直报错。

```java
Exception in thread "main" javax.script.ScriptException: ReferenceError: "window" is not defined in <eval> at line number 123
	at jdk.nashorn.api.scripting.NashornScriptEngine.throwAsScriptException(NashornScriptEngine.java:470)
	at jdk.nashorn.api.scripting.NashornScriptEngine.invokeImpl(NashornScriptEngine.java:392)
	at jdk.nashorn.api.scripting.NashornScriptEngine.invokeFunction(NashornScriptEngine.java:190)
    
Exception in thread "main" javax.script.ScriptException: ReferenceError: "document" is not defined in <eval> at line number 123
	at jdk.nashorn.api.scripting.NashornScriptEngine.throwAsScriptException(NashornScriptEngine.java:470)
	at jdk.nashorn.api.scripting.NashornScriptEngine.invokeImpl(NashornScriptEngine.java:392)
	at jdk.nashorn.api.scripting.NashornScriptEngine.invokeFunction(NashornScriptEngine.java:190)    
```



在尝试了各种方法都有问题时，改变了思路，不在用java直接调用，用Node.js将js代码部署成服务，远程调用。

## 三、解决方案

1. 确保已经安装Node.js环境

   ```
   #使用node -v 来确认已安装
   node -v
   ```

   

2. 创建一个新目录并初始化项目

   ```sh
   # 在命令行中，使用 mkdir myService 创建一个新的目录。
   mkdir myService
   # 进入该目录，然后运行 npm init 命令来初始化一个新的 Node.js 项目。根据提示输入相关信息。
   npm init
   # 这将创建一个 package.json 文件，其中包含了你的项目的配置信息和依赖项。
   ```

   

3. 安装Express.js

   ```sh
   # 在命令行中，运行 npm install express 命令来安装 Express.js 框架。
   npm install express
   # Express.js 是一个流行的、简洁而灵活的 Web 框架，可以帮助你快速搭建基于 Node.js 的 Web 服务。
   ```

   

4. 创建服务代码

   ```js
   // 在项目目录中，创建一个名为 server.js 的文件，并打开它。
   // 在 server.js 文件中，首先导入 Express 框架和其他所需模块：
   
   const express = require('express');
   const app = express();
   
   // 使用 app.get()、app.post() 等方法来定义路由和处理函数，监听特定的 URL 路径，并在请求到达时执行相应的处理函数。
   // 在处理函数中，获取传递的参数，可以通过 req.query 来获取查询字符串参数，通过 req.params 来获取路径参数。
   app.get('/example', (req, res) => {
     const param1 = req.query.param1; // 获取查询字符串参数
     const param2 = req.params.param2; // 获取路径参数
   
     // 执行方法并返回响应
     const result = yourMethod(param1, param2);
     res.send(result);
   });
   
   // 使用 app.use() 方法将 Express 中间件设置为解析请求体的 JSON 数据
   app.use(express.json());
   // 创建一个 POST 路由，通过调用 app.post() 方法监听 POST 请求并处理请求数据。你可以在处理函数中获取通过请求体传递的数据。
   app.post('/examplepost', (req, res) => {
     const data = req.body; // 获取请求体数据
       
     // 执行方法并返回响应
     const result = yourMethod(data);
     res.send(result);
   });
   
   // 在上述代码中，yourMethod() 是你自己定义的方法，根据传递的参数执行相应的逻辑，并返回结果。
   // 此时可以将需要待用的js代码全部复制到这里
   ```

   

5. 增加启动代码

   ```js
   // 在 server.js 文件的末尾，添加代码来启动你的服务
   app.listen(3333, () => {
     console.log('Server is running on port 3333');
   });
   
   // 这将在端口号 3333 上启动你的服务。也可以选择其他端口号。
   ```

   

6. 运行服务

   ```sh
   # 在命令行中，进入到你的项目目录，并运行 node server.js 命令。
   node server.js
   
   # 会看到一条消息表示你的服务已成功启动
   ```

   ![执行结果](https://raw.githubusercontent.com/li123sai/myPictures/main/img/js1.png)

7. 调用服务

   ```java
   // 现在JavaScript 方法已经被部署为一个 Node.js 服务，并且可以接收参数
   // 可以通过访问下边这样来传递查询字符串参数
   http://localhost:3333/example?param1=value1  
   // 或者这样来传递路径参数
   http://localhost:3333/example/value2 
   
   // 在处理函数中，你可以通过 req.query 来获取查询字符串参数，通过 req.params 来获取路径参数。
   ```

   

8. 测试调用

   ![执行结果](https://raw.githubusercontent.com/li123sai/myPictures/main/img/js3.png)

   还是报 document is not defined 。

   原因：

   ```
   当在 Node.js 环境中运行代码时，像 document 这样的对象是浏览器环境中的全局对象，而在 Node.js 中是不存在的。这就是为什么会收到 "document is not defined" 的错误消息。
   
   如果你想在 Node.js 中执行 DOM 操作，可以考虑使用一些类似于 jsdom 的第三方模块。
   jsdom 是一个流行的 Node.js 模块，可以模拟浏览器环境，使你能够在 Node.js 中使用类似于浏览器环境的 API。
   ```

   

9. 安装 jsdom 模块

   ```sh
   npm install jsdom
   ```

   

10. 在代码中使用jsdom

    ```js
    const { JSDOM } = require('jsdom');
    
    // 创建一个虚拟的浏览器环境
    const dom = new JSDOM('<!DOCTYPE html><p>Hello world</p>');
    
    // 获取文档对象
    const document = dom.window.document;
    
    // 在文档中进行 DOM 操作
    const paragraph = document.querySelector('p');
    console.log(paragraph.textContent);  // 输出: Hello world
    
    // 上边就是使用 JSDOM 类来创建一个虚拟的浏览器环境，并获取了 document 对象，以便执行 DOM 操作。
    ```

    ## 四、总结

在尝试Kettle 中的 JavaScript 脚本调用不行后，只了解Kettle 的 JavaScript没有直接访问DOM的能力，没有思考问题本身。

Kettle 中的 JavaScript 脚本环境是基于 Rhino，Rhino 引擎主要用于服务器端编程，与浏览器中的 JavaScript 环境有所不同，无法直接使用 `document` 对象进行 DOM 操作。Kettle 是java编写的，所以在后边直接用java调用也报错。

因为在 Java 中调用 JavaScript 时，通常使用的是 Java 的 JavaScript 引擎，例如 Rhino 或 Nashorn。这些引擎可以用于在 Java 程序中执行 JavaScript 代码，但它们并没有提供完整的浏览器环境，因此不支持像 `document` 这样的浏览器特定对象。



在寻找解决方案时，一直想怎么解决js调用报错问题，结果陷入循环，这时候不妨跳出来，看清问题本质，换个思路。
