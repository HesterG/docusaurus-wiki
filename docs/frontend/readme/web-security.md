# Web Security

## OWASP

[OWASP Web Application Security Testing Checklist](https://github.com/0xRadi/OWASP-Web-Checklist)

[cheatsheet](https://cheatsheetseries.owasp.org/cheatsheets/AJAX\_Security\_Cheat\_Sheet.html)



## Vulnerabilities

[top 10 security vulnerabilities](https://sucuri.net/guides/owasp-top-10-security-vulnerabilities-2020)

[前端常见安全汇总](https://blog.flqin.com/390.html)

1. [点击劫持](https://zh.javascript.info/clickjacking)

2. [XSS](https://juejin.cn/post/6844903685122703367)

3. [CSRF](https://juejin.cn/post/6844903689702866952)

   CSRF自动防御策略：
   同源检测（Origin 和 Referer 验证）。[HTTP headers 之 host, referer, origin](https://juejin.cn/post/6844903954455724045#heading-6)

   CSRF主动防御措施：
   Token验证 或者 双重Cookie验证 以及配合Samesite Cookie。
   保证页面的幂等性，后端接口不要在GET页面中做用户操作。

4 - 6.[opener、cdn劫持、中间人攻击](https://blog.flqin.com/390.html)



## CSP

[MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CSP)

[Content Security Policy 入门教程](http://www.ruanyifeng.com/blog/2016/09/csp.html)



## 可阅读资料

[web安全](https://websec.readthedocs.io/zh/latest/)