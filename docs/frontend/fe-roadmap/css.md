# CSS

## CSS单位

[ch](https://www.jianshu.com/p/1e9f39421b14)



## CSS字体

[字体格式指南：TTF、OTF、WOFF、EOT和SVG](https://www.jianshu.com/p/e554f914e024)



## CSS specificity

计算方式：相加

![Screen Shot 2022-03-14 at 3 50 52 PM](https://user-images.githubusercontent.com/17645053/158127893-654d7fd7-5415-47d0-9e83-0bc3ea6bc313.png)



## 清除浮动

[清除浮动的四种方式及其原理理解](https://juejin.cn/post/6844903504545316877)

* `clear`: 通俗理解就是，哪边不允许有浮动元素，`clear`就是对应方向的值，两边都不允许就是both。使用`clear`清除浮动的方法还是有适用范围的。

我们需要更加通用和可靠的方法：

1. 父元素结束标签之前插入清除浮动的块级元素
2. 通过伪元素清理浮动
3. BFC(父元素设置`overflow`不为`visible`)

[BFC布局规则](https://juejin.cn/post/7029622804739784717)

1. 内部的Box会在垂直方向，一个接一个地放置。
2. Box垂直方向的距离由margin决定。属于同一个BFC的两个相邻Box的margin会发生重叠
3. 每个元素的margin box的左边， 与包含块border box的左边相接触(对于从左往右的格式化，否则相反)。即使存在浮动也是如此。
4. BFC的区域不会与float box重叠。
5. BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。反之也如此。
6. 计算BFC的高度时，浮动元素也参与计算



## 盒模型

```css
/*默认为conten-box， width/height + padding + border = actual width/height of an element*/
box-sizing:conent-box; 
/*width/height 包括了border和padding*/
box-sizing:border-box;
```



## CSS Masking

![image](https://user-images.githubusercontent.com/17645053/217246015-bd3cb32a-6992-4ade-a9c4-c1e2781e8ed2.png)

```css
.mask1 {
  -webkit-mask-image: url(w3logo.png);
  mask-image: url(w3logo.png);  /*mask-image为漏出来的图片部分*/
  -webkit-mask-repeat: no-repeat;
  mask-repeat: no-repeat;    
}
```



## `animation-fill-mode`

`none` - Default value. Animation will not apply any styles to the element before or after it is executing

`forwards` - The element will retain the style values that is set by the last keyframe (depends on animation-direction and animation-iteration-count) 动画运行完后保持住最后一桢

`backwards` - The element will get the style values that is set by the first keyframe (depends on animation-direction), and retain this during the animation-delay period 在`delay`期间将style设置为第一帧(`from`里的样式)

`both` - The animation will follow the rules for both forwards and backwards, extending the animation properties in both directions 同时应用前两项



## Flex

[参考: flex浏览器支持](https://www.runoob.com/cssref/css3-pr-flex.html)

**flex-container属性**

[justify-content](https://css-tricks.com/almanac/properties/j/justify-content/)

* `space-between`: 第一个在start-line 最后一个在end-line，其余平均分配
* `space-around`: item间的空间平均分配，第一个不用在start-line 最后一个不用在end-line
* `space-evenly`: item间以及第一个item前和最后一个item后空间都平均分配

`justify-content` vs `align-item`

* `justify-content`设置主轴上的对齐方式，`align-item`设置交叉轴上的对齐方式

[align-items vs align-content](https://juejin.cn/post/6844903911690600456)

* 主要区别体现在多行时， `align-item`设置`flex-item`在每一行的对齐方式；`align-content`在多行时会将`flex-item`们作为一个整体。(需要指定container的高度)

**flex-item属性**

`flex-basis` 属性用于设置或检索弹性盒伸缩基准值

`flex-grow`是一个比例，决定了和其他`flex-item`所占空间的比例

`flex-shrink`具体计算方式：[CSS flex-shrink](https://www.runoob.com/cssref/css3-pr-flex-shrink.html)

`align-self`The `align-self` property overrides the default alignment set by the container's `align-items` property.



## Grid

`grid-area`: grid-row-start / grid-column-start / grid-row-end / grid-column-end

e.g. grid-area: 2 / 1 / span 2 / span 3; grid-area: 1 / 2 / 5 / 6;

[fr是什么](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS\_Grid\_Layout/Basic\_Concepts\_of\_Grid\_Layout#fr\_%E5%8D%95%E4%BD%8D)

[auto-fill vs auto-fit](https://segmentfault.com/a/1190000040116599)

[minmax](https://www.w3cplus.com/css3/how-the-minmax-function-works.html)

[minmax一个应用](https://www.zhangxinxu.com/wordpress/2019/11/css-grid-minmax/)

```css
.container {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
}
```



## CSS Responsive

@media应用之一: [针对不同媒体](https://web.dev/learn/design/media-queries/#target-different-types-of-output)

[国际化](https://web.dev/learn/design/internationalization/) 有的语言从右到左读，所以使用类似`margin-inline-end`代替`margin-right`;`margin-block-end`代替`margin-bottom`, i.e., 使用[logical properties](https://web.dev/learn/design/internationalization/#logical-properties)

[HTML & CSS – dir, direction, writing-mode, ltr (left to rigth), rtl (right to left)](https://www.cnblogs.com/keatkeat/p/15967175.html)

[`clamp()`](https://developer.mozilla.org/en-US/docs/Web/CSS/clamp) 

[variable fonts](https://juejin.cn/post/6844903728772808711#heading-11)

[responsive images](https://web.dev/learn/design/responsive-images/)、[阮一峰-响应式图像教程](http://www.ruanyifeng.com/blog/2019/06/responsive-images.html)

```html
<img
 src="small-image.png"
 alt="A description of the image."
 width="300"
 height="200"
 loading="lazy"
 decoding="async"
 srcset="small-image.png 300w,
  medium-image.png 600w,
  large-image.png 1200w"
>
```

[优化滚动的小技巧：使用CSS Scroll Snap](https://www.php.cn/css-tutorial-468774.html)



## CSS methodologies

[科普](https://www.webfx.com/blog/web-design/css-methodologies/)

[BEM](https://en.bem.info/)

[SMACSS](https://smacss-zh.vercel.app/core/3-%E5%AF%B9CSS%E8%A7%84%E5%88%99%E8%BF%9B%E8%A1%8C%E5%88%86%E7%B1%BB.html#%E5%91%BD%E5%90%8D%E8%A7%84%E5%88%99)

[BEM、SMACSS、OOCSS — CSS 三種常見命名原則](https://medium.com/@k2307874/css-%E5%91%BD%E5%90%8D%E5%8E%9F%E5%89%87-bem-smacss-oocss-84e843a263d1)



## CSS transition

[教程链接](https://doc.houdunren.com/%E7%B3%BB%E7%BB%9F%E8%AF%BE%E7%A8%8B/css/12%20%E5%8F%98%E5%BD%A2%E5%8A%A8%E7%94%BB.html#%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86)

[transition视频教程](https://www.bilibili.com/video/BV1QJ411g7Uh?spm\_id\_from=333.337.search-card.all.click)

[`transitionend`](https://doc.houdunren.com/%E7%B3%BB%E7%BB%9F%E8%AF%BE%E7%A8%8B/css/13%20%E8%BF%87%E6%B8%A1%E5%BB%B6%E8%BF%9F.html)可以用来监听transition是否已经完成

[`transition-duration`](https://doc.houdunren.com/%E7%B3%BB%E7%BB%9F%E8%AF%BE%E7%A8%8B/css/13%20%E8%BF%87%E6%B8%A1%E5%BB%B6%E8%BF%9F.html#transition-duration)可以设置多个值，与`transition-property`配合

[`transition-property`](https://doc.houdunren.com/%E7%B3%BB%E7%BB%9F%E8%AF%BE%E7%A8%8B/css/13%20%E8%BF%87%E6%B8%A1%E5%BB%B6%E8%BF%9F.html#transition-property)可以设置多个或者`all`、`none`

[`transition-timing-function`](https://doc.houdunren.com/%E7%B3%BB%E7%BB%9F%E8%AF%BE%E7%A8%8B/css/13%20%E8%BF%87%E6%B8%A1%E5%BB%B6%E8%BF%9F.html#transition-timing-function)

* 用于设置过渡效果的速度, 可在 https://cubic-bezier.com网站在线体验效果差异;
* [步进形态](https://doc.houdunren.com/css/13%20%E8%BF%87%E6%B8%A1%E5%BB%B6%E8%BF%9F.html#%E6%AD%A5%E8%BF%9B%E5%BD%A2%E6%80%81)，可以一步步走: [时钟效果](https://doc.houdunren.com/css/13%20%E8%BF%87%E6%B8%A1%E5%BB%B6%E8%BF%9F.html#%E6%97%B6%E9%92%9F%E6%95%88%E6%9E%9C);
* [`transform-origin`](https://doc.houdunren.com/%E7%B3%BB%E7%BB%9F%E8%AF%BE%E7%A8%8B/css/12%20%E5%8F%98%E5%BD%A2%E5%8A%A8%E7%94%BB.html#%E5%8F%98%E5%BD%A2%E5%9F%BA%E7%82%B9);

[`transition-delay`](https://doc.houdunren.com/%E7%B3%BB%E7%BB%9F%E8%AF%BE%E7%A8%8B/css/13%20%E8%BF%87%E6%B8%A1%E5%BB%B6%E8%BF%9F.html#transition-delay)也可以设置多个值

[`transition`](https://doc.houdunren.com/%E7%B3%BB%E7%BB%9F%E8%AF%BE%E7%A8%8B/css/13%20%E8%BF%87%E6%B8%A1%E5%BB%B6%E8%BF%9F.html#transition)将过渡规则统一设置

* 必须设置过渡时间
* 延迟时间放在逗号或结束前



## CSS animation

[使用多个动画](https://doc.houdunren.com/%E7%B3%BB%E7%BB%9F%E8%AF%BE%E7%A8%8B/css/14%20%E5%B8%A7%E5%8A%A8%E7%94%BB.html#%E4%BD%BF%E7%94%A8%E5%8A%A8%E7%94%BB)

* 可以在元素身上同时使用多个动画。
* 多个动画有相同属性时，**后面动画的属性优先使用**
* 尽量不要在两个动画中控制相同的属性，因为不同浏览器的对待方式略有不同

[中间值](https://doc.houdunren.com/%E7%B3%BB%E7%BB%9F%E8%AF%BE%E7%A8%8B/css/14%20%E5%B8%A7%E5%8A%A8%E7%94%BB.html#%E4%B8%AD%E9%97%B4%E5%80%BC)

* 如果没办法计算出中间值就不会有动画效果

[`animation-iteration-count`](https://doc.houdunren.com/%E7%B3%BB%E7%BB%9F%E8%AF%BE%E7%A8%8B/css/14%20%E5%B8%A7%E5%8A%A8%E7%94%BB.html#%E9%87%8D%E5%A4%8D%E5%8A%A8%E7%94%BB)

* 可同时设置元素的多个动画重复，使用逗号分隔
* 如果动画数量大于重复数量定义，后面的动画将重新计算重复

动画库：

* [swiper](https://swiperjs.com/swiper-api)
* [animate.css](https://animate.style/)

贝塞尔曲线：

* 需要设置四个值 `cubic-bezier(<x1>, <y1>, <x2>, <y2>)`，来控制曲线速度，可在 [cubic-bezier](https://cubic-bezier.com/)网站在线体验效果。

[步进动画](https://doc.houdunren.com/%E7%B3%BB%E7%BB%9F%E8%AF%BE%E7%A8%8B/css/14%20%E5%B8%A7%E5%8A%A8%E7%94%BB.html#%E6%AD%A5%E8%BF%9B%E9%80%9F%E5%BA%A6)

* `steps(n,start)` 可以简单理解为从第二个开始，`steps(n,end)` 从第一个开始。
* `step-start` 效果等于 `steps(1,start)` ,`step-end` 效果等同于 `steps(1,end)`。

[`animation-play-state`](https://doc.houdunren.com/%E7%B3%BB%E7%BB%9F%E8%AF%BE%E7%A8%8B/css/14%20%E5%B8%A7%E5%8A%A8%E7%94%BB.html#%E6%92%AD%E6%94%BE%E7%8A%B6%E6%80%81)

* 使用 `animation-play-state` 可以控制动画的暂停与运行。
* [幻灯片效果](https://doc.houdunren.com/css/14%20%E5%B8%A7%E5%8A%A8%E7%94%BB.html#%E5%B9%BB%E7%81%AF%E7%89%87)

[`animation-fill-mode`](https://doc.houdunren.com/%E7%B3%BB%E7%BB%9F%E8%AF%BE%E7%A8%8B/css/14%20%E5%B8%A7%E5%8A%A8%E7%94%BB.html#%E5%A1%AB%E5%85%85%E6%A8%A1%E5%BC%8F)用于定义动画播放结束后的处理模式，是回到原来状态还是停止在动画结束状态。

| 选项      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| none      | 需要等延迟结束，起始帧属性才应用                             |
| backwards | 动画效果在起始帧，不等延迟结束                               |
| forwards  | 结束后停留动画的最后一帧                                     |
| both      | 包含backwards与forwards规则，即动画效果在起始帧，不等延迟结束，并且在结束后停止在最后一帧 |

[animation组合定义](https://doc.houdunren.com/%E7%B3%BB%E7%BB%9F%E8%AF%BE%E7%A8%8B/css/14%20%E5%B8%A7%E5%8A%A8%E7%94%BB.html#%E7%BB%84%E5%90%88%E5%AE%9A%E4%B9%89)



参考：

[animation视频教程](https://www.bilibili.com/video/BV1uJ411T7NN?p=2)

[教程文档](https://doc.houdunren.com/%E7%B3%BB%E7%BB%9F%E8%AF%BE%E7%A8%8B/css/14%20%E5%B8%A7%E5%8A%A8%E7%94%BB.html)





## CSS in JS

[CSS in JS](https://www.ruanyifeng.com/blog/2017/04/css\_in\_js.html)

[CSS in JS的好与坏](https://zhuanlan.zhihu.com/p/103522819)



## 学习链接：[CSS Tutorial](https://www.w3schools.com/css/default.asp)、[Learn CSS](https://web.dev/learn/css/)、[CSS响应式](https://web.dev/learn/design/)

