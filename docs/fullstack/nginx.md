---
title: nginx
---

# 负载均衡

[视频](https://www.bilibili.com/video/BV1yS4y1N76R?p=26)

负载均衡轮询有个问题：session不能在server间共享

负载均衡解决会话不能共享的一种方式：把session存在redis server上

真正高并发不会用，会使用无状态的方案，下发token (类似jwt)

