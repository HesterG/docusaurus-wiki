# 计算机网络笔记(三)

## 应用层

### 应用层概述

*   应用层解决通过应用进程的交互来实现特定网络应用的问题

    <img src="https://user-images.githubusercontent.com/17645053/159821855-4f13bfbf-6d14-48fb-a6a6-41071b8fa42d.png" alt="Screen Shot 2022-03-24 at 9 07 42 AM" data-size="original" /><img src="https://user-images.githubusercontent.com/17645053/159821725-d2332aa4-0ab4-4865-9b81-0fd3f9498023.png" alt="Screen Shot 2022-03-24 at 9 06 27 AM" data-size="original" />

### 客户/服务器方式(C/S)和对等方式(P2P)

#### 客户/服务器（C/S）方式

![Screen Shot 2022-03-24 at 9 28 08 AM](https://user-images.githubusercontent.com/17645053/159823819-f1f732e6-1cbe-424f-9bac-35a236402d90.png)

* 很多网络应用都是C/S方式，包括WWW、电子邮件、文件传输FTP等。
* C/S方式的应用通常是**服务集中型**的，一台服务器要服务很多客户机，**常会出现服务器计算机跟不上众多客户机请求的情况**。
* 因此在C/S应用中，常用**计算机群集**构建一个强大的**虚拟服务器**。

#### P2P方式

![Screen Shot 2022-03-24 at 9 33 38 AM](https://user-images.githubusercontent.com/17645053/159824303-5053516c-ee59-4086-8107-a09f6508128d.png)

* P2P最突出的特性之一就是它的**可扩展性**。因为系统每增加一个对等方，不仅增加的是服务的请求者，同时也增加了服务的提供者，**系统性能不会因为规模的增大而降低**。
* P2P方式**具有成本上的优势**，因为它通常不需要庞大的服务器设施和服务器带宽。为了降低成本，服务提供商对于将P2P方式用于应用的兴趣越来越大。

### 动态主机配置协议DHCP

#### DHCP服务器作用

* 客户机开机开机自动启动DHCP，向DHCP服务器请求获取配置信息。这样就不用手动配置了。

![Screen Shot 2022-03-24 at 9 41 15 AM](https://user-images.githubusercontent.com/17645053/159825009-22369a7f-0ba1-48fd-8d4c-48ca4368c9b4.png)

#### DHCP的工作过程

* 使用C/S方式，它使用UDP所提供的服务，DHCP报文在运输层会被封装成UDP用户数据报。
* DHCP服务器使用UDP端口67；DHCP客户端使用UDP端口68。
* `DHCP DISCOVER`源地址`0.0.0.0`因为主机尚未分配到IP地址，因此使用该地址来代替。目的地址`255.255.255.255` 为广播发送，因为不知道有哪几台DHCP服务器。对于一般客户，其应用层没有监听该UDP用户数据报目的端口67的进程（DHCP服务进程），因此只能丢弃；而对于DHCP服务器，其应用层运行着DHCP服务器进程，因此会接受该`DHCP DISCOVER`报文并作出响应。
* DHCP服务器收到DHCP发现报文后，根据其中封装的DHCP客户端的MAC地址来找自己的数据库，看是否有针对该MAC的配置信息，如果有，则使用这些配置信息来构建并发送`DHCP OFFER`报文，如果没有，则采用默认配置信息来构建并发送`DHCP OFFER`报文：目的地址仍为`255.255.255.255`广播地址，因为主机还没有配置IP地址。对于DHCP服务器，其应用层没有监听该UDP用户数据报目的端口68的进程（DHCP客户进程），因此只能丢弃；而对于DHCP客户，其应用层运行着DHCP客户进程，因此会接受该`DHCP OFFER`报文并作出响应。DHCP客户会根据`DHCP OFFER`报文的事务ID来判断该报文是否是自己所请求的报文(i.e., 如果该事物ID与自己之前发送的`DHCP DISCOVER`报文中的事务ID相等，就表明这是自己所请求的报文，就可以接受该报文，否则就丢弃该报文)。配置信息中的`IP地址`是DHCP从自己数据库中挑选出来的，并使用`ARP`确保所选`IP地址`没有被其他主机占用。

![Screen Shot 2022-03-24 at 10 04 55 AM](https://user-images.githubusercontent.com/17645053/159827478-fc0c2403-bbf3-495e-aa1d-bdb64faa0df0.png)

* 本例中，DHCP客户会收到两个DHCP服务器发来的IP，DHCP客户会选一个，一般会选先来的那一个，并发送`DHCP REQUEST`报文。源地址仍为`0.0.0.0`，因为此时DHCP客户才从多个DHCP服务器中挑选一个作为自己的DHCP服务器，它首先需要征得该服务器的同意，之后才能正式使用向该DHCP服务器租用的IP地址。目的地址仍为`255.255.255.255`广播地址。

![Screen Shot 2022-03-24 at 10 04 44 AM](https://user-images.githubusercontent.com/17645053/159827470-f36dd7e4-b528-4474-a850-7a0d44ba8987.png)

* 假设DHCP客户选择DHCP服务器1作为自己的DHCP服务器。于是DHCP服务器1给DHCP客户发送`DHCP ACK`报文段，目的地址仍为广播地址`255.255.255.255` 。收到`DHCP ACK`后主机还会使用ARP检测该IP地址是否被网络中其他主机占用，如果占用会发送DHCP谢绝报文来谢绝租约，如果没有占用DHCP客户就可以使用那个IP地址了。

![Screen Shot 2022-03-24 at 1 03 54 PM](https://user-images.githubusercontent.com/17645053/159845930-8053b1ae-b313-4516-a56d-d90393bb4e4f.png)

* 租用期过半后主机会向DHCP服务器发送续约请求;主机可以随时发送`DHCP RELEASE`来提前终止`DHCP`服务器所提供的租用期。

![Screen Shot 2022-03-24 at 1 07 45 PM](https://user-images.githubusercontent.com/17645053/159846314-5d96ac5b-fd4f-485d-a0f5-1d1e488c0e4c.png)

#### DHCP中继代理

* 路由器会丢弃广播报文，所以要给路由器配置DHCP服务器的IP地址并使之成为DHCP中继代理。我们并不愿意在每一个网络上都配置一个DHCP服务器。所以要用中继代理。

![Screen Shot 2022-03-24 at 1 37 01 PM](https://user-images.githubusercontent.com/17645053/159849604-2995d836-b637-4bfb-8b92-6fe936dcfd9b.png)

### DNS

#### DNS作用

* 用户主机首先在自己的`DNS高速缓存`中查找`www.hnust.cn`对应的`IP地址`。如果没有找到，就会向网络中某台DNS服务器查询，DNS服务器中有域名、`IP地址`映射关系的数据库，DNS服务器在数据库中查找后将结果发送给用户主机。
* 理论上可以只用一台DNS服务器，但是不可取

![Screen Shot 2022-03-24 at 1 55 31 PM](https://user-images.githubusercontent.com/17645053/159851667-b2c2d99d-3017-40f0-9ca9-19d39633ef61.png)

#### 因特网采用**层次树状结构的域名结构**

* 级别低的域名在最左边，顶级域名在最右边

![Screen Shot 2022-03-24 at 1 57 36 PM](https://user-images.githubusercontent.com/17645053/159851915-5be34cb3-38de-414a-888e-d257519317e3.png)

*   顶级域名TLD(Top Level Domain)分以下三类

    * 国家顶级域名、通用顶级域名、反向域

    <img src="https://user-images.githubusercontent.com/17645053/159852091-7ccbde69-c15d-4154-b683-b3071ca8a8b1.png" alt="Screen Shot 2022-03-24 at 1 59 05 PM" data-size="original" />

    * 国家顶级域名下注册的二级域名均由该国家自行确定。

    <img src="https://user-images.githubusercontent.com/17645053/159852384-b3120d27-9300-45a7-848f-131a2529489d.png" alt="Screen Shot 2022-03-24 at 2 01 27 PM" data-size="original" />
*   因特网域名空间

    <img src="https://user-images.githubusercontent.com/17645053/159852548-8abc3943-c0cf-41d3-b12e-5b1b6a49245b.png" alt="Screen Shot 2022-03-24 at 2 03 07 PM" data-size="original" />
*   DNS使用分布在各地的域名服务器来实现域名到IP地址的转换

    DNS可以划分为以下四种不同的类型

    1. **根**域名服务器，知道所有顶级域名服务器的域名及其IP地址
    2. **顶级**域名服务器，管理该顶级域名下所有二级域名
    3. **权限**域名服务器，管理某个区的域名
    4.  **本地**域名服务器，不属于上述DNS等级结构中。当一个主机发起DNS报文请求时，这个报文就先被送往该主机的本地域名服务器。本地DNS起代理作用，会将该报文转发到上述域名服务器的等级结构中。每一个ISP、大学、甚至一个大学里的学院都可以拥有一个本地域名服务器。也被称为**默认域名服务器**。本地域名服务器离用户较近，也可能就在一个局域网中。需要配置在主机中。

        <img src="https://user-images.githubusercontent.com/17645053/159854367-860f4bf6-03f7-474b-809c-11362009d957.png" alt="Screen Shot 2022-03-24 at 2 18 42 PM" data-size="original" />

#### 域名解析的过程

1. 递归查询
2. 迭代查询

![Screen Shot 2022-03-24 at 2 23 22 PM](https://user-images.githubusercontent.com/17645053/159855134-c90d8d9f-8e99-4151-aba9-8087c51f851a.png)

*   DNS广泛使用高速缓存

    <img src="https://user-images.githubusercontent.com/17645053/159855348-4d76657a-289a-4288-8c59-1ebe7ed9079f.png" alt="Screen Shot 2022-03-24 at 2 25 05 PM" data-size="original" />

    因为域名、IP地址映射关系不是永久不变，所以需要设置计时器；用户主机也会缓存域名

    <img src="https://user-images.githubusercontent.com/17645053/159855570-5ca071a8-efa4-4ea8-b131-9605ec507f6d.png" alt="Screen Shot 2022-03-24 at 2 26 52 PM" data-size="original" />

### 文件传送协议FTP

![Screen Shot 2022-03-24 at 2 35 59 PM](https://user-images.githubusercontent.com/17645053/159856818-84a08504-4748-4c0c-9c9c-e01bbaef3672.png) ![Screen Shot 2022-03-24 at 2 46 52 PM](https://user-images.githubusercontent.com/17645053/159858371-a4d34d82-4563-4986-ae84-c3b4fa5dbdff.png)

#### 工作原理

* FTP客户与FTP服务器建立TCP连接(端口21) 控制连接，会话期间保持打开
* 另一条TCP连接用于传送文件(端口20) 数据通道，每次文件传输才建立，传输结束就关闭
* 主动模式

![Screen Shot 2022-03-24 at 2 50 45 PM](https://user-images.githubusercontent.com/17645053/159858923-f03127bd-002c-4e13-98fd-5478e33e2610.png)

* 被动模式

![Screen Shot 2022-03-24 at 2 51 44 PM](https://user-images.githubusercontent.com/17645053/159859065-740dc835-8069-4840-be89-1423b9133c14.png)

### 电子邮件

#### 介绍

![Screen Shot 2022-03-24 at 3 05 13 PM](https://user-images.githubusercontent.com/17645053/159861108-5d8d8f6c-7b97-4c17-a44e-5b329b503110.png)

*   用户代理：电子邮件客户端软件，如QQ mail、gmail

    包括发送协议和读取协议

    <img src="https://user-images.githubusercontent.com/17645053/159861524-68093bdb-35b0-484e-8f22-c5955c1420ce.png" alt="Screen Shot 2022-03-24 at 3 08 26 PM" data-size="original" />
*   SMTP/POP3客户/服务器进行TCP连接

    <img src="https://user-images.githubusercontent.com/17645053/159861769-8a4416ef-df3a-4116-932d-784842c82bce.png" alt="Screen Shot 2022-03-24 at 3 10 04 PM" data-size="original" />

#### SMTP基本工作原理

![Screen Shot 2022-03-24 at 3 35 29 PM](https://user-images.githubusercontent.com/17645053/159865631-1af0e13b-765b-4a3a-91a1-ccfce4a2ab7e.png)

*   电子邮件信息格式不是SMTP定义的，而是RFC 822单独定义的。

    有信封、内容两部分。内容有首部、主体。

    <img src="https://user-images.githubusercontent.com/17645053/159865979-0c1a6cf3-fcb2-404a-bcd3-8c6629f41285.png" alt="Screen Shot 2022-03-24 at 3 37 43 PM" data-size="original" />
*   SMTP只能传送`ASCII`码文本数据

    所以不能传送图片、音频、视频、非英文国家文字等

    所以提出`MIME`, 可以将非`ASCII`码转换成`ASCII`码

    <img src="https://user-images.githubusercontent.com/17645053/159866229-893c14af-da11-42d1-a765-a33b4d9a4713.png" alt="Screen Shot 2022-03-24 at 3 39 30 PM" data-size="original" />

    MIME新增5个邮件首部字段，也用于后来同样面向ASCII字符的HTTP

    <img src="https://user-images.githubusercontent.com/17645053/159866431-1fe9ba47-a4f8-4cb6-b7f3-68756e1cac30.png" alt="Screen Shot 2022-03-24 at 3 40 51 PM" data-size="original" />

#### 常用邮件读取协议：POP、IMAP

* 都采用基于TCP连接C/S服务器方式，POP3使用熟知端口110，IMAP4使用熟知端口143

![Screen Shot 2022-03-24 at 3 43 12 PM](https://user-images.githubusercontent.com/17645053/159866813-3b38b22e-cd00-4316-ac2a-7b49aabec297.png)

#### 基于万维网的电子邮件

* 不需要安装专门的用户代理程序，只需要使用通用的万维网**浏览器**
* 使用HTTP协议而不是SMTP协议

![Screen Shot 2022-03-24 at 3 46 29 PM](https://user-images.githubusercontent.com/17645053/159867324-5769506e-6349-424b-bf4f-f88f579dda8d.png)

### 万维网WWW

![Screen Shot 2022-03-24 at 3 50 23 PM](https://user-images.githubusercontent.com/17645053/159867964-4e552a9d-9d53-4aa3-aa2f-05cf948b6233.png) ![Screen Shot 2022-03-24 at 3 52 30 PM](https://user-images.githubusercontent.com/17645053/159868248-b6321d07-3462-4df5-b1b4-4a1d60c58f2f.png)

#### HTTP

* 客户进程与服务器建立TCP连接，客户进程通过TCP连接发送HTTP请求

![Screen Shot 2022-03-24 at 3 57 00 PM](https://user-images.githubusercontent.com/17645053/159868949-d2325f5d-f5f4-453d-9351-8bff18cb2fde.png)

​ HTTP/1.0 采用非持续连接。每次浏览器都要与服务器建立TCP连接，收到响应后就立即关闭连接。

![Screen Shot 2022-03-24 at 3 58 35 PM](https://user-images.githubusercontent.com/17645053/159869276-8dd9f3b0-2c72-461f-b773-96181eadd593.png)

HTTP/1.1 采用持续连接方式。使用**流水线**方式工作，即浏览器在收到HTTP响应报文之前就能连续发送多个请求报文。使TCP连接中空闲时间减少，提高了下载文档的效率。

![Screen Shot 2022-03-24 at 4 02 10 PM](https://user-images.githubusercontent.com/17645053/159869696-141b490d-8189-421a-8592-099672ab5fff.png)

#### HTTP请求报文格式

* 面向文本，每一个字段是ASCII码串，且每个字段长度都不确定

![Screen Shot 2022-03-24 at 4 08 48 PM](https://user-images.githubusercontent.com/17645053/159870818-c3c659e3-f450-4aff-95c8-66759be3fbe9.png)

​ 支持以下方法

![Screen Shot 2022-03-24 at 4 09 10 PM](https://user-images.githubusercontent.com/17645053/159870864-64acf008-7ce5-4c92-9124-14018af71ff7.png)

*   HTTP响应报文格式

    <img src="https://user-images.githubusercontent.com/17645053/159871063-570a6e1b-1712-440a-8f72-ea68567848bd.png" alt="Screen Shot 2022-03-24 at 4 10 16 PM" data-size="original" />

#### Cookie

* 使用cookie在服务器上记录用户信息，是对无状态HTTP进行状态化的技术

![Screen Shot 2022-03-24 at 4 12 10 PM](https://user-images.githubusercontent.com/17645053/159871390-ee9eb4d8-e823-4592-a5d7-74c837744c15.png)

*   Cookie的工作原理

    www服务器为用户生成唯一cookie识别码，并以此为index在后端数据库中创建一个项目，用来记录用户访问该网站的各种信息。用户将cookie信息存入文件，再发http请求时带上cookie，就可以让服务器识别出该用户，并返回该用户的个性化网页。

    <img src="https://user-images.githubusercontent.com/17645053/159871936-86c0d2c0-b51e-4c2d-b820-180bb93707cb.png" alt="Screen Shot 2022-03-24 at 4 13 52 PM" data-size="original" />

#### WWW缓存与代理服务器

* web缓存可以位于客户机，也可以位于中间系统，位于中间系统上的web缓存又称为代理服务器。

![Screen Shot 2022-03-24 at 4 21 49 PM](https://user-images.githubusercontent.com/17645053/159873083-800401bc-ec3b-4db7-a2cc-7916171bb1ef.png)

* 如果已过期，就发送一个请求，包括字段`if-modified-since`，值为`last-modified`，服务器判断是否一致后响应304，并不携带实体。代理服务器重新更新该文档的有效日期，代理服务器再发给主机

![Screen Shot 2022-03-24 at 4 24 36 PM](https://user-images.githubusercontent.com/17645053/159873774-c547dceb-eb9b-4549-b3b2-e627ae35f879.png)

* 如果不一致，就响应200，并携带实体

![Screen Shot 2022-03-24 at 4 27 09 PM](https://user-images.githubusercontent.com/17645053/159874114-053676d5-5e0b-466c-a9f3-207774f2c0a3.png)

### [HTTP/0.9 1.0 1.1 2.0 3.0](https://blog.poetries.top/browser-working-principle/guide/part6/lesson29.html)

#### [HTTP/0.9](https://blog.poetries.top/browser-working-principle/guide/part6/lesson29.html#%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE-http-0-9)

HTTP/0.9 的一个完整的请求流程。

* 因为 HTTP 都是基于 TCP 协议的，所以客户端先要根据 IP 地址、端口和服务器建立 TCP 连接，而建立连接的过程就是 TCP 协议三次握手的过程。
* 建立好连接之后，会发送一个 GET 请求行的信息，如GET /index.html用来获取 index.html。
* 服务器接收请求信息之后，读取对应的 HTML 文件，并将数据以 ASCII 字符流返回给客户端。
* HTML 文档传输完成后，断开连接。

当时的需求很简单，就是用来传输体积很小的 HTML 文件，所以 HTTP/0.9 的实现有以下三个特点。

* 第一个是**只有一个请求行**，并**没有HTTP 请求头和请求体**，因为只需要一个请求行就可以完整表达客户端的需求了。
* 第二个是**服务器也没有返回头信息**，这是因为服务器端并不需要告诉客户端太多信息，只需要返回数据就可以了。
* 第三个是返回的文件内容是以 **ASCII** 字符流来传输的，因为**都是 HTML 格式**的文件，所以使用 ASCII 字节码来传输是最合适的。

#### [HTTP/1.0](https://blog.poetries.top/browser-working-principle/guide/part6/lesson29.html#%E8%A2%AB%E6%B5%8F%E8%A7%88%E5%99%A8%E6%8E%A8%E5%8A%A8%E7%9A%84-http-1-0)

1994 年底出现了拨号上网服务，同年网景又推出一款浏览器，从此万维网就不局限于学术交流了，而是进入了高速的发展阶段。随之而来的是万维网联盟（W3C）和 HTTP 工作组（HTTP-WG）的创建，它们致力于 HTML 的发展和 HTTP 的改进。

新兴网络都带来了的新需求：

1. 在浏览器中展示的不单是 HTML 文件了，还包括了 JavaScript、CSS、图片、音频、视频等不同类型的文件。因此支持多种类型的文件下载是 HTTP/1.0 的一个核心诉求
2. 文件格式不仅仅局限于 ASCII 编码，还有很多其他类型编码的文件。

那么该如何实现多种类型文件的下载呢？

**HTTP/1.0 引入了请求头和响应头**，它们都是以为 Key-Value 形式保存的，在 HTTP 发送请求时，会带上请求头信息，**服务器**返回数据时，会**先返回响应头信息**。

有了请求头和响应头，浏览器和服务器就能进行更加深入的交流了。

那 HTTP/1.0 是怎么通过请求头和响应头来支持多种不同类型的数据呢？

要支持多种类型的文件，我们就需要解决以下几个问题

1. 浏览器需要知道服务器返回的**数据是什么类型**的，然后浏览器才能根据不同的数据类型做针对性的处理。
2. 由于万维网所支持的应用变得越来越广，所以单个文件的数据量也变得越来越大。为了减轻传输性能，服务器会对数据进行压缩后再传输，所以浏览器需要知道服务器**压缩的方法**。
3. 由于万维网是支持全球范围的，所以需要提供国际化的支持，服务器需要对不同的地区提供不同的语言版本，这就需要浏览器告诉服务器它想要什么**语言版本**的页面。
4. 最后，由于增加了各种不同类型的文件，而每种文件的编码形式又可能不一样，为了能够准确地读取文件，浏览器需要知道**文件的编码类型**

基于以上问题，HTTP/1.0 的方案是通过请求头和响应头来进行协商，在发起请求时候会通过 HTTP 请求头告诉服务器它期待服务器返回什么类型的文件、采取什么形式的压缩、提供什么语言的文件以及文件的具体编码。最终发送出来的请求头内容如下：

```
accept: text/html
accept-encoding: gzip, deflate, br
accept-Charset: ISO-8859-1,utf-8
accept-language: zh-CN,zh
```

HTTP/1.0 除了对多文件提供良好的支持外，还依据当时实际的需求引入了很多其他的特性，这些特性都是通过请求头和响应头来实现的。下面我们来看看新增的几个典型的特性

1. 有的请求服务器可能无法处理，或者处理出错，这时候就需要告诉浏览器服务器最终处理该请求的情况，这就**引入了状态码**。状态码是通过响应行的方式来通知浏览器的。
2. 为了减轻服务器的压力，在 HTTP/1.0 中提供了**Cache 机制**，用来缓存已经下载过的数据。
3. 服务器需要**统计客户端的基础信息**，比如 Windows 和 macOS 的用户数量分别是多少，所以 HTTP/1.0 的请求头中还加入了**用户代理的字段**

#### [HTTP/1.1](https://blog.poetries.top/browser-working-principle/guide/part6/lesson29.html#%E7%BC%9D%E7%BC%9D%E8%A1%A5%E8%A1%A5%E7%9A%84-http-1-1)

1.  支持长连接`PersistentConnection`

    如果在下载每个文件的时候，都需要经历建立 TCP 连接、传输数据和断开连接这样的步骤，无疑会增加大量无谓的开销。

    为了解决这个问题，HTTP/1.1 中增加了持久连接的方法，它的特点是在一个 TCP 连接上可以传输多个 HTTP 请求，只要浏览器或者服务器**没有明确断开连接**，那么该 **TCP 连接会一直保持**。

    **持久连接在 HTTP/1.1 中是默认开启的**`Connection： keep-alive`，如果你不想要采用持久连接，可以在 HTTP 请求头中加上`Connection: close`。目前浏览器中对于**同一个域名**，**默认**允许**同时建立 6 个 TCP 持久连接**。

    [更多解释](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Connection\_management\_in\_HTTP\_1.x#%E9%95%BF%E8%BF%9E%E6%8E%A5): 短连接有两个比较大的问题：创建新连接耗费的时间尤为明显，另外 TCP 连接的性能只有在该连接被使用一段时间后(热连接)才能得到改善。为了缓解这些问题，_长连接_ 的概念便被设计出来了，甚至在 HTTP/1.1 之前。或者这被称之为一个 _keep-alive_ 连接。

    一个长连接会保持一段时间，重复用于发送一系列请求，节省了新建 TCP 连接握手的时间，还可以利用 TCP 的性能增强能力。当然这个连接也不会一直保留着：连接在空闲一段时间后会被关闭(服务器可以使用 [`Keep-Alive`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Keep-Alive) 协议头来指定一个最小的连接保持时间)。

    长连接也还是有缺点的；就算是在空闲状态，它还是会消耗服务器资源，而且在重负载时，还有可能遭受 [DoS attacks](https://developer.mozilla.org/zh-CN/docs/Glossary/DOS\_attack) 攻击。这种场景下，可以使用非长连接，即尽快关闭那些空闲的连接，也能对性能有所提升。

    HTTP/1.0 里默认并不使用长连接。把 [`Connection`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Connection) 设置成 `close` 以外的其它参数都可以让其保持长连接，通常会设置为 `retry-after。`

    在 HTTP/1.1 里，默认就是长连接的，协议头都不用再去声明它(但我们还是会把它加上，万一某个时候因为某种原因要退回到 HTTP/1.0 呢)。
2.  请求流水线`pipeline`

    持久连接虽然能减少 TCP 的建立和断开次数，但是它**需要等待前面的请求返回之后，才能进行下一次请求**。如果 TCP 通道中的某个请求因为某些原因没有及时返回，那么就会阻塞后面的所有请求，这就是著名的**队头阻塞**的问题。

    HTTP/1.1 中试图通过管线化的技术来解决队头阻塞的问题。HTTP/1.1 中的管线化是指将**多个 HTTP 请求整批提交给服务器的技术**，虽然可以整批发送请求，不过服务器依然需要根据请求顺序来回复浏览器的请求。

    FireFox、Chrome 都做过管线化的试验，但是由于各种原因，它们最终都放弃了管线化技术。

    [更多解释](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Connection\_management\_in\_HTTP\_1.x#http\_%E6%B5%81%E6%B0%B4%E7%BA%BF)

    默认情况下，[HTTP](https://developer.mozilla.org/en-US/HTTP) 请求是按顺序发出的。下一个请求只有在当前请求收到应答过后才会被发出。由于会受到网络延迟和带宽的限制，在下一个请求被发送到服务器之前，可能需要等待很长时间。

    流水线是在同一条长连接上发出连续的请求，而不用等待应答返回。这样可以避免连接延迟。理论上讲，性能还会因为两个 HTTP 请求有可能被打包到一个 TCP 消息包中而得到提升。就算 HTTP 请求不断的继续，尺寸会增加，但设置 TCP 的 [MSS](https://en.wikipedia.org/wiki/Maximum\_segment\_size)(Maximum Segment Size) 选项，仍然足够包含一系列简单的请求。

    并不是所有类型的 HTTP 请求都能用到流水线：只有 [idempotent](https://developer.mozilla.org/zh-CN/docs/Glossary/Idempotent) 方式，比如 [`GET`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/GET)、[`HEAD`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/HEAD)、[`PUT`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/PUT) 和 [`DELETE`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/DELETE) 能够被安全的重试：如果有故障发生时，流水线的内容要能被轻易的重试。

    今天，所有遵循 HTTP/1.1 的代理和服务器都应该支持流水线，虽然实际情况中还是有很多限制：一个很重要的原因是，目前没有现代浏览器默认启用这个特性。

    [域名分片](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Connection\_management\_in\_HTTP\_1.x#%E5%9F%9F%E5%90%8D%E5%88%86%E7%89%87)可以增加同时建立的TCP连接

    <img src="https://user-images.githubusercontent.com/17645053/160309836-82739766-be17-4742-9100-fabf8525cbaf.png" alt="Screen Shot 2022-03-28 at 9 02 46 AM" data-size="original" />
3.  提供虚拟主机Host头的支持

    在 HTTP/1.0 中，每个域名绑定了一个唯一的 IP 地址，因此一个服务器只能支持一个域名。但是随着虚拟主机技术的发展，需要实现在[一台物理主机上绑定多个虚拟主机](https://www.zhihu.com/question/29390934)，每个虚拟主机都有自己的单独的域名，这些单独的域名都公用同一个 IP 地址。(多个域名公用一个IP)

    因此，HTTP/1.1 的请求头中增加了Host 字段，用来表示当前的域名地址，这样服务器就可以根据不同的 Host 值做不同的处理。
4.  对动态生成的内容提供了完美支持

    在设计 HTTP/1.0 时，需要在响应头中设置完整的数据大小，如`Content-Length: 901`，这样浏览器就可以根据设置的数据大小来接收数据。不过随着服务器端的技术发展，**很多页面的内容都是动态生成的**，因此**在传输数据之前并不知道最终的数据大小**，这就导致了**浏览器不知道何时会接收完所有的文件数据**。

    HTTP/1.1 通过引入`Chunk transfer` 机制来解决这个问题，服务器会将数据分割成若干个任意大小的数据块，每个数据块发送时会**附上上个数据块的长度**，最后使用一个零长度的块作为发送数据完成的标志。这样就**提供了对动态内容**的支持。
5.  客户端 Cookie、安全机制

    除此之外，HTTP/1.1 还引入了客户端 **Cookie 机制和安全机制**。其中，Cookie 机制我们在《03 | HTTP 请求流程：为什么很多站点第二次打开速度会很快？》这篇文章中介绍过了，而安全机制我们会在后面的安全模块中再做介绍，这里就不赘述了。
6.  错误通知的管理

    在HTTP1.1中新增了24个错误状态响应码，如409（Conflict）表示请求的资源与资源的当前状态发生冲突；410（Gone）表示服务器上的某个资源被永久性的删除。
7.  带宽优化及网络连接的使用

    HTTP1.0中，存在一些浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了，并且不支持断点续传功能，HTTP1.1则在请求头引入了range头域，它允许只请求资源的某个部分，即返回码是206（Partial Content），这样就方便了开发者自由的选择以便于充分利用带宽和连接。

#### [SPDY](https://cloud.tencent.com/developer/article/1082516)

来源：[HTTP,HTTP2.0,SPDY,HTTPS](https://cloud.tencent.com/developer/article/1082516)

SPDY可以说是综合了HTTPS和HTTP两者有点于一体的传输协议，**主要解决：**

**1、降低延迟**，针对HTTP高延迟的问题，SPDY优雅的采取了多路复用（multiplexing）。多路复用通过多个请求stream共享一个tcp连接的方式，解决了HOL blocking的问题，降低了延迟同时提高了带宽的利用率。

**2、请求优先级（request prioritization）**。多路复用带来一个新的问题是，在连接共享的基础之上有可能会导致关键请求被阻塞。SPDY允许给每个request设置优先级，这样重要的请求就会优先得到响应。比如浏览器加载首页，首页的html内容应该优先展示，之后才是各种静态资源文件，脚本文件等加载，这样可以保证用户能第一时间看到网页内容。

**3、header压缩**。前面提到HTTP1.x的header很多时候都是重复多余的。选择合适的压缩算法可以减小包的大小和数量。

**4、基于HTTPS的加密协议传输**，大大提高了传输数据的可靠性。

**5、服务端推送（server push）**，采用了SPDY的网页，例如我的网页有一个sytle.css的请求，在客户端收到sytle.css数据的同时，服务端会将sytle.js的文件推送给客户端，当客户端再次尝试获取sytle.js时就可以直接从缓存中获取到，不用再发请求了。

位于HTTP之下，TCP和SSL之上，这样可以轻松兼容老版本的HTTP协议(将HTTP1.x的内容封装成一种新的frame格式)，同时可以使用已有的SSL功能。

HTTP2.0可以说是SPDY的升级版（其实原本也是基于SPDY设计的），但是，HTTP2.0 跟 SPDY 仍有不同的地方，主要是以下两点：

* HTTP2.0 支持明文 HTTP 传输，而 SPDY 强制使用 HTTPS
* HTTP2.0 消息头的压缩算法采用 `HPACK`，而非 SPDY 采用的 `DEFLATE`

#### [HTTP/2.0](https://blog.poetries.top/browser-working-principle/guide/part6/lesson30.html)

HTTP/1.1 为网络效率做了大量的优化，最核心的有如下三种方式：

1. 增加了持久连接；
2. 浏览器为每个域名最多同时维护 6 个 TCP 持久连接；
3.  使用 CDN 的实现域名分片机制。(把资源的域名分为多个二级域名，指向原本的同一资源，这样就避免了同一域名的并发连接数量）

    来自[知乎](https://www.zhihu.com/question/34401250): 平均一个域名有18个TCP连接。所以为了突破浏览器的限制，大部分网站使用了域名分片，[domain sharding](https://www.zhihu.com/search?q=domain+sharding\&search\_source=Entity\&hybrid\_search\_source=Entity\&hybrid\_search\_extra=\{%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A58746920})。即建立多个域名，这样每个域名都能建立6个连接，就能提升并发连接数了。域名分片有如下缺点

    * 增加域名管理、IDC容灾和运维的难度。这个是大部分FE考虑不到的，但很重要。
    * domain sharding推荐的数量是2个，可以参考这个数据：[http://www.mobify.com/blog/domain-sharding-bad-news-mobile-performance/](https://link.zhihu.com/?target=http%3A//www.mobify.com/blog/domain-sharding-bad-news-mobile-performance/) 和这篇文章：[Domain Sharding revisited](https://link.zhihu.com/?target=http%3A//www.stevesouders.com/blog/2013/09/05/domain-sharding-revisited/)
    * 盲目增加[TCP连接数](https://www.zhihu.com/search?q=TCP%E8%BF%9E%E6%8E%A5%E6%95%B0\&search\_source=Entity\&hybrid\_search\_source=Entity\&hybrid\_search\_extra=\{%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A58746920})会极大破坏TCP拥塞机制。因为TCP拥塞和流控的前提假设是建立在一个TCP连接上的，如果同时建立多个TCP连接，对于多个连接并发流量的突增和暴涨，TCP无法有效控制。

[http/1.1的主要问题](https://blog.poetries.top/browser-working-principle/guide/part6/lesson30.html#http-1-1-%E7%9A%84%E4%B8%BB%E8%A6%81%E9%97%AE%E9%A2%98)

前面我们分析了 HTTP/1.1 所存在的一些主要问题：

1. TCP慢启动。
2. TCP 连接之间相互竞争: 同时开启多条TCP一定会竞争带宽。关键资源不能优先下载
3. 队头阻塞是由于 HTTP/1.1 的机制导致的。HTTP/1.1中只有该请求响应后才能重用当前连接发送下一次请求，即使管道化技术使得可以在本次响应完成之前发送下次请求，但响应依然严格按照顺序返回，也就是如果前一个响应被阻塞，后边的响应将不会到来。

[http.2.0的多路复用](https://blog.poetries.top/browser-working-principle/guide/part6/lesson30.html#http-2-%E7%9A%84%E5%A4%9A%E8%B7%AF%E5%A4%8D%E7%94%A8)

虽然 TCP 有问题，但是我们依然没有换掉 TCP 的能力，所以我们就要想办法去规避 TCP 的慢启动和 TCP 连接之间的竞争问题。

基于此，HTTP/2 的思路就是**一个域名**只使用**一个 TCP 长连接**来传输数据，这样整个页面资源的下载过程只需要一次慢启动，同时也避免了多个 TCP 连接竞争带宽所带来的问题。

HTTP/2 的解决方案可以总结为：一个域名只使用一个 TCP 长连接和消除队头阻塞问题。

每个请求都有一个对应的 ID，如 stream1 表示 index.html 的请求，stream2 表示 foo.css 的请求。这样在浏览器端，就可以随时将请求发送给服务器了。

服务器端接收到这些请求后，会**根据自己的喜好来决定优先返回哪些内容**，比如服务器可能早就缓存好了 index.html 和 bar.js 的响应头信息，那么当接收到请求的时候就可以立即把 index.html 和 bar.js 的响应头信息返回给浏览器，然后再将 index.html 和 bar.js 的响应体数据返回给浏览器。之所以可以随意发送，是因为每份数据都有对应的 ID，浏览器接收到之后，会筛选出相同 ID 的内容，将其拼接为完整的 HTTP 响应数据。

可以将请求分成一帧一帧的数据去传输，这样带来了一个额外的好处，就是当收到一个优先级高的请求时，比如接收到 **JavaScript 或者 CSS 关键资源**的请求，服务器可以**暂停之前的请求来优先处理关键资源**的请求

从图中可以看出，HTTP/2 添加了一个**二进制分帧层**，那我们就结合图来分析下 HTTP/2 的请求和接收过程。

* 首先，浏览器准备好请求数据，包括了请求行、请求头等信息，如果是 POST 方法，那么还要有请求体。
* 这些数据经过**二进制分帧层**处理之后，会被转换为**一个个带有请求 ID 编号的帧**，通过协议栈将这些帧发送给服务器。
* 服务器接收到所有帧之后，会将**所有相同 ID 的帧合并为一条完整的请求信息**。
* 然后**服务器**处理该条请求，并**将处理的响应行、响应头和响应体分别发送至二进制分帧层**。
* 同样，二进制分帧层会将这些响应数据转换为一个个带有请求 ID 编号的帧，**经过协议栈发送给浏览器**。
* 浏览器接收到响应帧之后，会根据 ID 编号将帧的数据提交给对应的请求

从上面的流程可以看出，通过引入\*\*二进制分帧层\*\*，就实现了 HTTP 的多路复用技术。

上一篇文章我们介绍过，HTTP 是浏览器和服务器通信的语言，在这里虽然 HTTP/2 引入了二进制分帧层，不过 HTTP/2 的语义和 HTTP/1.1 依然是一样的，也就是说它们通信的语言并没有改变，比如开发者依然可以通过 Accept 请求头告诉服务器希望接收到什么类型的文件，依然可以使用 Cookie 来保持登录状态，依然可以使用 Cache 来缓存本地文件，这些都没有变，发生改变的只是传输方式。这一点对开发者来说尤为重要，这意味着我们不需要为 HTTP/2 去重建生态，并且 HTTP/2 推广起来会也相对更轻松了

[HTTP/2 其他特性](https://blog.poetries.top/browser-working-principle/guide/part6/lesson30.html#http-2-%E5%85%B6%E4%BB%96%E7%89%B9%E6%80%A7)

1.  可以设置请求的优先级

    我们知道浏览器中有些数据是非常重要的，但是在发送请求时，重要的请求可能会晚于那些不怎么重要的请求，如果服务器按照请求的顺序来回复数据，那么这个重要的数据就有可能推迟很久才能送达浏览器，这对于用户体验来说是非常不友好的。

    为了解决这个问题，HTTP/2 提供了请求优先级，可以在发送请求时，**标上该请求的优先级**，这样服务器接收到请求之后，会优先处理优先级高的请求。
2.  服务器推送Server Push

    除了设置请求的优先级外，HTTP/2 还**可以直接将数据提前推送到浏览器**。你可以想象这样一个场景，当用户请求一个 HTML 页面之后，服务器知道该 HTML 页面会引用几个重要的 JavaScript 文件和 CSS 文件，那么在接收到 HTML 请求之后，附带将要使用的 CSS 文件和 JavaScript 文件一并发送给浏览器，这样当浏览器解析完 HTML 文件之后，就能直接拿到需要的 CSS 文件和 JavaScript 文件，这对首次打开页面的速度起到了至关重要的作用。

    [更多解释](https://juejin.cn/post/6844903503983280141#heading-8): 简单来讲，Server Push就是允许Server对还没有发出的请求进行响应。如下图所示，在第一个请求中，客户端发出了一条对index.html的请求。通过对index.html的分析，服务器可以推测出其该html的加载还需要style.css和script.js，因此在第一条请求的响应中可以将这些内容一并发送过去，避免了接下来两条请求、响应带来的延迟（RTT, Round-trip Time）。虽然在HTTP/1.x中采用inline的方式，也可以达到类似Server Push的效果，但inline具有以下弊端：

    * CSS或js没有单独的缓存，如果html的缓存方式与被inline的内容不一致，在之后对该页面访问时，本来可以缓存的css或js内容被重复请求加载
    * 其他页面用到该样式会被重复请求加载
3.  头部压缩HPack

    无论是 HTTP/1.1 还是 HTTP/2，它们都有请求头和响应头，这是浏览器和服务器的通信语言。HTTP/2 对**请求头和响应头**进行了**压缩**，你可能觉得一个 HTTP 的头文件没有多大，压不压缩可能关系不大，但你这样想一下，在浏览器发送请求的时候，基本上都是发送 HTTP 请求头，很少有请求体的发送，通常情况下页面也有 100 个左右的资源，如果将这 100 个请求头的数据压缩为原来的 20%，那么传输效率肯定能得到大幅提升。

[去除某些http/1.1的优化](https://juejin.cn/post/6844903503983280141#heading-12):

* 域名分片
* 雪碧图
* 拼接JS、CSS

**总结**

我们首先分析了影响 HTTP/1.1 效率的三个主要因素：**TCP 的慢启动、多条 TCP 连接竞争带宽和队头阻塞**。

接下来我们分析了 HTTP/2 是如何采用多路复用机制来解决这些问题的。多路复用是通过在协议栈中添加二进制分帧层来实现的，有了二进制分帧层还能够实现请求的优先级、服务器推送、头部压缩等特性，从而大大提升了文件传输效率。

HTTP/2 协议规范于 2015 年 5 月正式发布，在那之后，该协议已在互联网和万维网上得到了广泛的实现和部署。从目前的情况来看，国内外一些排名靠前的站点基本都实现了 HTTP/2 的部署。使用 HTTP/2 能带来 20%～60% 的效率提升，至于 20% 还是 60% 要看优化的程度。总之，我们也应该与时俱进，放弃 HTTP/1.1 和其性能优化方法，去“拥抱”HTTP/2

#### [深入理解http/2.0协议](https://juejin.cn/post/6844903984524705800)

*   Connection、Message、Stream、Frame

    <img src="https://user-images.githubusercontent.com/17645053/160074556-8ffe249a-5523-4fd4-a61c-34dc829150a2.png" alt="Screen Shot 2022-03-25 at 3 28 06 PM" data-size="original" />
*   二进制分帧 - http2的基石

    流(Stream)是连接中的一个虚拟信道，可以承载双向消息传输。每个流有唯一整数标识符。为了防止两端流ID冲突，**客户端**发起的流具有奇数ID，服务器端发起的流具有偶数ID。

    同一个Stream内Frame是有序的。
*   多路复用

    每个域名一个连接
*   HPACK

    原理：用header字段表里的索引代替实际的header。

    只要给服务端发送一个 Frame，该 Frame 的 Payload 部分存储 0x8285，Frame 的 Type 设置为 Header 类型，便可表示这个 Frame 属于 http Header

    为什么是 0x8285，而不是 0x0205？这是因为高位设置为 **1** 表示这个字节是一个**完全索引值（key 和 value 都在索引中）**。

    类似的，通过高位的标志位可以区分出这个字节是属于一个**完全索引值**，还是**仅索引了 key**，还是 **key和value 都没有索引**(参见：HTTP/2首部压缩的OkHttp3实现④)。

    因为索引表的**大小的是有限**的，它仅保存了一些常用的 http Header，同时每次请求还可以在表的末尾**动态追加新的 http Header** 缓存，动态部分称之为 **Dynamic Table**。[Static Table](https://datatracker.ietf.org/doc/html/rfc7541) 和 Dynamic Table 在一起组合成了索引表。

    HPACK 不仅仅通过索引键值对来降低数据量，同时还会将字符串进行**霍夫曼编码来压缩**字符串大小。
*   Request Priorities

    每个流都可以带有一个31比特的优先值：0 表示最高优先级；2的31次方-1 表示最低优先级。

    ● 优先级最高：主要的html

    ● 优先级高：CSS文件

    ● 优先级中：js文件

    ● 优先级低：图片
* Server Push

#### [HTTP/3.0](https://blog.poetries.top/browser-working-principle/guide/part6/lesson31.html#http3%EF%BC%9A%E7%94%A9%E6%8E%89tcp%E3%80%81tcl%E5%8C%85%E8%A2%B1-%E6%9E%84%E5%BB%BA%E9%AB%98%E6%95%88%E7%BD%91%E7%BB%9C)

HTTP/2.0的问题：

1.  [TCP队头阻塞](https://blog.poetries.top/browser-working-principle/guide/part6/lesson31.html#tcp-%E7%9A%84%E9%98%9F%E5%A4%B4%E9%98%BB%E5%A1%9E)

    <img src="https://user-images.githubusercontent.com/17645053/159911823-4a4109ba-fc76-4cbd-991b-4a724cd69072.png" alt="Screen Shot 2022-03-24 at 7 59 59 PM" data-size="original" />

    在 HTTP/2 中，多个请求是跑在一个 TCP 管道中的，如果其中任意一路数据流中出现了丢包的情况，那么就会阻塞该 TCP 连接中的所有请求。这不同于 HTTP/1.1，使用 HTTP/1.1 时，浏览器为每个域名开启了 6 个 TCP 连接，如果其中的 1 个 TCP 连接发生了队头阻塞，那么其他的 5 个连接依然可以继续传输数据。

    丢包率的增加，HTTP/2 的传输效率也会越来越差。有测试数据表明，当系统达到了 2% 的丢包率时，HTTP/1.1 的传输效率反而比 HTTP/2 表现得更好。
2.  [TCP建立时延](https://blog.poetries.top/browser-working-principle/guide/part6/lesson31.html#tcp-%E5%BB%BA%E7%AB%8B%E8%BF%9E%E6%8E%A5%E7%9A%84%E5%BB%B6%E6%97%B6)

    HTTP/1 和 HTTP/2 都是使用 TCP 协议来传输的，而如果使用 HTTPS 的话，还需要使用 TLS 协议进行安全传输，而使用 TLS 也需要一个握手过程，这样就需要有两个握手延迟过程。

    * 在建立 TCP 连接的时候，需要和服务器进行三次握手来确认连接成功，也就是说需要在消耗完 1.5 个 RTT 之后才能进行数据传输。
    * 进行 TLS 连接，TLS 有两个版本——TLS1.2 和 TLS1.3，每个版本建立连接所花的时间不同，大致是需要 1～2 个 RTT，关于 HTTPS 我们到后面到安全模块再做详细介绍

    总之，在传输数据之前，我们需要花掉 3～4 个 RTT。如果浏览器和服务器的物理距离较近，那么 1 个 RTT 的时间可能在 10 毫秒以内，也就是说总共要消耗掉 30～40 毫秒。这个时间也许用户还可以接受，但如果服务器相隔较远，那么 1 个 RTT 就可能需要 100 毫秒以上了，这种情况下整个握手过程需要 300～400 毫秒，这时用户就能明显地感受到“慢”了。
3.  [TCP协议僵化](https://blog.poetries.top/browser-working-principle/guide/part6/lesson31.html#tcp-%E5%8D%8F%E8%AE%AE%E5%83%B5%E5%8C%96)

    如果我们在客户端升级了 TCP 协议，但是当新协议的数据包经过这些中间设备时，它们可能不理解包的内容，于是这些数据就会被丢弃掉。这就是中间设备僵化，它是阻碍 TCP 更新的一大障碍。

    除了中间设备僵化外，操作系统也是导致 TCP 协议僵化的另外一个原因。因为 TCP 协议都是通过操作系统内核来实现的，应用程序只能使用不能修改。通常操作系统的更新都滞后于软件的更新，因此要想自由地更新内核中的 TCP 协议也是非常困难的

[QUIC协议](https://blog.poetries.top/browser-working-principle/guide/part6/lesson31.html#quic-%E5%8D%8F%E8%AE%AE)

![Screen Shot 2022-03-24 at 8 03 22 PM](https://user-images.githubusercontent.com/17645053/159912343-47547e62-a2e5-417b-b1d1-5336dbcad4c0.png)

QUIC协议建立在UDP之上。基于 UDP 实现了类似于 TCP 的多路数据流、传输可靠性等功能

* 实现了类似 TCP 的流量控制、传输可靠性的功能。虽然 UDP 不提供可靠性的传输，但 QUIC 在 UDP 的基础之上增加了一层来保证数据可靠性传输。它提供了数据包重传、拥塞控制以及其他一些 TCP 中存在的特性。
* 集成了 TLS 加密功能。目前 QUIC 使用的是 TLS1.3，相较于早期版本 **TLS1.3 有更多的优点**，其中最重要的一点是**减少了握手所花费的 RTT 个数**。
*   实现了 HTTP/2 中的多路复用功能。和 TCP 不同，QUIC 实现了在同一物理连接上可以有**多个独立的逻辑数据流**（如下图）。实现了数据流的单独传输，就解决了 TCP 中队头阻塞的问题。

    <img src="https://user-images.githubusercontent.com/17645053/159911646-818255e9-1ef9-4aea-ab5f-f93d4c89e9de.png" alt="Screen Shot 2022-03-24 at 7 58 41 PM" data-size="original" />

    实现了快速握手功能。由于 QUIC 是基于 UDP 的，所以 QUIC 可以实现使用 0-RTT 或者 1-RTT 来建立连接，这意味着 QUIC 可以用最快的速度来发送和接收数据，这样可以大大提升首次打开页面的速度。

[HTTP/3 的挑战](https://blog.poetries.top/browser-working-principle/guide/part6/lesson31.html#http-3-%E7%9A%84%E6%8C%91%E6%88%98)

1. 从目前的情况来看，服务器和浏览器端都没有对 HTTP/3 提供比较完整的支持。Chrome 虽然在数年前就开始支持 Google 版本的 QUIC，但是这个版本的 QUIC 和官方的 QUIC 存在着非常大的差异。
2. 部署 HTTP/3 也存在着非常大的问题。因为系统内核对 UDP 的优化远远没有达到 TCP 的优化程度，这也是阻碍 QUIC 的一个重要原因。
3. 中间设备僵化的问题。这些设备对 UDP 的优化程度远远低于 TCP，据统计使用 QUIC 协议时，大约有 3%～7% 的丢包率。

**总结**

我们首先分析了 HTTP/2 中所存在的一些问题，主要包括了 TCP 的队头阻塞、建立 TCP 连接的延时、TCP 协议僵化等问题。

这些问题都是 TCP 的内部问题，因此要解决这些问题就要优化 TCP 或者“另起炉灶”创造新的协议。由于优化 TCP 协议存在着诸多挑战，所以官方选择了创建新的 QUIC 协议。

HTTP/3 正是基于 QUIC 协议的，你可以把 QUIC 看成是集成了“**TCP+HTTP/2 的多路复用 +TLS 等功能**”的一套协议。这是集众家所长的一个协议，从协议最底层对 Web 的文件传输做了比较彻底的优化，所以等生态相对成熟时，可以用来打造比现在的 HTTP/2 还更加高效的网络。

虽说这套协议解决了 HTTP/2 中因 TCP 而带来的问题，不过由于是改动了底层协议，所以推广起来还会面临着巨大的挑战。

**关于 HTTP/3 的未来，我有下面两点判断：**

* 从标准制定到实践再到协议优化还需要走很长一段路；
* 因为动了底层协议，所以 HTTP/3 的增长会比较缓慢，这和 HTTP/2 有着本质的区别

#### [HTTP/3原理与实践](http://www.alloyteam.com/2020/05/14385/#prettyPhoto)

* 互联网工程任务组正式将基于 QUIC 协议的 HTTP（HTTP over QUIC）重命名为 HTTP/3
*   建立连接

    QUIC 从请求连接到正式接发 HTTP 数据一共花了 1 RTT，这 1 个 RTT 主要是为了获取 Server Config，后面的连接如果客户端缓存了 Server Config，那么就可以直接发送 HTTP 数据，实现 0 RTT 建立连接。
*   连接迁移

    TCP 连接基于四元组（源 IP、源端口、目的 IP、目的端口），切换网络时至少会有一个因素发生变化，导致连接发生变化。当连接发生变化时，如果还使用原来的 TCP 连接，则会导致连接失败，就得等原来的连接超时后重新建立连接，所以我们有时候发现切换到一个新网络时，即使新网络状况良好，但内容还是需要加载很久。如果实现得好，当检测到网络变化时立刻建立新的 TCP 连接，即使这样，建立新的连接还是需要几百毫秒的时间。

    QUIC 的连接不受四元组的影响，当这四个元素发生变化时，原连接依然维持。那这是怎么做到的呢？道理很简单，QUIC 连接不以四元组作为标识，而是使用一个 64 位的随机数，这个随机数被称为 `Connection ID`，即使 IP 或者端口发生变化，只要 Connection ID 没有变化，那么连接依然可以维持。
*   队头阻塞(TCP层面)

    假设一个Connection里有多个Stream，其中一个Stream中Frame3丢失，那么必须等待Frame3重传，虽然别的Stream都已到达，但是不能被处理，那么这时整条连接都被阻塞。

    不仅如此，由于 \*\*HTTP/2 必须使用 HTTPS，\*\*而 HTTPS 使用的 **TLS 协议也存在队头阻塞问题**。TLS 基于 Record 组织数据，将一堆数据放在一起（即一个 Record）加密，加密完后又拆分成多个 TCP 包传输。一般每个 Record 16K，包含 12 个 TCP 包，这样如果 12 个 TCP 包中有任何一个包丢失，那么整个 Record 都无法解密。

    更多解释：[**队头阻塞问题**](https://blog.csdn.net/fishmai/article/details/110487273)

    无论HTTP2(HTTP/2 over TCP)、HTTP1.x协议还是SPDY协议都基于TCP协议，上层可以解决HOL问题，但是传输层的HOL依然存在。说起传输层TCP协议的队头阻塞，这要从TCP的流量控制策略说起了。

    TCP的流量控制策略

    TCP保证了数据的有序和可达性，所以原则上是数据按照序号依次发送和接收，**下一个包的发送需要等到上一个包Ack到达**。这样的话，在相邻两个包的发送间隙存在很长时间的空闲等待，好在TCP采用了**滑动窗口机制**来减少了排队等待时间，双方约定一定大小的窗口，在这个窗口内的包都可以同步发送，接收者收到一个packet时会回复Ack给发送者，发送者收到Ack后移动发送窗口，发送后续数据。

    但是如果某个packet丢失或者其对应的Ack包丢失，同样会出现一方不必要的等待。如下图情况，packet 5的Ack包丢失，导致发送端无法移动发送窗口，但接收者已经在等待后面的包了。必须等到接收者超时重传这个确认包，服务器收到这个ACK包后，发送窗口才会移动，继续后面的发送行为。

    同时，对于已经被接收者接收的多个packets（比如B->A->C请求的三个包），在TCP层面是无法区分这三个包对应于上层的哪三个请求的，因此如果B包出现缺失，会导致后续已经被接收的包无法被应用层读取，请求A、C的包被B包阻塞。
* QUIC 是如何解决队头阻塞问题的呢？主要有两点
  1. QUIC 的传输单元是 Packet，加密单元也是 Packet，整个加密、传输、解密都基于 Packet，这样就能避免 TLS 的队头阻塞问题；
  2. QUIC 基于 UDP，UDP 的数据包在接收端没有处理顺序，即使中间丢失一个包，也不会阻塞整条连接，其他的资源会被正常处理。
*   拥塞控制

    QUIC 重新实现了 TCP 协议的 Cubic 算法进行拥塞控制，并在此基础上做了不少改进。下面介绍一些 QUIC 改进的拥塞控制的特性。

    **热插拔**

    TCP 中如果要修改拥塞控制策略，需要在系统层面进行操作。QUIC 修改拥塞控制策略只需要在应用层操作，并且 QUIC 会根据不同的网络环境、用户来动态选择拥塞控制算法。

    **前向纠错 FEC**

    QUIC 使用前向纠错 (FEC，Forward Error Correction) 技术增加协议的容错性。一段数据被切分为 10 个包后，依次对每个包进行异或运算，运算结果会作为 FEC 包与数据包一起被传输，如果不幸在传输过程中有一个数据包丢失，那么就可以根据剩余 9 个包以及 FEC 包推算出丢失的那个包的数据，这样就大大增加了协议的容错性。

    这是符合现阶段网络技术的一种方案，现阶段带宽已经不是网络传输的瓶颈，往返时间才是，所以新的网络传输协议可以适当增加数据冗余，减少重传操作。

    **单调递增的 Packet Number**

    QUIC使用packet seq number代替TCP的seq number重传时packet seq num会变，比如原来是N，重传的是M+N

    **ACK Delay**

    TCP 计算 RTT 时没有考虑接收方接收到数据到发送确认消息之间的延迟，如下图所示，这段延迟即 ACK Delay。QUIC 考虑了这段延迟，使得 RTT 的计算更加准确。

    **更多的 ACK Block**

    一般来说，接收方收到发送方的消息后都应该发送一个 ACK 回复，表示收到了数据。但每收到一个数据就返回一个 ACK 回复太麻烦，所以一般不会立即回复，而是接收到多个数据后再回复，TCP SACK 最多提供 3 个 ACK block。但有些场景下，比如下载，只需要服务器返回数据就好，但按照 TCP 的设计，每收到 3 个数据包就要 “礼貌性” 地返回一个 ACK。而 QUIC 最多可以捎带 256 个 ACK block。在丢包率比较严重的网络下，更多的 ACK block 可以减少重传量，提升网络效率。

    **流量控制**
*   X5 内核与 STGW

    X5 内核是腾讯开发的适用于安卓系统的浏览器内核，为了解决传统安卓系统浏览器内核适配成本高、不安全、不稳定等问题而开发的统一的浏览器内核。STGW 是 Secure Tencent Gateway 的缩写，意思是腾讯安全云网关。两者早在前两年便支持了 QUIC 协议。

#### 更多文章

[发展史](https://juejin.cn/post/6844903503983280141#heading-5)

[preload prefetch http2 serverpush](https://blog.fundebug.com/2019/04/11/understand-preload-and-prefetch/)

[http代理原理1](https://imququ.com/post/web-proxy.html)

[http代理原理2](../network-note01/\[https:/imququ.com/post/web-proxy.html]\(https:/imququ.com/post/web-proxy-2.html\))
