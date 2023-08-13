# 浏览器原理

## [How Browser Works](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/)

* 主要组件

1. **用户界面**：这包括地址栏、后退/前进按钮、书签菜单等。除了您看到请求页面的窗口之外，浏览器的每个部分都会显示。
2. **浏览器引擎**：在 UI 和渲染引擎之间编组动作。
3.  **渲染引擎**：负责显示请求的内容。例如，如果请求的内容是 HTML，则渲染引擎会解析 HTML 和 CSS，并将解析后的内容显示在屏幕上。

    <img src="https://user-images.githubusercontent.com/17645053/158114839-071a4e38-a141-459e-afad-4332323b2edd.png" alt="Screen Shot 2022-03-14 at 10 45 30 AM" data-size="original" />

    [主要流程](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/#The\_main\_flow)

    [更多细节](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/#HTML\_Parser)
4. **Networking**：对于HTTP请求等网络调用，在一个独立于平台的接口后面使用不同平台的不同实现。
5. **UI 后端**：用于绘制基本小部件，如组合框和窗口。此后端公开了一个非平台特定的通用接口。在它下面使用操作系统用户界面方法。
6. **JavaScript 解释器**。用于解析和执行 JavaScript 代码。
7. **数据存储**。这是一个持久层。浏览器可能需要在本地保存各种数据，例如 cookie。浏览器还支持 localStorage、IndexedDB、WebSQL 和 FileSystem 等存储机制。

## [浏览器工作原理与实践](https://blog.poetries.top/browser-working-principle/)

### 宏观上的浏览器

最新的Chrome浏览器包括：1个浏览器（Browser）主进程、1个 GPU 进程、1个网络（NetWork）进程、多个渲染进程和多个插件进程

* **浏览器进程**。主要负责界面显示、用户交互、子进程管理，同时提供存储等功能。
* **渲染进程**。核心任务是将 HTML、CSS 和 JavaScript 转换为用户可以与之交互的网页，排版引擎**Blink**和JavaScript引擎**V8**都是运行在该进程中，默认情况下，Chrome会为每个Tab标签创建一个渲染进程。出于安全考虑，渲染进程都是运行在沙箱模式下。
* **GPU进程**。其实，Chrome刚开始发布的时候是没有GPU进程的。而GPU的使用初衷是为了实现3D CSS的效果，只是随后网页、Chrome的UI界面都选择采用GPU来绘制，这使得GPU成为浏览器普遍的需求。最后，Chrome在其多进程架构上也引入了GPU进程。
* **网络进程**。主要负责页面的网络资源加载，之前是作为一个模块运行在浏览器进程里面的，直至最近才独立出来，成为一个单独的进程。
* **插件进程**。主要是负责插件的运行，因插件易崩溃，所以需要通过插件进程来隔离，以保证插件进程崩溃不会对浏览器和页面造成影响

#### [渲染流程](https://blog.poetries.top/browser-working-principle/guide/part1/lesson05.html#%E6%B8%B2%E6%9F%93%E6%B5%81%E7%A8%8B%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9Ahtml%E3%80%81css%E5%92%8Cjavascript%E6%98%AF%E5%A6%82%E4%BD%95%E5%8F%98%E6%88%90%E9%A1%B5%E9%9D%A2%E7%9A%84)

* [英文](https://cabulous.medium.com/how-does-browser-work-in-2019-part-iii-rendering-phase-i-850c8935958f)
* 按照渲染的时间顺序，流水线可分为如下几个子阶段：构建DOM树、样式计算、布局阶段、分层、绘制、分块、光栅化和合成。

[**构建dom树(dom construction)**](https://blog.poetries.top/browser-working-principle/guide/part1/lesson05.html#%E6%9E%84%E5%BB%BAdom%E6%A0%91)

* The input is an **HTML document** carried over from the network process, and the output is a **DOM tree**.
* A DOM tree is a representation of HTML codes. We can visit it by entering the “_document”_ in the Chrome console.

[**样式计算(style computation)**](https://blog.poetries.top/browser-working-principle/guide/part1/lesson05.html#%E6%A0%B7%E5%BC%8F%E8%AE%A1%E7%AE%97)

The input is CSS styles, and the output is a computed style structure.

[1. 把CSS转换为浏览器能够理解的结构](https://blog.poetries.top/browser-working-principle/guide/part1/lesson05.html#\_1-%E6%8A%8Acss%E8%BD%AC%E6%8D%A2%E4%B8%BA%E6%B5%8F%E8%A7%88%E5%99%A8%E8%83%BD%E5%A4%9F%E7%90%86%E8%A7%A3%E7%9A%84%E7%BB%93%E6%9E%84)(“Translating” CSS)

* 渲染引擎会把获取到的CSS文本全部转换为styleSheets结构中的数据，并且该结构同时具备了查询和修改功能，这会为后面的样式操作提供基础
*   Similar to HTML, the browser doesn’t speak CSS. The translated result is **style sheets**.

    Entering “_document.styleSheets”_ in the Chrome browser console, we can see all parsed style sheets.

    These style sheets come from

    * a source linked in the `<link>` tag,
    * styles inside of a `<style>`, and
    * inline styles

    Same as a DOM tree, the style sheets look like a JavaScript object structure and can be visited and edited in the memory.

[2. 转换样式表中的属性值，使其标准化](https://blog.poetries.top/browser-working-principle/guide/part1/lesson05.html#\_2-%E8%BD%AC%E6%8D%A2%E6%A0%B7%E5%BC%8F%E8%A1%A8%E4%B8%AD%E7%9A%84%E5%B1%9E%E6%80%A7%E5%80%BC%EF%BC%8C%E4%BD%BF%E5%85%B6%E6%A0%87%E5%87%86%E5%8C%96)(standardizing values and unites)

We use all kinds of values and units in our CSS.

* width: 50%
* padding: 2em 0
* font-size: 1rem

They are relative values.

At this step, **all relative values are converted to the absolute ones, pixels**.

Why pixels? At the end of the rendering phase, a browser displays bitmaps on the screen. Bitmaps are made of pixels.

It is time to do our math.

At the end of the standardization, the previous CSS values are changed to the following:

* width: 500px (Assuming the width of its parent element is 1000px.)
* padding: 32px 0 (Assuming the font size of the element is 16px.)
* font-size: 16px (Assuming the font size of the root element is 16px.)

[3. 计算出DOM树中每个节点的具体样式](https://blog.poetries.top/browser-working-principle/guide/part1/lesson05.html#\_3-%E8%AE%A1%E7%AE%97%E5%87%BAdom%E6%A0%91%E4%B8%AD%E6%AF%8F%E4%B8%AA%E8%8A%82%E7%82%B9%E7%9A%84%E5%85%B7%E4%BD%93%E6%A0%B7%E5%BC%8F)

[**布局(Layout)**](https://blog.poetries.top/browser-working-principle/guide/part1/lesson05.html#%E5%B8%83%E5%B1%80%E9%98%B6%E6%AE%B5)

[1. 创建布局树/渲染树](https://blog.poetries.top/browser-working-principle/guide/part1/lesson05.html#\_1-%E5%88%9B%E5%BB%BA%E5%B8%83%E5%B1%80%E6%A0%91)(constructing the layout tree)

* dom、styleSheets(cssom)生成布局树(layout tree/render tree)
*   The layout tree looks like a duplication of the DOM tree.

    What are the differences? First of all, “invisible” elements in the DOM tree won’t be included in the layout tree.

    For example, an element with _display: none_ is excluded from the layout tree. Same as all elements inside of the

    tag. There is an exception: an element with _visibility: hidden;_ is in the layout tree.

    Secondly, some CSS properties add content to the layout.

    Take a pseudo-class, for example, _div::after {content: ‘I’m here’;}_ creates a content included in the layout tree though it is not existing in the DOM tree.

[2. 布局计算](https://blog.poetries.top/browser-working-principle/guide/part1/lesson05.html#\_2-%E5%B8%83%E5%B1%80%E8%AE%A1%E7%AE%97)(calculating the geometry information)

* 计算布局树节点的坐标位置
*   To paint a box on a blank canvas, we need to know:

    * the starting x, y coordinates, and
    * the size of the box

    Each element in the layout tree is a box.

    Calculating the final geometry of each element is a mighty job. Think about it. In CSS, many properties modify the geometry of an element.

    * float: left;
    * width: 100px;
    * display: absolute;
    * and more…

    It is challenging to design an effective layout system to work with it. [Developers on BlinkOn shared some knowledge](https://www.youtube.com/watch?v=Y5Xa4H2wtVA) with us if you are interested.

    The main thread does the heavy lifting for us and completes all the calculations.

    Now it has a layout tree. On the layout tree, **each element has precise coordinates and size information**.

[**分层(Layer)**](https://blog.poetries.top/browser-working-principle/guide/part1/lesson06.html#%E5%88%86%E5%B1%82)

* The input is the **layout tree**, and the output is a **layer tree**.
* 通常满足下面两点中任意一点的元素就可以被提升为单独的一个图层。
  1. 拥有层叠上下文属性的元素会被提升为单独的一层
  2. 需要剪裁（clip）的地方也会被创建为图层

[**图层绘制(Paint)**](https://blog.poetries.top/browser-working-principle/guide/part1/lesson06.html#%E5%9B%BE%E5%B1%82%E7%BB%98%E5%88%B6)

* In this phase, the inputs are the **layout tree** and **layer tree**, and the output is **paint records**.
* 渲染引擎会把一个图层的绘制拆分成很多小的绘制指令，然后再把这些指令按照顺序组成一个待绘制列表
* 可以打开“开发者工具”的“Layers”标签，选择“document”层，来实际体验下绘制列表

[**栅格化操作(tiliing and raster)**](https://blog.poetries.top/browser-working-principle/guide/part1/lesson06.html#%E6%A0%85%E6%A0%BC%E5%8C%96%EF%BC%88raster%EF%BC%89%E6%93%8D%E4%BD%9C)

* 绘制列表只是用来记录绘制顺序和绘制指令的列表，而实际上绘制操作是由渲染引擎中的合成线程(**compositor thread**)来完成的。
* 当图层的绘制列表准备好之后，主线程会把该绘制列表提交（commit）给合成线程
* 合成线程会将图层划分为图块（tile），这些图块的大小通常是256x256或者512x512
* 合成线程会按照视口附近的图块来优先生成位图，**实际生成位图的操作是由栅格化来执行的**。所谓栅格化，是指**将图块转换为位图**。而图块是栅格化执行的最小单位。渲染进程维护了一个栅格化的线程池，所有的图块栅格化都是在线程池内执行的
* 通常，栅格化过程都会使用GPU来加速生成，使用GPU生成位图的过程叫快速栅格化，或者GPU栅格化，生成的位图被保存在GPU内存中。
* tiling: The input are the paint records and layers, and the output is tiles.
* raster: The input is tiles, and the output is bitmaps.
* [参考](https://cabulous.medium.com/how-does-browser-work-in-2019-part-iii-rendering-phase-i-850c8935958f)

[**合成和显示(Draw Quad and Display)**](https://blog.poetries.top/browser-working-principle/guide/part1/lesson06.html#%E5%90%88%E6%88%90%E5%92%8C%E6%98%BE%E7%A4%BA)

* 一旦所有图块都被光栅化，合成线程就会生成一个绘制图块的命令——“DrawQuad”，然后将该命令提交给浏览器进程。
* 浏览器进程里面有一个叫viz的组件，用来接收合成线程发过来的DrawQuad命令，然后根据DrawQuad命令，将其页面内容绘制到内存中，最后再将内存显示在屏幕上。
* 到这里，经过这一系列的阶段，编写好的HTML、CSS、JavaScript等文件，经过浏览器就会显示出漂亮的页面了。
* The input is bitmaps, and the output is a compositor frame.

**总结**

* 渲染进程将HTML内容转换为能够读懂的DOM树结构。
* 渲染引擎将CSS样式表转化为浏览器可以理解的styleSheets(CSSOM)，计算出DOM节点的样式。
* 创建布局树，并计算元素的布局信息。
* 对布局树进行分层，并生成分层树。
* 为每个图层生成绘制列表，并将其提交到合成线程。
* 合成线程将图层分成图块，并在光栅化线程池中将图块转换成位图。
* 合成线程发送绘制图块命令DrawQuad给浏览器进程。
* 浏览器进程根据DrawQuad消息生成页面，并显示到显示器上

![img](https://miro.medium.com/max/1400/1\*3NRQcXczIankRLLph-dUdw.png)

8 phases of rendering:

1. The main thread in the renderer process “translates” the HTML document to **DOM tree**
2. It conveys CSS into the **computed style**
3. It creates a **layout tree** based on the DOM tree and computed style
4. From a layout tree, it generates a **layer tree**
5. Lastly, it establishes **paint records**
6. The compositor thread carries over the paint records, starts **tilling** based on the current viewport
7. Multiple raster threads **raster** tiles into bitmaps with the help of the GPU process.
8. The browser process receives the **Draw Quad** command from the browser process, then it displays a frame of the page in front of us.

[**重排、重绘、直接合成**](https://blog.poetries.top/browser-working-principle/guide/part1/lesson06.html#%E7%9B%B8%E5%85%B3%E6%A6%82%E5%BF%B5)

* 重排：通过JavaScript或者CSS修改元素的**几何位置属性**，例如改变元素的宽度、高度等，那么浏览器会触发重新布局，解析之后的一系列子阶段，这个过程就叫重排。无疑，重排需要更新完整的渲染流水线，所以**开销也是最大**的
* 重绘：比如通过JavaScript更改某些元素的**背景颜色**。那么布局阶段将不会被执行，因为并没有引起几何位置的变换，所以就直接进入了绘制阶段，然后执行之后的一系列子阶段，这个过程就叫重绘。**相较于重排操作，重绘省去了布局和分层阶段，所以执行效率会比重排操作要高一些**。
* 直接合成：我们使用了CSS的transform来实现动画效果，这可以避开重排和重绘阶段，直接在非主线程上执行合成动画操作。这样的效率是最高的，因为是在非主线程上合成，并没有占用主线程的资源，另外也避开了布局和绘制两个子阶段，所以相对于重绘和重排，合成能大大提升绘制效率。

### [V8工作原理(基础)](https://blog.poetries.top/browser-working-principle/guide/part3/lesson12.html)

#### [栈空间和堆空间：数据是如何存储的](https://blog.poetries.top/browser-working-principle/guide/part3/lesson12.html#%E6%A0%88%E7%A9%BA%E9%97%B4%E5%92%8C%E5%A0%86%E7%A9%BA%E9%97%B4%EF%BC%9A%E6%95%B0%E6%8D%AE%E6%98%AF%E5%A6%82%E4%BD%95%E5%AD%98%E5%82%A8%E7%9A%84)

*   [引用类型](https://segmentfault.com/a/1190000006752076)储存在堆(`heap`)，基本类型储存在栈(`stack`)

    [闭包在内存中的形成](https://blog.poetries.top/browser-working-principle/guide/part3/lesson12.html#%E5%86%8D%E8%B0%88%E9%97%AD%E5%8C%85) 例：在`heap`创建`closure(foo)`

#### [垃圾回收](https://blog.poetries.top/browser-working-principle/guide/part3/lesson13.html#%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%EF%BC%9A%E5%9E%83%E5%9C%BE%E6%95%B0%E6%8D%AE%E5%A6%82%E4%BD%95%E8%87%AA%E5%8A%A8%E5%9B%9E%E6%94%B6)

*   [栈stack](https://blog.poetries.top/browser-working-principle/guide/part3/lesson13.html#%E8%B0%83%E7%94%A8%E6%A0%88%E4%B8%AD%E7%9A%84%E6%95%B0%E6%8D%AE%E6%98%AF%E5%A6%82%E4%BD%95%E5%9B%9E%E6%94%B6%E7%9A%84)

    ESP下移，代表`stack`中该执行上下文变成了无效空间
*   [堆heap](https://blog.poetries.top/browser-working-principle/guide/part3/lesson13.html#%E5%A0%86%E4%B8%AD%E7%9A%84%E6%95%B0%E6%8D%AE%E6%98%AF%E5%A6%82%E4%BD%95%E5%9B%9E%E6%94%B6%E7%9A%84)

    V8 把堆分成两个区域——新生代和老生代

    1. 标记空间中活动对象和非活动对象。所谓活动对象就是还在使用的对象，非活动对象就是可以进行垃圾回收的对象。
    2. 回收非活动对象所占据的内存。其实就是在所有的标记完成之后，统一清理内存中所有被标记为可回收的对象。
    3. 内存整理。一般来说，频繁回收对象后，内存中就会存在大量不连续空间，我们把这些不连续的内存空间称为内存碎片。当内存中出现了大量的内存碎片之后，如果需要分配较大连续内存的时候，就有可能出现内存不足的情况。所以最后一步需要整理这些内存碎片，但这步其实是可选的，因为有的垃圾回收器不会产生内存碎片，比如接下来我们要介绍的[副垃圾回收器](https://blog.poetries.top/browser-working-principle/guide/part3/lesson13.html#%E5%89%AF%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E5%99%A8)(新生代垃圾回收器)。

    因为新生区的空间不大，所以很容易被存活的对象装满整个区域。为了解决这个问题，JavaScript 引擎采用了**对象晋升**策略，也就是经过两次垃圾回收依然还存活的对象，会被移动到老生区中。

    [主垃圾回收器](https://blog.poetries.top/browser-working-principle/guide/part3/lesson13.html#%E4%B8%BB%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E5%99%A8)负责老生区中的垃圾回收

    [全停顿](https://blog.poetries.top/browser-working-principle/guide/part3/lesson13.html#%E5%85%A8%E5%81%9C%E9%A1%BF): 一旦执行垃圾回收算法，都需要将正在执行的 JavaScript 脚本暂停下来，待垃圾回收完毕后再恢复脚本执行。新生代因为不多所以全停顿影响不大；老生代需要使用增量标记（Incremental Marking）算法

#### [编译器和解析器：V8如何执行一段JavaScript代码的](https://blog.poetries.top/browser-working-principle/guide/part3/lesson14.html#v8-%E6%98%AF%E5%A6%82%E4%BD%95%E6%89%A7%E8%A1%8C%E4%B8%80%E6%AE%B5-javascript-%E4%BB%A3%E7%A0%81%E7%9A%84)

*   [编译器和解释器](https://blog.poetries.top/browser-working-principle/guide/part3/lesson14.html#%E7%BC%96%E8%AF%91%E5%99%A8%E5%92%8C%E8%A7%A3%E9%87%8A%E5%99%A8)

    编译型语言在程序执行之前，需要经过编译器的编译过程，并且编译之后会直接保留机器能读懂的二进制文件，这样每次运行程序时，都可以直接运行该二进制文件，而不需要再次重新编译了。比如 C/C++、GO 等都是编译型语言。

    而由解释型语言编写的程序，在每次运行时都需要通过解释器对程序进行动态解释和执行。比如 Python、JavaScript 等都属于解释型语言。
* [V8如何执行一段JavaScript代码](https://blog.poetries.top/browser-working-principle/guide/part3/lesson14.html#v8-%E6%98%AF%E5%A6%82%E4%BD%95%E6%89%A7%E8%A1%8C%E4%B8%80%E6%AE%B5-javascript-%E4%BB%A3%E7%A0%81%E7%9A%84)
  1.  生成抽象语法树（AST）和执行上下文

      可以把 AST 看成代码的结构化的表示，编译器或者解释器后续的工作都需要依赖于 AST，而不是源代码

      `Babel`工作原理:`ES6`源码 -> `ES6的AST` ->`ES5的AST` -> `JavaScript` 源代码。

      两个阶段：分词、语法分析
  2.  生成字节码

      解释器Ignition会根据 AST 生成字节码，并解释执行字节码, 机器码所占用的空间远远超过了字节码，所以使用字节码可以减少系统的内存使用
  3.  执行代码

      第一次执行的字节码，解释器 Ignition 会逐条解释执行。在执行字节码的过程中，如果发现有热点代码（HotSpot），比如一段代码被重复执行多次，这种就称为**热点代码**，那么后台的编译器 **TurboFan** 就会把该段热点的字节码**编译为高效的Machine Code**，然后当再次执行这段被优化的代码时，只需要执行编译后的机器码就可以了，这样就大大提升了代码的执行效率。
*   [JavaScript 的性能优化](https://blog.poetries.top/browser-working-principle/guide/part3/lesson14.html#v8-%E6%98%AF%E5%A6%82%E4%BD%95%E6%89%A7%E8%A1%8C%E4%B8%80%E6%AE%B5-javascript-%E4%BB%A3%E7%A0%81%E7%9A%84)

    主要关注以下三点内容

    1. 提升单次脚本的执行速度，避免 JavaScript 的长任务霸占主线程，这样可以使得页面快速响应交互；
    2. 避免大的内联脚本，因为在解析 HTML 的过程中，解析和编译也会占用主线程；
    3. 减少 JavaScript 文件的容量，因为更小的文件会提升下载速度，并且占用更低的内存

### [浏览器中的页面循环系统](https://blog.poetries.top/browser-working-principle/guide/part4/lesson15.html)

#### [消息队列和事件循环](https://blog.poetries.top/browser-working-principle/guide/part4/lesson15.html#%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97%E5%92%8C%E4%BA%8B%E4%BB%B6%E5%BE%AA%E7%8E%AF%EF%BC%9A%E9%A1%B5%E9%9D%A2%E6%98%AF%E6%80%8E%E4%B9%88%E6%B4%BB%E8%B5%B7%E6%9D%A5%E7%9A%84)

* [主线程如何处理其他线程发送过来的任务](https://blog.poetries.top/browser-working-principle/guide/part4/lesson15.html#%E5%A4%84%E7%90%86%E5%85%B6%E4%BB%96%E7%BA%BF%E7%A8%8B%E5%8F%91%E9%80%81%E8%BF%87%E6%9D%A5%E7%9A%84%E4%BB%BB%E5%8A%A1)
  * 添加一个消息队列；
  * IO 线程中产生的新任务添加进消息队列尾部；
  * 渲染主线程会循环地从消息队列头部中读取任务，执行任务。
* [主线程如何处理其他进程发送过来的任务](https://blog.poetries.top/browser-working-principle/guide/part4/lesson15.html#%E5%A4%84%E7%90%86%E5%85%B6%E4%BB%96%E8%BF%9B%E7%A8%8B%E5%8F%91%E9%80%81%E8%BF%87%E6%9D%A5%E7%9A%84%E4%BB%BB%E5%8A%A1)
* 渲染进程专门有一个 IO 线程用来接收其他进程传进来的消息，接收到消息之后，会将这些消息组装成任务发送给渲染主线程，后续的步骤就和前面讲解的“处理其他线程发送的任务”一样了，这里就不再重复了。
* [消息队列中的任务类型](https://blog.poetries.top/browser-working-principle/guide/part4/lesson15.html#%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97%E4%B8%AD%E7%9A%84%E4%BB%BB%E5%8A%A1%E7%B1%BB%E5%9E%8B)
* 可以参考下[Chromium 的官方源码](https://cs.chromium.org/chromium/src/third\_party/blink/public/platform/task\_type.h)，这里面包含了很多内部消息类型，如输入事件（鼠标滚动、点击、移动）、微任务、文件读写、WebSocket、JavaScript 定时器等等
*   [页面使用单线程的缺点](https://blog.poetries.top/browser-working-principle/guide/part4/lesson15.html#%E9%A1%B5%E9%9D%A2%E4%BD%BF%E7%94%A8%E5%8D%95%E7%BA%BF%E7%A8%8B%E7%9A%84%E7%BC%BA%E7%82%B9)

    * 如何处理高优先级的任务

    通常我们把消息队列中的任务称为宏任务，每个宏任务中都包含了一个微任务队列，在执行宏任务的过程中，如果 DOM 有变化，那么就会将该变化添加到微任务列表中，这样就不会影响到宏任务的继续执行，因此也就解决了执行效率的问题。

    * 如何解决单个任务执行时长过久的问题
* [总结](https://blog.poetries.top/browser-working-principle/guide/part4/lesson15.html#%E6%80%BB%E7%BB%93)
  * 如果有一些确定好的任务，可以使用一个单线程来按照顺序处理这些任务，这是第一版线程模型。
  * 要在线程执行过程中接收并处理新的任务，就需要引入循环语句和事件系统，这是第二版线程模型。
  * 如果要接收其他线程发送过来的任务，就需要引入消息队列，这是第三版线程模型。
  * 如果其他进程想要发送任务给页面主线程，那么先通过 IPC 把任务发送给渲染进程的 IO 线程，IO 线程再把任务发送给页面主线程。
  * 消息队列机制并不是太灵活，为了适应效率和实时性，引入了微任务

#### [setTimeout是怎么实现的](https://blog.poetries.top/browser-working-principle/guide/part4/lesson16.html#webapi%EF%BC%9Asettimeout%E6%98%AF%E6%80%8E%E4%B9%88%E5%AE%9E%E7%8E%B0%E7%9A%84)

* 在 Chrome 中除了正常使用的消息队列之外，还有另外一个消息队列`delayed_incoming_queue`，这个队列中维护了需要延迟执行的任务列表，包括了定时器和 Chromium 内部一些需要延迟执行的任务。所以当通过 JavaScript 创建一个定时器时，渲染进程会将该定时器的回调任务添加到延迟队列中
* `ProcessDelayTask` 函数是专门用来处理延迟执行任务的。这里我们要重点关注它的执行时机，在处理完消息队列中的一个任务之后，就开始执行 `ProcessDelayTask` 函数。`ProcessDelayTask` 函数会根据发起时间和延迟时间**计算出到期的任务**，然后依次执行这些到期的任务。等到期的任务执行完成之后，再继续下一个循环过程。通过这样的方式，一个完整的定时器就实现了。
* 设置一个定时器，JavaScript 引擎会返回一个定时器的 ID。那通常情况下，当一个定时器的任务还没有被执行的时候，也是可以取消的，具体方法是调用clearTimeout 函数，并传入需要取消的定时器的 ID。浏览器内部实现取消定时器的操作也是非常简单的，就是直接从 delayed\_incoming\_queue 延迟队列中，通过 ID 查找到对应的任务，然后再将其从队列中删除掉就可以了。

[使用settimeout-的一些注意事项](https://blog.poetries.top/browser-working-principle/guide/part4/lesson16.html#%E4%BD%BF%E7%94%A8-settimeout-%E7%9A%84%E4%B8%80%E4%BA%9B%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9)

* **如果当前任务执行时间过久，会影延迟到期定时器任务的执行**
* 如果 setTimeout 存在嵌套调用，那么系统会设置最短时间间隔为 4 毫秒
* 未激活的页面，setTimeout 执行最小间隔是 1000 毫秒
* 延时执行时间有最大值
* **使用 setTimeout 设置的回调函数中的 this 不符合直觉**，如果被 setTimeout 推迟执行的回调函数是某个对象的方法，那么该方法中的 this 关键字将指向全局环境，而不是定义时所在的那个对象。
  * 使用匿名函数
  * 使用 bind 方法

### [浏览器中的页面](https://blog.poetries.top/browser-working-principle/guide/part5/lesson21.html)

#### [javascript-是如何影响-dom-生成的](https://blog.poetries.top/browser-working-principle/guide/part5/lesson22.html#javascript-%E6%98%AF%E5%A6%82%E4%BD%95%E5%BD%B1%E5%93%8D-dom-%E7%94%9F%E6%88%90%E7%9A%84)

* DOM 的具体生成流程：字节流 -> 分词器 -> 生成节点 -> 转换为dom （后两个同步进行）
*   如果js脚本内嵌在html中：当解析到 script 脚本标签时，HTML 解析器暂停工作，JavaScript 引擎介入，脚本执行完成之后，HTML 解析器恢复解析过程，继续解析后续的内容，直至生成最终的 DOM。

    如果js脚本是引入的：执行到 JavaScript 标签时，暂停整个 DOM 的解析，执行 JavaScript 代码，不过这里执行 JavaScript 时，需要先下载这段 JavaScript 代码。JavaScript 文件的下载过程会阻塞 DOM 解析，而通常下载又是非常耗时的，会受到网络环境、JavaScript 文件大小等因素的影响。

    不过 Chrome 浏览器做了很多优化，其中一个主要的优化是**预解析操作**。当渲染引擎收到字节流之后，会开启一个**预解析线程**，用来分析 HTML 文件中包含的 **JavaScript、CSS 等相关文件**，解析到相关文件之后，预解析线程会**提前下载**这些文件。

    我们知道引入 JavaScript 线程会阻塞 DOM，不过也有一些相关的策略来规避，比如：

    1. 使用 **CDN** 来加速 JavaScript 文件的加载。
    2. 压缩 JavaScript 文件的体积。
    3. 如果 JavaScript 文件中没有操作 DOM 相关代码，就可以将该 JavaScript 脚本设置为异步加载，通过 async 或 defer 来标记代码

#### [渲染流水线：CSS如何影响首次加载时白屏时间以及如何优化](https://blog.poetries.top/browser-working-principle/guide/part5/lesson23.html#%E6%B8%B2%E6%9F%93%E6%B5%81%E6%B0%B4%E7%BA%BF%EF%BC%9Acss%E5%A6%82%E4%BD%95%E5%BD%B1%E5%93%8D%E9%A6%96%E6%AC%A1%E5%8A%A0%E8%BD%BD%E6%97%B6%E7%9A%84%E7%99%BD%E5%B1%8F%E6%97%B6%E9%97%B4%EF%BC%9F)

*   在执行 JavaScript 脚本之前，如果页面中包含了外部 CSS 文件的引用，或者通过 style 标签内置了 CSS 内容，那么渲染引擎还需要将这些内容转换为 CSSOM，因为 JavaScript 有修改 CSSOM 的能力，所以在执行 JavaScript 之前，还需要依赖 CSSOM。也就是说 CSS 在部分情况下也会阻塞 DOM 的生成。

    在接收到 HTML 数据之后的预解析过程中，HTML 预解析器识别出来了有 CSS 文件和 JavaScript 文件需要下载，然后就同时发起这两个文件的下载请求，需要注意的是，这两个文件的下载过程是重叠的，所以下载时间按照最久的那个文件来算。

    后面的流水线就和前面是一样的了，不管 CSS 文件和 JavaScript 文件谁先到达，都要先等到 CSS 文件下载完成并生成 CSSOM，然后再执行 JavaScript 脚本，最后再继续构建 DOM，构建布局树，绘制页面。
*   [影响页面展示的因素以及优化策略](https://blog.poetries.top/browser-working-principle/guide/part5/lesson23.html#%E5%BD%B1%E5%93%8D%E9%A1%B5%E9%9D%A2%E5%B1%95%E7%A4%BA%E7%9A%84%E5%9B%A0%E7%B4%A0%E4%BB%A5%E5%8F%8A%E4%BC%98%E5%8C%96%E7%AD%96%E7%95%A5)

    我们重点关注第二个阶段，这个阶段的主要问题是白屏时间，如果白屏时间过久，就会影响到用户体验。为了缩短白屏时间，我们来挨个分析这个阶段的主要任务，包括了解析 HTML、下载 CSS、下载 JavaScript、生成 CSSOM、执行 JavaScript、生成布局树、绘制页面一系列操作。

    **所以要想缩短白屏时长，可以有以下策略**

    * 通过内联 JavaScript、内联 CSS 来移除这两种类型的文件下载，这样获取到 HTML 文件之后就可以直接开始渲染流程了。
    * 但并不是所有的场合都适合内联，那么还可以尽量减少文件大小，比如通过 webpack 等工具移除一些不必要的注释，并压缩 JavaScript 文件。
    * 还可以将一些**不需要在解析 HTML** 阶段使用的 **JavaScript** 标记上 **async 或者 defer**。
    * 对于大的 CSS 文件，可以通过媒体查询属性，将其拆分为多个不同用途的 CSS 文件，这样只有在特定的场景下才会加载特定的 CSS 文件。
    * 通过以上策略就能缩短白屏展示的时长了，不过在实际项目中，总是存在各种各样的情况，这些策略并不能随心所欲地去引用，所以还需要结合实际情况来调整最佳方案。

#### [分层和合成机制：为什么css动画比JavaScript高效](https://blog.poetries.top/browser-working-principle/guide/part5/lesson24.html)

*   每个显示器都有固定的刷新频率，通常是 60HZ，也就是每秒更新 60 张图片，更新的图片都来自于**显卡中一个叫前缓冲区**的地方。 显示器任务: 每秒固定读取 60 次前缓冲区中的图像，并将读取的图像显示到显示器上。

    显卡的职责: **合成新的图像**，并将图像**保存到后缓冲区**中，一旦显卡把合成的图像写到后缓冲区，系统就会让**后缓冲区和前缓冲区互换**，这样就能**保证显示器能读取到最新显卡合成的图像**。
*   通常情况下，**显卡的更新频率和显示器的刷新频率是一致的**。但有时候，在一些复杂的场景中，显卡处理一张图片的速度会变慢，这样就会造成视觉上的卡顿

    要解决卡顿问题，就要解决每帧生成时间过久的问题，为此 Chrome 对浏览器渲染方式做了大量的工作，其中最卓有成效的策略就是引入了分层和合成机制。
*   任意一帧的生成方式，有**重排、重绘和合成**三种方式

    这三种方式的**渲染路径是不同**的，通常渲染路径越长，生成图像花费的时间就越多。比如重排，它需要重新根据 CSSOM 和 DOM 来计算布局树，这样生成一幅图片时，会让整个渲染流水线的每个阶段都执行一遍，如果布局复杂的话，就很难保证渲染的效率了。而重绘因为没有了重新布局的阶段，操作效率稍微高点，但是依然需要重新计算绘制信息，并触发绘制操作之后的一系列操作。

    相较于重排和重绘，合成操作的路径就显得非常短了，并不需要触发布局和绘制两个阶段，如果采用了 GPU，那么合成的效率会非常高。

    所以，关于渲染引擎生成一帧图像的几种方式，按照效率我们推荐合成方式优先，若实在不能满足需求，那么就再退后一步使用重绘或者重排的方式。

    本文我们的焦点在合成上, Chrome 中的合成技术，可以用三个词来概括总结：**分层、分块和合成**。
*   将素材分解为多个图层的操作就称为分层，最后将这些图层合并到一起的操作就称为合成。所以，分层和合成通常是一起使用的。

    分层体现在生成布局树之后，渲染引擎会根据布局树的特点将其转换为层树（Layer Tree），层树是渲染流水线后续流程的基础结构。层树中的每个节点都对应着一个图层，下一步的绘制阶段就依赖于层树中的节点。

    绘制阶段其实并不是真正地绘出图片，而是将绘制指令组合成一个列表，比如一个图层要设置的背景为黑色，并且还要在中间画一个圆形，那么绘制过程会生成|Paint BackGroundColor:Black | Paint Circle|这样的绘制指令列表，绘制过程就完成了。

    有了绘制列表之后，就需要进入光栅化阶段了，光栅化就是按照绘制列表中的指令生成图片。每一个图层都对应一张图片

    合成线程有了这些图片之后，会将这些图片合成为“一张”图片，并最终将生成的图片发送到后缓冲区。这就是一个大致的分层、合成流程。

    需要重点关注的是，**合成操作**是在**合成线程**上完成的，这也就意味着在执行合成操作时，是**不会影响到主线程执行**的。这就是为什么经常主线程卡住了，但是 CSS 动画依然能执行的原因。
*   如果说分层是从宏观上提升了渲染效率，那么分块则是从微观层面提升了渲染效率。

    通常情况下，页面的内容都要比屏幕大得多，显示一个页面时，如果等待所有的图层都生成完毕，再进行合成的话，会产生一些不必要的开销，也会让合成图片的时间变得更久。

    因此，合成线程会将每个图层分割为大小固定的图块，然后**优先绘制靠近视口的图**块，这样就可以大大加速页面的显示速度。不过有时候， 即使只绘制那些优先级最高的图块，也要耗费不少的时间，因为涉及到一个很关键的因素——**纹理上传**，这是因为**从**计算机**内存**上传**到 GPU 内存**的操作会**比较慢**。

    为了解决这个问题，Chrome 又采取了一个策略：在首次合成图块的时候使用一个低分辨率的图片。比如可以是正常分辨率的一半，分辨率减少一半，纹理就减少了四分之三。在首次显示页面内容的时候，将这个低分辨率的图片显示出来，然后合成器继续绘制正常比例的网页内容，**当正常比例的网页内容绘制完成后，再替换掉当前显示的低分辨率内容**。这种方式尽管会让用户在开始时看到的是低分辨率的内容，但是也比用户在开始时什么都看不到要好。
* [如何利用分层技术优化代码](https://blog.poetries.top/browser-working-principle/guide/part5/lesson24.html#%E5%A6%82%E4%BD%95%E5%88%A9%E7%94%A8%E5%88%86%E5%B1%82%E6%8A%80%E6%9C%AF%E4%BC%98%E5%8C%96%E4%BB%A3%E7%A0%81)
* 总结
  * 首先我们介绍了显示器显示图像的原理，以及帧和帧率的概念，然后基于帧和帧率我们又介绍渲染引擎是如何实现一帧图像的。通常渲染引擎生成一帧图像有三种方式：重排、重绘和合成。其中**重排和重绘**操作都是在渲染进程的**主线程**上执行的，比较耗时；而**合成**操作是在渲染进程的**合成线程**上执行的，执行速度快，且不占用主线程。
  * 然后我们重点介绍了浏览器是怎么**实现合成**的，其技术细节主要可以使用三个词来概括：**分层、分块和合成**。
  * 最后我们还讲解了 CSS 动画比 JavaScript 动画高效的原因，以及怎么使用 `will-change` 来优化动画或特效。

#### [页面性能：如何系统优化页面](https://blog.poetries.top/browser-working-principle/guide/part5/lesson25.html#%E9%A1%B5%E9%9D%A2%E6%80%A7%E8%83%BD%EF%BC%9A%E5%A6%82%E4%BD%95%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96%E9%A1%B5%E9%9D%A2)

* 通常一个页面有三个阶段：加载阶段、交互阶段和关闭阶段
  1. **加载阶段**，是指从发出请求到渲染出完整页面的过程，影响到这个阶段的主要因素有**网络**和 **JavaScript** 脚本。
  2. **交互阶段**，主要是从页面加载完成到用户交互的整合过程，影响到这个阶段的主要因素是 **JavaScript** 脚本。
  3. **关闭阶段**，主要是用户发出关闭指令后页面所做的一些清理操作
*   [加载阶段](https://blog.poetries.top/browser-working-principle/guide/part5/lesson25.html#%E5%8A%A0%E8%BD%BD%E9%98%B6%E6%AE%B5)

    我们把这些能**阻塞网页首次渲染**的资源称为关键资源。基于**关键资源**(`JS`、首次请求的`HTML`资源、`CSS`文件)，我们可以继续细化出来三个影响页面首次渲染的核心因素。

    1. **关键资源个数**。关键资源个数越多，首次页面的加载时间就会越长。比如上图中的关键资源个数就是 3 个，1 个 HTML 文件、1 个 JavaScript 和 1 个 CSS 文件。
    2. **关键资源大小**。通常情况下，所有关键资源的内容越小，其整个资源的下载时间也就越短，那么阻塞渲染的时间也就越短。上图中关键资源的大小分别是 6KB、8KB 和 9KB，那么整个关键资源大小就是 23KB。
    3. **请求关键资源需要多少个 RTT**。 通常 1 个 HTTP 的数据包在 14KB 左右，所以 1 个 0.1M 的页面就需要拆分成 8 个包来传输了，也就是说需要 8 个 RTT。首先是请求 HTML 资源，大小是 6KB，小于 14KB，所以 1 个 RTT 就可以解决了。至于 JavaScript 和 CSS 文件，这里需要注意一点，由于渲染引擎有一个**预解析**的线程，在接收到 HTML 数据之后，预解析线程会快速扫描 HTML 数据中的关键资源，一旦扫描到了，会立马发起请求，你可以认为 **JavaScript 和 CSS 是同时发起请求的**，所以它们的请求是重叠的，那么**计算它们的 RTT 时，只需要计算体积最大的那个数据就可以了**。这里最大的是 CSS 文件（9KB），所以我们就按照 9KB 来计算，同样由于 9KB 小于 14KB，所以 JavaScript 和 CSS 资源也就可以算成 1 个 RTT。也就是说，上图中关键资源请求共花费了 2 个 RTT。

    **总的优化原则就是减少关键资源个数，降低关键资源大小，降低关键资源的 RTT 次数**

    1. **减少关键资源的个数**。
       * 可以将 JavaScript 和 CSS 改成内联的形式，比如上图的 JavaScript 和 CSS，若都改成内联模式，那么关键资源的个数就由 3 个减少到了 1 个。
       * 如果 JavaScript 代码没有 DOM 或者 CSSOM 的操作，则可以改成 **sync** 或者 **defer** 属性; 对于 CSS，如果不是在构建页面之前加载的，则可以添加媒体取消阻止显现的标志。当 JavaScript 标签加上了 sync 或者 defer、CSSlink 属性之前加上了取消阻止显现的标志后，它们就变成了非关键资源了
    2. **减少关键资源的大小**
       * 可以压缩 CSS 和 JavaScript 资源，移除 HTML、CSS、JavaScript 文件中一些注释内容
       * 也可以通过前面讲的取消 CSS 或者 JavaScript 中关键资源的方式。
    3. **减少关键资源 RTT 的次数**
       * 可以通过减少关键资源的个数和减少关键资源的大小搭配来实现。
       * 除此之外，还可以使用 CDN 来减少每次 RTT 时长
*   [交互阶段](https://blog.poetries.top/browser-working-principle/guide/part5/lesson25.html#%E4%BA%A4%E4%BA%92%E9%98%B6%E6%AE%B5)

    交互阶段是如何生成一个帧的。大部分情况下，生成一个新的帧都是由 JavaScript 通过修改 DOM 或者 CSSOM 来触发的。还有另外一部分帧是由 CSS 来触发的。

    如果在**计算样式阶段发现有布局信息的修改**，那么就会触发**重排**操作，然后触发后续渲染流水线的一系列操作，这个代价是非常大的。

    如果在计算样式阶段没有发现有布局信息的修改，只是**修改了颜色一类的信息**，那么就不会涉及到布局相关的调整，所以可以跳过布局阶段，**直接进入绘制阶段**，这个过程叫**重绘**。不过重绘阶段的代价也是不小的。

    还有另外一种情况，通过 CSS 实现一些**变形、渐变、动画**等特效，这是**由 CSS 触发**的，并且是**在合成线程上执行**的，这个过程称为**合成**。因为它**不会触发重排或者重绘**，而且合成操作本身的速度就非常快，所以执行**合成是效率最高**的方式。

    **优化方案: 一个大的原则就是让单个帧的生成速度变快**

    1. **减少 JavaScript 脚本执行时间**
    2. 将一次执行的函数分解为多个任务，使得每次的执行时间不要过久
    3. 采用 [Web Workers](https://www.cnblogs.com/goloving/p/13962441.html)。你可以把 Web Workers 当作主线程之外的一个线程，在 Web Workers 中是可以执行 JavaScript 脚本的，不过 Web Workers 中没有 DOM、CSSOM 环境，这意味着在 Web Workers 中是无法通过 JavaScript 来访问 DOM 的，所以我们可以把一些和 DOM 操作无关且耗时的任务放到 Web Workers 中去执行
    4.  **避免强制同步布局**

        执行 JavaScript 添加元素是在一个任务中执行的，重新计算样式布局是在另外一个任务中执行，这就是正常情况下的布局操作。

        所谓强制同步布局，是指 JavaScript 强制将计算样式和布局操作提前到当前的任务中。

        为了避免强制同步布局，我们可以调整策略，在**修改 DOM 之前查询相关值**。
    5.  **避免布局抖动**

        在一次 JavaScript 执行过程中，多次执行强制布局和抖动操作。

        ```js
        function foo() {
            let time_li = document.getElementById("time_li")
            // 在 foo 函数内部重复执行计算样式和布局，这会大大影响当前函数的执行效率
            for (let i = 0; i < 100; i++) {
                let main_div = document.getElementById("mian_div")
                let new_node = document.createElement("li")
                let textnode = document.createTextNode("time.geekbang")
                new_node.appendChild(textnode);
              	// 在一个 for 循环语句里面不断读取属性值，每次读取属性值之前都要进行计算样式和布局 
              	// 尽量不要在修改 DOM 结构后再去查询一些相关值
                new_node.offsetHeight = time_li.offsetHeight;
                document.getElementById("mian_div").appendChild(new_node);
            }
        }
        ```
    6. **合理利用 CSS 合成动画**
       * 合成动画是直接在合成线程上执行的，这和在主线程上执行的布局、绘制等操作不同，如果主线程被 JavaScript 或者一些布局任务占用，CSS 动画依然能继续执行。
       * 另外，如果能提前知道对某个元素执行动画操作，那就最好将其标记为 will-change，这是告诉渲染引擎需要将该元素单独生成一个图层。
    7.  **避免频繁的垃圾回收**

        我们知道 JavaScript 使用了自动垃圾回收机制，如果在一些函数中频繁创建临时对象，那么垃圾回收器也会频繁地去执行垃圾回收策略。这样当垃圾回收操作发生时，就会占用主线程，从而影响到其他任务的执行，严重的话还会让用户产生掉帧、不流畅的感觉。

        所以要尽量避免产生那些临时垃圾数据。那该怎么做呢？可以尽可能优化储存结构，尽可能避免小颗粒对象的产生。
*   总结

    在加载阶段，核心的优化原则是：优化关键资源的加载速度，减少关键资源的个数，降低关键资源的 RTT 次数。

    在交互阶段，核心的优化原则是：尽量减少一帧的生成时间。可以通过减少单次 JavaScript 的执行时间、避免强制同步布局、避免布局抖动、尽量采用 CSS 的合成动画、避免频繁的垃圾回收等方式来减少一帧生成的时长。

## 问题 JS解释器在哪一步发挥了作用？

### [浏览器是如何工作的：Chrome V8让你更懂JavaScript](https://segmentfault.com/a/1190000037435824)

* 内部结构
  * Parser: JS -> AST
  * Intrepretor Ignition: AST -> Bytecode, 并执行Bytecode同时收集 TurboFan 优化编译所需的信息
    * 有\*\*基于栈 (Stack-based)和基于寄存器 (Register-based)\*\*的Interpretor。
    * **大多数解释器都是基于栈的**，比如 Java 虚拟机，.Net 虚拟机，还有早期的 V8 虚拟机。基于堆栈的虚拟机在处理函数调用、解决递归问题和切换上下文时简单明快。而**现在的 V8 虚拟机则采用了基于寄存器的设计**，它将一些中间数据保存到寄存器中。
    * [V8引擎详解（四）——字节码是如何执行的](https://juejin.cn/post/6844904161163608078): 通常基于Register的虚拟机比基于Stack的虚拟机执行的更快，但是指令相对较长
  * Compiler TurboFan
    * 利用 Ignition 所收集的类型信息，将 Bytecode 转换为优化的汇编代码；
  * Garbage Collector Orinoco
* 在运行 C、C++以及 Java 等程序之前，需要进行编译，不能直接执行源码；但对于 JavaScript 来说，我们可以直接执行源码(比如：node test.js)，它是在运行的时候先编译再执行，这种方式被称为**即时编译(Just-in-time compilation)**，简称为 JIT。因此，V8 也属于 **JIT 编译器**。
* **V8 执行一段 JavaScript 代码所经历的主要流程**包括：
  * 初始化基础环境；
  * 解析源码生成 AST 和作用域；
  * 依据 AST 和作用域生成字节码；
  * 解释执行字节码；
  * 监听热点代码；
  * 优化热点代码为二进制的机器代码；
  * 反优化生成的二进制机器代码。
*   惰性解析、预解析

    **惰性解析**是指解析器在解析的过程中，如果遇到函数声明，那么会跳过函数内部的代码，并不会为其生成 AST 和字节码，而仅仅生成顶层代码的 AST 和字节码。

    **闭包给惰性解析带来的问题**：上文的 d 不能随着 foo 函数的执行上下文被销毁掉。

    **预解析器**，比如当解析顶层代码的时候，遇到了一个函数，那么预解析器并不会直接跳过该函数，而是对该函数做一次快速的预解析。

    **判断当前函数是不是存在一些语法上的错误**，发现了语法错误，那么就会向 V8 抛出语法错误；

    **检查函数内部是否引用了外部变量，如果引用了外部的变量，预解析器会将栈中的变量复制到堆中，在下次执行到该函数的时候，直接使用堆中的引用，这样就解决了闭包所带来的问题**。

### [图解Google V8](https://time.geekbang.org/column/intro/100048001?tab=catalog)

#### [笔记一](tu-jie-googlev8/bi-ji-1.md)

#### [笔记二](tu-jie-googlev8/bi-ji-2.md)

### [JavaScriptCore 中的推测](https://webkit.org/blog/10308/speculation-in-javascriptcore/)
