# Frontend Engineering

## StoryBook

* [StoryBook文档](https://storybook.js.org/)
*   Storybook + Vue项目创建方式

    ```
    vue create vue_demo
    cd vue_demo/
    npx -p @storybook/cli sb init --type vue
    npm run storybook
    ```

## Lerna

* [Lerna 从原理到实战详解](https://toutiao.io/posts/pz8foi6/preview)

## CSS预处理器

* [Sass.vs.Less](https://juejin.cn/post/6844904169313140749#heading-0)
* [sass](https://sass-lang.com/documentation)

## postcss

* [postcss简介](https://www.axihe.com/edu/postcss/home.html)
* PostCSS 本身是一个功能比较单一的工具。它提供了一种方式用 JavaScript 代码来处理 CSS。它**负责把 CSS 代码解析成抽象语法树结构（Abstract Syntax Tree，AST），再交由插件来进行处理**。插件基于 CSS 代码的 AST 所能进行的操作是多种多样的，比如可以支持变量和混入（mixin），增加浏览器相关的声明前缀，或是把使用将来的 CSS 规范的样式规则转译（transpile）成当前的 CSS 规范支持的格式。从这个角度来说，PostCSS 的**强大之处在于其不断发展的插件体系**。目前 PostCSS 已经有 200 多个功能各异的插件。开发人员也可以根据项目的需要，开发出自己的 PostCSS 插件。
* [Postcss 简明教程 及 css module](https://www.jianshu.com/p/2e9117c81792)

## sourcemap

[sourcemap简介](https://www.cnblogs.com/jaxu/p/11358129.html)

[JavaScript Source Map 详解](http://www.ruanyifeng.com/blog/2013/01/javascript\_source\_map.html)

## 构建工具

[构建工具 和webpack模块打包工具区别](https://static.kancloud.cn/cyyspring/webpack/1995849)

[参考一](https://www.dazhuanlan.com/cnbinco/topics/1462463)

```js
1.Grunt和Gulp的工作方式是：在一个配置文件中，指明对某些文件进行类似编译，组合，压缩等任务的具体步骤，工具之后可以自动替你完成这些任务。
Gulp/Grunt 是一种能够优化前端的开发流程的工具。

2.Webpack的工作方式是：把你的项目当做一个整体，通过一个给定的主文件（如：index.js），Webpack将从这个文件开始找到你的项目的所有依赖文件，使用loaders处理它们，最后打包为一个（或多个）浏览器可识别的JavaScript文件。
Webpack 是一种模块打包工具，将 js 文件，css 文件和其他一些请求资源打包合并成一个请求文件，从而减少了 http 请求的次数。

3.总结'Grunt和Gulp' 像是一个流程，例如先执行语法转换-》在执行压缩 项链式一步一步执行，'webpack'遇到需要处理的就通过'loaders'处理，这个执行的步骤是看通过入口进入的文件，里面依赖的内容，再根据配置规则处理依赖的内容。
```

[参考二](https://blog.csdn.net/qq\_36671474/article/details/82227369)

```js
Webpack与Gulp、Grunt没有什么可比性，它可以看作模块打包机，通过分析你的项目结构，找到JavaScript模块以及其它的一些浏览器不能直接运行的拓展语言（Scss，TypeScript等），并将其转换和打包为合适的格式供浏览器使用

Gulp/Grunt是一种能够优化前端的开发流程的工具，而WebPack是一种模块化的解决方案，不过Webpack的优点使得Webpack在很多场景下可以替代Gulp/Grunt类的工具。
```

[参考三](https://juejin.cn/post/6967271573258502152)

```js
webpack是一个现代化的JavaScript应用的静态模块打包工具。webpack依赖node，在使用之前必须先安装node

grunt/gulp的核心是task，通过配置一系列task，并且定义task要处理的事务（例如es6、ts转化、图片压缩，scss转成css）,之后让grunt/gulp来依次执行这些task，而且让整个流程自动化。所以grunt/gulp也被称为前端自动化任务管理工具。

什么时候用grunt/gulp呢？
如果工程模块依赖非常简单，甚至没有用到模块化的概念。只需要进行简单的合并、压缩，就使用grunt/gulp即可。但是如果整个项目使用了模块化管理，而且相互以来非常强，我们就可以使用更加强大的webpack了。

所以，grunt/gulp和webpack有什么不同呢？
grunt/gulp更加强调的是前端流程的自动化，模块化不是它的核心。
webpack更加强调模块化开发管理，而文件压缩合并、预处理等功能，是它的附带功能。
```

## Webpack

* [参考链接](https://static.kancloud.cn/cyyspring/webpack/2005652)
* 前端模块化
  1. 前端在整个历史的演变，最后提出的解决方案是'esm',使得前端可以解决以前模块化里程的各种问题，这些问题解决的都是开发阶段多文件的划分的模块化，但是在实际正式环境然需要考虑下面几个问题
     1. ES Modules 存在环境兼容问题 这个问题随着时间推移将会被解决
     2. 模块文件过多，网络请求频繁，影响工作效率，模块化导致问题就是我们会细致的划分文件，导致文件请求变多，这个解决方法也可以通过http2 /http3 来解决
     3. 所有的前端资源都需要模块化，**现有的'esm'只是把js 进行了模块化**，但是前端是由'css','图片'.'字体'这些个个文件组成，因此需要模块化的是前端所有资源不是单单的js文件
  2. 解决上面问题就需要引入新的前端开发工具
     1. 新特性代码编译，将包含新特性代码转换成大多数浏览器能够兼容的代码 如ES6 => ES5，这些可以通过glup 这些自动化构建工具帮助实现
     2. 解决文件过多，导致请求过多的问题，我们可以设想在开发阶段依旧是模块化多文件的形式，但是在打包的时候，可以讲过这些模块化的文件打包成一个文件
     3. 支持不同种类的前端资源类型，可以将其当作模块使用
* 为什么打包工具会出现
  1. ES6语法已经在前端开发领域普遍使用，然而很多浏览器依旧对ES6没有提供全面的兼容和支持。
  2. 前端三大框架里，例如React的JSX，VUE和Angular的指令都是浏览器无法识别的，需要编译
  3. 在CSS开发中，我们已经习惯使用less、sass等预编译工具，有了构建工具做编译转化，可以让我们开发效率更高，代码更好维护。
  4. 前端必不可少要使用图片等多媒体资源，构建工具可以对它们进行压缩。
  5. 当前的前端开发都是模块开发，也引入了大量的依赖包，为了让浏览器对代码的加载更快，以及代码不会被轻易识别 和可读，需要构建工具对代码进行压缩和混淆。
* Webpack
  1. Webpack 作为一个模块打包工具，将零散的 JavaScript 代码打包到一个 JS 文件中。也可以通过拆包，它能够将应用中所有的模块按照我们的需要分块打包。这样一来，就不用担心全部代码打包到一起，产生单个文件过大，导致加载慢的问题。
  2. 对于有环境兼容问题的代码，Webpack 可以在打包过程中通过 Loader 机制对其实现编译转换，然后再进行打包。
  3. 对于不同类型的前端模块类型，Webpack 支持在 JavaScript 中以模块化的方式载入任意类型的资源文件，例如，我们可以通过 Webpack 实现在 JavaScript 中加载 CSS 文件，被加载的 CSS 文件将会通过 style 标签的方式工作
*   hash vs chunkhash vs contenthash

    参考: [https://blog.csdn.net/qq\_17175013/article/details/119250701](https://blog.csdn.net/qq\_17175013/article/details/119250701)







### Uglify vs Terser

*   [为什么 webpack4 默认支持 ES6 语法的压缩？(从uglify到terser)](https://zhuanlan.zhihu.com/p/81108236)

    **代码压缩插件的历史**

    ......

    * uglify-es 的停止维护导致了 terser 被 fork 出来了，并且 terser 处理了没有合入的 PRs，最终创建了一个独立的仓库： [https://github.com/fabiosantoscode/terser](https://link.zhihu.com/?target=https%3A//github.com/fabiosantoscode/terser)
    * 随后，terser-webpack-plugin 被创建出来, 它基于 terser，并且具备uglifyjs-webpack-plugin 的同等功能 : [https://github.com/webpack-cont](https://link.zhihu.com/?target=https%3A//github.com/webpack-contrib/terser-webpack-plugin)
    * 结论：terser-webpack-plugin 基于 terser 因此它具备 ES6 的压缩能力，uglifyjs-webpack-plugin v2.x 版本基于 uglify-js，无法支持 ES6 的压缩。

    **原理**

    代码压缩原理其实挺简单的，也是 AST 的一个经典的应用案例。它的压缩过程通常是：

    ```
    JS 源代码 -> AST -> 美化、压缩 -> 新的 AST -> 压缩后的代码 
    ```

    了解了代码压缩的基本流程后，接下来我们看看源码包含了哪些内容，由于 terser 是从 uglify-es Fork 出来进行修改的，因此它的代码结构和 uglify-js 基本一致，只不过 terser 使用了 ES6 模块的静态分析功能。我们以 terser 的源码为例分析下：

    * ast.js：JS 的抽象语法树的描述信息
    * parse.js：Parser，用于从 JS 源代码分析出 AST
    * minify.js：用于将 AST 优化成更简短的结构
    * output.js：代码生成器，从 AST 输出 压缩后的代码，支持 sourcemap 的生成
    * propmangle.js：对变量的长度进行压缩，通常是单个字符
    * scope.js：分析变量定义/引用位置的信息
    * transform.js：节点遍历

    然后，我们来一探 terser 和 uglify-js 的差异。对比了之后，发现一个很大的差异是 AST 的支持上面不同。

    <img src="https://pic4.zhimg.com/80/v2-dc674b32e36f029125e0fe3038cf5413_1440w.jpg" alt="img" data-size="original" />

### ESBuild & SWC

* [快70倍！新一代JS构建工具：ESBuild & SWC浅析](https://jishuin.proginn.com/p/763bfbd78686)
* [使用swc与esbuild闪电加速你的webpack打包链路](https://blog.csdn.net/qq\_21567385/article/details/121345739)
* [如何看待前端 bundler esbuild？](https://www.zhihu.com/question/394060026) : 目前缺失插件机制让它没有办法上生产，但是可以用 webpack 做构建，esbuild 做压缩。比起 UglifyjsPlugin TerserPlugin 可以节约10-20%左右的时间\~
*   [vite vs webpack vs esbuild](https://juejin.cn/post/7004377706653564935)

    vite: vite在配置难易度和速度方面确实较webpack有很大优势，对于新手而言极为友好，开发过程的体验可以说是三者中最为舒适的。

    webpack: webpack的速度是一个大问题，虽然借助esbuild-loader加快了打包速度，但是仍逊于vite，不过其优势在于插件及各类功能的丰富，而且不依赖于其他工具，当你的项目需要较多的插件时，很大概率你要使用webpack。

    esbuild: esbuild确实很快。但是除了快其他的体验都不好，这是一个不成熟的工具，期待其功能的完善。
* [swc官方文档](https://swc.rs/docs/usage/swc-loader)

### webpack sourcemap

* [webpack sourcemap](https://static.kancloud.cn/cyyspring/webpack/1849350)

## Bundle vs Bundless

* [聊聊ESM、Bundle 、Bundleless 、Vite 、Snowpack](https://segmentfault.com/a/1190000025137845)

## 学习资源

[前端工程化学习笔记](https://static.kancloud.cn/cyyspring/webpack/2287656)

[前端工程化38讲](https://q.shanyue.tech/engineering/)
