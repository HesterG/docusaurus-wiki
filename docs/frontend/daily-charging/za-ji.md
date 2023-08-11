# 杂记

## 用hexo搭建博客时遇到的小问题

`<span>`标签里用到了`data-url`

```html
<span class="exturl cc-opacity" title="Deploy with Netlify → https://www.netlify.com" data-url="aHR0cHM6Ly93d3cubmV0bGlmeS5jb20="><img width="80" src="https://www.netlify.com/img/global/badges/netlify-dark.svg" alt="Netlify"></span>
```

好奇到底是什么，就去查了一下，发现时html5的一个特性:

[`data-<x>属性`](https://blog.csdn.net/hxdafei1989/article/details/72869216)

html5提供了一个data-\*这个属性，你可以写任意一个data-one,data-two,都可以，

然后js的dom就会读到你写的这个属性，拿到里面的值。

当然你可以自己通过JS自己追加属性。也可以改变里面的值。

```javascript
<div id="test" data-age="24">       Click Here    </div>
var test = document.getElementById('test');
test.dataset.my = 'Byron';
// 这种元素都会有dataset属性。这样就追加了一个data-my的属性（id=test），
// getAttribute()取值属性
console.log(test.getAttribute("data-age"));
// setAttribute()赋值属性
test.setAttribute("data-age","48");

// data-前缀属性可以在JS中通过dataset取值，更加方便
console.log(test.dataset.age);
// 赋值
tree.dataset.age = "30";
tree.dataset.age--;

tree.dataset.age++;
```

这里还encode成了base64，所以源代码里想在页面展示时需要用[`atob`](https://www.runoob.com/jsref/met-win-atob.html)转码

```javascript
const link = document.createElement('a');
// https://stackoverflow.com/questions/30106476/using-javascripts-atob-to-decode-base64-doesnt-properly-decode-utf-8-strings
// atob() 方法用于解码使用 base-64 编码的字符串。
// base-64 编码使用方法是 btoa() 。
link.href = decodeURIComponent(atob(element.dataset.url).split('').map(c => {
  return '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2);
}).join(''));
```

## 使用netlify给网站加速

[使用netlify自动部署hexo博客，写博客更加方便](https://blog.csdn.net/RayDon03/article/details/104446832)
