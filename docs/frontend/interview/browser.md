# Browser

## 浏览器缓存过程

第一次请求:

![image](https://user-images.githubusercontent.com/17645053/216228152-f6e6463c-e764-4f1b-9797-c8f1d6d6590e.png)

第二次请求该网页:

![p8b1y8dfjq](https://user-images.githubusercontent.com/17645053/216227318-26856278-e125-4987-b44b-74961fa4c9d4.png)

1.浏览器第一次加载资源，服务器返回200，浏览器将资源文件从服务器上请求下载下来，并把response header及该请求的返回时间一并缓存；

2.下一次加载资源时，先比较当前时间和上一次返回200时的时间差，如果没有超过cache-control设置的max-age，则没有过期，**命中强缓存**，**不发请求直接从本地缓存读取该文件**（如果浏览器不支持HTTP1.1，则用expires判断是否过期）；**如果时间过期，则向服务器发送header带有If-None-Match和If-Modified-Since的请求**

3.服务器收到请求后，优先根据Etag的值判断被请求的文件有没有做修改，Etag值一致则没有修改，命中协商缓存，返回304；如果不一致则有改动，直接返回新的资源文件带上新的Etag值并返回200；；

4.如果服务器收到的请求没有Etag值，则将If-Modified-Since和被请求文件的最后修改时间做比对，一致则命中协商缓存，返回304；不一致则返回新的last-modified和文件并返回200；

## 强缓存、协商缓存区别

1. 强缓存：不会向服务器发送请求，直接从缓存中读取资源，在chrome控制台的network选项中可以看到该请求返回**200**的状态码;

![1](https://user-images.githubusercontent.com/17645053/216227808-3d9f747e-7853-41c1-ab53-a9861c4e20e1.jpg)

![2](https://user-images.githubusercontent.com/17645053/216227815-a27ff061-169a-4be5-9883-d1244b8721d6.jpg)

> 200 form memory cache : 不访问服务器，一般已经加载过该资源且缓存在了内存当中，直接从内存中读取缓存。浏览器关闭后，数据将不存在（**资源被释放掉了**），再次打开相同的页面时，不会出现from memory cache。

> 200 from disk cache： 不访问服务器，已经在之前的某个时间加载过该资源，直接从硬盘中读取缓存，关闭浏览器后，数据依然存在，此资源不会随着该页面的关闭而释放掉下次打开仍然会是from disk cache。

> 优先访问memory cache,其次是disk cache，最后是请求网络资源

2. 协商缓存：向服务器发送请求，服务器会根据这个请求的**request header**的一些参数来判断是否命中协商缓存，如果命中，则返回**304**状态码并带上新的response header通知浏览器从缓存中读取资源；

两者的共同点是，都是从客户端缓存中读取资源；区别是强缓存不会发请求，协商缓存会发请求。

## 强缓存和协商缓存的header参数

### 强缓存

**Expires** (http1.0的头字段)：过期时间，如果设置了时间，则浏览器会在设置的时间内直接读取缓存，不再请求

**Cache-Control** (http1.1的头字段)：当值设为max-age=300时，则代表在这个请求正确返回时间（浏览器也会记录下来）的5分钟内再次加载资源，就会命中强缓存。除了该字段外，还有下面几个比较常用的设置值：

```arduino
（1） max-age：用来设置资源（representations）可以被缓存多长时间，单位为秒；
（2） s-maxage：和max-age是一样的，不过它只针对代理服务器缓存而言；
（3）public：指示响应可被任何缓存区缓存；
（4）private：只能针对个人用户，而不能被代理服务器缓存；
（5）no-cache：强制客户端直接向服务器发送请求,也就是说每次请求都必须向服务器发送。服务器接收到     请求，然后判断资源是否变更，是则返回新内容，否则返回304，未变更。这个很容易让人产生误解，使人误     以为是响应不被缓存。实际上Cache-Control:     no-cache是会被缓存的，只不过每次在向客户端（浏览器）提供响应数据时，缓存都要向服务器评估缓存响应的有效性。
（6）no-store：禁止一切缓存（这个才是响应不被缓存的意思）。
复制代码
```

> cache-control是http1.1的头字段，expires是http1.0的头字段,**如果expires和cache-control同时存在，cache-control会覆盖expires，建议两个都写。**

### 协商缓存

Last-Modifed/If-Modified-Since和Etag/If-None-Match是分别成对出现的，呈一一对应关系

**Etag/If-None-Match**：

Etag：

> Etag是属于HTTP 1.1属性，它是由服务器（Apache或者其他工具）生成返回给前端，用来帮助服务器控制Web端的缓存验证。 Apache中，ETag的值，默认是对文件的索引节（INode），大小（Size）和最后修改时间（MTime）进行Hash后得到的。

If-None-Match:

> 当资源过期时，浏览器发现响应头里有Etag, 则**再次像服务器请求时**带上请求头if-none-match(值是Etag的值)。服务器收到请求进行比对，决定返回200或304

note: If-none-match是再次请求时带上上次response里的etag让服务器进行对比

![image](https://user-images.githubusercontent.com/17645053/216228266-ab0a7e07-e654-49bc-aca5-346c604e5a29.png)

**Last-Modifed/If-Modified-Since**(http1.0头字段):

Last-Modified：

> 浏览器向服务器发送资源最后的修改时间

If-Modified-Since：

> 当资源过期时（浏览器判断Cache-Control标识的max-age过期），发现响应头具有Last-Modified声明，则再次向服务器请求时带上头if-modified-since，表示请求时间。服务器收到请求后发现有if-modified-since则与被请求资源的最后修改时间进行对比（Last-Modified）,若最后修改时间较新（大），说明资源又被改过，则返回最新资源，HTTP 200 OK;若最后修改时间较旧（小），说明资源无新修改，响应HTTP 304 走缓存。

> * Last-Modifed/If-Modified-Since的时间精度是秒，而Etag可以更精确。
> * **Etag优先级是高于Last-Modifed的，所以服务器会优先验证Etag**
> * Last-Modifed/If-Modified-Since是http1.0的头字段

## From disk cache 与 from disk cache

### 三级缓存原理

1. 先去内存看，如果有，直接加载
2. 如果内存没有，择取硬盘获取，如果有直接加载
3. 如果硬盘也没有，那么就进行网络请求
4.  加载到的资源缓存到硬盘和内存

    不同用户交互产生的走缓存的方式:

    ![img](https://images2017.cnblogs.com/blog/580705/201712/580705-20171223225445271-471561818.jpg)

参考:

* [强制缓存和协商缓存的区别](https://cloud.tencent.com/developer/article/1985866)
* [http面试必会的：强制缓存和协商缓存](https://juejin.cn/post/6844903838768431118)
* [from disk cache 与 from memory cache](https://www.cnblogs.com/jpfss/p/9242797.html)
