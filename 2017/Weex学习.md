---
title: Weex学习
---

> Weex 是一套简单易用的跨平台开发方案，能以 web 的开发体验构建高性能、可扩展的 native 应用，为了做到这些，Weex 与 Vue 合作，使用 Vue 作为上层框架，并遵循 W3C 标准实现了统一的 JSEngine 和 DOM API，这样一来，你甚至可以使用其他框架驱动 Weex，打造三端一致的 native 应用。

[Weex官网](https://weex.apache.org/cn/)

[Weex代码地址](https://github.com/apache/incubator-weex)

### 工作原理

> Weex 是一个动态化的高扩展跨平台解决方案。 在 Weex 代码中，您可以使用 <template>，<style> 和 <script> 标签编写页面或组件，然后将它们转换为 JS bundle 以进行部署。当服务器返回给客户端 JS bundle 时，JS bundle 会被客户端的 JavaScript 引擎处理，并管理渲染 native 视图，调用原生 API 和用户交互。

> Weex 表面上是一个客户端技术，但实际上它串联起了从本地开发环境到云端部署和分发的整个链路。开发者首先可以在本地像撰写 web 页面一样撰写一个 app 的页面，然后编译成一段 JavaScript 代码，形成 Weex 的一个 JS bundle；在云端，开发者可以把生成的 JS bundle 部署上去，然后通过网络请求或预下发的方式传递到用户的移动应用客户端；在移动应用客户端里，WeexSDK 会准备好一个 JavaScript 引擎，并且在用户打开一个 Weex 页面时执行相应的 JS bundle，并在执行过程中产生各种命令发送到 native 端进行的界面渲染或数据存储、网络通信、调用设备功能、用户交互响应等移动应用的场景实践；同时，如果用户没有安装移动应用，他仍然可以在浏览器里打开一个相同的 web 页面，这个页面是使用相同的页面源代码，通过浏览器里的 JavaScript 引擎运行起来的。

![](http://7xqhx8.com1.z0.glb.clouddn.com/weex_1.jpg) 



