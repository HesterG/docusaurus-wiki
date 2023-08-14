---
title: Nginx
---

## 负载均衡

[负载均衡视频](https://www.bilibili.com/video/BV1yS4y1N76R?p=26)

负载均衡轮询有个问题：session不能在server间共享

负载均衡解决会话不能共享的一种方式：把session存在redis server上

真正高并发不会用，会使用无状态的方案，下发token (类似[jwt](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html))

## 高可用

[HA介绍视频](https://www.bilibili.com/video/BV1yS4y1N76R?p=36)

[keepalived](https://www.keepalived.org/): vip用于决定哪个node ip在被使用

[keepalived配置](https://www.bilibili.com/video/BV1yS4y1N76R/?p=37)

## HTTPS

### 对称算法

[为什么对称算法不安全](https://www.bilibili.com/video/BV1yS4y1N76R/?p=38)

![](https://raw.githubusercontent.com/HesterG/doc-images/main/Screen%20Shot%202023-08-14%20at%201.14.45%20PM.png)


### 非对称算法

[可以使用非对称算法来解决](https://www.bilibili.com/video/BV1yS4y1N76R/?p=39), 但是[非对称算法也不是最安全的](https://www.bilibili.com/video/BV1yS4y1N76R?p=40)

![](https://raw.githubusercontent.com/HesterG/doc-images/main/Screen%20Shot%202023-08-14%20at%201.14.51%20PM.png)

### CA认证

[CA认证介绍](https://www.bilibili.com/video/BV1yS4y1N76R/?p=40)

![](https://raw.githubusercontent.com/HesterG/doc-images/main/Screen%20Shot%202023-08-14%20at%201.14.57%20PM.png)

### 自签名

[自签名介绍](https://www.bilibili.com/video/BV1yS4y1N76R?p=42)

自签名就是自己签名，不用CA签名，非常非常少用


## 课件

[Nginx 基础使用.pdf](https://github.com/HesterG/docusaurus-wiki/files/12328730/Nginx.pdf)

[Nginx 安装.pdf](https://github.com/HesterG/docusaurus-wiki/files/12328731/Nginx.pdf)

[Nginx课件.pdf](https://github.com/HesterG/docusaurus-wiki/files/12328733/Nginx.pdf)

[Nginx高级.pdf](https://github.com/HesterG/docusaurus-wiki/files/12328734/Nginx.pdf)

[Nginx高级课程扩容与高效.md](https://github.com/HesterG/docusaurus-wiki/files/12328735/Nginx.md)