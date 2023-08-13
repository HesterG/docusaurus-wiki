# 计算机网络笔记(二)

### 网络层

#### 网络层概述

* 主要任务是**实现网络互连**，进而**实现数据在各网络之间的传输**。
* 要**实现网络互连**，需要解决以下主要问题：
  * 网络层向运输层提供怎样的服务（可靠or不可靠传输）
  * 网络层寻址问题
  *   路由选择问题

      路由器收到数据包以后依据什么决定将包转发出去？

      依据路由表，如何得出路由记录？

      人工配置或路由选择协议
*   TCP/IP中将网络层称为网际层（因为网际协议IP是核心），我们将学习TCP/IP协议栈的**网际层**

    <img src="https://user-images.githubusercontent.com/17645053/159415020-57cf5680-eb3c-4d2e-8ba8-1501e9b758eb.png" alt="Screen Shot 2022-03-22 at 1 36 32 PM" data-size="original" />

#### 网络层提供的两种服务

**面向连接的虚电路服务**

* 虚电路表示这是一条逻辑上的连接，分组沿着这条逻辑连接按存储转发方式传送，不是真正建立了物理连接；而采用电路交换的电话通信，则是先建立一条真正的连接。所以**分组交换的虚连接**和**电路交换的连接**只是类似。
*   因特网先驱者并没有采用这种设计思路

    <img src="https://user-images.githubusercontent.com/17645053/159415830-02c4f94e-fcb6-499d-ae04-7af05cfe6147.png" alt="Screen Shot 2022-03-22 at 1 44 19 PM" data-size="original" />

**无连接的数据报服务**

* 将复杂的网络处理功能置于因特网边缘（用户主机和其内部的运输层），将相对简单的尽最大努力的分组交付功能置于因特网的核心。
*   采用这种设计思想，网络造价大大降低；运行方式灵活，可以适应多种应用。

    <img src="https://user-images.githubusercontent.com/17645053/159416214-5a0ad064-4929-4ba3-a52c-d7ff59b2bca7.png" alt="Screen Shot 2022-03-22 at 1 48 02 PM" data-size="original" />
*   总结对比

    <img src="https://user-images.githubusercontent.com/17645053/159416327-893d0a94-1ab4-4738-b6bb-e9df2b33c5be.png" alt="Screen Shot 2022-03-22 at 1 49 05 PM" data-size="original" />

#### IPv4地址及其应用

**概述**

*   **每一台主机（或路由器）的每一个接口**分配一个在全世界范围内是唯一的32比特标识符

    <img src="https://user-images.githubusercontent.com/17645053/159416796-6fe987d6-5ad9-4e65-b2f2-8822b8bf243c.png" alt="Screen Shot 2022-03-22 at 1 52 56 PM" data-size="original" />
*   点分十进制表示方法

    32位分成8位一组，每组十进制，写成点分十进制

**分类编址的IPv4地址**

![Screen Shot 2022-03-22 at 2 00 16 PM](https://user-images.githubusercontent.com/17645053/159417573-bf3dba50-4fb8-4774-94f2-9e38584748f6.png)

*   A类地址

    网络号：

    * 0和127不指派
    * 可指派网络数量：2^(8-1) - 2 = 126

    IP地址数量：

    * 2^24 - 2

    <img src="https://user-images.githubusercontent.com/17645053/159418101-598f6eed-2178-47dc-8e64-ccaf2f885936.png" alt="Screen Shot 2022-03-22 at 2 05 06 PM" data-size="original" />
*   B类地址

    <img src="https://user-images.githubusercontent.com/17645053/159418496-33d15a8e-ae7d-4b8b-92ab-c2548310882f.png" alt="Screen Shot 2022-03-22 at 2 08 28 PM" data-size="original" />
*   C类地址

    <img src="https://user-images.githubusercontent.com/17645053/159418887-5f160bba-d0f7-4f4d-8cc0-69d68115a634.png" alt="Screen Shot 2022-03-22 at 2 11 36 PM" data-size="original" />
*   一般不使用的特殊IP地址

    <img src="https://user-images.githubusercontent.com/17645053/159419354-42f07798-eea8-46ff-928c-42876fafca89.png" alt="Screen Shot 2022-03-22 at 2 15 33 PM" data-size="original" />

#### 划分子网的IPv4地址

*   计算所在子网的网络地址

    <img src="https://user-images.githubusercontent.com/17645053/159420434-db44f75b-86c1-4b94-a051-524a6d0c8bfe.png" alt="Screen Shot 2022-03-22 at 2 23 47 PM" data-size="original" />
*   划分子网的细节

    C类网举例

    <img src="https://user-images.githubusercontent.com/17645053/159421228-c5e50aec-0644-424d-9816-57050c5f1f7b.png" alt="Screen Shot 2022-03-22 at 2 29 46 PM" data-size="original" />
*   默认子网掩码

    <img src="https://user-images.githubusercontent.com/17645053/159422328-8be9927a-9c23-4907-b342-b02cf8bd9126.png" alt="Screen Shot 2022-03-22 at 2 37 21 PM" data-size="original" />

#### 无分类编址的IPv4地址

*   CIDR

    <img src="https://user-images.githubusercontent.com/17645053/159423336-7ca3d989-6f02-4028-a42e-a5cd7821d094.png" alt="Screen Shot 2022-03-22 at 2 45 53 PM" data-size="original" />
*   CIDR例子

    <img src="https://user-images.githubusercontent.com/17645053/159423583-5007d32b-7045-4bcf-9b65-7cd03058498f.png" alt="Screen Shot 2022-03-22 at 2 47 30 PM" data-size="original" />
*   路由聚合

    <img src="https://user-images.githubusercontent.com/17645053/159424500-30df3916-eb86-443d-9738-7754a0eed26e.png" alt="Screen Shot 2022-03-22 at 2 53 57 PM" data-size="original" />

#### IPv4地址的应用规划

*   定长的子网掩码FLSM

    例子

    <img src="https://user-images.githubusercontent.com/17645053/159425759-bdde262f-0f92-408a-9084-bbb459a67dd9.png" alt="Screen Shot 2022-03-22 at 3 02 20 PM" data-size="original" />

    子网掩码确认

    <img src="https://user-images.githubusercontent.com/17645053/159425918-363aae24-6ec6-4113-8551-e3b7eaa72370.png" alt="Screen Shot 2022-03-22 at 3 03 51 PM" data-size="original" />

    划分细节

    <img src="https://user-images.githubusercontent.com/17645053/159426068-1b28f44f-f248-4e25-b697-5ad5a97a6d6f.png" alt="Screen Shot 2022-03-22 at 3 04 44 PM" data-size="original" />

    缺点：N5只需要4个IP地址，但是我们分配了32个，会造成浪费。
*   变长的子网掩码FLSM

    例子

    <img src="https://user-images.githubusercontent.com/17645053/159426691-954181dc-63b7-4bc2-bdaf-e762c0208117.png" alt="Screen Shot 2022-03-22 at 3 08 47 PM" data-size="original" />

    具体分配

    <img src="https://user-images.githubusercontent.com/17645053/159426913-a659bc2a-91c1-40c8-aff4-55193e9b6839.png" alt="Screen Shot 2022-03-22 at 3 10 40 PM" data-size="original" />

#### IP数据包的发送和转发过程

* 分为主机发送IP数据报路由器转发IP数据报
* 如何知道目的主机是否与自己在同一个网络中？

![Screen Shot 2022-03-22 at 3 55 37 PM](https://user-images.githubusercontent.com/17645053/159433703-dadaadbf-fa56-4a4c-9e27-a7b9b57d116c.png)

* 路由器收到IP数据报后如何转发？

![Screen Shot 2022-03-22 at 3 57 59 PM](https://user-images.githubusercontent.com/17645053/159434012-4e64184d-0f33-498e-8685-b2fdf0a818c7.png)

* 利用路由器的路由表得到目的网络地址（地址掩码与目的地址相与，得到目的网络地址）

![Screen Shot 2022-03-22 at 4 00 33 PM](https://user-images.githubusercontent.com/17645053/159434462-a0a5fa74-bee0-45d0-aafe-eda3b3085998.png)

*   一道习题

    <img src="https://user-images.githubusercontent.com/17645053/159435172-f78f8c75-d668-43c4-b5b1-4b061fc71ed3.png" alt="Screen Shot 2022-03-22 at 4 05 00 PM" data-size="original" />

#### 静态路由配置及其可能产生的路由环境问题

*   静态路由

    <img src="https://user-images.githubusercontent.com/17645053/159442322-3bc4958e-73f7-4347-a5ce-06d05e070287.png" alt="Screen Shot 2022-03-22 at 4 48 36 PM" data-size="original" />
*   默认路由

    <img src="https://user-images.githubusercontent.com/17645053/159443131-ba7148ab-1824-4731-ba4d-67156fcbfa71.png" alt="Screen Shot 2022-03-22 at 4 53 09 PM" data-size="original" />
*   静态路由配置错位导致路由环路，所以需要`TTL`

    <img src="https://user-images.githubusercontent.com/17645053/159443367-ec74021b-6bf1-4bbc-aa60-bef8b2a473b9.png" alt="Screen Shot 2022-03-22 at 4 54 33 PM" data-size="original" />
*   聚合了不存在的网络而导致路由环路，所以可以在路由表中添加黑洞路由

    <img src="https://user-images.githubusercontent.com/17645053/159445194-d5efb04e-a296-4a08-bc3b-71914bbd8146.png" alt="Screen Shot 2022-03-22 at 5 05 35 PM" data-size="original" />
* 网络故障导致的路由环路，所以可以在路由表中添加黑洞路由，黑洞路由可能失效
*   总结

    <img src="https://user-images.githubusercontent.com/17645053/159445668-3b3d0979-0d64-4925-998e-022f594d40dc.png" alt="Screen Shot 2022-03-22 at 5 07 44 PM" data-size="original" />

#### 路由选择协议

**概述**

*   分为静态路由选择和动态路由选择

    <img src="https://user-images.githubusercontent.com/17645053/159446545-07d2c3ce-7589-491b-8cf2-796b64d78a35.png" alt="Screen Shot 2022-03-22 at 5 12 42 PM" data-size="original" />
*   因特网所采用的路由选择协议的主要特点

    <img src="https://user-images.githubusercontent.com/17645053/159446784-74467369-cb2a-409f-8f29-d690a05c24eb.png" alt="Screen Shot 2022-03-22 at 5 13 59 PM" data-size="original" />
*   因特网所采用的分层次路由选择协议

    <img src="https://user-images.githubusercontent.com/17645053/159447184-e425075a-940b-455d-b439-4c3e620aaf56.png" alt="Screen Shot 2022-03-22 at 5 16 33 PM" data-size="original" />
*   常见路由选择协议

    <img src="https://user-images.githubusercontent.com/17645053/159447376-4e4edaa1-3672-427f-b4fb-3e6736a4d067.png" alt="Screen Shot 2022-03-22 at 5 17 37 PM" data-size="original" />
*   路由器基本结构

    <img src="https://user-images.githubusercontent.com/17645053/159448323-51d4a5fa-4c2f-4523-9ca2-7ec95cfa69c2.png" alt="Screen Shot 2022-03-22 at 5 21 27 PM" data-size="original" />

**RIP工作原理**

![Screen Shot 2022-03-22 at 5 28 40 PM](https://user-images.githubusercontent.com/17645053/159449350-43ef5392-903e-4d34-b956-185fe87d68d6.png)

*   RIP认为好的路由就是“距离短”的路由，也就是所通过路由数量最少的路由

    <img src="https://user-images.githubusercontent.com/17645053/159449895-6fe7b805-05f7-445d-ac5d-a780b1ff7acf.png" alt="Screen Shot 2022-03-22 at 5 31 34 PM" data-size="original" />
*   RIP的基本工作过程

    <img src="https://user-images.githubusercontent.com/17645053/159450505-79425a34-80ba-4983-8f5f-41299367b8b0.png" alt="Screen Shot 2022-03-22 at 5 34 39 PM" data-size="original" />
*   RIP路由条目的更新规则

    <img src="https://user-images.githubusercontent.com/17645053/159456447-158bb23a-7d7e-4327-8795-ed202396210e.png" alt="Screen Shot 2022-03-22 at 6 05 24 PM" data-size="original" />
*   修改N1距离为16，表示N1不可达，并告诉R2，如果R2的消息先到，就会被谣言误导。

    直到增加到16才都知道N1不可达。

    可以使用触发更新/水平分割，但是也不能彻底避免路由环路

    <img src="https://user-images.githubusercontent.com/17645053/159457463-c4b2f5aa-588f-483b-8e25-f0a9a1c8d1ce.png" alt="Screen Shot 2022-03-22 at 6 11 13 PM" data-size="original" />

**开放最短路径优先OSPF的基本工作原理**

![Screen Shot 2022-03-22 at 6 19 42 PM](https://user-images.githubusercontent.com/17645053/159458971-a13be633-7bad-4946-9148-4ddb9d72ac96.png) ![Screen Shot 2022-03-22 at 6 23 27 PM](https://user-images.githubusercontent.com/17645053/159459610-33f2733c-2f5b-4279-b37e-86601d3e3613.png) ![Screen Shot 2022-03-22 at 6 24 35 PM](https://user-images.githubusercontent.com/17645053/159459924-b6ef897c-86f7-4477-b77a-7cf91ca1304b.png)

*   基本工作过程

    <img src="https://user-images.githubusercontent.com/17645053/159460902-a39f6316-581b-45f6-840d-834ea3299991.png" alt="Screen Shot 2022-03-22 at 6 25 54 PM" data-size="original" />
*   OSPF在多点接入网络中路由器邻居关系的建立

    <img src="https://user-images.githubusercontent.com/17645053/159461726-c0c58bb8-5059-4c2f-95f7-33923e9d130d.png" alt="Screen Shot 2022-03-22 at 6 27 46 PM" data-size="original" />
*   区域

    <img src="https://user-images.githubusercontent.com/17645053/159463087-955aa4a0-bb47-42b5-be3b-fa53e65b7441.png" alt="Screen Shot 2022-03-22 at 6 35 53 PM" data-size="original" />

**边界网关协议BGP的基本工作原理**

![Screen Shot 2022-03-22 at 6 40 57 PM](https://user-images.githubusercontent.com/17645053/159463978-3e1af6f2-1c44-4682-b64e-d3b729b92c41.png) ![Screen Shot 2022-03-22 at 6 42 05 PM](https://user-images.githubusercontent.com/17645053/159464129-dda4d3c2-ef64-4a1a-add7-02f187eae96d.png)

*   BGP发言人

    <img src="https://user-images.githubusercontent.com/17645053/159464327-840abeed-b4f3-44cc-95f2-9046e4100a66.png" alt="Screen Shot 2022-03-22 at 6 43 16 PM" data-size="original" />
*   根据策略找出到达各自自治系统较好的路由

    <img src="https://user-images.githubusercontent.com/17645053/159464610-bfb62674-8081-4648-b83b-9989ad7fd6b3.png" alt="Screen Shot 2022-03-22 at 6 45 06 PM" data-size="original" />
*   BGP适用于多级结构的因特网，例子

    <img src="https://user-images.githubusercontent.com/17645053/159464782-d91dfec3-f0f6-4c3f-97da-cf70eeda4e01.png" alt="Screen Shot 2022-03-22 at 6 46 01 PM" data-size="original" />
*   BGP报文

    <img src="https://user-images.githubusercontent.com/17645053/159465042-282bf89d-efa4-4d5b-9659-21cb9846a816.png" alt="Screen Shot 2022-03-22 at 6 47 26 PM" data-size="original" />
*   封装关系：UDP - RIP， OSFP - IP， BGP - TCP

    <img src="https://user-images.githubusercontent.com/17645053/159465706-7522f969-6cf1-4bda-ad49-bf506fb9f61d.png" alt="Screen Shot 2022-03-22 at 6 49 46 PM" data-size="original" />

#### IPv4数据报的首部格式

*   版本、首部长度、可选字段

    每一行4个Byte，固定部分是必须有的，可变部分是可选的。

    固定部分有五行，所以一共20Byte；可变部分有40Byte

    <img src="https://user-images.githubusercontent.com/17645053/159466706-5aeee743-141e-46eb-9eba-9422a3b4b8a1.png" alt="Screen Shot 2022-03-22 at 6 57 00 PM" data-size="original" />
*   区分服务 - 一般情况不使用

    <img src="https://user-images.githubusercontent.com/17645053/159466931-2a0727d8-d978-4d1c-bebf-805a3e13e9c5.png" alt="Screen Shot 2022-03-22 at 6 58 20 PM" data-size="original" />
*   总长度

    <img src="https://user-images.githubusercontent.com/17645053/159467154-54dcc318-aa71-4401-be8d-24f84b5828b4.png" alt="Screen Shot 2022-03-22 at 6 59 38 PM" data-size="original" />
*   标识、标志、片偏移

    共同用于IP数据报分片，以太网规定帧的MTU是1500Byte，所以如果IPv4数据报>1500Byte,需要进行分片，再将各分片数据报封装成帧

    <img src="https://user-images.githubusercontent.com/17645053/159468158-3527812c-6c59-429f-9e5c-7621f56ec345.png" alt="Screen Shot 2022-03-22 at 7 05 52 PM" data-size="original" /><img src="https://user-images.githubusercontent.com/17645053/159468430-ca20e076-8ea5-4652-93c2-e67ca9c06b21.png" alt="Screen Shot 2022-03-22 at 7 07 49 PM" data-size="original" />

    举例：对IPv4数据报进行分片

    假设以太网帧进行传输

    <img src="https://user-images.githubusercontent.com/17645053/159468932-325ef4e2-51d2-4a91-ae17-b3be55755920.png" alt="Screen Shot 2022-03-22 at 7 10 58 PM" data-size="original" />

    假设首部2还要再分片

    <img src="https://user-images.githubusercontent.com/17645053/159469054-460726d1-bd72-4a26-aaed-76a8161c8068.png" alt="Screen Shot 2022-03-22 at 7 11 48 PM" data-size="original" />
*   生存时间字段

    <img src="https://user-images.githubusercontent.com/17645053/159469384-45b38c8d-c1be-44c4-87d0-d0a57eb66546.png" alt="Screen Shot 2022-03-22 at 7 13 42 PM" data-size="original" />

    举例 - 生存时间TTL字段的作用 - 防止IP数据报在网络中永久兜圈

    <img src="https://user-images.githubusercontent.com/17645053/159469626-8b369e54-084e-4aec-9ea5-d1c52a6b26ec.png" alt="Screen Shot 2022-03-22 at 7 15 10 PM" data-size="original" />
*   协议

    <img src="https://user-images.githubusercontent.com/17645053/159469793-b6f6e4da-965c-4da4-9d68-66f756fe5442.png" alt="Screen Shot 2022-03-22 at 7 15 54 PM" data-size="original" />
*   首部检验和

    IPv6中，不再计算首部校验和，更快转发数据报

    <img src="https://user-images.githubusercontent.com/17645053/159470037-fd85e973-dd0b-498b-9101-7b8ffc8da1d5.png" alt="Screen Shot 2022-03-22 at 7 17 22 PM" data-size="original" />
*   源IP和目的IP

    <img src="https://user-images.githubusercontent.com/17645053/159470147-e9e1e976-5a8f-418a-a626-fd2be68e75c7.png" alt="Screen Shot 2022-03-22 at 7 17 58 PM" data-size="original" />

#### 网际控制报文协议ICMP

![Screen Shot 2022-03-22 at 7 24 21 PM](https://user-images.githubusercontent.com/17645053/159471251-2f875b5a-8f81-4887-abb5-ad1370d23eec.png)

* ICMP**差错报告报文**有以下五种报文
  1.  终点不可达

      <img src="https://user-images.githubusercontent.com/17645053/159471563-84b88ca0-1804-42d0-bbae-a667b3ea3a4d.png" alt="Screen Shot 2022-03-22 at 7 26 12 PM" data-size="original" />
  2.  源点抑制

      <img src="https://user-images.githubusercontent.com/17645053/159471675-1968ae62-3c21-4fc4-bbf5-5e76e4cc4ffc.png" alt="Screen Shot 2022-03-22 at 7 27 02 PM" data-size="original" />
  3.  时间超过

      <img src="https://user-images.githubusercontent.com/17645053/159471785-b88372e4-24c6-4de5-98c5-42d4341ae688.png" alt="Screen Shot 2022-03-22 at 7 27 48 PM" data-size="original" /><img src="https://user-images.githubusercontent.com/17645053/159471841-f7a3d15d-c231-4d52-8c0a-05d8ca517ad8.png" alt="Screen Shot 2022-03-22 at 7 28 07 PM" data-size="original" />
  4.  参数问题

      <img src="https://user-images.githubusercontent.com/17645053/159472109-07430d74-87e5-4b99-bc35-2f3a7e68a5f9.png" alt="Screen Shot 2022-03-22 at 7 29 42 PM" data-size="original" />
  5.  改变路由(重定向)

      <img src="https://user-images.githubusercontent.com/17645053/159472219-d3f3b819-e3de-4156-9608-c33409498cee.png" alt="Screen Shot 2022-03-22 at 7 30 38 PM" data-size="original" />
*   以下情况不应发送ICMP差错报告报文

    <img src="https://user-images.githubusercontent.com/17645053/159472340-8bbcd7b9-15fb-44bc-9732-e073d7131f32.png" alt="Screen Shot 2022-03-22 at 7 31 28 PM" data-size="original" />
*   ICMP**询问报文**

    1. 回送请求和回答
    2. 时间戳请求和回答

    <img src="https://user-images.githubusercontent.com/17645053/159472703-f22245f5-bca3-4e99-bbc5-2a24fec9399f.png" alt="Screen Shot 2022-03-22 at 7 33 16 PM" data-size="original" />
* ICMP应用举例
  1.  PING

      <img src="https://user-images.githubusercontent.com/17645053/159472898-acf70ee6-5cc7-469f-b182-e47a1f6973da.png" alt="Screen Shot 2022-03-22 at 7 34 20 PM" data-size="original" />
  2.  traceroute

      <img src="https://user-images.githubusercontent.com/17645053/159473089-c0e6f583-8dba-43a9-b900-39f084200bf6.png" alt="Screen Shot 2022-03-22 at 7 35 38 PM" data-size="original" />

      实现原理：

      不听发送TTL递增给各结点，让各结点因为TTL=0给H1回送差错报告来知道当中经过了哪些主机/路由器。

      <img src="https://user-images.githubusercontent.com/17645053/159473565-b2e4ab3e-0397-47fd-b098-ccc79c056f93.png" alt="Screen Shot 2022-03-22 at 7 38 54 PM" data-size="original" />

#### 虚拟专用网VPN与网络地址转换NAT

*   举例说明VPN

    在因特网中所有路由器对目的地址是私有地址的IP数据报一律不进行转发

    <img src="https://user-images.githubusercontent.com/17645053/159474813-e224a346-0851-4247-a6ec-e39f4fc71b25.png" alt="Screen Shot 2022-03-22 at 7 46 11 PM" data-size="original" />

    又称为IP隧道技术

    <img src="https://user-images.githubusercontent.com/17645053/159475080-5c40c5d2-a74a-4d4d-998c-a17eb75ec098.png" alt="Screen Shot 2022-03-22 at 7 47 50 PM" data-size="original" />
*   NAT

    <img src="https://user-images.githubusercontent.com/17645053/159475285-8db6d770-c7d5-4e0d-bdaf-1d1b21fbfbd5.png" alt="Screen Shot 2022-03-22 at 7 49 00 PM" data-size="original" />

    举例说明

    <img src="https://user-images.githubusercontent.com/17645053/159476221-4b80b240-2cef-498c-8953-e3c03d2c39be.png" alt="Screen Shot 2022-03-22 at 7 54 07 PM" data-size="original" /><img src="https://user-images.githubusercontent.com/17645053/159476552-912c7a02-bb60-4e32-a25c-48e69e9153ad.png" alt="Screen Shot 2022-03-22 at 7 55 48 PM" data-size="original" /><img src="https://user-images.githubusercontent.com/17645053/159476760-835e7e11-cdd4-4314-a524-92b6a6c31a97.png" alt="Screen Shot 2022-03-22 at 7 57 01 PM" data-size="original" /><img src="https://user-images.githubusercontent.com/17645053/159476853-46694494-d02b-4518-b86b-ee3487e12907.png" alt="Screen Shot 2022-03-22 at 7 57 37 PM" data-size="original" />
* [NAT转换是怎么工作的？ - 网工Fox的回答 - 知乎](https://www.zhihu.com/question/31332694/answer/1917791148)
* [快速理解P2P技术中的NAT穿透原理](https://juejin.cn/entry/6844903497847013383)
* [NAT基本原理及穿透详解(打洞)](https://juejin.cn/post/6844904098572009485)

#### SDN (TODO)

### 运输层

#### 概述

已解决主机到主机通信，运输层要解决\*\*(应用进程到应用进程)端到端\*\*通信

应用进程：AP

![Screen Shot 2022-03-23 at 10 12 28 AM](https://user-images.githubusercontent.com/17645053/159610213-2d39967a-67c2-470e-ab77-454a07a354d0.png) ![Screen Shot 2022-03-23 at 10 15 28 AM](https://user-images.githubusercontent.com/17645053/159610504-6e49a23f-a5a6-4c3e-acd0-cc2e582741b5.png) ![Screen Shot 2022-03-23 at 10 15 19 AM](https://user-images.githubusercontent.com/17645053/159610495-8f6edebc-2e33-493c-93b9-ab9e42f1ecd4.png)

#### 运输层端口号、复用与分用的概念

*   不同操作系统有不同的进程标识符

    端口号可以用来统一对TCP/IP体系的应用进程进行标识

    分为熟知端口、登记端口号、短暂端口号

    <img src="https://user-images.githubusercontent.com/17645053/159611132-29543aa1-222b-4529-aee0-4b7b6a155e31.png" alt="Screen Shot 2022-03-23 at 10 20 48 AM" data-size="original" />
*   发送方的复用和接收方的分用

    <img src="https://user-images.githubusercontent.com/17645053/159611398-4c6c989b-ff9a-45b9-95ae-99c89f4b44f2.png" alt="Screen Shot 2022-03-23 at 10 22 57 AM" data-size="original" />
*   TCP/IP体系的应用层常用协议所使用的运输层熟知端口号

    <img src="https://user-images.githubusercontent.com/17645053/159611543-80b86fa4-6273-4ec6-835b-9461060f1fa3.png" alt="Screen Shot 2022-03-23 at 10 24 22 AM" data-size="original" />
*   举例 - 运输层端口号作用

    DNS响应设置源端口为53，表明这是DNS服务器端进程所发送的UDP用户数据报

    DNS客户端进程使用短暂端口号49152

    <img src="https://user-images.githubusercontent.com/17645053/159612094-96710740-4859-4d1b-bd91-91b522dbd15a.png" alt="Screen Shot 2022-03-23 at 10 29 11 AM" data-size="original" />

​ HTTP - TCP常用端口号是80

​ HTTP客户端也使用短暂端口号49152

![Screen Shot 2022-03-23 at 10 31 33 AM](https://user-images.githubusercontent.com/17645053/159612332-3151bd3c-52c5-42a7-a482-9b5f49ee4625.png)

#### UDP和TCP的对比

![Screen Shot 2022-03-23 at 10 49 09 AM](https://user-images.githubusercontent.com/17645053/159614166-3349fed3-3773-402d-a6b5-7d2c37e1e520.png)

*   UDP支持一对一、一对多、一对全的通信;TCP仅支持单播（一对一）

    <img src="https://user-images.githubusercontent.com/17645053/159614339-344f72bf-0a6d-4015-8d2d-a4bfd945fb12.png" alt="Screen Shot 2022-03-23 at 10 50 48 AM" data-size="original" />
*   UDP面向应用报文，不拆分；TCP面向字节流，并拆分/组装，不保证数据块个数一样，但是字节流保证一样，是实现可靠传输、流量控制以及拥塞控制的基础。TCP两端可以同时发送和接收报文段，也就是全双工通信

    <img src="https://user-images.githubusercontent.com/17645053/159614822-a1f7cd30-6a47-4d00-bd25-cee89bb5c6de.png" alt="Screen Shot 2022-03-23 at 10 54 40 AM" data-size="original" />
*   UDP: 不可靠；TCP：可靠

    UDP发现误码就丢掉；

    <img src="https://user-images.githubusercontent.com/17645053/159615109-d75b1959-3e3b-4ef4-bc70-0f4cd334c230.png" alt="Screen Shot 2022-03-23 at 10 56 37 AM" data-size="original" />
*   数据报首部对比

    <img src="https://user-images.githubusercontent.com/17645053/159615216-5dd1bc94-ae67-4a5f-af47-e0c966e60d93.png" alt="Screen Shot 2022-03-23 at 10 58 10 AM" data-size="original" />

#### TCP流量控制

* 流量控制就是让发送方的发送速率不要太快，让接收方来得及接收
* 利用**滑动窗口**机制来实现
*   举例说明

    <img src="https://user-images.githubusercontent.com/17645053/159616048-bc1c3e3e-a85a-4529-8fe8-e2cebb488b4d.png" alt="Screen Shot 2022-03-23 at 11 06 26 AM" data-size="original" />

    收到0窗口就启动计时器，如果超时，就发送一个零窗口探测报文，仅携带1字节，接收窗口会告知现在的窗口大小，如果还是0就重新计时，如果不是就继续发送报文，打破死锁局面。

    如果0窗口报文段丢了也会超时重传

    <img src="https://user-images.githubusercontent.com/17645053/159616325-402776ca-e69c-4064-add9-07d63d6b0939.png" alt="Screen Shot 2022-03-23 at 11 10 03 AM" data-size="original" />

#### TCP的拥塞控制

![Screen Shot 2022-03-23 at 1 40 18 PM](https://user-images.githubusercontent.com/17645053/159631447-964e518a-2686-429f-b9c2-dc5e5011872f.png)

*   四种拥塞控制算法

    <img src="https://user-images.githubusercontent.com/17645053/159631703-3ee3770a-bbca-41f9-8b3d-1500cdc12409.png" alt="Screen Shot 2022-03-23 at 1 42 27 PM" data-size="original" />

    1.  慢开始

        cwnd < ssthresh时采用,cwnd按指数增长

        <img src="https://user-images.githubusercontent.com/17645053/159632831-0d340da9-df86-41fa-9c77-ebb3d5a01fe7.png" alt="Screen Shot 2022-03-23 at 1 52 07 PM" data-size="original" />
    2.  拥塞避免

        cwnd = ssthresh时，cwnd线性+1

        当重传计时器超时，判断网络可能出现了拥塞，进行以下工作

        1）将ssthresh值更新为发生拥塞时cwnd的一半

        2）将cwnd减少为1，并重新开始慢开始算法

        <img src="https://user-images.githubusercontent.com/17645053/159633550-dfa64c96-4b33-4e98-9c65-2c19ec1be8c8.png" alt="Screen Shot 2022-03-23 at 1 58 38 PM" data-size="original" />
    3.  快重传

        用拥塞避免时，个别报文会在网络中丢失，但实际网络没有发生拥塞，因此需要快重传。

        <img src="https://user-images.githubusercontent.com/17645053/159633968-6e0b4c6d-f2d4-44c5-b1d6-af8774fc5b6b.png" alt="Screen Shot 2022-03-23 at 2 01 29 PM" data-size="original" />

        要求接收方立即发送确认，而不是等自己发送数据时才稍带确认

        如果收到了失序报文段也要立即发出已收到的报文段的**重复确认**（例：没有收到M3就算收到了M4也重复说自己确认收到了M2，即没有收到M3）

        <img src="https://user-images.githubusercontent.com/17645053/159634427-5875fbe8-89c2-407e-bd90-e0c81de3af45.png" alt="Screen Shot 2022-03-23 at 2 05 46 PM" data-size="original" />
    4.  快恢复

        <img src="https://user-images.githubusercontent.com/17645053/159634637-881c308d-8aee-40c2-9f9f-7c905a5add80.png" alt="Screen Shot 2022-03-23 at 2 07 37 PM" data-size="original" />
*   拥塞控制举例

    <img src="https://user-images.githubusercontent.com/17645053/159634915-12a7f36e-f871-429b-a1fe-1ad881feca48.png" alt="Screen Shot 2022-03-23 at 2 10 25 PM" data-size="original" />

#### TCP超时重传时间的选择

![Screen Shot 2022-03-23 at 2 15 21 PM](https://user-images.githubusercontent.com/17645053/159635510-a36befd3-bace-49b4-a92b-f88787991bf2.png)

*   不能直接使用某一个RTT

    <img src="https://user-images.githubusercontent.com/17645053/159637304-b1cba00e-d8d6-4aa4-8a1d-3ca5609eb602.png" alt="Screen Shot 2022-03-23 at 2 30 04 PM" data-size="original" />
*   RFC6298建议下式计算RTO

    <img src="https://user-images.githubusercontent.com/17645053/159637489-ccf6d816-8e0c-479b-9785-fc8b02e612a8.png" alt="Screen Shot 2022-03-23 at 2 31 32 PM" data-size="original" />
*   测量RTT测量比较复杂

    <img src="https://user-images.githubusercontent.com/17645053/159637838-2b63dbfc-7d62-4e2f-978c-82194f5e2536.png" alt="Screen Shot 2022-03-23 at 2 33 53 PM" data-size="original" />
*   Karn算法以及其优化

    <img src="https://user-images.githubusercontent.com/17645053/159638094-f264f816-a0a7-4503-acd8-301f1ebb6151.png" alt="Screen Shot 2022-03-23 at 2 35 59 PM" data-size="original" />

    算法应用

    <img src="https://user-images.githubusercontent.com/17645053/159638427-48b4ac46-9f6d-489b-9a6e-24b10ce25cd3.png" alt="Screen Shot 2022-03-23 at 2 38 10 PM" data-size="original" />

#### TCP可靠传输的实现

*   基于以字节为单位的滑动窗口来实现可靠传输

    确认报文段rwnd=20, ack=31

    <img src="https://user-images.githubusercontent.com/17645053/159639248-0575b94a-b75c-489b-b92f-9ef53aac84bc.png" alt="Screen Shot 2022-03-23 at 2 44 39 PM" data-size="original" /><img src="https://user-images.githubusercontent.com/17645053/159639424-d9c7886f-96d4-4447-bf68-617cb315979b.png" alt="Screen Shot 2022-03-23 at 2 46 02 PM" data-size="original" />
*   一些概念

    <img src="https://user-images.githubusercontent.com/17645053/159642772-722bd259-1056-4c43-8f64-c9f67ab5e1fc.png" alt="Screen Shot 2022-03-23 at 3 06 43 PM" data-size="original" />

#### TCP的运输连接管理——TCP的连接建立

*   TCP是面向连接的协议，它基于运输连接来传送TCP报文段

    <img src="https://user-images.githubusercontent.com/17645053/159644083-969c1dba-a413-4a8b-bbde-9b510898e5c7.png" alt="Screen Shot 2022-03-23 at 3 15 36 PM" data-size="original" />
*   TCP要解决以下三个问题

    <img src="https://user-images.githubusercontent.com/17645053/159644214-6ffde541-1c75-4343-8ded-ef7456b4774b.png" alt="Screen Shot 2022-03-23 at 3 16 21 PM" data-size="original" />
*   TCP三报文握手

    第一次握手：主机A发送位码为SYN=1，随机产生seq number=1234567的数据包到服务器，主机B由SYN=1知道，A要求建立联机；

    第二次握手，主机B收到请求后要确认联机信息，向A发送ack number=(主机A的seq+1)，SYN=1,ACK=1，随机产生seq number=7654321的包；

    第三次握手：主机A收到后检查ack number是否正确，即第一次发送的seq number+1，以及位码ACK是否为1，若正确，主机A会再发送ack number=（主机B的seq+1）,ACK=1，主机B收到后确认seq值与ACK=1则连接建立成功。

    **sequence number**：表示的是我方（发送方）这边，这个packet的数据部分的第一位应该在整个data stream中所在的位置。（注意这里使用的是“应该”。因为对于没有数据的传输，如ACK，虽然它有一个seq，但是这次传输在整个data stream中是不占位置的。所以下一个实际有数据的传输，会依旧从上一次发送ACK的数据包的seq开始

    **acknowledge number**：表示的是期望的对方（接收方）的下一次sequence number是多少。

    <img src="https://user-images.githubusercontent.com/17645053/159644775-6f342268-309f-4def-99ef-8dc2e21a598c.png" alt="Screen Shot 2022-03-23 at 3 19 52 PM" data-size="original" />
*   不能两报文握手

    客户端不会理睬第二次连接确认请求因为是关闭状态，但是服务器会一直等着，因为服务器以为TCP连接已经建立好了。

    <img src="https://user-images.githubusercontent.com/17645053/159645395-c4117bdc-9b86-437b-9e49-a6428018e9dd.png" alt="Screen Shot 2022-03-23 at 3 22 37 PM" data-size="original" />

    **TCP的运输连接管理——TCP的连接释放**

![Screen Shot 2022-03-23 at 4 04 16 PM](https://user-images.githubusercontent.com/17645053/159651853-b0ef5f30-82c1-4024-8e37-bb77668c6e18.png)

*   如果没有2MSL的时间等待状态，　服务器有可能无法进入closed状态

    <img src="https://user-images.githubusercontent.com/17645053/159652420-ed9ca327-d74a-47c6-b561-74bf06760aa4.png" alt="Screen Shot 2022-03-23 at 4 08 07 PM" data-size="original" />
*   使用保活计时器

    <img src="https://user-images.githubusercontent.com/17645053/159652656-9fab37d5-6595-406f-966a-df294ca9c57d.png" alt="Screen Shot 2022-03-23 at 4 09 32 PM" data-size="original" />

#### TCP报文段的首部格式

![Screen Shot 2022-03-23 at 4 15 27 PM](https://user-images.githubusercontent.com/17645053/159653611-c945e2b5-93d0-4c83-9a62-cc65933c5d96.png)

*   源端口、目的端口

    <img src="https://user-images.githubusercontent.com/17645053/159654092-f39a4c65-b4f6-4604-bc34-2db4833769d7.png" alt="Screen Shot 2022-03-23 at 4 16 51 PM" data-size="original" />
*   序号、确认号

    <img src="https://user-images.githubusercontent.com/17645053/159654378-60a44aa1-ec60-4eea-b126-2c9b5050f57b.png" alt="Screen Shot 2022-03-23 at 4 19 45 PM" data-size="original" />

    TCP规定，在连接建立后所有传送的TCP报文都必须把ACK置1

    <img src="https://user-images.githubusercontent.com/17645053/159654929-6c80e961-84cf-426c-a46d-80fedc852aec.png" alt="Screen Shot 2022-03-23 at 4 20 33 PM" data-size="original" />

    举例

    序号201: 该TCP报文段数据载荷的第一个字节的序号为201

    确认号800: TCP客户进程收到了TCP服务器进程发来的序号到799为止的全部数据，现在期望收到序号从800开始的数据

    服务端序号800: 该TCP报文段数据载荷的第一个字节的序号为800，与TCP进程的确认相匹配

    服务确认号301: TCP客户进程收到了TCP服务器进程发来的序号到300为止的全部数据，现在期望收到序号从301开始的数据

    <img src="https://user-images.githubusercontent.com/17645053/159655797-ab8444ef-1a31-46de-9377-75b2822063b2.png" alt="Screen Shot 2022-03-23 at 4 27 51 PM" data-size="original" />
*   数据偏移

    <img src="https://user-images.githubusercontent.com/17645053/159656005-c2cf3ecc-3a0c-4aba-8289-6b76c3e0d07d.png" alt="Screen Shot 2022-03-23 at 4 29 02 PM" data-size="original" />
*   保留

    占6比特，保留为今后使用，目前应置为0
*   窗口

    <img src="https://user-images.githubusercontent.com/17645053/159659011-dc028ca2-b1b8-405b-8ef9-865499ed0408.png" alt="Screen Shot 2022-03-23 at 4 45 31 PM" data-size="original" />
*   校验和

    <img src="https://user-images.githubusercontent.com/17645053/159659250-c9f60281-33b5-4c5b-8f07-37acc56e2e2a.png" alt="Screen Shot 2022-03-23 at 4 46 50 PM" data-size="original" />
*   SYN、FIN、RST、PSH

    <img src="https://user-images.githubusercontent.com/17645053/159659554-0e1e6393-f2b6-4e3f-928a-6015392b1b1a.png" alt="Screen Shot 2022-03-23 at 4 47 36 PM" data-size="original" /><img src="https://user-images.githubusercontent.com/17645053/159659571-4d1c17c4-a7ac-4c20-af57-1a97ce4080df.png" alt="Screen Shot 2022-03-23 at 4 47 46 PM" data-size="original" /><img src="https://user-images.githubusercontent.com/17645053/159659585-693b6f7a-e800-4dad-9b58-1d2008030cb7.png" alt="Screen Shot 2022-03-23 at 4 48 09 PM" data-size="original" /><img src="https://user-images.githubusercontent.com/17645053/159659752-f07136ea-2f00-4438-bd3a-93e0bbf7c1ea.png" alt="Screen Shot 2022-03-23 at 4 49 31 PM" data-size="original" />
*   紧急指针、URG

    <img src="https://user-images.githubusercontent.com/17645053/159659982-4069af82-beea-4ce8-8812-4edcf63cfef1.png" alt="Screen Shot 2022-03-23 at 4 50 41 PM" data-size="original" />
*   选项

    <img src="https://user-images.githubusercontent.com/17645053/159660138-3e6a91df-cb34-46b8-8fac-9e9a27a458e0.png" alt="Screen Shot 2022-03-23 at 4 51 38 PM" data-size="original" />
*   填充

    <img src="https://user-images.githubusercontent.com/17645053/159660298-979baa76-9a80-47cd-ae39-756905714be8.png" alt="Screen Shot 2022-03-23 at 4 51 57 PM" data-size="original" />
