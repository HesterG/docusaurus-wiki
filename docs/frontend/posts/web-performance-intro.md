---
title: 前端性能优化初探
date: 2022-05-20T15:31:21.000Z
tags:
  - Frontend
categories:
  - - Tech
    - Frontend
  - - Tech
    - Optimization
---

# 前端性能优化初探

这篇文章将使用 Chrome开发者工具在“实验室”环境查看天刀直购系列活动页面的性能指标，结合入职以来做过的几个需求，总结一下学习到的页面性能优化的方式以及未来还可以采取的优化措施。

\# 性能指标测试环境和工具

性能指标一般通过在实验室中或在实际情况中两种情况来测量:

1. 实验室中：使用工具在稳定、受控的环境中模拟页面加载 实测工具(field tools)例子：
   * [Chrome 用户体验报告](https://developers.google.com/web/tools/chrome-user-experience-report)
   * [PageSpeed Insights 网页速度测量工具](https://pagespeed.web.dev/)
   * [搜索控制台（核心 Web 指标报告）](https://support.google.com/webmasters/answer/9205520)
   * [`web-vitals` JavaScript 库](https://github.com/GoogleChrome/web-vitals)
2. 在实际情况中：基于真实用户的实际页面加载与页面交互 实验室工具(lab tools)例子:
   * [Chrome 开发者工具](https://developer.chrome.com/docs/devtools/)
   * [灯塔](https://developers.google.com/web/tools/lighthouse/)
   * [WebPageTest 网页性能测试工具](https://webpagetest.org/)
   * 在[实测 Web 指标的最佳实践](https://web.dev/vitals-field-measurement-best-practices/)中，推荐使用[web-vitals js库](https://github.com/GoogleChrome/web-vitals#install-and-load-the-library),为了不影响测量官方建议延迟加载分析代码

## 使用 Chrome开发者工具分析如何进行页面性能优化

用于分析和测试优化策略的页面：

[天刀华为直购活动页面一](https://ty.qq.com/act/a20220408lottery/huawei/index.html)、[天刀华为直购活动页面二](https://ty.qq.com/act/a20220516lottery/test/index.html)

这里分为两个阶段：加载阶段和交互阶段，这里主要关注网络进程和渲染进程。

### 加载阶段

通过 `chrome` 的 `performance` 中的 `timing` 可以观察到比较重要的几个时间节点:

* `First Paint` ：Google定义的web指标(web vitals)之一，首次绘制像素的时间（白屏时间）。
* `First Contentful Paint` ：Google定义的web指标(web vitals)之一，测量页面从开始加载到页面内容的任何部分在屏幕上完成渲染的时间，如文本、图像、非空 `svg/canvas` 等（首屏时间）。
* `Largest Contentful Paint` : Google定义的核心web指标(core web vitals)之一，测量页面从开始加载到最大文本块或图像元素在屏幕上完成渲染的时间，测量加载性能。从 `Screenshots` 中看到这个节点是可以看到比较完整的页面的时候，因此从用户体验来说这个时间点会比较重要。
* `DomContentLoaded` 事件： `dom` 树构建完毕，但是还没有进入渲染树的构建。体验上页面可以正常交互了，但是图片、字体等资源可能还没完全加载好。(在 `main` 上，`DCL`之后就没有 `Parse HTML` 了)
* `Load` 事件：页面上所有资源都被加载完毕。( `js` 、 `css` 、图片，以及 `js` 异步加载的 `js` 、 `css` 、图片) ![undefined](https://user-images.githubusercontent.com/17645053/168962380-9043016c-ce9e-4c16-a968-c7a3ca92caf1.png)

如果不想使用`web-vitals`想自己做埋点，也可以使用performance Windows API来做简易的埋点。看一下`performance.timing`都有什么: ![Screen Shot 2022-05-20 at 3 15 38 PM](https://user-images.githubusercontent.com/17645053/169474053-167baa8c-ab55-4957-809e-bddbfdfad419.png) 根据它们可以计算出FP/DCL/Load等关键事件的时间点

```js
// 页面首次渲染时间(firstPaint)
let FP = domLoading - navigationStart
// DOM加载完成(DOMContentEventLoad)
let DCL = domContentLoadedEventEnd - navigationStart
// 图片、样式等外链资源加载完成 (Load)
let L = loadEventEnd - navigationStart
```

不过这里因为Google Web Vitals更多考虑了用户体验，所以也使用了Web Vitals的时间点来作为阶段的分割线。

#### beforeunload、发起请求准备阶段

在发起 `get index.html` 请求前，有一小段时间是在执行 `beforeunload` ，其实应该还包括了发起 `get index.html` 请求前的准备工作，如果未来需求里需要引入跨域文件，也许可以使用 `dns-prefetch` 来进行预解析。即预解析之后可能会用到的域名，并将结果缓存到系统缓存中，缩短 `DNS` 的查询时间，以此来提高访问速度。这里有个误解，之前以为 `dns-prefetch` 会影响首屏加载，但其实如果使用的是 `chrome` ， `V8` 是会利用空闲的时间去做预解析，并且它的预解析是在异步线程上执行的，可以避免影响首屏加载。不过 `dns-prefetch` 也不是设置得越多越好，虽然是异步线程，也还是占用设备的带宽，可能会造成资源竞争。 ![undefined](https://user-images.githubusercontent.com/17645053/168993038-8342cb8f-c81f-49d7-bf9d-8c32d9366d08.png)

#### 发起请求到FCP阶段

在 `First Contentful Paint` 之前，可以看到优先级高的关键资源会被首先请求，如 `index.html` 、 `index.css` 、 `index.js` 等。

![undefined](https://user-images.githubusercontent.com/17645053/168964607-d996963f-7b25-41cd-9e2f-aba74d82a95f.png)

而关键资源阻塞渲染的时长取决于最大的那个资源，因此猜测这个阶段可以采取利用 `webpack` 压缩文件、移除非必要的代码等优化方式。

于是实践了一下，使用`optimize-css-assets-webpack-plugin`压缩`css`、`HtmlWebpackPlugin`压缩`html`，以及`webpack`生产环境自动开启的js压缩工具测试了一下，结果发现虽然压缩后大小是有所减小（共计缩小了`14.9k`,为原体积的`30%`左右），线上`network`请求时长没什么改善，`FCP`和`LCP`也没有有效提前，查看`network`发现其实这里的速度主要还是取决于`TTFB`,以`get index.js`请求为例，大部分时间在`Waiting`,对于这个项目而言，因为本身文件大小不算大，`webpack`压缩后并没有起到有效的优化效果。

![Screen Shot 2022-05-20 at 1 30 46 PM](https://user-images.githubusercontent.com/17645053/169457346-fa6a9a2b-7f47-429a-8957-308d2cffb31a.png)

另外，浏览器为了更好的用户体验会先解析一部分 `html` ，所以才有了 `firstpaint` 在 `DomContentLoaded` 之前，也因此需要把 `js` 放到 `html` 的最下面，因为 `js` 会阻塞 `dom` 的解析， `css` 可以放到前面避免页面因为没有样式过于丑陋。于是觉得理论上也许可以尝试将一些不需要在解析 `html` 阶段使用的 `js` 标记上 `async` 或者 `defer` 。关于`aync`和`defer`的区别:

![img](https://harttle.land/assets/img/blog/acyn-vs-defer.jpg)

这里也自己实践了一下，给不依赖其他脚本的`js script`，如上报js加上了`aync`, 然后把不需要在解析阶段使用的`js script`加上了`defer`，但是结果发现并没有想象中地被优化。因为还是存在无法使用`defer`和`aync`的js文件，这些文件依然会阻塞渲染时间。而那些加了`defer`和`async`的文件其实本身也得益于`http/2`的`server push`，所以在这个需求里、在这个实验环境下加`defer/async`优化方式也不适用。

那么究竟要怎么提前`FCP`时间呢，这里介绍一下，[`FCP` ](https://web.dev/fcp/)是从开始加载到页面内容的任何部分在屏幕上完成渲染的时间，是 `Google` 的 `web vitals` 之一，目前[灯塔](https://github.com/GoogleChrome/lighthouse)、[PageSpeed Insights](https://pagespeed.web.dev/)、[Chrome 开发者工具](https://developer.chrome.com/docs/devtools/)、[搜索控制台](https://search.google.com/search-console/about)、[web.dev 测量工具](https://web.dev/measure/)和[Web 指标 Chrome 扩展程序](https://chrome.google.com/webstore/detail/web-vitals/ahfhijdlegdabablpippeagghigmibma)等工具都提供对他的支持。可以使用这些工具获取`FCP`和优化建议。对于缩短`FCP`, Google也给出了[一些建议](https://web.dev/fcp/#fcp-4)。还是要根据实际情况分析适用于自己项目的方式来进行优化。

查看一下 `web-vitals js` 库中[`getFCP`源码](https://github.com/GoogleChrome/web-vitals/blob/main/src/getFCP.ts)

#### FCP到LCP阶段

在这个阶段，占用时间比较多有请求图片、字体资源、优先级低的 `js` 文件。这里用的是 `http/2` ，允许同时通过单一的 `http/2` 连接发起多重的请求-响应消息，减缓了 `tcp` 竞争以及队头阻塞。之前有个需求用到了雪碧图，经过指点才了解到因为 `http/2` 的普及现在其实拆成多个小图反而会更快一些，就是因为 `http/2` 有多路复用的特性，不再需要通过拼接来减少队头阻塞，多个小图可以同时被请求会比较快。除此之外，也了解到有些针对 `http1.1` 的性能优化也不再适用了，如域名分片(因为不需要新开连接来解决头阻塞问题)和拼接 `js/css` (理由同雪碧图) 在实际开发过程中，蜘蛛会自动上传 `ossweb-img` 中的资源到 `CDN` , 因此在这一步起到了减少 `RTT` 时间，优化图片等资源的传输速度的效果。在过去的几个静态页面开发过程中，主要是也是针对这一阶段进行了优化，如压缩图片、压缩字体等。这里想聊一下 `font-spider` 。他的原理是将页面引用到字体从字体包中拿出来，合成一个新的字体包，这样就不用引入全部字体包中的字体了，实现了按需引入。这里没了解原理之前踩坑了，把一个项目中压缩过的字体包拿到另一个项目用，结果页面中有的字体没有转化过来。因为页面上的字是不一样的，需要按需重新生成新的压缩字体包才可以。 ![undefined](https://user-images.githubusercontent.com/17645053/168980701-0c2aa8f3-cd0e-4b82-9b3a-bace85d87577.png) `LCP` 是测量页面从开始加载到最大文本块或图像元素在屏幕上完成渲染的时间。被 `Google` 列为了 `core web vitals` 之一，支持它的工具与 `FCP` 的一致，对于缩短 `LCP` ，Google官方也给出了[优化建议](https://web.dev/optimize-lcp/)，还是需要针对自己的情况采取相应的优化措施。

查看一下 `web-vitals js` 库中[`getLCP` 源码](https://github.com/GoogleChrome/web-vitals/blob/main/src/getLCP.ts)

#### LCP到Load阶段

在这个阶段，通过查看 `main` 发现主要是执行了 `milo` 的 `alert("活动已经结束")` ，所以可以暂且忽略。但是也验证了异步加载的 `js` 会影响 `load` 事件的触发。 ![undefined](https://user-images.githubusercontent.com/17645053/168980353-1303ecb6-f426-4938-bf16-d486768119cc.png)

### 交互阶段

华为这个活动页面交互比较少，主要是点击按钮，触发购买等请求 ，由此可见组件化时尽量排除无用的依赖可以加快响应。例子： ![undefined](https://user-images.githubusercontent.com/17645053/168985752-709ef3a4-acf9-4e73-b3c0-8f53fdf350ed.png) 未来如果接触到较为复杂的交互功能，理论上应该尽量减少对`dom`的重排重绘，尽量利用好`css`合成动画。因为渲染流程包括构建`dom`树、计算样式、布局、分层、分层、绘制、栅格化、合成和显示等阶段。重排会触发整个渲染流程；重绘会触发绘制后面的一系列渲染流程；而合成就不会再重新触发整个流程或者绘制流程，效率会高很多。对`chrome`来说，合成是在合成线程上完成的，他不会影响到主线程。因此合理利用合成可以带来较好的用户体验并且提高效率（至少对`chrome`是这样）

## 开发体验优化

* 未来可以就`css`开发做一些优化，比如使用`less/sass`等；取名方面可以更加规范一些，比如使用 `BEM` ；要注意去除多余样式、优化结构。
* 可以多关注组件库并且参与开发，以此减少对于`milo`等库的依赖。
* 也许可以试着引入`webpack`对代码进行压缩。
* 另外，在开发过程中需要多注意兼容的问题，避免使用覆盖率低的`css`或`html`特性。

以上为至今自己在需求中遇到过的性能优化的一些心得体会，未来也希望可以继续学习挖掘更多前端页面性能和开发体验的优化方式。如果有写得不准确的也希望可以多多指正。

## 参考

* [前端性能监控-基础指标定义&计算](https://juejin.cn/post/6887108904766046216)
* [HTTP/2特性及其在实际应用中的表现](https://juejin.cn/post/6844903503983280141#heading-12)
* [分层和合成机制](https://time.geekbang.org/column/article/141842)
* [如何系统地优化页面](https://time.geekbang.org/column/article/143889)
* [DNS prefetch原理](https://serverless-action.com/fontend/fe-optimization/DNS-Prefetch.html)
* [web vitals](https://web.dev/learn-web-vitals/)
* [埋点sdk设计](https://juejin.cn/post/7085679511290773534#heading-3)
* [defer vs async](https://juejin.cn/post/7031113938532040740#heading-1)
