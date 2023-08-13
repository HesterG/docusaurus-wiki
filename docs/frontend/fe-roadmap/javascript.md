# Javascript

## 概念

### 冒泡捕获

[原文链接](https://ost.51cto.com/posts/3825)

使用`addEventListener()` 方法，您可以使用“useCapture”参数指定传播类型：

```javascript
addEventListener(event, function, useCapture);
```

默认是假的，将使用冒泡传播，当为真时，活动使用设置捕获传播。

### prototype

[prototype练习代码](https://github.com/HesterG/js-advanced-study/blob/main/01-prototype/index.js)

[JS 原型与原型链](https://juejin.cn/post/6844903782229213197)

### 作用域、闭包

[讲清楚之javascript作用域](https://segmentfault.com/a/1190000014980841)

[JavaScript闭包 - Web前端工程师面试题讲解](https://www.bilibili.com/video/BV1iE411q7Qd/?spm\_id\_from=333.788.recommend\_more\_video.0); [Learn Closures In 7 Minutes](https://www.youtube.com/watch?v=3a0I8ICR1Vg\&ab\_channel=WebDevSimplified); [学习Javascript闭包（Closure）](https://www.ruanyifeng.com/blog/2009/08/learning\_javascript\_closures.html); [廖雪峰 闭包](https://www.liaoxuefeng.com/wiki/1022910821149312/1023021250770016)

[闭包在内存中，以及如何回收内存](https://time.geekbang.org/column/article/127495)

JavaScript 引擎会沿着“当前执行上下文–>foo 函数闭包–> 全局执行上下文”的顺序来查找 myName 变量

### this

[this](https://www.bilibili.com/video/BV1BE411677T?from=search\&seid=5670659888759078442\&spm\_id\_from=333.337.0.0)

### strict mode

[阮一峰strict mode](https://www.ruanyifeng.com/blog/2013/01/javascript_strict_mode.html)

### 防抖、节流

[函数防抖和节流](https://segmentfault.com/a/1190000018445196)

### 深拷贝vs浅拷贝

[js深拷贝和浅拷贝及其实现方式](https://segmentfault.com/a/1190000039310119)

[javascript 数组以及对象的深拷贝（复制数组或复制对象）的方法](https://blog.csdn.net/FungLeo/article/details/54931379)

[JSON.stringify深拷贝的缺点](https://www.jianshu.com/p/52db1d0c1780)

1. 如果obj里面有时间对象，则`JSON.stringify`后再`JSON.parse`的结果，时间将只是字符串的形式，而不是对象的形式
2. 如果obj里有RegExp(正则表达式的缩写)、Error对象，则序列化的结果将只得到空对象；
3. **如果obj里有函数，undefined，则序列化的结果会把函数或 undefined丢失；**
4. **如果obj里有NaN、Infinity和-Infinity，则序列化的结果会变成null**
5. JSON.stringify()只能序列化对象的可枚举的自有属性，例如 如果obj中的对象是有构造函数生成的， 则使用JSON.parse(JSON.stringify(obj))深拷贝后，会丢弃对象的constructor；
6. 如果对象中存在循环引用的情况也无法正确实现深拷贝；

React里可以使用`immutable.js`



## js函数

### js reduce常用

[js reduce常用](https://blog.csdn.net/a715167986/article/details/118599974)

### call vs bind vs apply

[call vs bind vs apply](https://zhuanlan.zhihu.com/p/82340026)

### 正则

[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular\_Expressions)

[\1\2和\1](https://blog.csdn.net/liangf05/article/details/79361191)



## ES6+

### String新增

[原文链接](https://es6.ruanyifeng.com/?search=%E7%AE%AD%E5%A4%B4\&x=0\&y=0#docs/string-methods#String-raw)

`Sring.raw`返回个斜杠都被转义（即斜杠前面再加一个斜杠）的字符串，往往用于模板字符串的处理方法。

### 箭头函数

[箭头函数](https://www.liaoxuefeng.com/wiki/1022910821149312/1031549578462080)

### 对象的扩展

[对象的扩展](https://es6.ruanyifeng.com/?search=%E7%AE%AD%E5%A4%B4\&x=0\&y=0#docs/object)

结构赋值必须是最后一个参数，否则会报错

### 运算符扩展

[运算符扩展](https://es6.ruanyifeng.com/?search=%E7%AE%AD%E5%A4%B4\&x=0\&y=0#docs/operator)

[ES2020](https://github.com/tc39/proposal-optional-chaining) 引入了“链判断运算符”（optional chaining operator）`?.`

```javascript
const firstName = message?.body?.user?.firstName || 'default';
const fooValue = myForm.querySelector('input[name=foo]')?.value
```

上面代码使用了`?.`运算符，直接在链式调用的时候判断，左侧的对象是否为`null`或`undefined`。如果是的，就不再往下运算，而是返回`undefined`

常见场景：

```javascript
a?.b
// 等同于
a == null ? undefined : a.b

a?.[x]
// 等同于
a == null ? undefined : a[x]

a?.b()
// 等同于
a == null ? undefined : a.b()

a?.()
// 等同于
a == null ? undefined : a()
```

[ES2020](https://github.com/tc39/proposal-nullish-coalescing) 引入了一个新的 Null 判断运算符`??`。它的行为类似`||`，但是只有运算符左侧的值为`null`或`undefined`时，才会返回右侧的值。

```javascript
const headerText = response.settings.headerText ?? 'Hello, world!';
const animationDuration = response.settings.animationDuration ?? 300;
const showSplashScreen = response.settings.showSplashScreen ?? true;
```

上面代码中，默认值只有在左侧属性值为`null`或`undefined`时，才会生效。

这个运算符的一个目的，就是跟链判断运算符`?.`配合使用，为`null`或`undefined`的值设置默认值。

```javascript
const animationDuration = response.settings?.animationDuration ?? 300;
```

上面代码中，如果`response.settings`是`null`或`undefined`，或者`response.settings.animationDuration`是`null`或`undefined`，就会返回默认值300。也就是说，这一行代码包括了两级属性的判断。

[逻辑赋值运算符](https://github.com/tc39/proposal-logical-assignment)（logical assignment operators），将逻辑运算符与赋值运算符进行结合。

```javascript
// 或赋值运算符
x ||= y
// 等同于
x || (x = y)

// 与赋值运算符
x &&= y
// 等同于
x && (x = y)

// Null 赋值运算符
x ??= y
// 等同于
x ?? (x = y)
```

这三个运算符`||=`、`&&=`、`??=`相当于先进行逻辑运算，然后根据运算结果，再视情况进行赋值运算。

### Symbol

[Symbol](https://es6.ruanyifeng.com/?search=%E7%AE%AD%E5%A4%B4\&x=0\&y=0#docs/symbol)

`Symbol`函数前不能使用`new`命令，否则会报错。这是因为生成的 Symbol 是一个**原始类型的值**，不是对象。也就是说，由于 Symbol 值不是对象，所以不能添加属性。基本上，它是一种类似于字符串的数据类型。

可以用`toString`区分值, `toString`的用法不是很方便。[ES2019](https://github.com/tc39/proposal-Symbol-description) 提供了一个实例属性`description`，直接返回 Symbol 的描述。

`Symbol`函数的参数只是表示对当前 Symbol 值的描述，因此**相同参数**的`Symbol`函数的**返回值是不相等**的。

在对象的内部，使用 Symbol 值定义属性时，Symbol 值必须放在方括号之中。不然会被认为属性名是一个`string`，而不是`symbol`

Symbol 作为属性名，遍历对象的时候，该属性不会出现在`for...in`、`for...of`循环中，也不会被`Object.keys()`、`Object.getOwnPropertyNames()`、`JSON.stringify()`返回。

但是，它也不是私有属性，有一个`Object.getOwnPropertySymbols()`方法，可以获取指定对象的所有 Symbol 属性名。该方法返回一个数组，成员是当前对象的所有用作属性名的 Symbol 值。

我们希望重新使用同一个 Symbol 值，`Symbol.for()`方法可以做到这一点。它接受一个字符串作为参数，然后搜索有没有以该参数作为名称的 Symbol 值。如果有，就返回这个 Symbol 值，否则就新建一个以该字符串为名称的 Symbol 值，并将其注册到**全局**。

### Reflect

[Reflect](https://es6.ruanyifeng.com/#docs/reflect)

Reflect方法是静态的，所以不可以`new Reflect`

`Reflect`对象的方法与`Proxy`对象的方法一一对应，只要是`Proxy`对象的方法，就能在`Reflect`对象上找到对应的方法。这就让`Proxy`对象可以方便地调用对应的`Reflect`方法，完成默认行为，作为修改行为的基础。也就是说，不管`Proxy`怎么修改默认行为，你总可以在`Reflect`上获取默认行为。

### Proxy

[Proxy](https://es6.ruanyifeng.com/#docs/proxy)

`proxy`无法代理一些原生对象，因为这些原生对象的内部属性，只有通过正确的`this`才能拿到。

`proxy`会改变`this`:目标对象内部的`this`关键字会指向 Proxy 代理

### Iterator

[Iterator](https://es6.ruanyifeng.com/?search=%E7%AE%AD%E5%A4%B4\&x=0\&y=0#docs/iterator)

Iterator 接口的目的，就是为所有数据结构，提供了一种统一的访问机制，即`for...of`循环

ES6 规定，默认的 Iterator 接口部署在数据结构的`Symbol.iterator`属性，或者说，一个数据结构只要具有`Symbol.iterator`属性，就可以认为是“可遍历的”（iterable）。

原生具备 Iterator 接口的数据结构如下。

* Array
* Map
* Set
* String
* TypedArray
* 函数的 arguments 对象
* NodeList 对象

有一些场合会默认调用 Iterator 接口（即`Symbol.iterator`方法），除了下文会介绍的`for...of`循环，还有几个别的场合。

1. 解构赋值

2. 扩展运算符

3. `yield*`

4. **其他场合**

   由于数组的遍历会调用遍历器接口，所以任何接受数组作为参数的场合，其实都调用了遍历器接口。下面是一些例子。

   ```javascript
   for...of
   Array.from()
   Map(), Set(), WeakMap(), WeakSet()（比如`new Map([['a',1],['b',2]])`）
   Promise.all()
   Promise.race()
   ```

JavaScript 原有的`for...in`循环，只能获得数组对象的键名，不能直接获取键值。ES6 提供`for...of`循环，允许遍历获得键值。

`for...in`循环主要是为遍历对象而设计的，不适用于遍历数组。

### Generator

**Generator介绍**

- [Generator](https://es6.ruanyifeng.com/?search=%E7%AE%AD%E5%A4%B4\&x=0\&y=0#docs/generator)

- `yield`返回`IteratorResult`（迭代器）对象，它有两个属性，value和done，分别代表返回值和是否完成。

**generator与-Iterator-接口的关系**

[generator与-Iterator-接口的关系](https://es6.ruanyifeng.com/?search=%E7%AE%AD%E5%A4%B4\&x=0\&y=0#docs/generator#%E4%B8%8E-Iterator-%E6%8E%A5%E5%8F%A3%E7%9A%84%E5%85%B3%E7%B3%BB)

**任意一个对象的`Symbol.iterator`方法，等于该对象的遍历器生成函数**，调用该函数会返回该对象的一个遍历器对象。

由于 **Generator 函数就是遍历器生成函数**，因此可以把 Generator 赋值给对象的`Symbol.iterator`属性，从而使得该对象具有 Iterator 接口。

```javascript
var myIterable = {};
myIterable[Symbol.iterator] = function* () {
  yield 1;
  yield 2;
  yield 3;
};

[...myIterable] // [1, 2, 3]
```

上面代码中，Generator 函数赋值给`Symbol.iterator`属性，从而使得`myIterable`对象具有了 Iterator 接口，可以被`...`运算符遍历了。

Generator 函数执行后，返回一个遍历器对象。该对象本身也具有`Symbol.iterator`属性，执行后返回自身。

```javascript
function* gen(){
  // some code
}

var g = gen();

g[Symbol.iterator]() === g
// true
```

上面代码中，`gen`是一个 Generator 函数，调用它会生成一个遍历器对象`g`。它的`Symbol.iterator`属性，也是一个遍历器对象生成函数，执行后返回它自己。

**next 方法的参数** [§](https://es6.ruanyifeng.com/?search=箭头\&x=0\&y=0#docs/generator#next-方法的参数) [⇧](https://es6.ruanyifeng.com/?search=箭头\&x=0\&y=0#docs/generator)

`yield`表达式本身没有返回值，或者说总是返回`undefined`。`next`方法可以带一个参数，该参数就会被当作上一个`yield`表达式的返回值。

`Generator.prototype.throw()`方法抛出的错误要被内部捕获，前提是必须至少执行过一次`next`方法。

`throw`方法被捕获以后，会附带执行下一条`yield`表达式。也就是说，会附带执行一次`next`方法。

任何数据结构只要有 Iterator 接口，就可以被`yield*`遍历。

**V8 是怎么实现生成器函数的暂停执行和恢复执行的呢**

[V8 是怎么实现生成器函数的暂停执行和恢复执行的呢](https://time.geekbang.org/column/article/230117) 

协程是一种比线程更加轻量级的存在。可以把协程看成是跑在线程上的任务，**一个线程上可以存在多个协程，但是在线程上同时只能执行一个协程**。

**[应用](https://es6.ruanyifeng.com/?search=%E7%AE%AD%E5%A4%B4%5C&x=0%5C&y=0#docs/generator#%E5%BA%94%E7%94%A8)**

（1）异步操作的同步化表达 [§](https://es6.ruanyifeng.com/?search=箭头\&x=0\&y=0#docs/generator#（1）异步操作的同步化表达) [⇧](https://es6.ruanyifeng.com/?search=箭头\&x=0\&y=0#docs/generator)

（2）控制流管理 [§](https://es6.ruanyifeng.com/?search=箭头\&x=0\&y=0#docs/generator#（2）控制流管理) [⇧](https://es6.ruanyifeng.com/?search=箭头\&x=0\&y=0#docs/generator)

（3）部署 Iterator 接口 [§](https://es6.ruanyifeng.com/?search=箭头\&x=0\&y=0#docs/generator#（3）部署-Iterator-接口) [⇧](https://es6.ruanyifeng.com/?search=箭头\&x=0\&y=0#docs/generator)

​	**利用 Generator 函数，可以在任意对象上部署 Iterator 接口。**

（4）作为数据结构 [§](https://es6.ruanyifeng.com/?search=箭头\&x=0\&y=0#docs/generator#（4）作为数据结构) [⇧](https://es6.ruanyifeng.com/?search=箭头\&x=0\&y=0#docs/generator)

### async/await

[async/await](https://es6.ruanyifeng.com/?search=%E7%AE%AD%E5%A4%B4\&x=0\&y=0#docs/async)

[Generator 函数的语法糖](https://es6.ruanyifeng.com/?search=%E7%AE%AD%E5%A4%B4\&x=0\&y=0#docs/async#async-%E5%87%BD%E6%95%B0%E7%9A%84%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86)

- `async`返回Promise

[async await使用同步方式写异步代码](https://blog.poetries.top/browser-working-principle/guide/part4/lesson20.html#async-await%E4%BD%BF%E7%94%A8%E5%90%8C%E6%AD%A5%E6%96%B9%E5%BC%8F%E5%86%99%E5%BC%82%E6%AD%A5%E4%BB%A3%E7%A0%81)

- Generator、`await`底层都应用了协程

[async/await执行顺序](https://zhuanlan.zhihu.com/p/97138729)

### [decorator](https://es6.ruanyifeng.com/#docs/decorator)

函数不能使用`decorator`: 因为有函数提升，使得装饰器不能用于函数。类是不会提升的，所以就没有这方面的问题。

### [ArrayBuffer](https://zhuanlan.zhihu.com/p/133491261)

### Fetch

[fetch vs axios](https://segmentfault.com/a/1190000038300383)

[fetch vs axios vs ajax](https://zhuanlan.zhihu.com/p/58062212)

### 学习链接：[js教程](https://www.javascripttutorial.net/javascript-event-loop/)、[阮一峰es6教程](https://es6.ruanyifeng.com/#README)



## JS模块化

### JavaScript 模块化发展历程

[JavaScript 模块化发展历程](https://segmentfault.com/a/1190000023839483)

无模块化 -> 命名空间 -> IIFE（自执行函数）-> CommonJS -> AMD -> CMD -> UMD -> ES Module

**AMD**

[amd规范](https://github.com/amdjs/amdjs-api)

[代表：Require.js](https://requirejs.org/)

**CMD**

[Common Module Definition / draft](https://github.com/cmdjs/specification/blob/master/draft/module.md)

[代表：Sea.js](https://github.com/seajs/seajs)

**模块化编译实例**

[「将 Vue SFC 编译为 ESM 」探索之路](https://segmentfault.com/a/1190000040059032###)



## CSS & JS兼容

[Modernizr的介绍和使用](https://www.cnblogs.com/JoannaQ/p/3281841.html)



## Web APIs

[官网介绍](https://developer.mozilla.org/en-US/docs/Web/API)



## MutationObserver

[MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/MutationObserver)

[MutationObserver的应用](https://juejin.cn/post/6866943424709263373#heading-8)

[MutationObserver产生微任务](https://blog.poetries.top/browser-working-principle/guide/part4/lesson18.html#%E5%BE%AE%E4%BB%BB%E5%8A%A1)

## 