# HTML

## link和@import的区别

[link vs @import](https://zhuanlan.zhihu.com/p/136047345)

**1. 从属关系区别**

@import是 CSS 提供的语法规则，只有导入样式表的作用；link是HTML提供的标签，不仅可以加载 CSS 文件，还可以定义 RSS、rel 连接属性等。

我的web前端学习交流群[**点击进入**](https://link.zhihu.com/?target=https%3A//jq.qq.com/%3F\_wv%3D1027%26k%3D5pIrfEw)**1045267283，欢迎加入！**

**2. 加载顺序区别**

加载页面时，link标签引入的 CSS 被同时加载；@import引入的 CSS 将在页面加载完毕后被加载。

**3. 兼容性区别**

@import是 CSS2.1 才有的语法，故只可在 IE5+ 才能识别；link标签作为 HTML 元素，不存在兼容性问题。

**4. DOM可控性区别**

可以通过 JS 操作 DOM ，插入link标签来改变样式；由于 DOM 方法是基于文档的，无法使用@import的方式插入样式。



## 浏览器及内核

* Google Chrome

  优点：不易崩溃，速度快，几乎隐身，搜索简单，标签简单，更加安全

  内核：统称为Chromium内核或Chrome内核，以前是Webkit内核，现在是Blink内核

* IE

  优点：部分只有IE内核才能打开所有网页;IE内核浏览器更安全;IE内核占用内存及CPU更少;IE最新版比chrome的速度快。

  缺点：拓展性几乎没有；容易中病毒，出现一些乱七八糟的东西，而且上网速度较慢

  内核：Trident内核，也是俗称的IE内核

* Firefox

  优点：风格简单，速度快，安全性高，拓展性强等。

  缺点：网页错位，媒体功能不强等。

  内核：Gecko内核，俗称Firefox内核

* Safari

  优点：界面也比较的简单

  缺点：与运行在macOS上的safari相比，有些功能出现丢失

  内核：Webkit内核。WebKit 是一个开源渲染引擎，最初是 Linux 平台的引擎，后来被 Apple 修改为支持 Mac 和 Windows。有关详细信息，请参阅[webkit.org](http://webkit.org/)。

* Opera

  内核：最初是自己的Presto内核，后来是Webkit，现在是Blink内核

Blink渲染引擎是开源引擎[WebKit](https://www.wikiwand.com/zh-hans/WebKit)中[WebCore](https://www.wikiwand.com/zh-hans/WebKit#WebCore)元件的一个分支



