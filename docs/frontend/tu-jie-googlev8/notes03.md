# 笔记3

## [消息队列：V8是怎么实现回调函数的？](https://time.geekbang.org/column/article/227926)

我们在使用 JavaScript 时，经常要用到大量的回调函数，比如

* 在浏览器中可以使用 setTimeout 来设置定时器，使用 XMLHTTPRequest 来异步下载资源文件；
* 在 Node 中可以使用 readFile 来读取文件

这些操作都有一个共同的特点，那就是需要给调用 API 传入回调函数，然后浏览器或者 Node 会将执行处理的结果通过回调函数来触发。

从内部了解回调函数，可以帮助我们梳理清楚很多问题：

* 有助于我们理解浏览器中的 `Web API` 到底是怎么工作的；
* 有助于我们理解宏任务和微任务到底有哪些区别；
* 理解回调函数，是理解异步编程模型 `async/await` 的基础。

这些内容在我们实际的项目中都会频繁使用到，所以理解 `V8` 是怎么实现回调函数的就显得至关重要了。

### 什么是回调函数？

* 那究竟什么是回调函数呢？其实回调函数也是个函数，就像白马也是马一样。它具有函数的所有特征，它可以有参数和返回值。如果单独给出一个函数，你是看不出来它是不是回调函数的。**回调函数区别于普通函数，在于它的调用方式**。只有**当某个函数被作为参数，传递给另外一个函数，或者传递给宿主环境，然后该函数在函数内部或者在宿主环境中被调用，我们才称为回调函数。**
* 具体地讲，回调函数有两种不同的形式，同步回调和异步回调。通常，我们需要将回调函数传入给另外一个执行函数，那么**同步回调和异步回调的最大区别在于同步回调函数是在执行函数内部被执行的，而异步回调函数是在执行函数外部被执行的**。
*   我们先看一个**同步回调**的例子，你可以先看下面这段代码：

    ```js
    var myArray = ["water", "goods", "123", "like"];
    function handlerArray(indexName,index){
        console.log(index + 1 + ". " + indexName); 
    }
    myArray.forEach(handlerArray)
    ```

    在这段代码中，我们通过 `JavaScript` 自带的 `forEach` 方法来枚举数字中的每个项，这里的逻辑很简单：

    * 调用 `forEach` 时，需要使用回调函数 `handlerArray` 作为其参数；
    * 在 `forEach` 方法内部，会遍历 `myArray` 数组，每遍历一次都会调用一次回调函数 `handlerArray`。

    因为 `handlerArray` 是 `forEach` 的参数，而且 **`handlerArray` 是在 `forEach` 函数内部执行，所以这是一个同步回调。**
*   和同步回调函数不同的是，异步回调函数并不是在它的执行函数内部被执行的，而是在其他的位置和其他的时间点被执行的，比如下面这段 `setTimeout` 代码：

    ```js
    function foo() {
        alert("Hello");
    }
    setTimeout(foo, 3000)
    ```

    在这段代码中，我们使用了 `setTimeout` 函数，`setTimeout` 的第一个参数 `foo` 就是一个回调函数，`V8` 执行 `setTimeout` 时，会立即返回，等待 3000 毫秒之后，`foo` 函数才会被 `V8` 调用，**`foo` 函数并不是在 `setTimeout` 函数内部被执行的，所以这是一个异步回调。**

    对于同步回调函数的执行时机，我们理解起来比较简单，就是回调函数在执行函数内部被执行，那么异步回调函数在什么时机和什么位置被调用的呢？

    要解释清楚这个问题，我们就需要了解 `V8` 在运行时的线程模型，因为这涉及到了消息队列，事件循环等概念，这些概念都和线程模型是直接相关的，所以接下来我们就先来分析下 `V8` 的线程架构模型。

### UI 线程的宏观架构

* 早期浏览器的页面是运行在一个单独的 UI 线程中的，所以要在页面中引入 `JavaScript`，那么 `JavaScript` 也必须要运行在和页面相同的线程上，这样才能方便使用 `JavaScript` 来操纵 `DOM`，所以从一开始，`JavaScript` 就被设计成了运行在 UI 线程中。
* 所谓 UI 线程，是指运行窗口的线程，当你运行一个窗口时，无论该页面是 Windows 上的窗口系统，还是 Android 或者 iOS 上的窗口系统，它们都需要处理各种事件，诸如有触发绘制页面的事件，有鼠标点击、拖拽、放大缩小的事件，有资源下载、文件读写的事件，等等。
* 在页面线程中，当一个事件被触发时，比如用户使用鼠标点击了页面，系统需要将该事件提交给 UI 线程来处理。
* 在大部分情况下，UI 线程并不能立即响应和处理这些事件，比如在你在移动鼠标的过程中，每移动一个像素都会产生一个事件，所以鼠标移动的事件会频繁地被触发。在这种情况下，页面线程可能正在处理前一个事件，那么最新的事件就无法被立即执行。
*   针对这种情况，我们为 UI 线程提供一个消息队列，并将这些待执行的事件添加到消息队列中，然后 UI 线程会不断循环地从消息队列中取出事件、执行事件。\*\*我们把 UI 线程每次从消息队列中取出事件，执行事件的过程称为一个任务。\*\*整个流程大致如下所示：

    <img src="https://static001.geekbang.org/resource/image/b5/e1/b5c6a4cd613d262047a4339adb4eb8e1.jpg" alt="img" data-size="original" />

    我们可以用一段 JavaScript 代码来模拟下这个过程：

    ```js
    function UIMainThread() {
        while (queue.waitForMessage()) {
            Task task = queue.getNext()
            processNextMessage(task)
        }
    }
    ```

    在这段代码中，`queue` 是消息队列，`queue.waitForMessage()` 会同步地等待消息队列中的消息到达，如果当前没有任何消息等待被处理，则这个函数会将 UI 线程挂起。如果消息队列中有消息，则使用 `queue.getNext()` 取出下一个要执行的消息，并交由 `processNextMessage` 函数来处理消息。

    这就是**通用的 UI 线程的结构，有消息队列**，通过鼠标、键盘、触控板等产生的消息都会被添加进消息队列，**主线程会循环地从消息队列中取出消息并执行**。

### 异步回调函数的调用时机

*   理解了 UI 线程的基础架构模型，下面我们就可以来解释下异步函数的执行时机了。

    <img src="https://static001.geekbang.org/resource/image/87/44/874d4d0f49645dc211899b224f146644.jpg" alt="img" data-size="original" />

    比如在页面主线程中正在执行 A 任务，在执行 A 任务的过程中调用 `setTimeout(foo, 3000)`，**在执行 `setTimeout` 函数的过程中，宿主就会将`foo`函数封装成一个事件，并添加到消息队列中，然后 `setTimeout` 函数执行结束。**

    主线程会不间断地从消息队列中取出新的任务，执行新的任务，**等到时机合适，便取出 `setTimeout` 设置的 `foo` 函数的回调的任务，然后就可以直接执行 `foo` 函数的调用了**。

    通过分析，相信你已经发现了，通过 `setTimeout` 的执行流程其实是比较简单的，在 `setTimeout` 函数内部封装回调消息，并将回调消息添加进消息队列，然后主线程从消息队列中取出回调事件，并执行。
*   还有一类比较复杂一点的流程，最典型的是通过`XMLHttpRequest`所触发的回调，它和 `setTimeout` 有一些区别。

    因为 `XMLHttpRequest` 是用来下载网络资源的，但是**实际的下载过程却并不适合在主线程上执行，因为下载任务会消耗比较久的时间，如果在 UI 线程上执行，那么会阻塞 UI 线程，这就会拖慢 UI 界面的交互和绘制的效果**。所以当**主线程从消息队列中取出来了这类下载任务之后，会将其分配给网络线程，让其在网络线程上执行下载过程，这样就不会影响到主线程的执行了。**

    那么下面我们就来分析下 `XMLHttpRequest` 是怎么触发回调函数的？具体流程你可以参看下图：

    <img src="https://static001.geekbang.org/resource/image/94/42/942abef74c09cb43c0ffc94d0e836142.jpg" alt="img" data-size="original" />

    结合上图，我们就可以来分析下通用的 UI 线程是如何处理下载事件的，大致可以分为以下几步：

    1. UI 线程会从消息队列中取出一个任务，并分析该任务。
    2. 分析过程中发现该任务是一个**下载请求，那么主线程就会将该任务交给网络线程去执行。**
    3. 网络线程接到请求之后，便会和服务器端建立连接，并发出下载请求；
    4. 网络线程不断地收到服务器端传过来的数据；
    5. **网络线程每次接收到数据时**，都会将设置的回调函数和返回的数据信息，如大小、返回了多少字节、返回的数据在内存中存放的位置等信息**封装成一个新的事件，并将该事件放到消息队列中** ;
    6. UI 线程继续循环地读取消息队列中的事件，如果是下载状态的事件，那么 UI 线程会执行回调函数，程序员便可以在回调函数内部编写更新下载进度的状态的代码；
    7. 直到最后接收到下载结束事件，UI 线程会显示该页面下载完成。
* 这就是`XMLHttpRequest`所触发的回调流程，除了下载以外，`JavaScript` 中获取系统设备信息、文件读取等都是采用了类似的方式来实现的，因此，理解了 `XMLHttpRequest` 的执行流程，你也就理解了这一类异步 `API` 的执行流程了。

### 思考题

*   分析 Node 中的 readFileSync 和 readFile 函数，其中一个是同步读文件操作，另外一个是异步读文件操作，这两段代码如下所示：

    ```js
    var fs = require('fs')
    var data = fs.readFileSync('test.js')

    fs.readFile('test.txt', function(err, data){ data.toString() })
    ```
*   那么请你分别分析下它们的执行流程。欢迎你在留言区与我分享讨论。

    `readFileSync`函数执行时会等待文件读取完毕，再执行下一条语句，在该语句后可正常访问其执行结果(获取data)；

    `readFile`函数执行时不会等待文件读取完毕就会执行下一条语句，如果直接在其后而不是回调函数中操作其执行结果`data`时，程序会出现报错；

    本质是`readFileSync`是在主线程上执行的，`readFile`在读写线程中执行的

## [异步编程（一）：V8是如何实现微任务的？](https://time.geekbang.org/column/article/229532)

上节我们介绍了通用的 UI 线程架构，**每个 UI 线程都拥有一个消息队列，所有的待执行的事件都会被添加进消息队列中，UI 线程会按照一定规则，循环地取出消息队列中的事件，并执行事件**。而 JavaScript 最初也是运行在 UI 线程中的。换句话说，**JavaScript 语言就是基于这套通用的 UI 线程架构而设计的**。

基于这套基础 UI 框架，JavaScript 又延伸出很多新的技术，其中应用最广泛的当属**宏任务和微任务**。

**宏任务**很简单，**就是指消息队列中的等待被主线程执行的事件**。每个宏任务在执行时，V8 都会**重新创建栈**，然后随着宏任务中函数调用，**栈也随之变化**，最终，当该宏任务执行结束时，**整个栈又会被清空，接着主线程继续执行下一个宏任务**。

**微任务**稍微复杂一点，其实你可以把**微任务看成是一个需要异步执行的函数，执行时机是在主函数执行结束之后、当前宏任务结束之前。**

`JavaScript` 中之所以要引入微任务，主要是由于**主线程执行消息队列中宏任务的时间颗粒度太粗了，无法胜任一些对精度和实时性要求较高的场景**，那么\*\*微任务可以在实时性和效率之间做一个有效的权衡。\*\*另外使用微任务，可以改变我们现在的异步编程模型，使得我们可以使用同步形式的代码来编写异步调用。

虽然微任务如此重要，但是理解起来并不是太容易。我们先看下和微任务相关的知识栈，具体内容如下图所示：

![img](https://static001.geekbang.org/resource/image/1b/46/1b0ea2180dd2406b988b424cc2933746.jpg)

从图中可以看出，微任务是基于消息队列、事件循环、UI 主线程还有堆栈而来的，然后基于微任务，又可以延伸出协程、`Promise`、`Generator`、`await/async` 等现代前端经常使用的一些技术。也就是说，如果对消息队列、主线程还有调用栈理解的不够深入，你在研究微任务时，就容易一头雾水。

今天，我们就先来打通微任务的底层技术，搞懂消息队列、主线程、调用栈的关联，然后抽丝剥茧地剖析微任务的实现机制。

### 主线程、调用栈、消息队列

*   我们先从**主线程和调用栈**开始分析。我们知道，\*\*调用栈是一种数据结构，用来管理在主线程上执行的函数的调用关系。\*\*接下来我们通过执行下面这段代码，来分析下调用栈是如何管理主线程上函数调用的。

    ```js
    function bar() {
    }
    foo(fun){
      fun()
    }
    foo(bar)
    ```

    当 V8 准备执行这段代码时，会**先将全局执行上下文压入到调用栈中**，如下图所示：

    <img src="https://static001.geekbang.org/resource/image/82/62/82360525fb2bb064eb6a9916e4a81062.jpg" alt="img" data-size="original" />

    然后 V8 便开始在主线程上执行 foo 函数，首先它会**创建 foo 函数的执行上下文，并将其压入栈中**，那么此时调用栈、主线程的关系如下图所示：

    <img src="https://static001.geekbang.org/resource/image/6b/1e/6ba964d00e3528b4040a4ae12cc91e1e.jpg" alt="img" data-size="original" />

    然后，`foo` 函数又调用了 `bar` 函数，那么当 V8 执行 `bar` 函数时，同样要**创建 `bar` 函数的执行上下文，并将其压入栈中**，最终效果如下图所示：

    <img src="https://static001.geekbang.org/resource/image/28/fa/288d80945985a1cf0c0a4d4be9674ffa.jpg" alt="img" data-size="original" />

    **等 bar 函数执行结束，V8 就会从栈中弹出 bar 函数的执行上下文**，此时的效果如下所示：

    <img src="https://static001.geekbang.org/resource/image/9b/a6/9b7474d3e2d5ca840f2119ca64d40da6.jpg" alt="img" data-size="original" />

    最后，**foo 函数执行结束，V8 会将 foo 函数的执行上下文从栈中弹出**，效果如下所示：

    <img src="https://static001.geekbang.org/resource/image/e9/ab/e9e2e01d56b9546745eaa24ac00d74ab.jpg" alt="img" data-size="original" />
*   以上就是调用栈管理主线程上函数调用的方式，不过，这种方式会带来一种问题，那就是栈溢出。比如下面这段代码：

    ```js
    function foo(){
      foo()
    }
    foo()
    ```

    由于 foo 函数内部嵌套调用它自己，所以在调用 foo 函数的时候，它的栈会一直向上增长，但是由于栈空间在内存中是连续的，所以通常我们都会限制调用栈的大小，如果当函数嵌套层数过深时，过多的执行上下文堆积在栈中便会导致栈溢出，最终如下图所示：

    <img src="https://static001.geekbang.org/resource/image/81/25/814b88dc157ef6f43403f46e271ae625.jpg" alt="img" data-size="original" />

    我们可以使用 `setTimeout` 来解决栈溢出的问题，`setTimeout` 的本质是将同步函数调用改成异步函数调用，这里的异步调用是将 `foo` 封装成事件，并将其添加进**消息队列**中，然后主线程再按照一定规则循环地从消息队列中读取下一个任务。使用 `setTimeout` 改造后代码代码如下所示：

    ```js
    function foo() {
      setTimeout(foo, 0)
    }
    foo()
    ```

    那么现在我们就可以从**调用栈、主线程、消息队列**这三者的角度来分析这段代码的执行流程了。

    首先，主线程会从消息队列中取出需要执行的宏任务，假设当前取出的任务就是要执行的这段代码，这时候主线程便会进入代码的执行状态。这时关于主线程、消息队列、调用栈的关系如下图所示：

    <img src="https://static001.geekbang.org/resource/image/bf/65/bfea2ecc835f8324e034db33339ed965.jpg" alt="img" data-size="original" />

    接下来 V8 就要执行 foo 函数了，同样**执行 foo 函数时，会创建 foo 函数的执行上下文，并将其压入栈中**，最终效果如下图所示：

    <img src="https://static001.geekbang.org/resource/image/48/dc/48706d55e9c490d84d676b5000d56bdc.jpg" alt="img" data-size="original" />

    当 V8 执行执行 foo 函数中的 `setTimeout` 时，`setTimeout` 会将`foo`函数封装成一个新的宏任务，并将其添加到消息队列中，在 V8 执行 `setTimeout` 函数时的状态图如下所示：

    <img src="https://static001.geekbang.org/resource/image/dc/e1/dc84bd1ee456789905e734415eecdce1.jpg" alt="img" data-size="original" />

    等 `foo` 函数执行结束，**V8 就会结束当前的宏任务，调用栈也会被清空**，调用栈被清空后状态如下图所示：

    <img src="https://static001.geekbang.org/resource/image/2a/89/2aeacb651b6aec591105ce2e4a8ed889.jpg" alt="img" data-size="original" />

    当一个宏任务执行结束之后，忙碌的主线程依然不会闲下来，它会一直重复这个取宏任务、执行宏任务的过程。刚才通过 `setTimeout` 封装的回调宏任务，也会在某一时刻被主线取出并执行，这个执行过程，就是 foo 函数的调用过程。具体示意图如下所示：

    <img src="https://static001.geekbang.org/resource/image/26/bc/260d8a7294472f4ee7b194bdb7d513bc.jpg" alt="img" data-size="original" />

    因为 `foo` 函数并不是在当前的父函数内部被执行的，而是**封装成了宏任务，并丢进了消息队列中，然后等待主线程从消息队列中取出该任务，再执行该回调函数 foo，这样就解决了栈溢出的问题。**

### 微任务解决了宏任务执行时机不可控的问题

* 不过，对于栈溢出问题，虽然我们可以通过将某些函数封装成宏任务的方式来解决，但是宏任务需要先被放到消息队列中，如果某些宏任务的执行时间过久，那么就会影响到消息队列后面的宏任务的执行，而且这个影响是不可控的，因为你无法知道前面的宏任务需要多久才能执行完成。
* 于是 JavaScript 中又引入了微任务，微任务会在当前的任务快要执行结束时执行，**利用微任务，你就能比较精准地控制你的回调函数的执行时机。**
* 通俗地理解，V8 会为每个宏任务维护一个微任务队列。当 V8 执行一段 JavaScript 时，会为这段代码创建一个环境对象，微任务队列就是存放在该环境对象中的。当你通过 `Promise.resolve` 生成一个微任务，该微任务会被 V8 自动添加进微任务队列，等整段代码快要执行结束时，该环境对象也随之被销毁，但是在销毁之前，V8 会先处理微任务队列中的微任务。
* 理解微任务的执行时机，你只需要记住以下两点：
  1. 首先，如果当前的任务中产生了一个微任务，通过 `Promise.resolve()` 或者 `Promise.reject()` 都会触发微任务，**触发的微任务不会在当前的函数中被执行，所以执行微任务时，不会导致栈的无限扩张**；
  2. 其次，和异步调用不同，微任务依然会在当前任务执行结束之前被执行，这也就意味着在当前微任务执行结束之前，消息队列中的其他任务是不可能被执行的。
*   因此在函数内部触发的微任务，一定比在函数内部触发的宏任务要优先执行。为了验证这个观点，我们来分析一段代码：

    ```js
    function bar(){
      console.log('bar')
      Promise.resolve().then(
        (str) =>console.log('micro-bar')
      ) 
      setTimeout((str) =>console.log('macro-bar'),0)
    }
    ```

    ```js
    function foo() {
      console.log('foo')
      Promise.resolve().then(
        (str) =>console.log('micro-foo')
      ) 
      setTimeout((str) =>console.log('macro-foo'),0)
      
      bar()
    }
    foo()
    console.log('global')
    Promise.resolve().then(
      (str) =>console.log('micro-global')
    ) 
    setTimeout((str) =>console.log('macro-global'),0)
    ```
    
    在这段代码中，包含了通过 `setTimeout` 宏任务和通过 `Promise.resolve` 创建的微任务，你认为最终打印出来的顺序是什么？
    
    执行这段代码，我们发现最终打印出来的顺序是：
    
    ```js
    foo
    bar
    global
    micro-foo
    micro-bar
    micro-global
    macro-foo
    macro-bar
    macro-global
    ```
    
    我们可以清晰地看出，微任务是处于宏任务之前执行的。接下来，我们就来详细分析下 V8 是怎么执行这段 JavaScript 代码的。
    
    首先，当 V8 执行这段代码时，**会将全局执行上下文压入调用栈中，并在执行上下文中创建一个空的微任务队列**。那么此时：
    
    * 调用栈中包含了全局执行上下文；
    * 微任务队列为空。
    
    此时的消息队列、主线程、调用栈的状态图如下所示：
    
    <img src="https://static001.geekbang.org/resource/image/b9/2a/b9bf0027185405e762b46cd4b77c892a.jpg" alt="img" data-size="original" />
    
    然后，执行`foo`函数的调用，V8 会先创建 `foo` 函数的执行上下文，并将其压入到栈中。接着执行 `Promise.resolve`，这会触发一个 `micro-foo1` 微任务，V8 会将该微任务添加进微任务队列。然后执行 `setTimeout` 方法。该方法会触发了一个 `macro-foo1` 宏任务，`V8` 会将该宏任务添加进消息队列。那么此时：
    
    * 调用栈中包含了全局执行上下文、`foo` 函数的执行上下文；
    * 微任务队列有了一个微任务，`micro-foo`；
    * 消息队列中存放了一个通过 `setTimeout` 设置的宏任务，`macro-foo`。
    
    此时的消息队列、主线程和调用栈的状态图如下所示(图画错了)：
    
    <img src="https://static001.geekbang.org/resource/image/f8/2d/f85b1b316f669316cef23c4714d4ce2d.jpg" alt="img" data-size="original" />
    
    接下来，foo 函数调用了 bar 函数，**那么 V8 需要再创建 bar 函数的执行上下文，并将其压入栈中**，接着执行 `Promise.resolve`，这会触发一个 `micro-bar` 微任务，该微任务会被添加进微任务队列。然后执行 `setTimeout` 方法，这也会触发一个 `macro-bar` 宏任务，宏任务同样也会被添加进消息队列。那么此时：
    
    * 调用栈中包含了**全局执行上下文、foo 函数的执行上下文、bar 的执行上下文**；
    * 微任务队列中的微任务是 `micro-foo`、`micro-bar`；
    * 消息队列中，宏任务的状态是 `macro-foo`、`macro-bar`。
    
    此时的消息队列、主线程和调用栈的状态图如下所示：
    
    <img src="https://static001.geekbang.org/resource/image/33/1c/338875c3ff58e389af86cf2acab5bd1c.jpg" alt="img" data-size="original" />
    
    接下来，`bar` 函数执行结束并退出，`bar` 函数的执行上下文也会从栈中弹出，紧接着 `foo` 函数执行结束并退出，`foo` 函数的执行上下文也随之从栈中被弹出。那么此时：
    
    * 调用栈中包含了全局执行上下文，因为 `bar` 函数和 `foo` 函数都执行结束了，所以它们的执行上下文都被弹出调用栈了；
    * 微任务队列中的微任务同样还是 `micro-foo`、`micro-bar`；
    * 消息队列中宏任务的状态同样还是 `macro-foo`、`macro-bar`。
    
    此时的消息队列、主线程和调用栈的状态图如下所示：
    
    <img src="https://static001.geekbang.org/resource/image/d2/e0/d24acef2cc39b1688dff9f19b9cdb9e0.jpg" alt="img" data-size="original" />
    
    主线程执行完了 `foo` 函数，紧接着就要执行全局环境中的代码 `Promise.resolve` 了，这会触发一个 `micro-global` 微任务，`V8` 会将该微任务添加进微任务队列。接着又执行 `setTimeout` 方法，该方法会触发了一个 `macro-global` 宏任务，`V8` 会将该宏任务添加进消息队列。那么此时：
    
    * 调用栈中包含的是全局执行上下文；
    * 微任务队列中的微任务同样还是 `micro-foo`、`micro-bar`、`micro-global`；
    * 消息队列中宏任务的状态同样还是 `macro-foo`、`macro-bar`、`macro-global`。
    
    此时的消息队列、主线程和调用栈的状态图如下所示：
    
    <img src="https://static001.geekbang.org/resource/image/34/db/34fb1a481b60708360b48ba04821f6db.jpg" alt="img" data-size="original" />
    
    等到这段代码即将执行完成时，`V8` 便要销毁这段代码的环境对象，此时环境对象的析构函数被调用（注意，这里的析构函数是 C++ 中的概念），这里就是 `V8` 执行微任务的一个检查点，这时候 **`V8` 会检查微任务队列，如果微任务队列中存在微任务，那么 V8 会依次取出微任务，并按照顺行执行。因为微任务队列中的任务分别是：`micro-foo、micro-bar、micro-global`，所以执行的顺序也是如此。**
    
    此时的消息队列、主线程和调用栈的状态图如下所示：
    
    <img src="https://static001.geekbang.org/resource/image/26/c9/267e549592913e25bdb6dfe716eeddc9.jpg" alt="img" data-size="original" />
    
    等微任务队列中的所有微任务都执行完成之后，当前的宏任务也就执行结束了，**接下来主线程会继续重复执行取出任务、执行任务的过程。由于正常情况下，取出宏任务的顺序是按照先进先出的顺序**，所有最后打印出来的顺序是：`macro-foo、macro-bar、macro-global`。
    
    等所有的任务执行完成之后，消息队列、主线程和调用栈的状态图如下所示：
    
    <img src="https://static001.geekbang.org/resource/image/6c/4a/6caa4a24f1ddd918af62f6dbcb1c464a.jpg" alt="img" data-size="original" />
    
    以上就是完整的执行流程的分析，到这里，相信你已经了解微任务和宏任务的执行时机是不同的了，**微任务是在当前的任务快要执行结束之前执行的，宏任务是消息队列中的任务，主线程执行完一个宏任务之后，便会接着从消息队列中取出下一个宏任务并执行。**

### 能否在微任务中循环地触发新的微任务？

既然宏任务和微任务都是异步调用，只是执行的时机不同，那能不能在 `setTimeout` 解决栈溢出的问题时，把触发宏任务改成是触发微任务呢？

比如，我们将代码改为：

```js
function foo() {
  return Promise.resolve().then(foo)
}
foo()
```

当执行 `foo` 函数时，由于`foo`函数中调用了 `Promise.resolve()`，这会触发一个微任务，那么此时，`V8` 会将该微任务添加进微任务队列中，退出当前 `foo` 函数的执行。

然后，`V8` 在准备退出当前的宏任务之前，会检查微任务队列，发现微任务队列中有一个微任务，于是先执行微任务。由于这个微任务就是调用 `foo` 函数本身，所以在执行微任务的过程中，需要继续调用`foo`函数，在执行 `foo` 函数的过程中，又会触发了同样的微任务。

**那么这个循环就会一直持续下去，当前的宏任务无法退出，也就意味着消息队列中其他的宏任务是无法被执行的，比如通过鼠标、键盘所产生的事件。这些事件会一直保存在消息队列中，页面无法响应这些事件，具体的体现就是页面的卡死**。

不过，由于`V8`每次执行微任务时，都**会退出当前 `foo` 函数的调用栈，所以这段代码是不会造成栈溢出的**。

### 总结

* 这节课我们主要从**调用栈、主线程、消息队列**这三者关联的角度来分析了微任务。
* 调用栈是一种数据结构，用来管理在主线程上执行的函数的调用关系。主线在执行任务的过程中，如果函数的调用层次过深，可能造成栈溢出的错误，我们可以使用`setTimeout`来解决栈溢出的问题。
* `setTimeout` 的本质是将同步函数调用改成异步函数调用，这里的异步调用是**将回调函数封装成宏任务，并将其添加进消息队列中，然后主线程再按照一定规则循环地从消息队列中读取下一个宏任务。**
* **消息队列中事件又被称为宏任务**，不过，宏任务的时间颗粒度太粗了，无法胜任一些对精度和实时性要求较高的场景，而**微任务可以在实时性和效率之间做有效的权衡**。
* 微任务之所以能实现这样的效果，主要**取决于微任务的执行时机，微任务其实是一个需要异步执行的函数，执行时机是在主函数执行结束之后、当前宏任务结束之前。**
* 因为**微任务依然是在当前的任务中执行的，所以如果在微任务中循环触发新的微任务，那么将导致消息队列中的其他任务没有机会被执行**。

### 思考题

*   浏览器中的 `MutationObserver` 接口提供了监视对 `DOM` 树所做更改的能力，它在内部也使用了微任务的技术，那么今天留给你的作业是，查找 `MutationObserver` 相关资料，分析它是如何工作的，其中微任务的作用是什么？

    `MutationObserver`和`IntersectionObserver`两个性质应该差不多。我这里简称`ob`。`ob`是一个微任务，通过浏览器的`requestIdleCallback`，在浏览器每一帧的空闲时间执行`ob`监听的回调，该监听是不影响主线程的，但是回调会阻塞主线程。当然有一个限制，如果100ms内主线程一直处于未空闲状态，那会强制触发`ob`。

## [异步编程（二）：V8是如何实现async/await的？](https://time.geekbang.org/column/article/230117)

上一节我们介绍了 JavaScript 是基于单线程设计的，最终造成了 JavaScript 中出现大量回调的场景。当 JavaScript 中有大量的异步操作时，会降低代码的可读性, 其中最容易造成的就是回调地狱的问题。

JavaScript 社区探索并推出了一系列的方案，**从“Promise 加 then”到“generator 加 co”方案，再到最近推出“终极”的 async/await 方案，完美地解决了回调地狱所造成的问题。**

今天我们就来分析下回调地狱问题是如何被一步步解决的，在这个过程中，你也就理解了 V8 实现 async/await 的机制。

### 什么是回调地狱？

*   我们先来看什么是回调地狱。

    假设你们老板给了你一个小需求，要求你从网络获取某个用户的用户名，获取用户名称的步骤是先通过一个 `id_url` 来获取用户 ID，然后再使用获取到的用户 ID 作为另外一个 `name_url` 的参数，以获取用户名。

    我做了两个 DEMO URL，如下所示：

    ```js
    const id_url = 'https://raw.githubusercontent.com/binaryacademy/geektime-v8/master/id'

    const name_url = 'https://raw.githubusercontent.com/binaryacademy/geektime-v8/master/name'
    ```

    那么你会怎么实现这个小小的需求呢？

    其中最容易想到的方案是使用 `XMLHttpRequest`，并**按照前后顺序异步请求这两个 URL**。具体地讲，你可以先定义一个 `GetUrlContent` 函数，这个函数**负责封装 `XMLHttpRequest` 来下载 `URL` 文件内容**，由于**下载过程是异步执行的，所以需要通过回调函数来触发返回结果**。那么我们需要给 `GetUrlContent` 传递一个回调函数 `result_callback`，来触发异步下载的结果。

    最终实现的业务代码如下所示：

    ```js
    //result_callback：下载结果的回调函数
    //url：需要获取URL的内容
    function GetUrlContent(result_callback,url) {
        let request = new XMLHttpRequest()

        request.open('GET', url)

        request.responseType = 'text'

        request.onload = function () {

        	result_callback(request.response)

    		}

    		request.send()

    }

    function IDCallback(id) {

      console.log(id)

      let new_name_url = name_url + "?id="+id

      GetUrlContent(NameCallback,new_name_url)

    }

    function NameCallback(name) {

    	console.log(name)

    }

    GetUrlContent(IDCallback,id_url)
    ```

    在这段代码中：

    1. 我们先使用 `GetUrlContent` 函数来异步下载用户 ID，之后再通过`IDCallback`回调函数来获取到请求的 ID；
    2. 有了 ID 之后，我们再在 `IDCallback` 函数内部，使用获取到的 ID 和 `name_url` 合并成新的获取用户名称的 URL 地址；
    3. 然后，再次使用`GetUrlContent`来获取用户名称，返回的用户名称会触发 `NameCallback` 回调函数，我们可以在 `NameCallback` 函数内部处理最终的返回结果。
* 可以看到，我们每次请求网络内容，都需要设置一个回调函数，用来返回异步请求的结果，这些穿插在代码之间的回调函数打乱了代码原有的顺序，比如正常的代码顺序是先获取 ID，再获取用户名。但是由于使用了异步回调函数，获取用户名代码的位置，反而在获取用户 ID 的代码之上了，这就直接导致了我们代码逻辑的不连贯、不线性，非常不符合人的直觉。
* 因此，异步回调模式影响到我们的编码方式，如果在代码中过多地使用异步回调函数，会将你的整个代码逻辑打乱，从而让代码变得难以理解，这也就是我们经常所说的回调地狱问题。

### 使用 Promise 解决回调地狱问题

*   为了解决回调地狱的问题，JavaScript 做了大量探索，最开始引入了 Promise 来解决部分回调地狱的问题，比如最新的 fetch 就使用 Promise 的技术，我们可以使用 fetch 来改造上面这段代码，改造后的代码如下所示：

    ```js
    fetch(id_url)

    .then((response) => {

    return response.text()

    })

    .then((response) => {

    let new_name_url = name_url + "?id=" + response

    return fetch(new_name_url)

    }).then((response) => {

    return response.text()

    }).then((response) => {

    console.log(response)//输出最终的结果

    })  
    ```

    我们可以看到，改造后的代码是先获取用户 ID，等到返回了结果之后，再利用用户 ID 生成新的获取用户名称的 URL，然后再获取用户名，最终返回用户名。使用 Promise，我们就可以按照线性的思路来编写代码，非常符合人的直觉。所以说，**使用 Promise 可以解决回调地狱中编码不线性的问题**。

### 使用 Generator 函数实现更加线性化逻辑

*   虽然使用 Promise 可以解决回调地狱中编码不线性的问题，但这种方式充满了 Promise 的 then() 方法，**如果处理流程比较复杂的话，那么整段代码将充斥着大量的 then，异步逻辑之间依然被 then 方法打断了，因此这种方式的语义化不明显，代码不能很好地表示执行流程**。那么我们就需要思考，能不能更进一步，像编写同步代码的方式来编写异步代码，比如：

    ```js
    function getResult(){
       let id = getUserID()
       let name = getUserName(id)
       return name
    }
    ```
*   由于`getUserID()`和 `getUserName()` 都是异步请求，如果要实现这种线性的编码方式，那么一个可行的方案就是执行到异步请求的时候，**暂停当前函数，等异步请求返回了结果，再恢复该函数**。

    具体地讲，执行到 `getUserID()` 时暂停 `GetResult` 函数，然后浏览器在后台处理实际的请求过程，待 ID 数据返回时，再来恢复`GetResult`函数。接下来再执行`getUserName`来获取到用户名，由于 `getUserName()` 也是一个异步请求，所以在使用 `getUserName()` 的同时，依然需要暂停 `GetResult` 函数的执行，等到 `getUserName()` 返回了用户名数据，再恢复 `GetResult` 函数的执行，最终`getUserName()`函数返回了 `name` 信息。

    这个思维模型大致如下所示：

    <img src="https://static001.geekbang.org/resource/image/48/65/485d9de2097107f818c622a83a953665.jpg" alt="img" data-size="original" />

    我们可以看出，这个模型的关键就是**实现函数暂停执行和函数恢复执行**，而生成器就是为了实现暂停函数和恢复函数而设计的。
*   **生成器函数是一个带星号函数，配合 yield 就可以实现函数的暂停和恢复**，我们看看生成器的具体使用方式：

    ```js
    function* getResult() {

    yield 'getUserID'

    yield 'getUserName'

    return 'name'

    }

    let result = getResult()

    console.log(result.next().value)

    console.log(result.next().value)
    console.log(result.next().value)
    ```

    执行上面这段代码，观察输出结果，你会发现函数 `getResul`t 并不是一次执行完的，而是全局代码和 `getResult` 函数交替执行。

    其实这就是生成器函数的特性，在生成器内部，如果遇到 `yield` 关键字，那么 `V8` 将返回关键字后面的内容给外部，并暂停该生成器函数的执行。生成器暂停执行后，外部的代码便开始执行，外部代码如果想要恢复生成器的执行，可以使用`result.next`方法。

    那么，V8 是怎么实现生成器函数的暂停执行和恢复执行的呢？

    这背后的魔法就是**协程，协程是一种比线程更加轻量级的存在**。你可以把协程看成是跑在线程上的任务，一个线程上可以存在多个协程，但是在线程上同时只能执行一个协程。比如，当前执行的是 A 协程，要启动 B 协程，那么 A 协程就需要将主线程的控制权交给 B 协程，这就体现在 A 协程暂停执行，B 协程恢复执行；同样，也可以从 B 协程中启动 A 协程。**通常，如果从 A 协程启动 B 协程，我们就把 A 协程称为 B 协程的父协程。**

    正如一个进程可以拥有多个线程一样，一个线程也可以拥有多个协程。\*\*每一时刻，该线程只能执行其中某一个协程。\*\*最重要的是，**协程不是被操作系统内核所管理，而完全是由程序所控制**（也就是在用户态执行）。这样带来的好处就是性能得到了很大的提升，不会像线程切换那样消耗资源。

    为了让你更好地理解协程是怎么执行的，我结合上面那段代码的执行过程，画出了下面的“协程执行流程图”，你可以对照着代码来分析：

    <img src="https://static001.geekbang.org/resource/image/f0/41/f0e0800ab9ec2f559fbb58ecabc76c41.jpg" alt="img" data-size="original" />
*   到这里，相信你已经弄清楚协程是怎么工作的了，其实在 JavaScript 中，**生成器就是协程的一种实现方式，这样，你也就理解什么是生成器了**。

    因为生成器可以暂停函数的执行，所以，我们将所有异步调用的方式，写成同步调用的方式，比如我们使用生成器来实现上面的需求，代码如下所示：

    ```js
    function* getResult() {
        let id_res = yield fetch(id_url);
        console.log(id_res)
        let id_text = yield id_res.text();
        console.log(id_text)
    ```

    ```js
        let new_name_url = name_url + "?id=" + id_text
        console.log(new_name_url)


        let name_res = yield fetch(new_name_url)
        console.log(name_res)
        let name_text = yield name_res.text()
        console.log(name_text)
    }


    let result = getResult()
    result.next().value.then((response) => {
        return result.next(response).value
    }).then((response) => {
        return result.next(response).value
    }).then((response) => {
        return result.next(response).value
    }).then((response) => {
        return result.next(response).value
    ```
    
    这样，我们可以将同步、异步逻辑全部写进生成器函数 `getResult` 的内部，然后，我们在外面依次使用一段代码来控制生成器的暂停和恢复执行。以上，就是协程和 Promise 相互配合执行的大致流程。
*   通常，我们把执行生成器的代码封装成一个函数，这个函数驱动了 getResult 函数继续往下执行，我们把这个执行生成器代码的函数称为**执行器（可参考著名的 co 框架）**，如下面这种方式：

    ```js
    function* getResult() {
        let id_res = yield fetch(id_url);
        console.log(id_res)
        let id_text = yield id_res.text();
        console.log(id_text)
    ```


        let new_name_url = name_url + "?id=" + id_text
        console.log(new_name_url)


        let name_res = yield fetch(new_name_url)
        console.log(name_res)
        let name_text = yield name_res.text()
        console.log(name_text)
    }
    co(getResult())
    ```

### async/await：异步编程的“终极”方案

* 由于生成器函数可以暂停，因此我们可以在生成器内部编写完整的异步逻辑代码，不过**生成器依然需要使用额外的 co 函数来驱动生成器函数的执行，这一点非常不友好。**
*   基于这个原因，ES7 引入了 `async/await`，这是 `JavaScript` 异步编程的一个重大改进，它改进了生成器的缺点，提供了在**不阻塞主线程的情况下使用同步代码实现异步访问资源的能力**。你可以参考下面这段使用 `async/await` 改造后的代码：

    ```js
    async function getResult() {
        try {
            let id_res = await fetch(id_url)
            let id_text = await id_res.text()
            console.log(id_text)
      
            let new_name_url = name_url+"?id="+id_text
            console.log(new_name_url)

            let name_res = await fetch(new_name_url)
            let name_text = await name_res.text()
            console.log(name_text)
        } catch (err) {
            console.error(err)
        }
    }
    getResult()
    ```
*   观察上面这段代码，你会发现整个异步处理的逻辑都是使用同步代码的方式来实现的，而且还支持 `try catch` 来捕获异常，这就是完全在写同步代码，所以非常符合人的线性思维。

    虽然这种方式看起来像是同步代码，但是**实际上它又是异步执行的**，也就是说，**在执行到 `await fetch` 的时候，整个函数会暂停等待 `fetch` 的执行结果，等到函数返回时，再恢复该函数，然后继续往下执行。**

    **其实 `async/await` 技术背后的秘密就是 `Promise` 和生成器应用**，往底层说，就是微任务和协程应用。要搞清楚 `async` 和 `await` 的工作原理，我们就得对 `async` 和`await`分开分析。

    我们先来看看 `async` 到底是什么。根据 `MDN` 定义，`async` 是一个通过异步执行并隐式返回 `Promise` 作为结果的函数。

    这里需要重点关注异步执行这个词，简单地理解，如果在 `async` 函数里面使用了 `await`，那么此时 `async` 函数就会暂停执行，并等待合适的时机来恢复执行，所以说 `async` 是一个异步执行的函数。

    那么暂停之后，什么时机恢复 `async` 函数的执行呢？

    要解释这个问题，我们先来看看，`V8` 是如何处理 `await` 后面的内容的。

    通常，`await` 可以等待两种类型的表达式：

    1. 可以是任何普通表达式 ;
    2. 也可以是一个 `Promise` 对象的表达式。

    **如果 `await` 等待的是一个 `Promise` 对象，它就会暂停执行生成器函数，直到 `Promise` 对象的状态变成 `resolve`，才会恢复执行，然后得到 `resolve` 的值，作为 `await` 表达式的运算结果**。
*   我们看下面这样一段代码：

    ```js
    function NeverResolvePromise(){
        return new Promise((resolve, reject) => {})
    }
    async function getResult() {
        let a = await NeverResolvePromise()
        console.log(a)
    }
    getResult()
    console.log(0)
    ```

    这一段代码，我们使用`await`等待一个没有 `resolve`的 `Promise`，那么这也就意味着，`getResult` 函数会一直等待下去。

    和生成器函数一样，**使用了 `async` 声明的函数在执行时，也是一个单独的协程，我们可以使用 `await` 来暂停该协程，由于 `await` 等待的是一个`Promise`对象，我们可以`resolve`来恢复该协程**。
*   下面是我从协程的视角，画的这段代码的执行流程图，你可以对照参考下：

    <img src="https://static001.geekbang.org/resource/image/5f/f6/5fd7a95e6dd2ee6dda588641e2eecaf6.jpg" alt="img" data-size="original" />
*   如果 await 等待的对象已经变成了 resolve 状态，那么 V8 就会恢复该协程的执行，我们可以修改下上面的代码，来证明下这个过程：

    ```js
    function HaveResolvePromise(){
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                resolve(100)
              }, 0);
          })
    }
    async function getResult() {
        console.log(1)
        let a = await HaveResolvePromise()
        console.log(a)
        console.log(2)
    }
    console.log(0)
    getResult()
    console.log(3)
    ```

    现在，这段代码的执行流程就非常清晰了，具体执行流程你可以参看下图：

    <img src="https://static001.geekbang.org/resource/image/1c/7f/1c7bc077282dfa996746e5c403c42f7f.jpg" alt="img" data-size="original" />
*   如果 `await` 等待的是一个非 `Promise` 对象，比如 `await 100`，那么 `V8` 会隐式地将`await`后面的 `100` 包装成一个已经`resolve`的对象，其效果等价于下面这段代码：

    ```js
    function ResolvePromise(){
        return new Promise((resolve, reject) => {
                resolve(100)
          })
    }
    async function getResult() {
        let a = await ResolvePromise()
        console.log(a)
    }
    getResult()
    console.log(3)
    ```

### 总结

* `Callback` 模式的异步编程模型需要实现大量的回调函数，大量的回调函数会打乱代码的正常逻辑，使得代码变得不线性、不易阅读，这就是我们所说的**回调地狱问题**。
* 使用 `Promise` 能很好地解决回调地狱的问题，我们可以按照线性的思路来编写代码，这个过程是线性的，非常符合人的直觉。
* 但是这种方式充满了 `Promise` 的 `then()` 方法，如果处理流程比较复杂的话，那么整段代码将充斥着大量的 `then`，**语义化不明显，代码不能很好地表示执行流程**。
* 我们想要通过线性的方式来编写异步代码，要实现这个理想，最关键的是要能实现函数暂停和恢复执行的功能。而生成器就可以实现函数暂停和恢复，我们可以在生成器中使用同步代码的逻辑来异步代码 (实现该逻辑的核心是协程)，**但是在生成器之外，我们还需要一个触发器来驱动生成器的执行，因此这依然不是我们最终想要的方案。**
* 我们的最终方案就是 `async/await`，`async` 是一个可以暂停和恢复执行的函数，我们会在 `async` 函数内部使用 `await` 来暂停 `async` 函数的执行，`await` 等待的是一个 `Promise` 对象，如果 `Promise` 的状态变成 `resolve` 或者 `reject`，那么 `async` 函数会恢复执行。因此，使用 `async/await` 可以实现以同步的方式编写异步代码这一目标。
*   你会发现，这节课我们讲的也是前端异步编程的方案史，我把这一过程也画了一张图供你参考：

    <img src="https://static001.geekbang.org/resource/image/6e/f3/6e0508be2a2444cba8ade6230610f4f3.jpg" alt="img" data-size="original" />

### 思考题

*   了解 async/await 的演化过程，对于理解 async/await 至关重要，在进化过程中，co+generator 是比较优秀的一个设计。今天留给你的思考题是，co 的运行原理是什么？

    co源码实现原理：其实就是通过不断的调用generator函数的next()函数，来达到自动执行generator函数的效果（类似async、await函数的自动自行）。 具体代码分析，我之前写过一篇文章： [学习 koa 源码的整体架构，浅析koa洋葱模型原理和co原理](https://juejin.im/entry/5e6a080af265da575b1bd160)

### [更多参考](https://blog.csdn.net/Water\_Cyan/article/details/110500498)

## [垃圾回收（一）：V8的两个垃圾回收器是如何工作的？](https://time.geekbang.org/column/article/230845)

我们都知道，JavaScript 是一门自动垃圾回收的语言，也就是说，我们不需要去手动回收垃圾数据，这一切都交给 V8 的垃圾回收器来完成。V8 为了更高效地回收垃圾，引入了两个垃圾回收器，它们分别针对着不同的场景。

那这两个回收器究竟是如何工作的呢，这节课我们就来分析这个问题。

### 垃圾数据是怎么产生的？

* 首先，我们看看垃圾数据是怎么产生的。
* 无论是使用什么语言，我们都会频繁地使用数据，这些数据会被存放到栈和堆中，通常的方式是在内存中创建一块空间，使用这块空间，在不需要的时候回收这块空间。
*   比如下面这样一句代码：

    ```js
    window.test = new Object()
    window.test.a = new Uint16Array(100)
    ```

    当 `JavaScript` 执行这段代码的时候，会先为`window`对象添加一个`test`属性，并在堆中创建了一个空对象，并将该对象的地址指向了 `window.test` 属性。随后又创建一个大小为 100 的数组，并将属性地址指向了 `test.a` 的属性值。此时的内存布局图如下所示：

    ![42b70203c6da641831d778ce08a7a5b1](https://user-images.githubusercontent.com/17645053/215237671-25b99a46-bffb-4ef1-b5af-5b6b8d4cc863.jpg)

    我们可以看到，**栈中保存了指向 `window` 对象的指针**，通过栈中 `window` 的地址，我们可以到达`window`对象，通过 `window` 对象可以到达 `test` 对象，通过`test`对象还可以到达 `a` 对象。
*   如果此时，我将另外一个对象赋给了 a 属性，代码如下所示：

    ```js
    window.test.a = new Object()
    ```

    那么此时的内存布局如下所示：

    ![](https://static001.geekbang.org/resource/image/9c/dc/9c44e8bb2a75a8da72877c8a192967dc.jpg)

    我们可以看到，`a` 属性之前是指向堆中数组对象的，现在已经指向了另外一个空对象，那么此时堆中的数组对象就成为了垃圾数据，因为我们无法从一个根对象遍历到这个`Array`对象。

    不过，你不用担心这个数组对象会一直占用内存空间，因为 V8 虚拟机中的垃圾回收器会帮你自动清理。

### 垃圾回收算法

* 那么垃圾回收是怎么实现的呢？大致可以分为以下几个步骤：
  1.  通过 GC Root 标记空间中活动对象和非活动对象。

      目前 V8 采用的**可访问性（reachability）**算法来判断堆中的对象是否是活动对象。具体地讲，这个算法是将一些 **GC Root** 作为初始存活的对象的集合，从 GC Roots 对象出发，遍历 GC Root 中的所有对象：

      * **通过 GC Root 遍历到的对象，我们就认为该对象是可访问的（reachable）**，那么必须保证这些对象应该在内存中保留，我们也称可访问的对象为活动对象；
      * **通过 GC Roots 没有遍历到的对象，则是不可访问的（unreachable）**，那么这些不可访问的对象就可能被回收，我们称不可访问的对象为非活动对象。

      在浏览器环境中，GC Root 有很多，通常包括了以下几种 (但是不止于这几种)：

      1. 全局的 window 对象（位于每个 iframe 中）；
      2. 文档 DOM 树，由可以通过遍历文档到达的所有原生 DOM 节点组成；
      3. 存放栈上变量。
  2. 回收非活动对象所占据的内存。其实就是在所有的标记完成之后，**统一清理内存中所有被标记为可回收的对象**。
  3. 做内存整理。一般来说，频繁回收对象后，内存中就会存在大量不连续空间，我们把这些**不连续的内存空间称为内存碎片**。当内存中出现了大量的内存碎片之后，如果需要分配较大的连续内存时，就有可能出现内存不足的情况，所以最后一步需要整理这些内存碎片。但这步其实是可选的，因为有的垃圾回收器不会产生内存碎片，比如接下来我们要介绍的副垃圾回收器。
*   以上就是大致的垃圾回收的流程。目前 V8 采用了两个垃圾回收器，**主垃圾回收器 -Major GC 和副垃圾回收器 -Minor GC (Scavenger)**。V8 之所以使用了两个垃圾回收器，主要是受到了**代际假说（The Generational Hypothesis）的影响**。

    代际假说是垃圾回收领域中一个重要的术语，它有以下两个特点：

    1. 大部分对象都是“朝生夕死”的，也就是说**大部分对象在内存中存活的时间很短**，比如函数内部声明的变量，或者块级作用域中的变量，当函数或者代码块执行结束时，作用域中定义的变量就会被销毁。因此这一类对象一经分配内存，很快就变得不可访问；
    2. 是不死的对象，会活得更久，比如全局的 `window`、`DOM`、`Web API` 等对象。

    其实这两个特点不仅仅适用于 `JavaScript`，同样适用于大多数的编程语言，如 `Java`、`Python` 等。

    `V8` 的垃圾回收策略，就是建立在该假说的基础之上的。接下来，我们来分析下 V8 是如何实现垃圾回收的。

    **如果我们只使用一个垃圾回收器，在优化大多数新对象的同时，就很难优化到那些老对象**，因此你需要权衡各种场景，根据对象生存周期的不同，而使用不同的算法，以便达到最好的效果。

    所以，在 V8 中，会把堆分为新生代和老生代两个区域，**新生代中存放的是生存时间短的对象，老生代中存放生存时间久的对象。**

    新生代通常只支持 1～8M 的容量，而老生代支持的容量就大很多了。对于这两块区域，V8 分别使用两个不同的垃圾回收器，以便更高效地实施垃圾回收。

    1. 副垃圾回收器 -**Minor GC (Scavenger)**，主要负责**新生代**的垃圾回收。
    2. 主垃圾回收器 -**Major GC**，主要负责**老生代**的垃圾回收。

### 副垃圾回收器

* 副垃圾回收器主要负责新生代的垃圾回收。通常情况下，大多数小的对象都会被分配到新生代，所以说这个区域虽然不大，但是垃圾回收还是比较频繁的。
*   新生代中的垃圾数据用 **Scavenge 算法**来处理。所谓 Scavenge 算法，是把新生代空间对半划分为两个区域，一半是**对象区域 (from-space)**，一半是**空闲区域 (to-space)**，如下图所示：

    <img src="https://static001.geekbang.org/resource/image/75/9d/75329eceafd88573097f8d073430bc9d.jpg" alt="img" data-size="original" />
* **新加入的对象都会存放到对象区域**，当**对象区域快被写满时，就需要执行一次垃圾清理操作**。
*   在垃圾回收过程中，首先要对对象区域中的垃圾做标记；标记完成之后，就进入垃圾清理阶段。**副垃圾回收器会把这些存活的对象复制到空闲区域中，同时它还会把这些对象有序地排列起来，所以这个复制过程，也就相当于完成了内存整理操作，复制后空闲区域就没有内存碎片了**。

    ![](https://static001.geekbang.org/resource/image/12/87/12519a0d1f2484cd24297e821f2f1887.jpg)
*   完成复制后，对象区域与空闲区域进行角色翻转，也就是原来的对象区域变成空闲区域，原来的空闲区域变成了对象区域。这样就完成了垃圾对象的回收操作，同时，**这种角色翻转的操作还能让新生代中的这两块区域无限重复使用下去。**

    ![](https://static001.geekbang.org/resource/image/79/bd/797db43b27c8a6add1ffa540910c7ebd.jpg)
* 不过，副垃圾回收器每次执行清理操作时，都需要将存活的对象从对象区域复制到空闲区域，复制操作需要时间成本，如果新生区空间设置得太大了，那么每次清理的时间就会过久，所以**为了执行效率，一般新生区的空间会被设置得比较小**。
* 也正是因为新生区的空间不大，所以很容易被存活的对象装满整个区域，副垃圾回收器一旦监控对象装满了，便执行垃圾回收。同时，副垃圾回收器还会采用**对象晋升策略**，也就是**移动那些经过两次垃圾回收依然还存活的对象到老生代中。**

### 主垃圾回收器

* 主垃圾回收器主要负责老生代中的垃圾回收。除了新生代中晋升的对象，一些大的对象会直接被分配到老生代里。因此，老生代中的对象有两个特点：
  1. 对象占用空间大；
  2. 对象存活时间长。
* 由于老生代的对象比较大，**若要在老生代中使用 Scavenge 算法进行垃圾回收，复制这些大的对象将会花费比较多的时间，从而导致回收执行效率不高**，同时还会浪费一半的空间。所以，主垃圾回收器是采用**标记 - 清除（Mark-Sweep）的算法**进行垃圾回收的。
* 那么，标记 - 清除算法是如何工作的呢？
  1. 标记过程阶段。标记阶段就是从一组根元素开始，递归遍历这组根元素，在这个遍历过程中，**能到达的元素称为活动对象，没有到达的元素就可以判断为垃圾数据**。
  2. 垃圾的清除过程。它和副垃圾回收器的垃圾清除过程完全不同，主垃圾回收器会直接将标记为垃圾的数据清理掉。
*   你可以理解这个过程是清除掉下图中红色标记数据的过程，你可参考下图大致理解下其清除过程：

    <img src="https://static001.geekbang.org/resource/image/c7/9b/c70cdb85c0b656061e4cc420efdaf59b.jpg" alt="img" data-size="original" />
* 对垃圾数据进行标记，然后清除，这就是**标记 - 清除算法**，不过对一块内存多次执行标记 - 清除算法后，会产生大量不连续的内存碎片。而碎片过多会导致大对象无法分配到足够的连续内存，于是又引入了另外一种算法——**标记 - 整理（Mark-Compact**）。
*   这个算法的标记过程仍然与标记 - 清除算法里的是一样的，\*\*先标记可回收对象，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉这一端之外的内存。\*\*你可以参考下图：

    <img src="https://static001.geekbang.org/resource/image/6a/d8/6a558a6731fd68757e1a43c1dbc27ed8.jpg" alt="img" data-size="original" />

### 总结

* 今天，我们先分析了什么是垃圾数据，从“GC Roots”对象出发，遍历 GC Root 中的所有对象，如果通过 GC Roots 没有遍历到的对象，则这些对象便是垃圾数据。V8 会有专门的垃圾回收器来回收这些垃圾数据。
* V8 依据代际假说，将堆内存划分为新生代和老生代两个区域，新生代中存放的是生存时间短的对象，老生代中存放生存时间久的对象。为了提升垃圾回收的效率，V8 设置了两个垃圾回收器，主垃圾回收器和副垃圾回收器。主垃圾回收器负责收集老生代中的垃圾数据，副垃圾回收器负责收集新生代中的垃圾数据。
* **副垃圾回收器采用了 Scavenge 算法**，是把新生代空间对半划分为两个区域，一半是对象区域，一半是空闲区域。新的数据都分配在对象区域，等待对象区域快分配满的时候，垃圾回收器便执行垃圾回收操作，之后将存活的对象从对象区域拷贝到空闲区域，并将两个区域互换。
* 主垃圾回收器回收器主要负责老生代中的垃圾数据的回收操作，会经历**标记、清除和整理**过程。

## [垃圾回收（二）：V8是如何优化垃圾回收器执行效率的？](https://time.geekbang.org/column/article/232138)

上节我们介绍了 V8 使用副垃圾回收器和主垃圾回收器来处理垃圾回收，这节课我们看看 V8 是如何优化垃圾回收器的执行效率的。

由于 JavaScript 是运行在主线程之上的，因此，一旦执行垃圾回收算法，都需要将正在执行的 JavaScript 脚本暂停下来，待垃圾回收完毕后再恢复脚本执行。我们把这种行为叫做**全停顿（Stop-The-World）**。

**一次完整的垃圾回收分为标记和清理两个阶段**，垃圾数据标记之后，V8 会继续执行清理和整理操作，虽然主垃圾回收器和副垃圾回收器的处理方式稍微有些不同，但它们都是主线程上执行的，执行垃圾回收过程中，会暂停主线程上的其他任务，具体全停顿的执行效果如下图所示：

![img](https://static001.geekbang.org/resource/image/90/23/9004196c53f2f381a1321bcbc346fc23.jpg?wh=2284\*709)

![image](https://img.imyangyong.com/blog/2019-12-19%2014-53-46.png)

可以看到，执行垃圾回收时会占用主线程的时间，如果在执行垃圾回收的过程中，垃圾回收器占用主线程时间过久，就像上面图片展示的那样，花费了 200 毫秒，在这 200 毫秒内，主线程是不能做其他事情的。**比如，页面正在执行一个 `JavaScript` 动画，因为垃圾回收器在工作，就会导致这个动画在这 200 毫秒内无法执行，造成页面的卡顿 (Jank)，用户体验不佳**。

为了解决全停顿而造成的用户体验的问题，V8 团队经过了很多年的努力，向现有的垃圾回收器添加**并行、并发和增量等垃圾回收技术**，并且也已经取得了一些成效。这些技术主要是从两方面来解决垃圾回收效率问题的：

1. **将一个完整的垃圾回收的任务拆分成多个小的任务**，这样就消灭了单个长的垃圾回收任务；
2. **将标记对象、移动对象等任务转移到后台线程进行**，这会大大减少主线程暂停的时间，改善页面卡顿的问题，让动画、滚动和用户交互更加流畅。

接下来，我们就来深入分析下，V8 是怎么向现有的垃圾回收器添加并行、并发和增量等技术，来提升垃圾回收执行效率的。

### 并行回收(Parallel)

* 既然执行一次完整的垃圾回收过程比较耗时，那么解决效率问题，**第一个思路就是主线程在执行垃圾回收的任务时，引入多个辅助线程来并行处理**，这样就会加速垃圾回收的执行速度，因此 V8 团队引入了并行回收机制。
*   所谓**并行回收，是指垃圾回收器在主线程上执行的过程中，还会开启多个协助线程，同时执行同样的回收工作**，其工作模式如下图所示：

    <img src="https://static001.geekbang.org/resource/image/00/1f/00537bdadac433a57c77c56c5cc33c1f.jpg?wh=2284*793" alt="img" data-size="original" />
* 采用并行回收时，**垃圾回收所消耗的时间，等于总体辅助线程所消耗的时间（辅助线程数量乘以单个线程所消耗的时间），再加上一些同步开销的时间**。这种方式比较简单，因为在执行垃圾标记的过程中，主线程并不会同时执行`JavaScript`代码，因此 `JavaScript` 代码也不会改变回收的过程。所以我们可以假定内存状态是静态的，因此只要确保同时只有一个协助线程在访问对象就好了。
* V8 的**副垃圾回收器所采用的就是并行策略**，它在执行垃圾回收的过程中，**启动了多个线程来负责新生代中的垃圾清理操作**，这些线程同时将对象空间中的数据移动到空闲区域。由于数据的地址发生了改变，所以还需要同步更新引用这些对象的指针。

### 增量回收(Incremental marking)

* 虽然**并行策略能增加垃圾回收的效率，能够很好地优化副垃圾回收器，但是这**仍然是一种全停顿的垃圾回收方式\*\*，在主线程执行回收工作的时候才会开启辅助线程，这依然还会存在效率问题。比如老生代存放的都是一些大的对象，如 window、DOM 这种，**完整执行老生代的垃圾回收，时间依然会很久。这**些大的对象都是主垃圾回收器的，所以在 2011 年，V8 又引入了增量标记的方式，我们把这种垃圾回收的方式称之为增量式垃圾回收\*\*。
*   所谓增量式垃圾回收，是指垃圾收集器将标记工作分解为更小的块，并且穿插在主线程不同的任务之间执行。采用增量垃圾回收时，垃圾回收器没有必要一次执行完整的垃圾回收过程，每次执行的只是整个垃圾回收过程中的一小部分工作，具体流程你可以参看下图：

    <img src="https://static001.geekbang.org/resource/image/be/6f/be18e6dc6c93e761a37d50aed48f246f.jpg?wh=2284*707" alt="img" data-size="original" />
* 增量标记的算法，比全停顿的算法要稍微复杂，这主要是因为**增量回收是并发的（concurrent）**，要实现增量执行，需要满足两点要求：
  1. 垃圾回收可以被随时暂停和重启，暂停时需要保存当时的扫描结果，等下一波垃圾回收来了之后，才能继续启动。
  2. 在暂停期间，被标记好的垃圾数据如果被 JavaScript 代码修改了，那么垃圾回收器需要能够正确地处理。
*   我们先来看看第一点，**V8 是如何实现垃圾回收器的暂停和恢复执行的**。

    这里我们需要知道，在没有采用增量算法之前，V8 使用黑色和白色来标记数据。在执行一次完整的垃圾回收之前，垃圾回收器会将所有的数据设置为白色，用来表示这些数据还没有被标记，然后垃圾回收器在会**从 GC Roots 出发，将所有能访问到的数据标记为黑色。遍历结束之后，被标记为黑色的数据就是活动数据，那些白色数据就是垃圾数据**。如下图所示：

    <img src="https://static001.geekbang.org/resource/image/e1/0c/e1409de965aaab9bbf401249b1e02d0c.jpg?wh=2284*1285" alt="img" data-size="original" />

    **如果内存中的数据只有两种状态，非黑即白，那么当你暂停了当前的垃圾回收器之后，再次恢复垃圾回收器，那么垃圾回收器就不知道从哪个位置继续开始执行了**。

    比如垃圾回收器执行了一小段增量回收后，被 V8 暂停了，然后主线程执行了一段 JavaScript 代码，然后垃圾回收器又被恢复了，那么恢复时内存状态就如下图所示：

    <img src="https://static001.geekbang.org/resource/image/2c/4d/2cd1ef856522ad0c8176c60b75e63c4d.jpg?wh=2284*1285" alt="img" data-size="original" />

    那么，当垃圾回收器再次被启动的时候，它到底是从 A 节点开始标记，还是从 B 节点开始执行标注过程呢？因为没有其他额外的信息，所以垃圾回收器也不知道该如何处理了。
* 为了解决这个问题，V8 采用了**三色标记法**，除了黑色和白色，还额外引入了灰色：
  1. **黑色**表示这个节点被 GC Root **引用到了，而且该节点的子节点都已经标记完成了** ;
  2. **灰色**表示这个节点被 GC Root 引用到，但**子节点还没被垃圾回收器标记处理，也表明目前正在处理这个节点**；
  3. **白色**表示这个节点**没有被访问到，如果在本轮遍历结束时还是白色，那么这块数据就会被收回**。
*   **引入灰色标记之后，垃圾回收器就可以依据当前内存中有没有灰色节点，来判断整个标记是否完成，如果没有灰色节点了，就可以进行清理工作了**。如果还有灰色标记，当下次恢复垃圾回收器时，便从灰色的节点开始继续执行。

    因此采用三色标记，可以很好地支持增量式垃圾回收。
*   接下来，我们再来分析下，标记好的垃圾数据被 JavaScript 修改了，V8 是如何处理的。我们看下面这样的一个例子：

    ```js
    window.a = Object()
    window.a.b = Object()
    window.a.b.c=Object() 
    ```

    执行到这段代码时，垃圾回收器标记的结果如下图所示：

    <img src="https://static001.geekbang.org/resource/image/19/17/19bbfb35d274064b253814b58413bc17.jpg?wh=2284*700" alt="img" data-size="original" />

    然后又执行了另外一个代码，这段代码如下所示：

    ```js
    window.a.b = Object() //d
    ```

    执行完之后，垃圾回收器又恢复执行了增量标记过程，由于 b 重新指向了 d 对象，所以 b 和 c 对象的连接就断开了。这时候代码的应用如下图所示：

    <img src="https://static001.geekbang.org/resource/image/75/15/759f6a8105d64d3aebdc16f81a2b5e15.jpg?wh=2284*767" alt="img" data-size="original" />

    这就说明一个问题，**当垃圾回收器将某个节点标记成了黑色，然后这个黑色的节点被续上了一个白色节点，那么垃圾回收器不会再次将这个白色(d)节点标记为黑色节点了，因为它已经走过这个路径了。**

    但是**这个新的白色节点(d)的确被引用了，所以我们还是需要想办法将其标记为黑色**。

    为了解决这个问题，增量垃圾回收器添加了一个约束条件：**不能让黑色节点指向白色节点**。

    通常我们使用**写屏障 (Write-barrier) 机制**实现这个约束条件，也就是说，**当发生了黑色的节点引用了白色的节点，写屏障机制会强制将被引用的白色节点变成灰色的，这样就保证了黑色节点不能指向白色节点的约束条件**。这个方法也被称为**强三色不变性**，它保证了垃圾回收器能够正确地回收数据，因为在标记结束时的所有白色对象，对于垃圾回收器来说，都是不可到达的，可以安全释放。

    所以在 V8 中，每次执行如 `window.a.b = value` 的写操作之后，**V8 会插入写屏障代码，强制将 value(d) 这块内存标记为灰色**。

### 并发 (concurrent) 回收

* 虽然通过\*\*三色标记法(_tri-color_ marking)和写屏障(Write-barrier)\*\*机制可以很好地实现增量垃圾回收，**但是由于这些操作都是在主线程上执行的，如果主线程繁忙的时候，增量垃圾回收操作依然会增加主线程处理任务的吞吐量 (throughput)**。
* 结合并行回收可以将一些任务分配给辅助线程，但是并行回收依然会阻塞主线程，那么，有没有办法在不阻塞主线程的情况下，执行垃圾回收操作呢？
* 还真有，这就是我们要来重点研究的并发回收机制了。
*   所谓并发回收，**是指主线程在执行 JavaScript 的过程中，辅助线程能够在后台完成执行垃圾回收的操作**。并发标记的流程大致如下图所示：

    <img src="https://static001.geekbang.org/resource/image/15/c2/157052aa087c840f5f58a7708f30bdc2.jpg?wh=2284*1213" alt="img" data-size="original" />

    并发回收的优势非常明显，**主线程不会被挂起，JavaScript 可以自由地执行 ，在执行的同时，辅助线程可以执行垃圾回收操作**。

    但是并发回收却是这**三种技术中最难的一种**，这主要由以下两个原因导致的：

    1. 当**主线程执行 JavaScript 时，堆中的内容随时都有可能发生变化**，从而使得辅助线程之前做的工作完全无效；
    2. **主线程和辅助线程极有可能在同一时间去更改同一个对象**，这就需要**额外实现读写锁**的一些功能了。
*   尽管并行回收要额外解决以上两个问题，但是权衡利弊，**并行回收这种方式的效率还是远高于其他方式的**。

    不过，这**三种技术在实际使用中，并不是单独的存在，通常会将其融合在一起使用，V8 的主垃圾回收器就融合了这三种机制，来实现垃圾回收**，那它具体是怎么工作的呢？你可以先看下图：

    <img src="https://static001.geekbang.org/resource/image/7b/42/7b8b901cb2eb575bb8907e1ad7dc1842.jpg?wh=2284*1105" alt="img" data-size="original" />

    可以看出来，主垃圾回收器同时采用了这三种策略：

    1. 首先**主垃圾回收器主要使用并发标记**，我们可以看到，在主线程执行 JavaScript，辅助线程就开始执行标记操作了，所以说**标记是在辅助线程中完成的**。
    2. 标记完成之后，再执行**并行清理操作**。主线程在执行清理操作时，多个辅助线程也在执行清理操作。
    3. 另外，**主垃圾回收器还采用了增量标记的方式**，清理的任务会穿插在各种 JavaScript 任务之间执行。

### [更多参考](https://www.imyangyong.com/blog/2019/07/node/V8%E5%BC%95%E6%93%8E%E7%9A%84%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E6%9C%BA%E5%88%B6/)

### [**V8当前垃圾回收机制参考**](https://developer.51cto.com/article/627316.html)

**副垃圾回收器** V8在**新生代垃圾回收中，使用并行（parallel）机制，在整理排序阶段**(如上总结部分)，也就是将活动对象从from-to复制到space-to的时候，**启用多个辅助线程，并行的进行整理**。由于多个线程竞争一个新生代的堆的内存资源，可能出现有某个活动对象被多个线程进行复制操作的问题，为了解决这个问题，V8在第一个线程对活动对象进行复制并且复制完成后，都必须去维护复制这个活动对象后的指针转发地址，以便于其他协助线程可以找到该活动对象后可以判断该活动对象是否已被复制。

![Screen Shot 2022-04-07 at 3 06 57 PM](https://user-images.githubusercontent.com/17645053/162140309-03201917-18f4-44f5-b459-85e955d82345.png)

**主垃圾回收器** V8在**老生代垃圾回收中，如果堆中的内存大小超过某个阈值之后，会启用并发（Concurrent）标记任务**(如上总结部分)。每个辅助线程都会去追踪每个标记到的对象的指针以及对这个对象的引用，而在JavaScript代码执行时候，并发标记也在后台的辅助进程中进行，当堆中的某个对象指针被JavaScript代码修改的时候，写入屏障（write barriers）技术会在辅助线程在进行并发标记的时候进行追踪。

当并发标记完成或者动态分配的内存到达极限的时候，主线程会执行最终的快速标记步骤，这个时候主线程会挂起，主线程会再一次的扫描根集以确保所有的对象都完成了标记，由于辅助线程已经标记过活动对象，主线程的本次扫描只是进行check操作，确认完成之后，某些辅助线程会进行清理内存操作，某些辅助进程会进行内存整理操作，由于都是并发的，并不会影响主线程JavaScript代码的执行。

## [答疑：几种常见内存问题的解决策略](https://time.geekbang.org/column/article/232492)

今天这节答疑课，我们来结合 Node 中的读文件操作，分析下消息循环系统是怎么影响到异步编程的，然后我们再来结合 JavaScript 中的几种常见的内存问题，来分析下内存问题出现的原因和解决方法。

### `Node` 中的 `readFile API` 工作机制

*   `Node` 中很多 `API` 都提供了同步和异步两种形式，下面我们来看下《17 | 消息队列：V8 是怎么实现回调函数的？》这节课留的思考题。思考题中有两段代码，我们通过这两段代码来分析下同步和异步读文件 API 的区别。

    ```js
    var fs = require('fs')
    var data = fs.readFileSync('test.js')
    ```

    ```js
    function fileHanlder(err, data){
      data.toString()  
    }

    fs.readFile('test.txt', fileHanlder)
    ```
*   在解答这个问题之前，我们来看看 Node 的体系架构。你可以先参考下图：

    <img src="https://static001.geekbang.org/resource/image/b2/eb/b2894f2297a23a9d706d0517610deeeb.jpg" alt="img" data-size="original" />

    **`Node` 是 `V8` 的宿主，它会给 `V8` 提供事件循环和消息队列**。在 `Node` 中，事件循环是由 `libuv` 提供的，`libuv` 工作在主线程中，它会从消息队列中取出事件，并在主线程上执行事件。

    同样，对**于一些主线程上不适合处理的事件**，比如消耗时间过久的网络资源下载、文件读写、设备访问等，**`Node` 会提供很多线程来处理这些事件，我们把这些线程称为线程池**。
*   通常，在 `Node` 中，我们认为读写文件是一个非常耗时的工作，因此**主线程会将回调函数和读文件的操作一道发送给文件读写线程**，并**让实际的读写操作运行在读写线程中**。

    比如当在 `Node` 的主线程上执行 `readFile` 的时候，主线程会将`readFile`的文件名称和回调函数，提交给文件读写线程来处理，具体过程如下所示：

    <img src="https://static001.geekbang.org/resource/image/65/ff/654cccf962dccd2797bd1267ab82b9ff.jpg" alt="img" data-size="original" />

    文件读写线程完成了文件读取之后，会**将结果和回调函数封装成新的事件，并将其添加进消息队列中**。比如文件线程将读取的文件内容存放在内存中，并将 `data` 指针指向了该内存，然后文件读写线程会将 `data` 和回调函数封装成新的事件，并将其丢进消息队列中，具体过程如下所示：

    <img src="https://static001.geekbang.org/resource/image/da/99/daaad54f06e7bb25dbb3b8174f55bf99.jpg" alt="img" data-size="original" />

    等到`libuv`从消息队列中读取该事件后，主线程就可以着手来处理该事件了。**在主线程处理该事件的过程中，主线程调用事件中的回调函数`fileHandler`，并将 `data` 结果数据作为参数**，如下图所示：

    <img src="https://static001.geekbang.org/resource/image/b9/c2/b9e3c603cfa7d3f47178c06ffd945fc2.jpg" alt="img" data-size="original" />

    然后在回调函数中，我们就可以拿到读取的结果来实现一些业务逻辑了。
*   不过，总有些人觉得异步读写文件操作过于复杂了，**如果读取的文件体积不大或者项目瓶颈不在文件读写，那么依然使用异步调用和回调函数的模式就显得有点过度复杂了**。

    因此 `Node` 还提供了一套**同步读写的 `API`**。第一段代码中的 `readFileSync` 就是同步实现的，同步代码非常简单，当 `libuv` 读取到 `readFileSync` 的任务后，就**直接在主线程上执行读写操作**，等待读写结束，直接返回读写的结果，这也是**同步回调的一种应用**。当然在读写过程中，消息队列中的其他任务是无法被执行的。

    所以在选择使用同步 API 还是异步 API 时，我们要看实际的场景，并不是非 A 即 B。

### 几种内存问题

* 分析了异步 `API`，接下来我们再来看看 `JavaScript` 中的内存问题，内存问题至关重要，因为通过内存而造成的问题很容易被用户察觉。总的来说，内存问题可以定义为下面这三类：
  1. 内存泄漏 (Memory leak)，它会导致页面的性能越来越差；
  2. 内存膨胀 (Memory bloat)，它会导致页面的性能会一直很差；
  3. 频繁垃圾回收，它会导致页面出现延迟或者经常暂停。

### 内存泄漏

* 我们先看内存泄漏。本质上，内存泄漏可以定义为：**当进程不再需要某些内存的时候，这些不再被需要的内存依然没有被进程回收**。
* 在 `JavaScript` 中，造成内存泄漏 (Memory leak) 的**主要原因是不再需要 (没有作用) 的内存数据依然被其他对象引用着**。
*   下面我们就来看几种实际的例子：

    我们知道，JavaScript 是一门非常宽松的语言，你甚至可以使用一个**未定义的变量**，比如下面这样一段代码：

    ```js
    function foo() {
        //创建一个临时的temp_array
        temp_array = new Array(200000)
       /**
        * 使用temp_array
        */
    }
    ```

    当执行这段代码时，由于函数体内的对象**没有被`var、let、const`这些关键字声明，那么 V8 就会使用 `this.temp_array` 替换 `temp_array`**。

    ```js
    function foo() {
        //创建一个临时的temp_array
        this.temp_array = new Array(200000)
       /**
        * this.temp_array
        */
    }
    ```

    在浏览器，**默认情况下，`this` 是指向 `window` 对象的，而 `window` 对象是常驻内存的，所以即便`foo`函数退出了，但是 `temp_array` 依然被`window` 对象引用了， 所以`temp_array`依然也会和 `window` 对象一样，会常驻内存**。因为 `temp_array` 已经是不再被使用的对象了，但是依然被`window`对象引用了，这就造成了 `temp_array` 的泄漏。

    为了解决这个问题，我们可以在 `JavaScript` 文件头部加上`use strict`，使用**严格模式避免意外的全局变量，此时上例中的 `this` 指向 `undefined`。**
*   另外，我们还要时刻**警惕闭包**这种情况，因为**闭包会引用父级函数中定义的变量，如果引用了不被需要的变量，那么也会造成内存泄漏**。比如你可以看下面这样一段代码：

    ```js
    function foo(){  
        var temp_object = new Object()
        temp_object.x = 1
        temp_object.y = 2
        temp_object.array = new Array(200000)
        /**
        *   使用temp_object
        */
        return function(){
            console.log(temp_object.x);
        }
    }
    ```

    可以看到，`foo` 函数使用了一个局部临时变量 `temp_object`，`temp_object` 对象有三个属性，`x、y`，还有一个非常占用内存的 `array` 属性。最后`foo`函数返回了一个匿名函数，该匿名函数引用了 `temp_object.x`。那么当调用完`foo`函数之后，由于返回的匿名函数引用了`foo`函数中的 `temp_object.x`，这会造成 `temp_object` 无法被销毁，即便只是引用了 `temp_object.x`，也会造成整个`temp_object`对象依然保留在内存中。我们可以通过 Chrome 调试工具查看下：

    <img src="https://static001.geekbang.org/resource/image/ff/10/ff81eec387d021a3b4a3d019c09cbb10.jpg" alt="img" data-size="original" />

    从上图可以看出，我们仅仅是需要 `temp_object.x` 的值，`V8` 却保留了整个 `temp_object` 对象。 要解决这个问题，我就需要根据实际情况，来判断闭包中返回的函数到底需要引用什么数据，不需要引用的数据就绝不引用，因为上面例子中，返回函数中只需要`temp_object.x`的值，因此我们可以这样改造下这段代码：

    ```js
    function foo(){  
        var temp_object = new Object()
        temp_object.x = 1
        temp_object.y = 2
        temp_object.array = new Array(200000)
        /**
        *   使用temp_object
        */
       let closure = temp_object.x
        return function(){
            console.log(closure);
        }
    }
    ```

    当再次执行这段代码时，我们就可以看到闭包引用的仅仅是一个 `closure` 的变量，最终如下图所示：

    <img src="https://static001.geekbang.org/resource/image/8c/ca/8c3309fd82201bf67d5c92b58d58e6ca.jpg" alt="img" data-size="original" />
*   我们再来看看由于`JavaScript`引用了`DOM`节点而造成的内存泄漏的问题，只有同时满足 `DOM` 树和 `JavaScript` 代码都不引用某个 `DOM` 节点，该节点才会被作为垃圾进行回收。 如果某个节点已从 `DOM` 树移除，但 `JavaScript` 仍然引用它，我们称此节点为“**detached** ”。**“detached ”节点是 DOM 内存泄漏的常见原因**。比如下面这段代码：

    ```js
    let detachedTree;
    function create() {

      var ul = document.createElement('ul');

      for (var i = 0; i < 100; i++) {

        var li = document.createElement('li');

        ul.appendChild(li);

      }

      detachedTree = ul;

    }

    create() 
    ```

    我们通过 `JavaScript` 创建了一些 `DOM` 元素，有了这些内存中的`DOM`元素，当有需要的时候，我们就快速地将这些 `DOM` 元素关联到 `DOM` 树上，一旦这些 `DOM` 元素从 `DOM` 上被移除后，它们并不会立即销毁，这主要是由于 `JavaScript` 代码中保留了这些元素的引用，导致这些 `DOM` 元素依然会呆在内存中。所以在保存`DOM`元素引用的时候，我们需要非常小心谨慎。

### 内存膨胀

*   了解几种可能造成内存泄漏的问题之后，接下来，我们再来看看另外一个和内存泄漏类似的问题：**内存膨胀（Memory bloat）**。

    内存膨胀和内存泄漏有一些差异，**内存膨胀主要表现在程序员对内存管理的不科学，比如只需要 50M 内存就可以搞定的，有些程序员却花费了 500M 内存。**

    额外使用过多的内存有可能是没有充分地利用好缓存，也有可能加载了一些不必要的资源。通常表现为内存在某一段时间内快速增长，然后达到一个平稳的峰值继续运行。

    比如一次性加载了大量的资源，内存会快速达到一个峰值。内存膨胀和内存泄漏的关系你可以参看下图：

    <img src="https://static001.geekbang.org/resource/image/99/10/992872337410ff5915e288e68f2c2e10.jpg" alt="img" data-size="original" />
* 我们可以看到，内存膨胀是快速增长，然后达到一个平衡的位置，而内存泄漏是内存一直在缓慢增长。**要避免内存膨胀，我们需要合理规划项目，充分利用缓存等技术来减轻项目中不必要的内存占用**。

### 频繁的垃圾回收

*   除了内存泄漏和内存膨胀，还有另外一类内存问题，那就是**频繁使用大的临时变量，导致了新生代空间很快被装满，从而频繁触发垃圾回收**。频繁的垃圾回收操作会让你感觉到页面卡顿。比如下面这段代码：

    ```js
    function strToArray(str) {
      let i = 0
      const len = str.length
      let arr = new Uint16Array(str.length)
      for (; i < len; ++i) {
        arr[i] = str.charCodeAt(i)
      }
      return arr;
    }
    ```

    ```js
    function foo() {
      let i = 0
      let str = 'test V8 GC'
      while (i++ < 1e5) {
        strToArray(str);
      }
    }


    foo()
    ```
    
    这段代码就会频繁创建临时变量，这种方式很快就会造成新生代内存内装满，从而频繁触发垃圾回收。**为了解决频繁的垃圾回收的问题，你可以考虑将这些临时变量设置为全局变量。**

### 总结

这篇答疑主要分析了两个问题，第一个是异步 API 和同步 API 的底层差异，第二个是 JavaScript 的主要内存问题的产生原因和解决方法。

Node 为读写文件提供了两套 API，一套是默认的异步 API，另外一套是同步 API。

readFile 就是异步 API，主线程在执行 readFile 的时候，会将实际读写操作丢给文件读写线程，文件读写线程处理完成之后，会将回调函数读取的结果封装成新的消息，添加到消息队列中，然后等主线执行该消息的时候，就会执行 readFile 设置的回调函数，这就是 Node 中的异步处理过程。readFileSync 是同步 API，同步 API 很简单，直接在主线程上执行，执行完成直接返回结果给它的调用函数。使用同步 API 会比较方便简单，但是你需要考虑项目能否接受读取文件而造成的暂停。

内存问题对于前端开发者来说也是至关重要的，通常有三种内存问题：内存泄漏 (Memory leak)、内存膨胀 (Memory bloat)、频繁垃圾回收。

在 JavaScript 中，造成内存泄漏 (Memory leak) 的主要原因，是不再需要 (没有作用) 的内存数据依然被其他对象引用着。所以要避免内存泄漏，我们需要避免引用那些已经没有用途的数据。

内存膨胀和内存泄漏有一些差异，内存膨胀主要是由于程序员对内存管理不科学导致的，比如只需要 50M 内存就可以搞定的，有些程序员却花费了 500M 内存。要解决内存膨胀问题，我们需要对项目有着透彻的理解，也要熟悉各种能减少内存占用的技术方案。

如果频繁使用大的临时变量，那么就会导致频繁垃圾回收，频繁的垃圾回收操作会让你感觉到页面卡顿，要解决这个问题，我们可以考虑将这些临时变量设置为全局变量。
