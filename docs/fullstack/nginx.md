---
title: Nginx
---

# 负载均衡

[视频](https://www.bilibili.com/video/BV1yS4y1N76R?p=26)

负载均衡轮询有个问题：session不能在server间共享

负载均衡解决会话不能共享的一种方式：把session存在redis server上

真正高并发不会用，会使用无状态的方案，下发token (类似[jwt](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html))

# 课件

[Nginx 基础使用.pdf](https://github.com/HesterG/docusaurus-wiki/files/12328730/Nginx.pdf)

[Nginx 安装.pdf](https://github.com/HesterG/docusaurus-wiki/files/12328731/Nginx.pdf)

[Nginx课件.pdf](https://github.com/HesterG/docusaurus-wiki/files/12328733/Nginx.pdf)

[Nginx高级.pdf](https://github.com/HesterG/docusaurus-wiki/files/12328734/Nginx.pdf)

[Nginx高级课程扩容与高效.md](https://github.com/HesterG/docusaurus-wiki/files/12328735/Nginx.md)