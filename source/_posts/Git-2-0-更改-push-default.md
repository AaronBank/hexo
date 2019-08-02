---
title: Git 2.0 更改 push default
date: 2019-07-31 13:43:30
tags: Git
categories: 版本管理
cover: http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/064701.jpg
---

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/064701.jpg)



### 问题处理

`git`更新后,项目`push`时如果提示这样的信息:

`warning: push.default尚未设置` ，是因为它的默认值在 Git 2.0 已从 'matching'
变更为 'simple'。若要不再显示本信息并保持传统习惯，进行如下设置：

 ```shell
git config --global push.default matching
 ```



若要不再显示本信息并从现在开始采用新的使用习惯，设置：

```shell
git config --global push.default simple
```



当 `push.default` 设置为 `matching` 后，`git` 将推送和远程同名的所有本地分支。

从 Git 2.0 开始，Git 默认采用更为保守的 `simple` 模式，只推送当前分支到远程关联的同名分支，即 `git push` 推送当前分支。

参见 `git help config` 并查找 `push.default` 以获取更多信息。
（`simple` 模式由 Git 1.7.11 版本引入。如果您有时要使用老版本的 Git，为保持兼容，请用 `current` 代替 `simple`。



### Matching

`matching` 参数是 `Git 1.x` 的默认行为，其意是如果你执行 `git push` 但没有指定分支，它将 `push` 所有你本地的分支到远程仓库中对应匹配的分支。



### Simple

而 `Git 2.x `默认的是 `simple`，意味着执行 `git push` 没有指定分支时，只有当前分支会被 `push` 到你使用 `git pull` 获取的代码。



### 修改默认设置

从上述消息提示中的解释，我们可以修改全局配置，使之不会每次 push 的时候都进行提示。对于 `matching` 输入如下命令即可：

```shell
git config --global push.default matching
```

而对于 `simple` ，请输入：

```shell
git config --global push.default simple
```



**参考地址：**[https://www.oschina.net/news/45585/git-2-x-change-push-default-to-simple](https://www.oschina.net/news/45585/git-2-x-change-push-default-to-simple)

