# HTML

## `<!DOCTYPE>`

[HTML中 !DOCTYPE 的解释与作用](https://blog.csdn.net/sunhl951/article/details/79763727)

DTD：Document Type Definition，中文翻译为：文档类型定义。DTD可定义合法的XML文档构建模块。它使用一系列合法的元素来定义文档的结构。因为早期的版本**基于SGML**，所以需要套用SGML的解析规则。DTD的作用在于定义SGML文档的文档类型以便于浏览器解析。

随着技术的进步，现在**HTML5 不基于 SGML，所以也就不需要引用 DTD了**

如果没有声明，那么不同的浏览器将会以自己不同的怪异的模式去解析渲染页面，这样页面在不同的浏览器上呈现出来的效果也就不一样，人们把这称之为“怪异模式”。

但是如果声明了，将会开启“严格模式”，又有人称之为“标准模式”，浏览器将已w3c标准来解析渲染页面。

## Accessibility

[W3School链接](https://www.w3schools.com/accessibility)

旨在帮助残障人士“浏览”网页

[可访问性标签](https://www.w3schools.com/accessibility/accessibility\_labels.php)

[role](https://www.w3schools.com/accessibility/accessibility\_role\_name\_value.php) 确保使用[辅助技术](https://www.w3schools.com/accessibility/accessibility\_keyboard\_assistive\_technology.php)的人能够使用它们。辅助技术的例子有屏幕阅读器、开关控制和语音识别软件

[combobox](https://weihongyu.com/wai-aria-%E6%97%A0%E9%9A%9C%E7%A2%8Dweb%E8%A7%84%E8%8C%83/)

## defer vs async

[异步加载脚本：Defer/Async](https://harttle.land/2016/05/18/async-javascript-loading.html#header-4)

[web性能优化基础篇之 async 与 defer 的区别](https://juejin.cn/post/7031113938532040740#heading-1)

[javaScript 是如何影响 DOM 生成的](https://blog.poetries.top/browser-working-principle/guide/part5/lesson22.html#javascript-%E6%98%AF%E5%A6%82%E4%BD%95%E5%BD%B1%E5%93%8D-dom-%E7%94%9F%E6%88%90%E7%9A%84)

## Document Fragment

[文档片段](https://juejin.cn/post/6952499015879507982)

与document相比，最大的区别是DocumentFragment 不是真实 DOM 树的一部分，它的变化不会触发 DOM 树的重新渲染，且不会导致性能等问题。

[createElement与createDocumentFragment的区别](https://juejin.cn/post/6981651859173802021)

`createDocumentFragment`是 `DOM` 节点。 它们不是主 `DOM` 树的一部分。通常的用例是创建文档片段，将元素附加到文档片段，然后将文档片段附加到DOM树。在DOM树中，文档片段被其所有的子元素所代替。

因为文档片段存在于内存中，并不在DOM树中，所以将子元素插入到文档片段时不会引起页面回流（对元素位置和几何上的计算）。因此，使用文档片段通常会带来更好的性能。

##
