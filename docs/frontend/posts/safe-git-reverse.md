---
title: 安全的git回退方式记录
date: 2022-05-06T17:05:56.000Z
tags:
  - git
categories:
  - - Tech
    - Git
---

# 安全的git回退方式记录

远程Git Repo的安全回退方式

## 常用回退指令: git revert, git reset

### git工作区

**Workspace**： 工作区，就是你平时存放项目代码的地方。 **Index / Stage**： 暂存区，用于临时存放你的改动，事实上它只是一个文件，保存即将提交到文件列表信息。`git add <file>`以后，文件就会被存储于这个区域。 **Repository**： 仓库区（或版本库），就是安全存放数据的位置，这里面有你提交到所有版本的数据。其中HEAD指向最新放入仓库的版本。`git commit <file>`以后，文件就会被存储于仓库区。 **Remote**： 远程仓库，托管代码的服务器，可以简单的认为是你项目组中的一台电脑用于远程数据交换。`git push <file>`以后，文件就会被上传到远程仓库。

### git reset

用于回退版本，可以遗弃不再使用的提交，根据参数的区别，可以指定回退的额范围(scope)，即是否回退暂存区(Index / Stage)和工作区(Workspace)的内容 `--hard` 参数: 把head移动回指定commit(默认为最新的一个commit)，并且工作区和暂存区内容都将回退 `--soft`参数：把head移动回指定commit(默认为最新的一个commit)，工作区和暂存区都不会发生变化。即运行`git log`发现head变化了，其余的都不变。 `--mixed`参数: 默认的参数，把head移动回指定commit(默认为最新的一个commit)，只有暂存区变化。

### git revert

在当前提交后面，新增一次提交，抵消掉上一次提交导致的所有变化，不会改变过去的历史，主要是用于安全地取消过去发布的提交。 注意：`git revert <commid>`是撤销commid，即撤销到该commid上一个commit,不是撤销到该commid

### git reset和git revert的区别

* `git revert`是用一次**新的commit来回滚之前的commit**，`git reset`是**直接删除**指定的commit；
* `git reset`是把`HEAD`向后移动了一下，而`git revert`是`HEAD`继续前进，只是新的commit的内容和要revert的内容正好相反，能够抵消要被revert的内容；
* 在操作上，`git reset`以后如果没有`pull`就要`push`的话必须要加`-f`,这也是`reset`不安全的理由；
* `git revert`被设计为撤销公开的提交(已经`push`了的commit)的安全方式，`git reset`被设计为重设本地更改。

## 如何安全地回滚远程分支

1. 使用`git revert`
2. 使用`merge -s`产生一个**代码状态与历史版本完全一致，但基于 `master`(要回退到的head) 的一个commit**。[具体方式](https://harttle.land/2018/03/12/reset-origin-without-force-push.html)

* 两者都是新提交一个commit来撤销操作（**忌对远程分支使用`reset`或`rebase`， git官方也推荐使用`revert`）**

## 其他撤销操作小贴士

1.  替换上一次的提交信息

    ```
    # 产生一个新的提交对象，替换掉上一次提交产生的提交对象。
    # 这时如果暂存区有发生变化的文件，会一起提交到仓库。所以，--amend不仅可以修改提交信息，还可以整个把上一次提交替换掉。
    git commit --amend -m "Fixes bug #42"
    ```
2.  撤销工作区的文件修改

    ```
    # 如果工作区的某个文件被改乱了，但还没有提交，可以用git checkout命令找回本次修改之前的文件。
    git checkout -- [filename]
    ```
3.  从暂存区撤销文件

    ```
    git rm --cached [filename]
    ```

## 参考

* [文章一](https://www.cnblogs.com/qdhxhz/p/9757390.html)
* [文章二](https://www.jianshu.com/p/c2ec5f06cf1a)
* [文章三](https://juejin.cn/post/6844903542931587086)
* [文章四](https://harttle.land/2018/03/12/reset-origin-without-force-push.html)
* [文章五](https://www.ruanyifeng.com/blog/2019/12/git-undo.html)
