# js-01

## 数据类型有哪几种？

* 7 种原始数据类型(primitive types)： `Null` `Undefined` `String` `Number` `Boolean` `BigInt` `Symbol`
* `Object` 对象类型，也称为引用类型
* The big differences between primitive types and objects are
  * primitive types are immutable, objects only have an immutable reference, but their value can change over time
  * primitive types are passed by value. Objects are passed by reference
  * primitive types are copied by value. Objects are copied by reference
  * primitive types are compared by value. Objects are compared by reference

#### 基本数据类型和引用类型在内存的区别

* 基本类型：存储在栈内存中，因为基本类型的大小是固定，**在栈内可以快速查找**。
* 引用类型：存储在堆内存中，因为引用类型的大小是不固定的，所以存储在堆内存中，然后栈内存中仅存储堆中的内存地址。
* 我们在查找**对象是从栈中查找**，那么可得知，对于基本对象我们是对它的值进行操作，而对于引用类型，我们是对其引用地址操作。

![stack-heap](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/2/23/1707148374e5c7ef\~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)

## 什么是包装对象

### 定义

*   对象是 JavaScript 语言最主要的数据类型，三种原始类型的值——数值、字符串、布尔值——在一定条件下，也会**自动转为对象**，也就是**原始类型的“包装对象”（wrapper）**。

    所谓“包装对象”，指的是与数值、字符串、布尔值分别相对应的`Number`、`String`、`Boolean`三个原生对象。这三个原生对象可以把原始类型的值变成（包装成）对象。

    包装对象的设计目的，首先是**使得“对象”这种类型可以覆盖 JavaScript 所有的值，整门语言有一个通用的数据模型，其次是使得原始类型的值也有办法调用自己的方法。**
*   这三个对象作为构造函数使用（带有`new`）时，可以将原始类型的值转为对象；

    ```javascript
    // 下面代码中，基于原始类型的值，生成了三个对应的包装对象。可以看到，v1、v2、v3都是对象，且与对应的简单类型值不相等。
    var v1 = new Number(123);
    var v2 = new String('abc');
    var v3 = new Boolean(true);

    typeof v1 // "object"
    typeof v2 // "object"
    typeof v3 // "object"

    v1 === 123 // false
    v2 === 'abc' // false
    v3 === true // false
    ```
*   作为普通函数使用时（不带有`new`），可以将任意类型的值，转为原始类型的值。

    ```javascript
    // 字符串转为数值
    Number('123') // 123
    // 数值转为字符串
    String(123) // "123"
    // 数值转为布尔值
    Boolean(123) // true
    ```

### 实例方法

*   三种包装对象各自提供了许多实例方法，详见后文。这里介绍两种它们共同具有、从`Object`对象继承的方法：`valueOf()`和`toString()`。

    `valueOf()`方法返回包装对象实例对应的原始类型的值。

    ```javascript
    new Number(123).valueOf()  // 123
    new String('abc').valueOf() // "abc"
    new Boolean(true).valueOf() // true
    ```

    `toString()`方法返回对应的字符串形式。

    ```javascript
    new Number(123).toString() // "123"
    new String('abc').toString() // "abc"
    new Boolean(true).toString() // "true"
    ```

### 原始类型与实例对象的自动转换

*   某些场合，**原始类型的值会自动当作包装对象调用**，即调用包装对象的属性和方法。这时，JavaScript 引擎会自动将原始类型的值转为包装对象实例，并在使用后立刻销毁实例。

    ```javascript
    // abc是一个字符串，本身不是对象，不能调用length属性。
    // JavaScript 引擎自动将其转为包装对象，在这个对象上调用length属性。
    // 调用结束后，这个临时对象就会被销毁。这就叫原始类型与实例对象的自动转换。
    'abc'.length // 3

    var str = 'abc';
    str.length // 3

    // 等同于
    var strObj = new String(str)
    // String {
    //   0: "a", 1: "b", 2: "c", length: 3, [[PrimitiveValue]]: "abc"
    // }
    strObj.length // 3
    // 上面代码中，字符串abc的包装对象提供了多个属性，length只是其中之一。
    // 自动转换生成的包装对象是只读的，无法修改。所以，字符串无法添加新属性。

    // 自动转换生成的包装对象是只读的，无法修改。所以，字符串无法添加新属性。
    var s = 'Hello World';
    s.x = 123;
    s.x // undefined
    ```

### 自定义方法

*   除了原生的实例方法，包装对象还可以自定义方法和属性，供原始类型的值直接调用。比如，我们可以新增一个`double`方法，使得字符串和数字翻倍。

    ```javascript
    String.prototype.double = function () {
      return this.valueOf() + this.valueOf();
    };

    'abc'.double()
    // abcabc

    Number.prototype.double = function () {
      return this.valueOf() + this.valueOf();
    };

    (123).double() // 246

    // 上面代码在String和Number这两个对象的原型上面，分别自定义了一个方法，从而可以在所有实例对象上调用。
    // 注意，最后一行的123外面必须要加上圆括号，否则后面的点运算符（.）会被解释成小数点。
    ```
*   参考:

    [js包装对象](https://blog.csdn.net/m0\_46404694/article/details/119859431)

    自动转换生成的包装对象是只读的，无法修改。所以，字符串无法添加新属性。除了原生的实例方法，包装对象还可以自定义方法和属性，供原始类型的值直接调用。通过往prototype上加东西来自定义方法和属性。

    [包装对象](https://wangdoc.com/javascript/stdlib/wrapper)

## this

[JavaScript的this关键字 - Web前端工程师面试题讲解](https://www.bilibili.com/video/BV1BE411677T/)

[阮一峰this](https://www.ruanyifeng.com/blog/2010/04/using\_this\_keyword\_in\_javascript.html)

[函数执行结果 - 考察this指针](https://github.com/a1029563229/InterviewQuestions/tree/master/execute/1)

```javascript
function Foo() {
    Foo.a = function() {
        console.log(1)
    }
    this.a = function() {
        console.log(2)
    }
}
Foo.prototype.a = function() {
    console.log(3)
}
Foo.a = function() {
    console.log(4)
}
Foo.a();
let obj = new Foo(); 
obj.a();
Foo.a();
```

打印结果： 4 2 1

## 箭头函数

箭头函数与普通函数的区别,构造函数（function）可以使用 new 生成实例，那么箭头函数可以吗？为什么？

箭头函数是普通函数的简写，可以更优雅的定义一个函数，和普通函数相比，有以下几点差异：

1. 函数体内的 `this` 对象，就是定义时所在的对象，而不是使用时所在的对象；
2. **不可以使用 `arguments` 对象，该对象在函数体内不存在,** [**例**](https://juejin.cn/post/6844904070579240974#heading-16)。如果要用，可以用 rest 参数代替；
3. 不可以使用 `yield` 命令，因此箭头函数不能用作 `Generator` 函数；
4.  不可以使用`new`命令，因为：

    * 没有自己的 `this`，无法调用 `call、apply`；
    * 没有 `prototype` 属性，而 `new` 命令在执行时需要将钩子函数的 `prototype` 赋值给新的对象的 `__proto__`

    [js new的时候干了什么](https://juejin.cn/post/6844904180864253965)

    大致如下

    ```javascript
    // 1.创建一个空对象
    var obj = new Object()
    // 2.将空对象的原型赋值为构造函数的原型
    obj.__proto__ = Perons.prototype
    // 3.变更构造函数内部this,将其指向刚刚创建出来的对象
    Perons.call(obj)
    // 4.返回对象
    return obj
    ```

## prototype

{% embed url="https://github.com/HesterG/js-advanced-study/blob/main/01-prototype/index.js" %}

![img](https://pic1.zhimg.com/80/e83bca5f1d1e6bf359d1f75727968c11\_720w.webp?source=1940ef5c)



## typeof、instanceof

### instanceof vs typeof

1.  typeof(a) 用于返回值的类型，有 "number"、"string"、"boolean"、"null"、"function" 和 "undefined"、"symble"、"object"

    ```javascript
    let a = 1
    let a1 = '1'
    let a2 = true
    let a3 = null
    let a4 = undefined
    let a5 = Symbol
    let a6 = {}
    console.log(typeof(a),typeof(a1),typeof(a2),typeof(a3),typeof(a4),typeof(a5),typeof(a6))
    // number string boolean object undefined function object
    ```
2. instanceof 用于判断该对象是否是目标实例，根据原型链 `__proto__` 逐层向上查找，通过 instanceof 也可以判断一个实例是否是其父类型或者祖先类型的实例。

### instanceof 的实现原理

```javascript
while (x.__proto__) {
  if (x.__proto__ === y.prototype) {
    return true;
  }
  x.__proto__ = x.__proto__.__proto__;
}va
// 到顶了(Object.__proto__ = null)
if (x.__proto__ === null) {
  return false;
}
```

有这么个面试题

```javascript
function person() {
    this.name = 10
}
console.log(person instanceof person)
复制代码
```

结果是 `false` ，看下 person 函数的原型链 `person.__proto__ => Function.prototype => Object.prototype => null` ，所以在原型链上是找不到 `person` 的

## 描述 NaN 指的是什么

[原文](https://juejin.cn/post/6844904070579240974#heading-7)

[关于NAN](https://segmentfault.com/a/1190000016088123)

* NaN(Not A Number) 属性是代表非数字值的特殊值，该属性用于表示某个值不是数字。
*   NaN 是 **Number 对象中的静态属性**

    ```javascript
    typeof(NaN) // "number"
    NaN == NaN // false
    ```
* 它有两个重要的性质：
  1.  NaN`与任何值都不相等，包括`NaN\`自身：

      ```
      alert(NaN == NaN)  // false
      alert(NaN === NaN)  // false
      ```
  2. 任何涉及 `NaN`的操作都会返回`NaN`。

### 哪些情况会产生`NaN`?

1.  计算

    JS 在进行加减乘除运算之前，会先调用 `Number()`方法，将非数值的运算项转化为数值，如果转换失败就返回`NaN`，比如：

    `1-'a'; // NaN`

    首先是执行`Number('a')`，不能成功转化为数值，返回`NaN`，再利用`NaN`的第二条性质：任何涉及 `NaN`的操作都会返回`NaN`，所以最终的结果是`NaN`。
2.  类型转换

    当一个值不能被`Number`，`parseInt`，`parseFloat`成功转换为数值，就返回`NaN`，举例：

    ```javascript
    Number('123.456abc');   // NaN
    parseInt('123.456abc');  // 123
    parseFloat('123.456abc'); // 123.456

    Number('abc');  // NaN
    parseInt('abc');  // NaN
    parseFloat('abc');  // NaN

    Number([]);  // 0
    parseInt([]);  // NaN
    parseFloat([]);  // NaN

    Number('');  // 0
    parseInt('');  // NaN
    parseFloat('');  // NaN

    Number({});  // NaN
    parseInt({});  // NaN
    parseFloat({});  // NaN
    ```

    这里要注意三者之间的差异，我的理解是 **`parseInt`，`parseFloat`会尽量将参数值转化为数值**。

### `Number.isNaN()`

* 这个与`window.isNaN()`不是同一个，`window.isNaN()`是用来是否不是数字，`Number.isNaN()`是用来判断是不是就是`NaN`
*   **ES6 中的`Number.isNaN()`是一个判断`NaN`的升级版**，和`isNaN()`不同的是，`Number.isNaN()`不会强制转化参数，直接对参数本身做判断，这样只有参数显示等于`NaN`，才会返回`true`

    ```javascript
    Number.isNaN(NaN);  // true，其他情况都返回 false
    ```
*   它的实现原理是：

    ```javascript
    function isNaN (value) {
        return typeof(value) === "number" && isNaN(value);
    }
    ```
*   那怎么判断一个值是否是 NAN 呢？ 若支持 `es6` ，可直接使用 `Number.isNaN()`, 若不支，可根据 `NAN !== NAN` 的特性

    ```javascript
    function isReallyNaN(val) {
        let x = Number(val);
        return x !== x;
    }
    ```

## class

### class的基本语法

* [阮一峰class](https://es6.ruanyifeng.com/#docs/class)
*   基本上，ES6 的`class`可以看作只是一个语法糖，它的绝大部分功能，ES5 都可以做到，新的`class`写法只是让对象原型的写法更加清晰、更像面向对象编程的语法而已。

    ```javascript
    // es5
    function Point(x, y) {
      this.x = x;
      this.y = y;
    }
    Point.prototype.toString = function () {
      return '(' + this.x + ', ' + this.y + ')';
    };
    var p = new Point(1, 2);

    // 用es6 class改写
    class Point {
      constructor(x, y) {
        this.x = x;
        this.y = y;
      }

      toString() {
        return '(' + this.x + ', ' + this.y + ')';
      }
    }
    ```
*   ES6 的类，完全可以看作构造函数的另一种写法。

    ```javascript
    class Point {
      // ...
    }
    typeof Point // "function"
    Point === Point.prototype.constructor // true
    ```
*   由于类的方法都定义在`prototype`对象上面，所以类的新方法可以添加在`prototype`对象上面。`Object.assign()`方法可以很方便地一次向类添加多个方法。

    ```javascript
    class Point {
      constructor(){
        // ...
      }
    }

    Object.assign(Point.prototype, {
      toString(){},
      toValue(){}
    });
    ```
*   类的内部所有定义的方法，都是**不可枚举的（non-enumerable）**。

    ```javascript
    class Point {
      constructor(x, y) {
        // ...
      }

      toString() {
        // ...
      }
    }

    Object.keys(Point.prototype)
    // []
    Object.getOwnPropertyNames(Point.prototype)
    // ["constructor","toString"]

    // 这一点与 ES5 的行为不一致。以下是es5 是可以枚举内部方法的
    var Point = function (x, y) {
      // ...
    };
    Point.prototype.toString = function () {
      // ...
    };
    Object.keys(Point.prototype)
    // ["toString"]
    Object.getOwnPropertyNames(Point.prototype)
    // ["constructor","toString"]
    ```
*   constructor

    ```javascript
    // 一个类必须有constructor()方法，如果没有显式定义，一个空的constructor()方法会被默认添加。
    class Point {
    }

    // 等同于
    class Point {
      constructor() {}
    }
    ```
*   实例属性的新写法

    [ES2022](https://github.com/tc39/proposal-class-fields) 为类的实例属性，又规定了一种新写法。实例属性现在除了可以定义在`constructor()`方法里面的`this`上面，也可以定义在类内部的最顶层。

    ```javascript
    // 原来的写法
    class IncreasingCounter {
      constructor() {
        this._count = 0;
      }
      get value() {
        console.log('Getting the current value!');
        return this._count;
      }
      increment() {
        this._count++;
      }
    }
    // 上面示例中，实例属性_count定义在constructor()方法里面的this上面。
    // 现在的新写法是，这个属性也可以定义在类的最顶层，其他都不变。
    class IncreasingCounter {
      _count = 0;
      get value() {
        console.log('Getting the current value!');
        return this._count;
      }
      increment() {
        this._count++;
      }
    }
    ```
*   取值函数（getter）和存值函数（setter）

    与 ES5 一样，在“类”的内部可以使用`get`和`set`关键字，对某个属性设置存值函数和取值函数，拦截该属性的存取行为。

    ```javascript
    class MyClass {
      constructor() {
        // ...
      }
      get prop() {
        return 'getter';
      }
      set prop(value) {
        console.log('setter: '+value);
      }
    }

    let inst = new MyClass();

    inst.prop = 123;
    // setter: 123

    inst.prop
    // 'getter'

    // 存值函数和取值函数是设置在属性的 Descriptor 对象上的。
    class CustomHTMLElement {
      constructor(element) {
        this.element = element;
      }
      get html() {
        return this.element.innerHTML;
      }
      set html(value) {
        this.element.innerHTML = value;
      }
    }
    var descriptor = Object.getOwnPropertyDescriptor(
      CustomHTMLElement.prototype, "html"
    );
    "get" in descriptor  // true
    "set" in descriptor  // true
    // 上面代码中，存值函数和取值函数是定义在html属性的描述对象上面，这与 ES5 完全一致。
    ```

### [类的注意点](https://es6.ruanyifeng.com/#docs/class#%E7%B1%BB%E7%9A%84%E6%B3%A8%E6%84%8F%E7%82%B9)

*   严格模式

    类和模块的内部，默认就是严格模式，所以不需要使用`use strict`指定运行模式。只要你的代码写在类或模块之中，就只有严格模式可用。考虑到未来所有的代码，其实都是运行在模块之中，所以 ES6 实际上把整个语言升级到了严格模式。
* [不存在提升](https://es6.ruanyifeng.com/#docs/class#%E4%B8%8D%E5%AD%98%E5%9C%A8%E6%8F%90%E5%8D%87)
*   `name` 属性

    由于本质上，ES6 的类只是 ES5 的构造函数的一层包装，所以函数的许多特性都被`Class`继承，包括`name`属性。
*   Generator 方法

    如果某个方法之前加上星号（`*`），就表示该方法是一个 Generator 函数。
*   this 的指向

    类的方法内部如果含有`this`，它默认指向类的实例。但是，必须非常小心，一旦单独使用该方法，很可能报错。

### class和function的区别

* [class和function的区别](https://juejin.cn/post/7028115551184502820)
  1. class构造函数必须使用new操作符
  2.  class声明不可以提升

      `function`构造函数声明存在提升，也就是定义构造函数的部分**可以**写在实例化对象的后面

      `class`声明不存在提升，声明类的部分**不能**写在实例化对象的后面，，如下代码所示，报错信息显示`Cannot access 'Person' before initialization`。

      ```javascript
      const usr = new Person('Jack');
      // ReferenceError: Cannot access 'Person' before initialization

      class Person {
          constructor(name) {
              this.name = name;
          }
      }
      ```
  3. class不可以用call、apply、bind改变执行上下文

## 实现继承

*   [javascript实现继承的七种方式](https://juejin.cn/post/6844904161071333384)

    **寄生组合式继承**能够很完美地实现继承，但也不是没有缺点。`inherit()` 方法中复制了父类的原型，赋给子类，假如子类原型上有自定的方法，也会被覆盖，因此可以通过`Object.defineProperty`的方式，将子类原型上定义的属性或方法添加到复制的原型对象上，如此，既可以保留子类的原型对象的完整性，又能够复制父类原型。

    ```javascript
    function Parent(name, age){
        this.name = name;
        this.age = age;
    }

    Parent.prototype.getName = function(){
        console.log(this.name)
    }

    function Child(name, age, grade){
        // 这里call的意思就是把Parent的方法应用到Child这个对象身上
        // 所以Child就继承了Parent的方法
        Parent.call(this, name, age);
        this.grade = grade;
    }

    // Child.prototype = new Parent();

    // function inherit(child, parent){
    //     let obj = parent.prototype;
    //     obj.constructor = child;
    //     child.prototype = obj;
    // }

    function inherit(child, parent){
        let obj = parent.prototype;
        obj.constructor = child;
        for(let key in child.prototype){
            Object.defineProperty(obj, key, {
                value: child.prototype[key]
            })
        }
        child.prototype = obj;
    }
    ```

## 作用域、作用域链、闭包

* 函数作用域：function scope 执行上下文：execution context 执行栈： execution stack
*   [讲清楚之javascript作用域](https://segmentfault.com/a/1190000014980841)

    | 作用域（Scope）          | -         |
    | ------------------- | --------- |
    | window/global Scope | 全局作用域     |
    | function Scope      | 函数作用域     |
    | Block Scope         | 块作用域（ES6） |
    | eval Scope          | eval作用域   |
*   [作用域&作用域链](https://segmentfault.com/a/1190000022751945)

    **作用域链**

    作用域链保证了执行上下文中的变量和函数的有序访问。**作用域链**是由**当前执行上下文以及上级执行上下文(上级的上级，直到全局执行上下文)的变量对象组成的集合**。在作用域链的最前端始终都是当前执行上下文中的**变量对象(VO)**，如果是函数，则称其为**活动对象(AO)**，最底端总是全局执行上下文中的**变量对象(VO)**。说白了就是变量和函数的查找规则。

    函数也是对象，所以也可以有属性。当函数被创建时，默认就有一个`[[Scope]]`属性，它是一个内部属性仅供`JavaScript`引擎内部使用。这个属性里存放着就是当前执行上下文对应的作用域链。当当前执行上下文中访问一个属性或者函数时，就会从作用域链中自上而下进行查找。

    ```javascript
    var scope = 'scope';
    function foo(){
        var a = 1; 
        console.log(scope);    // scope
    }
    foo();
    ```

    上述代码中 foo 函数内部访问 scope 变量，但是当前作用域内并没有声明 scope 变量，所以就会去上级作用域中查找。最终在全局作用域中找到了变量 scope。这种链式的查找规则就是作用域链。

    用图来表示函数 foo 创建时和执行时，作用域链的变化。

    当foo函数创建时：

    ![3292740486-60272d6b9447c7c9_fix732](https://user-images.githubusercontent.com/17645053/213858622-b9901eaa-88c0-4501-9160-75a82bbc3542.jpeg)

    当foo函数执行时：

    ![1083231135-8bc3d40c27d2ea18_fix732](https://user-images.githubusercontent.com/17645053/213858659-288e99b8-d580-4466-86bf-b195e6cda789.jpeg)

    当联系着图再来看上述代码：当foo函数内部使用 scope 变量时，**在作用域链的顶端(foo执行上下文的活动对象AO)中查找，如果找不到就往作用域链的下一级寻找，这里的下一级为全局作用域**，所有就在全局执行上下文的变量对象上查找，刚好全局执行上下文的变量对象上有一个属性 scope ，其值为 “scope”。所以打印出 “scope”。

    再看一个例子：

    ```javascript
    var scope='scope';
    function foo() {
        var a=1;
        function bar() {
            var b=2;
            console.log(a); // 1
            console.log(b); // 2
            console.log(scope); // scope
            console.log(c); // Uncaught ReferenceError: c is not defined
        }
        bar();
    }
    foo();
    ```

    当foo被创建和执行时，作用域链的变化跟上述例子一致。但是这里当foo执行时，又遇到了 bar 函数的创建与执行。所以 bar函数执行上下文对应的作用域链为：

    ![1390143431-d2b22bbefb9ca559_fix732](https://user-images.githubusercontent.com/17645053/213858675-7d7b1112-cd08-4b64-bb43-6d65a6144355.jpeg)

    从上面的图我们可以分析上述代码的输出结果：

    当 bar 访问属性 a 时，先从作用域链的最顶端查找，也就是 bar执行上下文的活动对象(AO)，但是AO中并不存在a属性，所以就往作用域链的上层去查找，找到了 foo执行上下文的活动对象(AO)，碰巧foo执行上下文的活动对象(AO)上有一个属性a ，其值为 1，因此打印出1。当访问b、scope、c时跟访问a时查找规则一致。但是当访问c 时，查找到作用域链的最底端，也就是全局执行上下文的变量对象上仍未查找到，这时认为该变量未定义，抛出`ReferenceError`。

    \
    总结：

    * 作用域分为全局作用域和局部作用域。
    * 作用域其实就是规定了当前作用域中的变量和函数可被作用的范围。
    * 作用域链其实就是规定了变量和函数的查找规则、是当前执行上下文的变量对象以及所有父级执行上下文的变量对象的集合。
    * 当查找一个变量时，先从作用域的顶端查找，一直查找到作用域的底端，若查找完仍未找到，抛出错误。
*   [作用域和闭包](https://juejin.cn/post/6844904165672484871)

    **词法作用域**

    **词法作用域**（`Lexical Scopes`）是 `javascript` 中使用的作用域类型，**词法作用域** 也可以被叫做 **静态作用域**，与之相对的还有 **动态作用域**。那么 `javascript` 使用的 **词法作用域** 和 **动态作用域** 的区别是什么呢？看下面这段代码：

    ```scss
    var value = 1;

    function foo() {
      console.log(value);
    }

    function bar() {
      var value = 2;
      foo();
    }

    bar();

    // 结果是 ???
    // 结果是1
    复制代码
    ```

    上面这段代码中，一共有三个作用域：

    * 全局作用域
    * `foo` 的函数作用域
    * `bar` 的函数作用域

    一直到这边都好理解，可是 `foo` 里访问了本地作用域中没有的变量 `value` 。根据前面说的，引擎为了拿到这个变量就要去 `foo` 的上层作用域查询，那么 `foo` 的上层作用域是什么呢？是它 **调用时** 所在的 bar 作用域？还是它 **定义时** 所在的全局作用域？

    这个关键的问题就是 `javascript` 中的作用域类型——**词法作用域。**

    > 词法作用域，就意味着函数**被定义的时候，它的作用域就已经确定了**，和拿到哪里执行没有关系，因此词法作用域也被称为 “静态作用域”。

    如果是动态作用域类型，那么上面的代码运行结果应该是 `bar` 作用域中的 `2` 。也许你会好奇什么语言是动态作用域？`bash` 就是动态作用域，感兴趣的小伙伴可以了解一下。

    **Note:**

    * 词法作用域就是函数在定义时，它的作用域已经被确定了，和在哪执行没有关系。
    * 作用域的应用场景: 模块化

    \
    **闭包**

    > 能够访问其他函数内部变量的函数，被称为 **闭包**。

    上面这个定义比较难理解，简单来说，**闭包就是函数内部定义的函数，被返回了出去并在外部调用**。

    \
    **闭包的应用场景**

    1 单例模式

    单例模式是一种常见的涉及模式，它保证了一个类只有一个实例。实现方法一般是先判断实例是否存在，如果存在就直接返回，否则就创建了再返回。单例模式的好处就是避免了重复实例化带来的内存开销：

    ```javascript
    // 单例模式
    function Singleton(){
      this.data = 'singleton';
    }

    Singleton.getInstance = (function () {
      var instance;
        
      return function(){
        if (instance) {
          return instance;
        } else {
          instance = new Singleton();
          return instance;
        }
      }
    })();

    var sa = Singleton.getInstance();
    var sb = Singleton.getInstance();
    console.log(sa === sb); // true
    console.log(sa.data); // 'singleton'
    ```

    \
    \
    2 模拟私有属性

    `javascript` 没有 `java` 中那种 `public` `private` 的访问权限控制，对象中的所用方法和属性均可以访问，这就造成了安全隐患，内部的属性任何开发者都可以随意修改。虽然语言层面不支持私有属性的创建，但是我们可以用闭包的手段来模拟出私有属性：

    ```php
    // 模拟私有属性
    function getGeneratorFunc () {
      var _name = 'John';
      var _age = 22;
        
      return function () {
        return {
          getName: function () {return _name;},
          getAge: function() {return _age;}
        };
      };
    }

    var obj = getGeneratorFunc()();
    obj.getName(); // John
    obj.getAge(); // 22
    obj._age; // undefined
    ```
    
    \
    **闭包的问题**
    
    容易引起[内存泄露](https://juejin.cn/post/6844904165672484871#heading-15), [如何排查以及解决](https://juejin.cn/post/6844904165672484871#heading-16)



*   作用域教学视频:

    [JavaScript函数作用域 - Web前端工程师面试题讲解](https://www.bilibili.com/video/BV1AJ41137cW)

    [Execution context and execution stack](https://www.youtube.com/watch?v=9-GR58IqGCc\&t=449s\&ab\_channel=procademy)
*   闭包教学:

    [JavaScript闭包 - Web前端工程师面试题讲解](https://www.bilibili.com/video/BV1iE411q7Qd/?spm\_id\_from=333.788.recommend\_more\_video.0)

    [Learn Closures In 7 Minutes](https://www.youtube.com/watch?v=3a0I8ICR1Vg\&ab\_channel=WebDevSimplified)

    [学习Javascript闭包（Closure）](https://www.ruanyifeng.com/blog/2009/08/learning\_javascript\_closures.html)

    [廖雪峰 闭包](https://www.liaoxuefeng.com/wiki/1022910821149312/1023021250770016)
