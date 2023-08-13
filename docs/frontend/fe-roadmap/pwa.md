# PWA

## PWA应用实践

[PWA 应用实战](https://lavas-project.github.io/pwa-book/)


## Service Worker

**[Service Worker简介](https://x5.tencent.com/product/service-worker.html)**

Service Worker从英文翻译过来就是一个服务工人，服务于前端页面的后台线程，基于Web Worker实现。有着独立的js运行环境，分担、协助前端页面完成前端开发者分配的需要在后台悄悄执行的任务。基于它可以实现拦截和处理网络请求、消息推送、静默更新、事件同步等服务。

1. Service Worker线程运行的是js，有着独立的js环境，不能直接操作DOM树，但可以通过postMessage与其服务的前端页面通信。
2. Service Worker服务的不是单个页面，它服务的是当前网络path下所有的页面，只要当前path 的Service Worker被安装，用户访问当前path下的任意页面均会启动该Service Worker。当一段时间没有事件过来，浏览器会自动停止Service Worker来节约资源，所以Service Worker线程中不能保存需要持久化的信息。
3. Service Worker安装是在后台悄悄执行，更新也是如此。每次新唤起Service Worker线程，它都会去检查Service Worker脚本是否有更新，如有一个字节的变化，它都会新起一个Service Worker线程类似于安装一样去安装新的Service Worker脚本，当旧的Service Worker所服务的页面都关闭后，新的Service Worker便会生效。

**[Servie Worker应用与实践](https://mp.weixin.qq.com/s/3Ep5pJULvP7WHJvVJNDV-g)**

**生命周期**
![image](https://user-images.githubusercontent.com/17645053/180904436-d66241b7-1937-4919-b6bd-579ddbcd88a4.png)