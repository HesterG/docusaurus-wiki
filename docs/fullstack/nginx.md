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

![WechatIMG2047](https://github-production-user-asset-6210df.s3.amazonaws.com/17645053/260286110-26f00ed6-b38b-4cff-8a4d-42decf97e6be.jpeg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20230813%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20230813T065922Z&X-Amz-Expires=300&X-Amz-Signature=c976fa137127ae7d64dda3dd8cc691606e75750ef9e475bbfaf73c28a7e48987&X-Amz-SignedHeaders=host&actor_id=17645053&key_id=0&repo_id=677180325)

## 课件

[Nginx 基础使用.pdf](https://github.com/HesterG/docusaurus-wiki/files/12328730/Nginx.pdf)

[Nginx 安装.pdf](https://github.com/HesterG/docusaurus-wiki/files/12328731/Nginx.pdf)

[Nginx课件.pdf](https://github.com/HesterG/docusaurus-wiki/files/12328733/Nginx.pdf)

[Nginx高级.pdf](https://github.com/HesterG/docusaurus-wiki/files/12328734/Nginx.pdf)

[Nginx高级课程扩容与高效.md](https://github.com/HesterG/docusaurus-wiki/files/12328735/Nginx.md)