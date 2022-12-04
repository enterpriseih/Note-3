# 概念

## nodejs是什么

- nodejs是基于Chome V8引擎的Javascript运行时（运行环境）
- nodejs出现之前，js只能在浏览器运行
- nodejs出现之后，js可以在任何安装nodejs的环境运行

## commonjs和ES6 Module有何区别

- 语法不同

- commonjs是动态引入，执行时引入

  <img src="img/image-20221204201550462.png" alt="image-20221204201550462" style="zoom:60%;" />

- ES6 Module是静态引入，编译时引入

  <img src="img/image-20221204201422600.png" alt="image-20221204201422600" style="zoom:67%;" />

### commonjs require的三个级别

- nodejs自带模块，如erquire（‘http’）
- npm安装的模块，如require（‘lodash’）
- 自定义的模块，如require（‘../utils.js’）

## nodejs和前端js的区别

- 都使用ES语法
- 前端js使用JS Web API
- nodejs使用node API
- 前端js用于网页，在浏览器运行
- nodejs可用于服务端，如开发web server
- nodejs也可用于本机，如webpack等本机工具

<img src="img/image-20221204202412036.png" alt="image-20221204202412036" style="zoom:50%;" />

# 框架的作用和价值

- 封装原生的api，让开发更简单
- 规范流程和格式，如koa2的中间件
- 让开发人员专注于业务和功能开发

# async/await执行顺序

<img src="img/image-20221204164423328.png" alt="image-20221204164423328" style="zoom:67%;" />

<img src="img/image-20221204164444377.png" alt="image-20221204164444377" style="zoom: 67%;" />

> await后面的内容相当于异步回调的内容！！

执行结果：

![image-20221204164838255](img/image-20221204164838255.png)

# koa2如何处理路由

<img src="img/image-20221204165247614.png" alt="image-20221204165247614" style="zoom:60%;" />

# koa2洋葱圈模型

<img src="img/image-20221204165449955.png" alt="image-20221204165449955" style="zoom:50%;" />

<img src="img/image-20221204165556244.png" alt="image-20221204165556244" style="zoom:80%;" />

# 服务端经典分层模型

<img src="img/image-20221204165849757.png" alt="image-20221204165849757" style="zoom: 67%;" />

# Http

## 如何启动http服务

<img src="img/image-20221204194636543.png" alt="image-20221204194636543" style="zoom: 60%;" />

## 如何定义路由

- 定义路由的两大要素：method+pathname
- 从req中获取method和url
- 再通过url获取pathname

<img src="img/image-20221204195418891.png" alt="image-20221204195418891" style="zoom:65%;" />

## 如何获取querystring

<img src="img/image-20221204195600535.png" alt="image-20221204195600535" style="zoom:65%;" />

## 如何获取Request body

<img src="img/image-20221204195925519.png" alt="image-20221204195925519" style="zoom:72%;" />

## 如何返回json格式的数据

<img src="img/image-20221204200215520.png" alt="image-20221204200215520" style="zoom:60%;" />

## session如何实现登录

- cookie可实现登录校验
- cookie不能泄漏用户信息，所以有了session
- session存储用户信息，cookie和session关联

<img src="img/image-20221204200557798.png" alt="image-20221204200557798" style="zoom:60%;" />

<img src="img/image-20221204200657933.png" alt="image-20221204200657933" style="zoom:60%;" />









