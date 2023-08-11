# Tech Tools

## npm

* [pnpm 可以给你带来什么](https://juejin.cn/post/6951190236290351118)
*   [`npm run xxx`发生了什么](https://juejin.cn/post/7078924628525056007)

    `npm i`的时候，`npm`就帮我们配置好了软连接，这种软连接相当于一种映射，执行`npm run xxx`的时候，就会到 `node_modules/bin`中找对应的映射文件，然后再找到相应的`js`文件来执行。

    `node_modules/bin`中 有三个`vue-cli-service`文件。

    为什么会有三个文件呢？

    ```
    # unix 系默认的可执行文件，必须输入完整文件名
    vue-cli-service
    ​# windows cmd 中默认的可执行文件，当我们不添加后缀名时，自动根据 pathext 查找文件
    vue-cli-service.cmd​
    # Windows PowerShell 中可执行文件，可以跨平台
    vue-cli-service.ps1
    ```

    **总结**

    * 运行 npm run xxx的时候，npm 会先在当前目录的 node\_modules/.bin 查找要执行的程序，如果找到则运行；
    * 没有找到则从全局的 node\_modules/.bin 中查找，npm i -g xxx就是安装到到全局目录；
    * 如果全局目录还是没找到，那么就从 path 环境变量中查找有没有其他同名的可执行程序。


## git

* [Git 对象模型](https://ruby-china.org/topics/20723) 底下评论有大佬画图解释
* git ssh config file参考
    ```
    Host *
      AddKeysToAgent yes
      UseKeychain yes
      IdentityFile ~/.ssh/id_rsa

    Host *
      HostName github.com
      AddKeysToAgent yes
      UseKeychain yes
      IdentityFile ~/.ssh/id_rsa
    ```
    查看ssh相关debug信息:
    `ssh -vT git@github.com`
