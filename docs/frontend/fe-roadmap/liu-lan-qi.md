# 浏览器

## 主流浏览器及内核

Google Chrome

* 优点：不易崩溃，速度快，几乎隐身，搜索简单，标签简单，更加安全
* 内核：统称为Chromium内核或Chrome内核，以前是Webkit内核，现在是Blink内核

IE

* 优点：部分只有IE内核才能打开所有网页;IE内核浏览器更安全;IE内核占用内存及CPU更少;IE最新版比chrome的速度快。
* 缺点：拓展性几乎没有；容易中病毒，出现一些乱七八糟的东西，而且上网速度较慢
* 内核：Trident内核，也是俗称的IE内核

Firefox

* 优点：风格简单，速度快，安全性高，拓展性强等。
* 缺点：网页错位，媒体功能不强等。
* 内核：Gecko内核，俗称Firefox内核

Safari

* 优点：界面也比较的简单
* 缺点：与运行在macOS上的safari相比，有些功能出现丢失
* 内核：Webkit内核。WebKit 是一个开源渲染引擎，最初是 Linux 平台的引擎，后来被 Apple 修改为支持 Mac 和 Windows。有关详细信息，请参阅[webkit.org](http://webkit.org/)。

Opera

* 内核：最初是自己的Presto内核，后来是Webkit，现在是Blink内核



## 浏览器原理

{% embed url="https://hestergong.gitbook.io/my-wiki/liu-lan-qi-yuan-li" %}



## 浏览器数据库

[IndexedDB入门](https://www.ruanyifeng.com/blog/2018/07/indexeddb.html)

IndexedDB 具有以下特点。

1. 键值对储存。 IndexedDB 内部采用对象仓库（object store）存放数据。所有类型的数据都可以直接存入，包括 JavaScript 对象。对象仓库中，数据以"键值对"的形式保存，每一个数据记录都有对应的主键，主键是独一无二的，不能有重复，否则会抛出一个错误。
2. 异步。 IndexedDB 操作时不会锁死浏览器，用户依然可以进行其他操作，这与 LocalStorage 形成对比，后者的操作是同步的。异步设计是为了防止大量数据的读写，拖慢网页的表现。
3. 支持事务。 IndexedDB 支持事务（transaction），这意味着一系列操作步骤之中，只要有一步失败，整个事务就都取消，数据库回滚到事务发生之前的状态，不存在只改写一部分数据的情况。
4. 同源限制 IndexedDB 受到同源限制，每一个数据库对应创建它的域名。网页只能访问自身域名下的数据库，而不能访问跨域的数据库。
5. 储存空间大 IndexedDB 的储存空间比 LocalStorage 大得多，一般来说不少于 250MB，甚至没有上限。
6. 支持二进制储存。 IndexedDB 不仅可以储存字符串，还可以储存二进制数据（ArrayBuffer 对象和 [Blob](http://www.semlinker.com/you-dont-know-blob/) 对象）。
