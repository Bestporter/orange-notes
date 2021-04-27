# Django Channels 笔记

“实时”网络应用：在这类应用中客户端和服务器之间、点对点通信交互非常频繁。包含很多服务（又名微服务）的应用也变成是常态。新的web技术允许web应用程序走向十年前我们只敢在梦里想象的方向。这些核心技术之一就是[WebSockets](https://en.wikipedia.org/wiki/WebSocket)：一种新的提供全双工通信的协议——一个持久的，允许任何时间发送数据的客户端和服务器之间的连接。

​	在这个新的世界，Django显示出了它的老成。在其核心，Django是建立在请求和响应的简单概念之上的：浏览器发出请求，Django调用一个视图，它返回一个响应并发送回浏览器。

这在WebSockets中是行不通的 ！**视图的生命周期只在一个请求当中，没有一种机制能打开一个连接不断的发送数据到客户端，而不发送相关请求。**

Channels允许Django以非常类似于传统HTTP的方式支持WebSockets。Channels也允许在运行Django的服务器上运行后台任务。HTTP请求表现以前一样,但也通过Channels进行路由。因此,在Channels 支持下Django现在看起来像这样:

