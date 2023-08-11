# css

## BFC

* [什么是BFC](https://juejin.cn/post/7005791554170257415)
* [BFC(边距重叠解决方案)](https://juejin.cn/post/7029622804739784717)
* [CSS外边距坍塌的三种情况](https://blog.csdn.net/qq\_27127385/article/details/103850561)
* Block Formatting Context
* 块格式化上下文是页面上的**一个独立的渲染区域**，容器里面的子元素**不会在布局上影响外面的元素**。
* 它是决定块盒子的布局及浮动元素相互影响的一个因素。

### css在什么情况会创建一个块格式化上下文

* 根元素`<html>`
* 浮动元素：float不是none
* 定位元素：position是absolute或fixed
* overflow不是visible
* display为inline-block或flex
* display为table、table-cell等HTML表格相关属性

### BFC的作用

**浮动高度坍塌**

*   [浮动高度塌陷](https://juejin.cn/post/7005791554170257415#heading-2)

    通过给父元素添加`overflow: hidden;`(只要不是visible都可以)属性来触发BFC，形成独立的一块区域

    ```javascript
    .wrapper {
      border: 1px solid red;
      overflow: hidden;
    }
    // 这种现象的原因是子元素为浮动定位，元素浮动了起来，导致子元素脱离了文档流，造成父元素wrapper的高度塌陷
    .box1 {
      width: 200px;
      height: 200px;
      background: blue;
      float: left;
    }
    ```

    这样一来，计算BFC的高度的时候，会将浮动元素的高度也计算在内，这个时候我们可以看到BFC把box1包裹了起来

[**解决外边距坍塌**](https://blog.csdn.net/qq\_27127385/article/details/103850561)

*   父子元素/margin越界

    这种情况也叫做子元素越界或者margin越界（第一个子元素的margin-top和最后一个子元素的margin-bottom的越界问题）： **如果块级父元素中，不存在border, padding**（当上border及上padding宽度为0时）, **行内元素以及清除浮动这四条属性**，那么这个块级元素和其第一个子元素的上外边距（margin-top）就可以说”挨到了一起“。此时这个**块级父元素和其第一个子元素**就会发生**上外边距**塌陷现象。

    类似地，若块级**父元素的下外边距**（margin-bottom）与它的**最后一个子元素**的下外边距之间没有父元素的border, padding, 行内元素，height, min-height, max-height分隔时，就会发生**下外边距**塌陷现象。

    方法一：为父元素设置上边框，并将父元素高度减少

    方法二：父元素增加上内边距，并减少父元素高度

    方法三：父元素增加属性 overflow: hidden，即触发 BFC
*   同级相邻

    同级相邻元素之间的外边距会塌陷，塌陷后的间距等于两个元素外边距的较大值（如果后者被清除浮动，不遵循此规则）:

    两个相邻的外边距都是正数时，折叠结果是它们两者之间较大的值。

    两个相邻的外边距都是负数时，折叠结果是两者绝对值的较大值。

    两个外边距一正一负时，折叠结果是两者的相加的和。

    解决方法之触发 BFC：

    ```html
    <div class='div1'></div>

    <div style="overflow:hidden">
    	<div class='div2'></div>
    </div>
    ```
* 空块元素

### [BFC布局规则](https://juejin.cn/post/7029622804739784717)

1. 内部的Box会在垂直方向，一个接一个地放置。
2. Box垂直方向的距离由margin决定。属于同一个BFC的两个相邻Box的margin会发生重叠
3. 每个元素的margin box的左边， 与包含块border box的左边相接触(对于从左往右的格式化，否则相反)。即使存在浮动也是如此。
4. BFC的区域不会与float box重叠。
5. BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。反之也如此。
6. 计算BFC的高度时，浮动元素也参与计算

## 清除浮动

[清除浮动的四种方式及其原理理解](https://juejin.cn/post/6844903504545316877)

## DPR

[像素和DPR](https://www.cnblogs.com/xiaohuochai/p/5494624.html)

[DPR 与 viewport](https://zhuanlan.zhihu.com/p/26131956)

像素分为两种：设备像素和CSS像素

1. 设备像素(device independent pixels): 设备屏幕的物理像素，任何设备的物理像素的数量都是固定的
2. CSS像素(CSS pixels): 又称为逻辑像素，是为web开发者创造的，在CSS和javascript中使用的一个抽象的层

例子:

MacBook Pro 13.3 英寸的显示器分辨率是 2560 x 1600，这个 2560px 就是我们前面说的设备上的物理像素值，而浏览器全屏显示的宽度只有 1280px，这个就是 dips 值，最终可以知道这台 MacBook Pro 电脑屏幕的 DPR 为 2，DPR 在这里所表达的意思就是：1280 dips 在实际显示的时候，被硬件扩展到了 2560 的硬件像素宽度，**2 个物理像素对应 1 个 CSS 像素（这个指的水平方向或垂直方向，如果在一个平面内的话 4 个物理像素点对应 1 个 CSS 像素点）**。

如果现在上面这台电脑里有一张实际宽度为 200px 的高清图片，在浏览器里被 css 设置宽度为 200px，那么这张图片看起来就会有点模糊，因为它实际被硬件扩展到了 400px 的硬件像素宽度，是它实际宽度的两倍。但是，如果它被 css 设置宽度为 100px，这时候它实际被硬件扩展到了 200px 的硬件像素宽度，和它实际像素一致，就不会模糊了。

## 1px问题

[1px问题原理](https://segmentfault.com/a/1190000037790305#item-1)

在讲原理之前，先跟大家说一个概念，就是设备像素比DPR(devicePixelRatio)是什么

> DPR = 设备像素 / CSS像素(某一方向上)

这句话看起来很难理解，可以结合下面这张图（1px在各个DPR上面的展示），一般我们h5拿到的设计稿都是750px的，但是如果在DPR为2的屏幕上，手机的最小像素却是要用2 \* 2px来进行绘制，这也就导致了为什么1px会比较粗了。

![25](https://user-images.githubusercontent.com/17645053/215938916-b11e4375-aae2-46c9-a059-ebdeed59dec6.png)

[移动端1px问题解决方案](https://juejin.cn/post/6959736170158751780)

1px 问题的解决方案是其实非常多的。不过从实用的角度出发，建议大家掌握3～4种就可以了，其他方法了解一下就行。

| 方案               | 优点         | 缺点                |
| ---------------- | ---------- | ----------------- |
| 直接写 0.5px        | 代码简单       | IOS及Android老设备不支持 |
| 用图片代替边框          | 全机型兼容      | 修改颜色及不支持圆角        |
| background渐变     | 全机型兼容      | 代码多及不支持圆角         |
| box-shadow模拟边框实现 | 全机型兼容      | 有边框和虚影无法实现        |
| 伪元素先放大后缩小        | 简单实用       | 缺点不明显             |
| 设置viewport解决问题   | 一套代码适用所有页面 | 缺点不明显             |

## &#x20;用CSS创建三角形

[css三角形](https://segmentfault.com/a/1190000039319618)

一个向下的三角形：

![image](https://segmentfault.com/img/bVcO8F8)

```css
.triangle {
    width: 0;
    height: 0;
    border-top: 100px solid #000;
    border-right: 100px solid transparent;
    border-left: 100px solid transparent;
    border-bottom: 100px solid transparent;
}
```

这个三角形的宽正好是左右边框的总和，也就是200px,而它的高当然就是上边框的宽度了，也就是100px。 以此类推，如果设计稿给的是一个长50px高60px的一个向上的三角形，那么就应该这样写：

```css
.triangle {
    width: 0;
    height: 0;
    border-top: 60px solid #000;
    border-right: 25px solid transparent;
    border-left: 25px solid transparent;
}
```

## display:none vs visibility:hidden

[display:none和visibility:hidden的区别](https://zhuanlan.zhihu.com/p/37221519)

1.  是否占据空间

    display:none，该元素不占据任何空间，在文档渲染时，该元素如同不存在（但依然存在文档对象模型树中）。

    visibility:hidden，该元素空间依旧存在。

    即一个（display:none）不会在渲染树中出现，一个（visibility :hidden）会。
2.  是否渲染

    display:none，会触发reflow（回流），进行渲染。

    visibility:hidden，只会触发repaint（重绘），因为没有发现位置变化，不进行渲染。
3.  是否是继承属性

    display:none，display不是继承属性，元素及其子元素都会消失。

    visibility:hidden，visibility是继承属性，若子元素使用了visibility:visible，则不继承，这个子孙元素又会显现出来。
4.  读屏器是否读取

    读屏器不会读取display：none的元素内容，而会读取visibility：hidden的元素内容。

### [回流和重绘](https://www.cnblogs.com/songyao666/p/15891383.html#%E4%BB%80%E4%B9%88%E6%98%AF%E5%9B%9E%E6%B5%81reflow%E4%B8%8E%E9%87%8D%E7%BB%98repaint)

**回流（Reflow）**

当渲染树`render tree`中的一部分(或全部)因为元素的规模尺寸，布局，隐藏等改变而需要重新构建。这就称为**回流(reflow)**。每个页面至少需要一次回流，就是在页面第一次加载的时候，这时候是一定会发生回流的，因为要构建`render tree`。在回流的时候，浏览器会使渲染树中受到影响的部分失效，并重新构造这部分渲染树，完成回流后，浏览器会重新绘制受影响的部分到屏幕中，该过程称为**重绘**。

**简单来说，回流就是计算元素在设备内的确切位置和大小并且重新绘制回流的代价要远大于重绘。并且回流必然会造成重绘，但重绘不一定会造成回流。**

**重绘（Repaint）**

当渲染树`render tree`中的一些元素需要更新样式，但这些样式属性只是改变元素的外观，风格，而不会影响布局的，比如`background-color`。则就叫称为**重绘(repaint)**。

**简单来说，重绘就是将渲染树节点转换为屏幕上的实际像素，不涉及重新布局阶段的位置与大小计算**

## block，inline和inline-block

[block，inline和inline-block概念和区别](https://developer.aliyun.com/article/338942)

1. block和inline这两个概念是简略的说法，完整确切的说应该是 block-level elements (块级元素) 和 inline elements (内联元素)。block元素通常被现实为独立的一块，会单独换一行；inline元素则前后不会产生换行，一系列inline元素都在一行内显示，直到该行排满。
2. 大体来说HTML元素各有其自身的布局级别（block元素还是inline元素）：
   * 常见的块级元素有 DIV, FORM, TABLE, P, PRE, H1\~H6, DL, OL, UL 等。
   * 常见的内联元素有 SPAN, A, STRONG, EM, LABEL, INPUT, SELECT, TEXTAREA, IMG, BR 等。
3. block元素可以包含block元素和inline元素；但inline元素只能包含inline元素。要注意的是这个是个大概的说法，每个特定的元素能包含的元素也是特定的，所以具体到个别元素上，这条规律是不适用的。比如 P 元素，只能包含inline元素，而不能包含block元素。
4. 一般来说，可以通过display:inline和display:block的设置，改变元素的布局级别。

### display:block

1. block元素会独占一行，多个block元素会各自新起一行。默认情况下，block元素宽度自动填满其父元素宽度。
2. block元素可以设置width,height属性。块级元素即使设置了宽度,仍然是独占一行。
3. block元素可以设置margin和padding属性。

### display:inline

1. inline元素不会独占一行，多个相邻的行内元素会排列在同一行里，直到一行排列不下，才会新换一行，其宽度随元素的内容而变化。
2. inline元素设置width,height属性无效。
3. inline元素的margin和padding属性，水平方向的padding-left, padding-right, margin-left, margin-right都产生边距效果；但竖直方向的padding-top, padding-bottom, margin-top, margin-bottom不会产生边距效果。

### display:inline-block

1. 简单来说就是将对象呈现为inline对象，但是对象的内容作为block对象呈现。之后的内联对象会被排列在同一行内。比如我们可以给一个link（a元素）inline-block属性值，使其既具有block的宽度高度特性又具有inline的同行特性。

[三者对比例子](https://codepen.io/huijing/pen/PNMxXL/)

## px、em、rem、%、vw、vh、vm

[px、em、rem、%、vw、vh、vm这些单位的区别](https://www.jianshu.com/p/82f02af17e78)

**rem**

Font size of the root element

css3新单位，相对于根元素html（网页）的font-size，不会像em那样，依赖于父元素的字体大小，而造成混乱。

**%**

一般宽泛的讲是相对于父元素，但是并不是十分准确。

1、对于普通定位元素就是我们理解的父元素

2、对于position: absolute;的元素是相对于已定位的父元素

3、对于position: fixed;的元素是相对于 ViewPort（可视窗口）

**vw**

css3新单位，viewpoint width的缩写，视窗宽度，1vw等于视窗宽度的1%。

举个例子：浏览器宽度1200px, 1 vw = 1200px/100 = 12 px。

## rem

Font size of the root element

```js
function setFont() {
	var cliWidth = html.clientWidth;
  // root element的font是乘以100以后的，所以如果想要11px,就是0.11rem
  // 假如一个图的宽是750px, 那它的宽就是7.5rem
	html.style.fontSize = 100 * (cliWidth / 750) + "px";
}
```

## CSS specificity

计算方式：相加

优先级关系：内联样式 > ID 选择器 > 类选择器 = 属性选择器 = 伪类选择器 > 标签选择器 = 伪元素选择器

![Screen Shot 2022-03-14 at 3 50 52 PM](https://user-images.githubusercontent.com/17645053/158127893-654d7fd7-5415-47d0-9e83-0bc3ea6bc313.png)

## 英文换行

[word-break 和 word-wrap 的区别](https://blog.csdn.net/Bule\_daze/article/details/103837307)

`word-wrap`：决定**句尾**放不下单词时，单词是否换行。

* `normal`：只在允许的断字点换行（浏览器保持默认处理）。
* `break-word`：在长单词或 URL 地址内部进行换行。

`word-break`：决定**单词内**该怎么换行。

* `normal`：使用浏览器默认的换行规则。
* `break-all`：允许在单词内换行。
* `keep-all`：只能在半角空格或连字符处换行。

平文本可以配合 white-space: pre-wrap 来解决多空格压缩显示问题。 富文本采用的解决方案是对空格进行间隔 html 转义，这种方法更灵活，可以适应不同的场景，也适用于平文本。
