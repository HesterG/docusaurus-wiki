# 计算机网络笔记(一)



### 概述

#### 因特网

![Screen Shot 2022-03-16 at 9 17 21 AM](https://user-images.githubusercontent.com/17645053/158497661-32d57f7d-83cf-4d59-80c1-c04141b68280.png)

*   特点：面向公众

    所有RFC可以下载

    ISOC负责全面管理
* 组成：边缘（主机）、核心（大量网络和路由器，为边缘提供服务的）

#### 交换方式

*   电路交换

    线路传输效率低
* 报文交换
* 分组交换
  * 最重要的分组交换机：路由器，负责将各种网络互联起来，并对接收到的分组进行转换
  * 分组交换机收到一个分组后，先将分组保存下来，按照首部中的目的地址进行查表转发，找到合适的转发接口，通过该接口将分组转发给下一个分组交换机。
  *   发送方：构造分组、发送分组

      路由器：缓存分组、转发分组

      接收方：接收分组、还原报文
  * 构成原始报文的分组，依次在各结点交换机上\*\*存储转发\*\*，各结点交换机在发送分组的同时还\*\*缓存接受的分组\*\*
*   三方对比：

    <img src="https://user-images.githubusercontent.com/17645053/158504631-1d1e8b38-826c-4ef0-b3d8-b2a31fb7ec5d.png" alt="Screen Shot 2022-03-16 at 10 28 48 AM" data-size="original" />

    *   报文：

        缺点1）存储转发会有时延
    *   分组：

        优点4）分组是逐个传输的，所以后一个分组的存储操作和前一个分组的转发操作可以同时进行。

        优点5）因为分组比报文小，所以出错概率减小。即便分组出错，也只需重传出错的部分。

        缺点2）每个地址要加上源地址、目的地址等信息，所以传送的信息量变大了

#### 计算机网络定义和分类

* 计算机网络简单定义：互连的、自治的计算机的集合
  * 互连 计算机间通过有线/无线方式进行数据通信
  * 自治 独立计算机，有自己的硬件和软件，可以单独运行
  * 集合 至少两台计算机
* 计算机网络分类
  * 按交换技术分类：电路/报文/分组交换
  * 按使用者：公用网、专用网
  * 按传输介质：有线网络（双绞线网络、光线网络等）、无线网络
  *   按覆盖范围：广域网wan、城域网man(大约5-50公里)、局域网lan、个域网pan

      [什么是互联网、以太网、广域网、局域网](https://cloud.tencent.com/developer/article/1825132)
  *   按拓扑结构：总线型网络、星型网络、环型网络、网状型网络

      <img src="https://user-images.githubusercontent.com/17645053/158506779-3cc7002e-c74a-41db-886a-3c456341a797.png" alt="Screen Shot 2022-03-16 at 10 48 24 AM" data-size="original" />

#### 计算机网络性能指标

*   速率

    <img src="https://user-images.githubusercontent.com/17645053/158507390-5e962ace-0319-4cbb-a837-3edd66799e48.png" alt="Screen Shot 2022-03-16 at 10 53 42 AM" data-size="original" />
*   带宽

    分为模拟信号(单位Hz)系统和计算机网络(单位b/s)的意义

    <img src="https://user-images.githubusercontent.com/17645053/158507724-7125c2cd-73af-49a9-b4df-bd28602c8d37.png" alt="Screen Shot 2022-03-16 at 10 55 54 AM" data-size="original" />
*   吞吐量

    实际上通过的数据，受带宽和额定速率的限制，下面的例子，带宽1Gb/s，实际吞吐量只有700Mb/s

    <img src="https://user-images.githubusercontent.com/17645053/158507876-71d49285-fdb4-406b-8a96-a352350b58b8.png" alt="Screen Shot 2022-03-16 at 10 58 09 AM" data-size="original" />
*   时延

    发送时延中发送速率取决于max(网卡发送速率、信道带宽、接口速率)

    <img src="https://user-images.githubusercontent.com/17645053/158508477-f602d2b6-de29-44d7-9c7c-5cbc9eed4259.png" alt="Screen Shot 2022-03-16 at 11 03 50 AM" data-size="original" />

    发送时延和传播时延哪个占主导是不一定的

    <img src="https://user-images.githubusercontent.com/17645053/158508665-b85bb6be-cd5b-484a-a1ba-b1348e8048eb.png" alt="Screen Shot 2022-03-16 at 11 05 12 AM" data-size="original" />

    n个分组、m段链路的时延

    <img src="https://user-images.githubusercontent.com/17645053/158535556-70474067-2b0e-4d32-8794-f86389294caa.png" alt="Screen Shot 2022-03-16 at 3 09 25 PM" data-size="original" />
*   时延带宽积

    <img src="https://user-images.githubusercontent.com/17645053/158509742-5b067fad-e446-462b-b15d-223d8bcf6905.png" alt="Screen Shot 2022-03-16 at 11 16 14 AM" data-size="original" />
*   往返时间RTT

    <img src="https://user-images.githubusercontent.com/17645053/158509899-d73520bc-0997-41ed-ae85-bb202fccbccd.png" alt="Screen Shot 2022-03-16 at 11 17 52 AM" data-size="original" />
*   利用率

    利用率越高，引起时延也越高

    <img src="https://user-images.githubusercontent.com/17645053/158510127-360f2786-0763-4f43-b05a-acee8fbb2907.png" alt="Screen Shot 2022-03-16 at 11 19 44 AM" data-size="original" />
*   丢包率

    <img src="https://user-images.githubusercontent.com/17645053/158510287-e947a670-bd4a-42cb-8e45-a33395df426d.png" alt="Screen Shot 2022-03-16 at 11 21 27 AM" data-size="original" />

#### 计算机网络体系结构

**常见网络体系结构**

*   OSI体系结构 - 法律国际标准

    OSI失败原因：

    1. 专家缺乏实际经验，没有商业驱动力
    2. 协议实现起来过分复杂，且效率低
    3. OSI标准制定周期太长，设备无法及时进入市场
    4. 层次划分不合理：有些功能重复出现
*   TCP/IP体系结构 - 事实国际标准，有市场背景，不代表技术水平最先进

    IP协议可以将不同网络接口互联，并向TCP、UDP提供网络互联服务

    TCP在享受IP协议提供的服务的基础上，向应用层相应协议提供可靠传输服务

    UDP在享受IP协议提供的服务的基础上，向应用层相应协议提供不可靠传输服务

    <img src="https://user-images.githubusercontent.com/17645053/158511632-36c1b13a-376e-4c8b-8326-78c298965291.png" alt="Screen Shot 2022-03-16 at 11 33 07 AM" data-size="original" />
*   原理体系结构

    <img src="https://user-images.githubusercontent.com/17645053/158511642-2ae93902-d9a7-43bb-a934-c65656fe6c30.png" alt="Screen Shot 2022-03-16 at 11 33 36 AM" data-size="original" />

**分层必要性**

分层解决不同的问题

![Screen Shot 2022-03-16 at 11 46 43 AM](https://user-images.githubusercontent.com/17645053/158512876-b3fe8aa2-63ee-4891-8a84-2feb488feb76.png)

**专用术语**

*   实体、对等实体

    <img src="https://user-images.githubusercontent.com/17645053/158528779-38e6e71f-3ad0-4ba4-b115-02760d95946c.png" alt="Screen Shot 2022-03-16 at 2 20 34 PM" data-size="original" />
*   协议

    <img src="https://user-images.githubusercontent.com/17645053/158528889-aaa28591-0091-49d3-9c98-a606532d74c5.png" alt="Screen Shot 2022-03-16 at 2 21 26 PM" data-size="original" />

    * 协议三要素：
      1. 语法 - 定义交换信息的格式
      2. 语义 - 定义双方要完成的操作
      3. 同步 - 定义双方的时序关系, 例：TCP三次握手
*   服务

    本层使用下一层提供的服务，并给上一层提供服务

    **Note：** 协议是水平的，服务是垂直的；实体看得见下层提供的服务，但是不知道实现该服务的具体协议。
*   服务访问点

    上下都通过这个访问点来进行访问，所以就中间三层有

    <img src="https://user-images.githubusercontent.com/17645053/158530201-30aba5e0-9f0b-449b-85f7-88804ca6b628.png" alt="Screen Shot 2022-03-16 at 2 31 02 PM" data-size="original" />
*   协议数据单元PDU， 服务数据单元SDU

    物理PDU：比特流 bit stream

    链路PDU：帧 frame

    网络PDU: IP数据报或分组 packet

    运输PDU: TCP报文 segment 或UDP用户数据 datagram

    应用PDU: 报文 message

    <img src="https://user-images.githubusercontent.com/17645053/158530500-7babae94-4a25-4965-9f84-57d9f6e24780.png" alt="Screen Shot 2022-03-16 at 2 32 52 PM" data-size="original" />

### 物理层

#### 基本概念

*   传输媒体

    *   导引型传输媒体

        双绞线、同轴电缆、光纤
    *   非导引型传输媒体

        微波通信

    为链路层提供透明传输比特流的服务（链路层不知道物理层用什么方式传播比特流）
* 物理层协议主要任务
  * 机械特性、电气特性、功能特性、过程特性

![Screen Shot 2022-03-17 at 10 01 36 AM](https://user-images.githubusercontent.com/17645053/158721701-46952d62-0187-4b41-98a8-a3f1d59015c4.png)

#### 物理层下面的传输媒体

传输媒体不属于计算机网络任何一层。

**导引型传输媒体**

*   同轴电缆

    <img src="https://user-images.githubusercontent.com/17645053/158722173-e3b12c7b-82be-4bab-b7bd-13e1318b4cc2.png" alt="Screen Shot 2022-03-17 at 10 06 44 AM" data-size="original" />
*   双绞线

    <img src="https://user-images.githubusercontent.com/17645053/158722486-16050f59-4922-424a-b02d-2eb7beb82a9c.png" alt="Screen Shot 2022-03-17 at 10 10 14 AM" data-size="original" />
*   光纤

    多模光纤：许多条不同角度入射的光线在一条光纤中传输

    单模光纤：光在纤芯中一直向前传播而不发生全反射

    <img src="https://user-images.githubusercontent.com/17645053/158722691-7e8cf7b9-0f74-4b38-8664-cf17d4996d3d.png" alt="Screen Shot 2022-03-17 at 10 12 17 AM" data-size="original" /><img src="https://user-images.githubusercontent.com/17645053/158723169-26d81667-e334-4314-a42c-9ebd16444840.png" alt="Screen Shot 2022-03-17 at 10 16 29 AM" data-size="original" />
*   电力线

    <img src="https://user-images.githubusercontent.com/17645053/158723309-660bf6f9-c5ea-46fb-9153-73accbccd3d5.png" alt="Screen Shot 2022-03-17 at 10 17 34 AM" data-size="original" />

**非导引型传输媒体**

*   微波通信

    微波：主要是直线传播，主要的两种方式：地面微波接力通信、卫星通信
*   无线电波

    中低频：地面波，靠近地球表面；（甚）高频：利用电离层反射
*   红外线

    点对点；直线传播；传输速率低
*   可见光

    LiWi，实验阶段

![Screen Shot 2022-03-17 at 10 23 15 AM](https://user-images.githubusercontent.com/17645053/158723866-f376d0d0-3fd7-4460-9ae1-081827668bb1.png)

#### 传输方式

![Screen Shot 2022-03-17 at 10 33 00 AM](https://user-images.githubusercontent.com/17645053/158724892-3e1dc35e-32da-4a90-b395-4a2eb05a9d6a.png)

#### 编码与调制

![Screen Shot 2022-03-17 at 10 39 12 AM](https://user-images.githubusercontent.com/17645053/158725544-d1cf1460-d8a1-48b1-8bda-2097a229d3be.png)

**常用编码**

* 不归零编码：不归零，要么正要么负，存在同步问题
* 归零编码: 将时钟用归零方式作为时钟信号，称为“自同步”，但是大部分带宽浪费在了“归零”
* 曼彻斯特编码：传统以太网
*   差分曼彻斯特编码

    <img src="https://user-images.githubusercontent.com/17645053/158726072-f4cdfb01-0f7c-4696-85e2-64e03c550e8a.png" alt="Screen Shot 2022-03-17 at 10 43 44 AM" data-size="original" />

**基本调制方法**

![Screen Shot 2022-03-17 at 10 48 55 AM](https://user-images.githubusercontent.com/17645053/158726603-c3de55f7-4885-4b62-885e-b9c1f6c3ecf6.png) ![Screen Shot 2022-03-17 at 10 52 09 AM](https://user-images.githubusercontent.com/17645053/158726942-a5bec13a-dd77-4d05-a6fc-aaa8af3bd7ec.png)

#### 信道极限容量

![Screen Shot 2022-03-17 at 11 08 23 AM](https://user-images.githubusercontent.com/17645053/158728791-9cf94547-1955-4b95-b665-11428304bcb4.png) ![Screen Shot 2022-03-17 at 11 15 08 AM](https://user-images.githubusercontent.com/17645053/158729548-b647e1c7-33d7-4efe-9df9-7d001b65e99b.png) ![Screen Shot 2022-03-17 at 11 09 28 AM](https://user-images.githubusercontent.com/17645053/158728909-122cd348-07ac-4d52-b624-6a7d4458ad99.png)

### 数据链路层

#### 概述

![Screen Shot 2022-03-18 at 10 06 58 AM](https://user-images.githubusercontent.com/17645053/158923789-f069dc58-eca9-4bc8-8746-35d9401c189c.png)

* 三个重要问题（点对点）
  1.  封装成帧

      <img src="https://user-images.githubusercontent.com/17645053/158923911-fbdd339a-1fa5-45e7-9db9-6f910d246bfc.png" alt="Screen Shot 2022-03-18 at 10 08 25 AM" data-size="original" />
  2.  差错检测

      通过帧尾检错码检测是否有错
  3. 可靠传输

![Screen Shot 2022-03-18 at 10 13 29 AM](https://user-images.githubusercontent.com/17645053/158924421-934e9a33-42c3-4d87-9d99-d3f1b9eb0deb.png)

#### 封装成帧

*   帧定界

    `PPP`一头一尾有帧定界标志

    以太网V2的MAC帧有前导码
* 透明传输
  1.  面向字节的物理链路：字节填充

      如果上层数据中出现与帧定界符一样的数据，在它前方插入一个转义字符，这样就知道这个字节是数据而不是帧定界符。

      如果上层数据中出现与帧定界符、转译字符一样的数据，出现帧定界符、转译字符，则插入转义字符。
  2.  面向比特的物理链路：比特填充

      0比特填充法: 如果上层数据中出现与帧定界符一样的数据，每5个1插入一个0

![Screen Shot 2022-03-18 at 10 26 08 AM](https://user-images.githubusercontent.com/17645053/158925695-bf138435-08a5-4d65-8e36-3907e421ff09.png)

#### 差错检测

*   奇偶校验

    使1的个数为奇数或偶数，一般不采用
*   CRC(Cyclic Redundancy Check)

    <img src="https://user-images.githubusercontent.com/17645053/158926740-af09fac1-e737-4c0e-a08f-7c27b7fcd7ce.png" alt="Screen Shot 2022-03-18 at 10 35 19 AM" data-size="original" />

可靠：重传；不可靠：丢弃

![Screen Shot 2022-03-18 at 10 37 05 AM](https://user-images.githubusercontent.com/17645053/158926962-4891c7d5-69a1-4371-b5e0-5bc9d2506f82.png)

#### 可靠传输

![Screen Shot 2022-03-21 at 10 08 50 AM](https://user-images.githubusercontent.com/17645053/159197007-1df77990-45aa-42bc-9428-902ef6a62a77.png)

**可靠传输的实现 - 停止-等待协议SW(Stop-andWait)**

![Screen Shot 2022-03-21 at 2 11 16 PM](https://user-images.githubusercontent.com/17645053/159212674-7d0c7076-335a-4d02-9537-48c4a8fead79.png) ![Screen Shot 2022-03-21 at 2 13 10 PM](https://user-images.githubusercontent.com/17645053/159212799-f52c4615-8153-4bf3-9a84-5910470696e3.png)

**可靠传输的实现 - Go-Back-N**

![Screen Shot 2022-03-21 at 3 25 19 PM](https://user-images.githubusercontent.com/17645053/159219313-9ef3d949-654a-4a9d-8ee8-aa3ce9d556a1.png)

**可靠传输的实现 - 选择重传协议SR**

![Screen Shot 2022-03-21 at 3 56 07 PM](https://user-images.githubusercontent.com/17645053/159222451-1791f77b-4a62-4cba-9037-915653820d60.png)

#### 点对点协议PPP

*   应用：通常，通过连接到某个ISP，ISP已经从因特网管理机构，申请到了一批IP地址，用户只有获取到ISP分配的合法IP地址后，才能成为因特网上的主机。

    **用户主机与ISP通信**时，所使用的数据链路层协议，通常就是**PPP**协议（1999年在以太网上公布的PPP协议就是PPPoE协议，它使得ISP可以通过DSL、以太网等宽带接入技术，以以太网接口的形式，为用户提供接入服务）。另外，PPP协议，也应用于广域网路由器之间的专用线路。

![Screen Shot 2022-03-21 at 6 44 27 PM](https://user-images.githubusercontent.com/17645053/159245919-56a4c360-1ca0-4adf-8750-648257b853e2.png) ![Screen Shot 2022-03-21 at 4 13 59 PM](https://user-images.githubusercontent.com/17645053/159224537-f5586bd7-fd84-4e9e-8d0e-a291e066f4c7.png)

*   帧格式

    <img src="https://user-images.githubusercontent.com/17645053/159225042-8166b950-a8c7-4ab7-9e16-16c8a4d73d92.png" alt="Screen Shot 2022-03-21 at 4 17 49 PM" data-size="original" />
*   点对点协议PPP透明传输

    面向**字节**的**异步**链路 字节填充法

    <img src="https://user-images.githubusercontent.com/17645053/159225344-5e7e656a-17f3-4a82-aeb8-5c12f6b29ee7.png" alt="Screen Shot 2022-03-21 at 4 19 55 PM" data-size="original" />

    面向**比特**的**同步**链路 比特填充法

    <img src="https://user-images.githubusercontent.com/17645053/159226636-47faaff9-cfd3-485f-b9dc-a7b4962a81c2.png" alt="Screen Shot 2022-03-21 at 4 29 15 PM" data-size="original" />
* 差错检测

![Screen Shot 2022-03-21 at 4 30 12 PM](https://user-images.githubusercontent.com/17645053/159226790-45f5f814-5725-402f-9501-13f4b0084a29.png)

**Note** 使用**PPP**的数据链路层向上**不提供可靠**的传输服务

*   PPP工作状态

    <img src="https://user-images.githubusercontent.com/17645053/159227112-ecd642cc-6232-410f-9f18-6bfb040f11c5.png" alt="Screen Shot 2022-03-21 at 4 33 04 PM" data-size="original" />

#### 媒体接入控制MAC

**基本概念**

![Screen Shot 2022-03-21 at 4 51 23 PM](https://user-images.githubusercontent.com/17645053/159229479-a654f2fe-aec5-4d6a-8430-8ce5c016e701.png)

**静态划分信道（通常应用于物理层）**

![Screen Shot 2022-03-21 at 4 53 33 PM](https://user-images.githubusercontent.com/17645053/159229758-df8a069f-9fd8-4da5-b5cf-6fb7178e3da8.png)

*   频分复用FDM

    <img src="https://user-images.githubusercontent.com/17645053/159230021-eee00c60-e8d0-47f5-94a9-9303d9245b11.png" alt="Screen Shot 2022-03-21 at 4 55 35 PM" data-size="original" />
*   时分复用TDM

    <img src="https://user-images.githubusercontent.com/17645053/159230092-9fae76d8-f0ba-4511-b5f8-76f45ec0cd32.png" alt="Screen Shot 2022-03-21 at 4 56 19 PM" data-size="original" />
*   波分复用WDM

    光信号传输一段距离会衰减，所以需要进行放大才能继续传输

    <img src="https://user-images.githubusercontent.com/17645053/159230329-be1b85ac-3e21-4560-b919-4b6259d454cb.png" alt="Screen Shot 2022-03-21 at 4 57 58 PM" data-size="original" />
* 码分复用CDM

**动态划分信道 - 随机接入 - CSMA/CD协议**

* 用于**总线型局域网以太网**
* 概念

![Screen Shot 2022-03-21 at 5 22 01 PM](https://user-images.githubusercontent.com/17645053/159233642-2780ac5a-ce9f-4c2d-9096-92e1b01c4e09.png)

*   例子

    <img src="https://user-images.githubusercontent.com/17645053/159234762-5507e126-4b64-4038-9028-ad2d129b89b9.png" alt="Screen Shot 2022-03-21 at 5 29 51 PM" data-size="original" />
*   争用期

    <img src="https://user-images.githubusercontent.com/17645053/159235053-c51802bb-6cd4-4658-83a5-6b612175f589.png" alt="Screen Shot 2022-03-21 at 5 31 47 PM" data-size="original" />
*   最小帧长

    <img src="https://user-images.githubusercontent.com/17645053/159235307-9fd82267-1352-4555-a290-5e34182343b5.png" alt="Screen Shot 2022-03-21 at 5 33 18 PM" data-size="original" />
*   最大帧长

    <img src="https://user-images.githubusercontent.com/17645053/159235514-f30f6dd3-412a-4e52-9aeb-73fe080cce05.png" alt="Screen Shot 2022-03-21 at 5 34 33 PM" data-size="original" />
*   截断二进制指数退避算法

    <img src="https://user-images.githubusercontent.com/17645053/159235766-e4faaf0a-c778-4fdf-bf5e-5e7d271a19c8.png" alt="Screen Shot 2022-03-21 at 5 35 38 PM" data-size="original" />
*   信道利用率

    <img src="https://user-images.githubusercontent.com/17645053/159236007-f4d51917-df50-48b7-a157-7ac5be655232.png" alt="Screen Shot 2022-03-21 at 5 37 14 PM" data-size="original" />
*   帧发送流程

    <img src="https://user-images.githubusercontent.com/17645053/159236019-de9d7630-8608-4137-ab22-9ca0d3cef012.png" alt="Screen Shot 2022-03-21 at 5 37 34 PM" data-size="original" />
*   帧接受流程

    <img src="https://user-images.githubusercontent.com/17645053/159236203-1468bd13-e50b-424e-b26d-e45e89b28e94.png" alt="Screen Shot 2022-03-21 at 5 38 54 PM" data-size="original" />

**动态划分信道 - 随机接入 - CSMA/CA**

*   用于无线局域网

    因为存在**隐蔽站问题**，进行碰撞检测的意义也不大

    <img src="https://user-images.githubusercontent.com/17645053/159237494-ac8620c7-1132-4c25-ba83-325e5fce9a05.png" alt="Screen Shot 2022-03-21 at 5 47 18 PM" data-size="original" />
*   802.11无线局域网使用CSMA/CA协议

    <img src="https://user-images.githubusercontent.com/17645053/159238669-562b10b3-607b-4182-a400-b2600ec852d6.png" alt="Screen Shot 2022-03-21 at 5 55 31 PM" data-size="original" />
*   帧间间隔

    <img src="https://user-images.githubusercontent.com/17645053/159239067-5954316d-e4b2-4747-bb27-7a7608f14687.png" alt="Screen Shot 2022-03-21 at 5 58 07 PM" data-size="original" />
*   CSMA/CA协议的工作原理

    <img src="https://user-images.githubusercontent.com/17645053/159240391-9529c8f2-eef3-42fd-967d-0194146cf63c.png" alt="Screen Shot 2022-03-21 at 6 06 54 PM" data-size="original" />
*   CSMA/CA退避算法

    <img src="https://user-images.githubusercontent.com/17645053/159241506-78f3867b-ef4d-475a-b749-9db369723d60.png" alt="Screen Shot 2022-03-21 at 6 14 28 PM" data-size="original" /><img src="https://user-images.githubusercontent.com/17645053/159241918-75146df9-5928-47f2-b812-f6308778214b.png" alt="Screen Shot 2022-03-21 at 6 17 05 PM" data-size="original" />
*   CSMA/CA协议的信道预约和虚拟载波监听

    <img src="https://user-images.githubusercontent.com/17645053/159242378-4e410205-43f9-4e0f-a8b3-f5b8dfaec3a7.png" alt="Screen Shot 2022-03-21 at 6 20 04 PM" data-size="original" />

    用小代价进行信道预约时值得的

    <img src="https://user-images.githubusercontent.com/17645053/159242588-cc98e8ec-150c-46cb-9ea1-28c4ac8e4763.png" alt="Screen Shot 2022-03-21 at 6 21 41 PM" data-size="original" /><img src="https://user-images.githubusercontent.com/17645053/159242999-b33145ce-0b2c-4818-ba22-581edc6684b8.png" alt="Screen Shot 2022-03-21 at 6 24 35 PM" data-size="original" />

#### MAC地址、IP地址、ARP协议

![Screen Shot 2022-03-21 at 6 49 11 PM](https://user-images.githubusercontent.com/17645053/159246585-f712cb1b-7b6f-4ebc-8353-88647df8ca1a.png)

**MAC地址**

![Screen Shot 2022-03-21 at 6 50 06 PM](https://user-images.githubusercontent.com/17645053/159246711-8f12b619-3afa-43de-ac7d-100528d5f4ae.png) ![Screen Shot 2022-03-21 at 6 51 11 PM](https://user-images.githubusercontent.com/17645053/159246836-fec024bb-3112-499c-94c3-07219546f01d.png) ![Screen Shot 2022-03-21 at 6 53 30 PM](https://user-images.githubusercontent.com/17645053/159247132-f20a2adc-a6e5-450d-be35-aae23f26fbf7.png)

* 可以通过MAC地址查找厂商

![Screen Shot 2022-03-21 at 6 57 54 PM](https://user-images.githubusercontent.com/17645053/159247762-53dbae59-c683-4c79-b9c5-0cd5ea50cb8c.png) ![Screen Shot 2022-03-21 at 6 58 46 PM](https://user-images.githubusercontent.com/17645053/159247897-7fd18f17-6e4b-44bb-8f29-e39678be577f.png)

**IP地址**

![Screen Shot 2022-03-21 at 7 24 00 PM](https://user-images.githubusercontent.com/17645053/159251464-59eeb0cb-6146-4894-9c76-3338db1c6bcd.png) ![Screen Shot 2022-03-21 at 7 27 10 PM](https://user-images.githubusercontent.com/17645053/159251908-92f662ff-96ae-453c-928d-bac28af8a4f0.png) ![Screen Shot 2022-03-21 at 7 28 57 PM](https://user-images.githubusercontent.com/17645053/159252356-d2c9f148-7f57-4cd6-9957-e160f6516608.png)

**ARP协议**

*   地址解析协议: 把IP地址转成MAC地址

    主机有ARP高速缓存，先查缓存

    <img src="https://user-images.githubusercontent.com/17645053/159253023-5f5e0cac-9c77-4b90-90d8-2e97b4525777.png" alt="Screen Shot 2022-03-21 at 7 32 42 PM" data-size="original" />

    如果没有，就发送ARP请求广播帧，将请求报文发到总线上的各主机

    <img src="https://user-images.githubusercontent.com/17645053/159253287-dc64cb92-a224-43ee-8e70-862f802bc179.png" alt="Screen Shot 2022-03-21 at 7 34 54 PM" data-size="original" />

    C主机会将B的IP地址与MAC地址记录到自己的ARP缓存

    然后给B发送ARP响应（单播帧），以告知自己的MAC地址

    响应报文内容：

    <img src="https://user-images.githubusercontent.com/17645053/159253693-612552af-b613-4822-ae4e-0b647deeb9fc.png" alt="Screen Shot 2022-03-21 at 7 37 24 PM" data-size="original" />
* ARP缓存 动态 vs 静态

![Screen Shot 2022-03-21 at 7 38 51 PM](https://user-images.githubusercontent.com/17645053/159253895-1b0de4cc-1bbe-4912-806c-f8246ed38e3a.png)

* ARP协议只能在一段链路或一个网络上使用

![Screen Shot 2022-03-21 at 7 39 42 PM](https://user-images.githubusercontent.com/17645053/159253995-2da41c25-8e20-4ae0-9e02-0b4834e6a855.png)

* 除ARP请求和相应以外，ARP还有其他类型的报文（例如用于检查IP地址冲突的“无故ARP、免费ARP”）。
* ARP没有安全验证机制，存在ARP欺骗（攻击）问题。

#### 集线器与交换机的区别

![Screen Shot 2022-03-21 at 7 50 52 PM](https://user-images.githubusercontent.com/17645053/159255575-6896cb84-9dcd-428b-a740-21ce87f53fe9.png)

* 集线器![Screen Shot 2022-03-21 at 7 58 23 PM](https://user-images.githubusercontent.com/17645053/159256654-14e82bde-ac02-4b60-bc60-669b0c8d7719.png)
*   交换机

    <img src="https://user-images.githubusercontent.com/17645053/159257273-9393277c-edf0-4650-a686-93e6e4a49f90.png" alt="Screen Shot 2022-03-21 at 8 02 48 PM" data-size="original" />
*   对比集线器和交换机

    交换机不使用`CSMA/CD`;集线器使用`CSMA/CD`

    如果用集线器将原本的广播域相连会扩大广播域并扩大碰撞域；如果用交换机将原本的广播域相连会扩大广播域并不会扩大碰撞域(隔离碰撞域名)

    集线器工作在物理层；交换机工作在数据链路层

    [集线器vs交换机](https://cn.fs.com/hk/blog/23715.html) 交换机优势明显
*   总结

    <img src="https://user-images.githubusercontent.com/17645053/159258200-1d3b55d4-ba04-47d0-9c83-18e740dc6083.png" alt="Screen Shot 2022-03-21 at 8 09 18 PM" data-size="original" />

#### 以太网交换机自学习和转发帧的流程

![Screen Shot 2022-03-21 at 8 30 50 PM](https://user-images.githubusercontent.com/17645053/159261451-248c6733-af9e-402d-8453-e9c6cc3ca28b.png)

#### 以太网交换机的生成树协议STP

![Screen Shot 2022-03-21 at 8 44 00 PM](https://user-images.githubusercontent.com/17645053/159263354-6e3c3cd6-c29a-47ba-9f38-e80a8eb36aa8.png) ![Screen Shot 2022-03-21 at 8 45 55 PM](https://user-images.githubusercontent.com/17645053/159263668-330487e0-abbb-4790-bd40-bff38cbd701b.png)

#### 虚拟局域网VLAN

**虚拟局域网VLAN概述**

![Screen Shot 2022-03-21 at 8 49 23 PM](https://user-images.githubusercontent.com/17645053/159264135-b5865d18-3bb1-4a19-b228-1840db1ea361.png)

*   网络中频繁出现广播协议

    <img src="https://user-images.githubusercontent.com/17645053/159264396-f740ac6c-91d4-41d4-a5a7-b73a28a0e83f.png" alt="Screen Shot 2022-03-21 at 8 50 51 PM" data-size="original" />
* **分割广播域**
  * 使用路由器可以隔离广播域，但路由器成本较高。
  * 虚拟局域网VLAN技术应运而生。
*   VLAN

    <img src="https://user-images.githubusercontent.com/17645053/159265597-f6e53172-cb0d-4af9-beaf-bd1a56b4f945.png" alt="Screen Shot 2022-03-21 at 8 57 49 PM" data-size="original" />

**虚拟局域网VLAN的实现机制**

*   IEEE802.1Q帧

    <img src="https://user-images.githubusercontent.com/17645053/159266194-af96d36e-19a8-453d-bfad-80831878bd15.png" alt="Screen Shot 2022-03-21 at 9 01 33 PM" data-size="original" />
*   交换机的端口类型

    <img src="https://user-images.githubusercontent.com/17645053/159266402-d8d4f480-3a34-4096-a765-65d9ed73d2f1.png" alt="Screen Shot 2022-03-21 at 9 02 45 PM" data-size="original" />

    *   Access端口

        <img src="https://user-images.githubusercontent.com/17645053/159266961-4b9f18be-231e-4c95-bcc1-ac1126f4caa1.png" alt="Screen Shot 2022-03-21 at 9 06 16 PM" data-size="original" />
    *   Trunk端口

        在由多个交换机互联而成的交换式以太网中划分VLAN时，连接主机的交换机端口应设置为Access类型，交换机之间互连的端口应设置为Trunk类型。

        <img src="https://user-images.githubusercontent.com/17645053/159267396-4d5de394-f7ea-4171-a903-614cacfd3be8.png" alt="Screen Shot 2022-03-21 at 9 08 51 PM" data-size="original" /><img src="https://user-images.githubusercontent.com/17645053/159267693-882aaa27-ca3e-4a55-8c8f-b7d349a94e3f.png" alt="Screen Shot 2022-03-21 at 9 10 35 PM" data-size="original" />
    *   Hybrid端口

        绝大部分跟Trunk相同，不同在于需要看VID是否在端口的“去标签”列表中

        主机A给C发送实例：

        <img src="https://user-images.githubusercontent.com/17645053/159268607-1193c3f6-cb97-4f2f-be1b-a696b82bcefd.png" alt="Screen Shot 2022-03-21 at 9 15 08 PM" data-size="original" />

        主机A给B发送实例：

        B可以识别普通帧，无法识别802.1Q帧，只能丢弃该帧。

        <img src="https://user-images.githubusercontent.com/17645053/159268887-8ce51dfb-f85f-4eba-b0af-ef31fb0b0213.png" alt="Screen Shot 2022-03-21 at 9 17 26 PM" data-size="original" />
