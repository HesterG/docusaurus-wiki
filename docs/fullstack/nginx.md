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

[keepalived配置](https://www.bilibili.com/video/BV1yS4y1N76R/?p=37)

## HTTPS

### 对称算法

[为什么对称算法不安全](https://www.bilibili.com/video/BV1yS4y1N76R/?p=38)

![](https://private-user-images.githubusercontent.com/17645053/260349096-6ea892dc-8005-4335-b71d-15277a6723db.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTEiLCJleHAiOjE2OTE5ODI4NTQsIm5iZiI6MTY5MTk4MjU1NCwicGF0aCI6Ii8xNzY0NTA1My8yNjAzNDkwOTYtNmVhODkyZGMtODAwNS00MzM1LWI3MWQtMTUyNzdhNjcyM2RiLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFJV05KWUFYNENTVkVINTNBJTJGMjAyMzA4MTQlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjMwODE0VDAzMDkxNFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTExZTA4YzJjNDU4ODNjNGFlMjZiZjYxZDMxOWU1NTg1MGNlMGJlMDc0MDQ3MmJlY2MzNTZiNTA2ZjNiYThkYTImWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.hHL1lH7vIL89_zxy5armmU-2n-r8HKHKsOUYxdS4MRM)

### 非对称算法

[可以使用非对称算法来解决](https://www.bilibili.com/video/BV1yS4y1N76R/?p=39), 但是[非对称算法也不是最安全的](https://www.bilibili.com/video/BV1yS4y1N76R?p=40)

![](https://private-user-images.githubusercontent.com/17645053/260359675-13f88008-94f0-4ff6-b08a-ff730e52e703.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTEiLCJleHAiOjE2OTE5ODg0MzAsIm5iZiI6MTY5MTk4ODEzMCwicGF0aCI6Ii8xNzY0NTA1My8yNjAzNTk2NzUtMTNmODgwMDgtOTRmMC00ZmY2LWIwOGEtZmY3MzBlNTJlNzAzLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFJV05KWUFYNENTVkVINTNBJTJGMjAyMzA4MTQlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjMwODE0VDA0NDIxMFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTRhYWNjMWM2NTg5NzBlZWFmYjUxOTY2ZTIxODdhNWIwMzg0YzE5MzY5MTYzMmNjM2M1OWU3ZWRmMzBiOGYwZjgmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.U7aYYbjwfT3ZFZ7yURI_nnPJJ1wAIoOtQ3ah6N8irv0)

### CA认证

[CA认证介绍](https://www.bilibili.com/video/BV1yS4y1N76R/?p=40)

![](https://private-user-images.githubusercontent.com/17645053/260361623-7e1d8a57-e83f-4122-96e3-3f1b629fc226.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTEiLCJleHAiOjE2OTE5ODk1MTgsIm5iZiI6MTY5MTk4OTIxOCwicGF0aCI6Ii8xNzY0NTA1My8yNjAzNjE2MjMtN2UxZDhhNTctZTgzZi00MTIyLTk2ZTMtM2YxYjYyOWZjMjI2LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFJV05KWUFYNENTVkVINTNBJTJGMjAyMzA4MTQlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjMwODE0VDA1MDAxOFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTFmYTdhOWNlNzJhNTg1NTBjOWY0MTI4NDZmY2U4MmZmM2U1NzFjZDk0NzI0NDE0ZTkyMjQ1ZGU3ODI4YWIxNGUmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.Zi0oRFBOjkJIcHDIWFb-Is2xAvLFe1KZcfqz85YxCwE)

### 自签名

[自签名介绍](https://www.bilibili.com/video/BV1yS4y1N76R?p=42)

自签名就是自己签名，不用CA签名，非常非常少


## 课件

[Nginx 基础使用.pdf](https://github.com/HesterG/docusaurus-wiki/files/12328730/Nginx.pdf)

[Nginx 安装.pdf](https://github.com/HesterG/docusaurus-wiki/files/12328731/Nginx.pdf)

[Nginx课件.pdf](https://github.com/HesterG/docusaurus-wiki/files/12328733/Nginx.pdf)

[Nginx高级.pdf](https://github.com/HesterG/docusaurus-wiki/files/12328734/Nginx.pdf)

[Nginx高级课程扩容与高效.md](https://github.com/HesterG/docusaurus-wiki/files/12328735/Nginx.md)