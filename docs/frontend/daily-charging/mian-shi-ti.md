# 面试题

## 前端面试题

### 用户一段时间内无操作，自动退出登录功能

* [实现检测用户一段时间内无操作，自动退出登录功能](http://www.4k8k.xyz/article/think\_A\_lot/100855169)

### 如何实现语音对讲

* [WEB前端语音对讲实现方案以及示例](https://blog.csdn.net/klsstt/article/details/94616528?utm\_source=app\&app\_version=5.3.0)

### 要开发一个前端性能监控的工具，需要上报哪些内容？

* [前端性能监控-基础指标定义&计算](https://juejin.cn/post/6887108904766046216)

### css中的子元素是如何继承父元素的line-height？

* [line-height继承问题](https://chenoge.github.io/2018/05/27/CSS-line-height%E7%BB%A7%E6%89%BF%E9%97%AE%E9%A2%98/)
  * 如果父级的`line-height`属性值**有单位或百分比**，那么**子级继承的值则是换算后的一个具体的px级别的值**；
  * 而如果父级的`line-height`属性值**没有单位**，则**子级会直接继承这个“数值”**，而非计算后的具体值，此时子级的`line-height`会根据本身的`font-size`值重新计算得到新的`line-height`值。
* [CSS有哪些属性可以继承？](https://www.jianshu.com/p/34044e3c9317)

### [使用JS实现链式调用](https://zhuanlan.zhihu.com/p/498209186?utm\_source=wechat\_session\&utm\_medium=social\&utm\_oi=545907922706681856)

核心就在于调用完的方法**将自身实例返回**

1）示例一

```javascript
function Class1() {
    console.log('初始化')
}
Class1.prototype.method = function(param) {
    console.log(param)
    return this
}
let cl = new Class1()
// 由于new 在实例化的时候this会指向创建的对象， 所以this.method这个方法会在原型链中找到。
cl.method('第一次调用').method('第二次链式调用').method('第三次链式调用')
```

2）示例二

```javascript
var obj = {
    a: function() {
        console.log("a");
        return this;
    },
    b: function() {
        console.log("b");
        return this;
    },
};
obj.a().b();
```

3\) 示例三见链接

### 写一个方法将DOM里的类数组对象转换为数组

*   [什么是类数组对象(array-like object)](https://blog.axiu.me/what-is-array-like-object/)

    指一个对象包含**非负**的整数型length属性，通常还包含索引属性。例如：

    ```javascript
    var ao1 = {length:0}; // like []
    var ao2 = {0: 'foo', 5: 'bar', length: 6}; // like ['foo', undefined x 4, 'bar']
    ```

    类数组对象可以像数组一样执行读写、获取长度(length)、遍历的操作。

    ```javascript
    console.log(ao2[0]); // foo 
    ao2[0] = 1;
    console.log(ao2.length); // 6
    ```

    类数组对象不能使用数组的方法(push、pop等)，但是可以使用`Array.prototype.xxx.call()`来调用。

    ```javascript
    Array.prototype.push.call(ao2, '6');
    console.log(ao2); // {0: "foo", 5: "bar", 6: "6", length: 7}
    ```

    常用类数组

    ```javascript
    // NodeList
    var nodes = document.querySelectorAll('c');
    nodes.length; // 5
    Array.isArray(nodes); // false

    // 字符串
    ar str = '123456';
    str.length; // 6
    Array.isArray(str); // false

    // 函数的arguments参数
    function foo (argA, argB, argC) {
    console.log(arguments.length);
    console.log(Array.isArray(arguments));
    }
    foo('a', 'b', 'c');
    ```
*   [类数组/对象转换成数组](https://juejin.cn/post/6844903920637050888)

    1. `[].slice.call(arguments)`或`Array.prototype.slice.call(arguments)`

    `slice()` 方法返回一个新的数组对象，这一对象是一个由 `begin` 和 `end` 决定的原数组的**浅拷贝**（包括 `begin`，不包括`end`）。原始数组不会被改变。

    [原理](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global\_Objects/Array/slice#%E7%B2%BE%E7%AE%80%E8%B7%A8%E6%B5%8F%E8%A7%88%E5%99%A8%E8%A1%8C%E4%B8%BA)

    ```javascript
    // 例1
    var foo = function() {
        var arr = [].slice.call(arguments)
        var result = Object.prototype.toString.call(arr)
        console.log(arr) 
    }()

    // 例2
    function list() {
      return Array.prototype.slice.call(arguments);
    }
    var list1 = list(1, 2, 3); // [1, 2, 3]

    // 例3 除了使用 Array.prototype.slice.call(arguments)，你也可以简单的使用 [].slice.call(arguments) 来代替。另外，你可以使用 bind 来简化该过程。
    var unboundSlice = Array.prototype.slice;
    var slice = Function.prototype.call.bind(unboundSlice);
    function list() {
      return slice(arguments);
    }
    var list1 = list(1, 2, 3); // [1, 2, 3]
    ```

    2\. [`Array.from`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global\_Objects/Array/from)

    `Array.from(arrayLike[, mapFn[, thisArg]])`方法对一个类似数组或可迭代对象创建一个新的，浅拷贝的数组实例。

    `mapFn` - 如果指定了该参数，新数组中的每个元素会执行该回调函数。

    `thisArg` - 可选参数，执行回调函数 `mapFn` 时 `this` 对象。

    ```javascript
    // 例1
    const arrayLike = {
      length: 5
    }
    let arr2 = Array.from(arrayLike)
    //[undefined, undefined, undefined, undefined, undefined]

    // 例2
    function combine(){ 
        let arr = [].concat.apply([], arguments);  // 没有去重复的新数组 
        return Array.from(new Set(arr));
    } 
    var m = [1, 2, 2], n = [2,3,3]; 
    console.log(combine(m, n));           // [1, 2, 3]
    ```

    3\. `[...xxx]` 扩展运算符

    ES6中的扩展运算符`...`也能将某些数据结构转换成数组，这种数据结构必须有便利器接口。

    但是展开运算符只能将某些数据结构转换成数组, 因为本质是调用遍历器接口（Symbol.iterator ），如果一个对象没有部署这个接口，就无法转换 --- [阮一峰ES6](https://link.juejin.cn/?target=http%3A%2F%2Fes6.ruanyifeng.com%2F%23docs%2Farray)

    ```javascript
    // arguments对象
    function foo() {
      var args = [...arguments];
    }
    // NodeList对象
    [...document.querySelectorAll('div')]
    // 兼容写法
    const toArray = (() => Array.from ? Array.from : obj => [].slice.call(obj))();
    ```

### JS中Number表示

* [深入JavaScript中的Number类型](https://codeantenna.com/a/gW2Nj4WqDz)

