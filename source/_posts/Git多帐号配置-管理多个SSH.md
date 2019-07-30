---
title: 'Git多帐号配置,管理多个SSH'
date: 2019-07-29 18:09:58
tags: Git
categories: 版本管理
cover: http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/070816.jpg
---

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/070816.jpg)

当我们存在多个`git`账号的时候,比如公司用`Gitlib`或者`Gerrit`等等,而日常自己做的一些东西则会托管到`Github`上面。这些`git`账号的邮箱如果使用的是不同的话，就会存在生成一个`git`的`key`的时候会覆盖另外其他的的key 的问题，下面说一下怎么处理😸

### 查看自己现有的SSH-Key

```python
cd ~/.ssh
ls

id_rsa
id_rsa.pub
known_hosts
```

如果你已经创建过git账号那你可能和我一样会看到只有一个`SSH-Key`,

这个`SSH-Key`是我在公司`Gerrit`(你也可以理解为`Gitlib`一样的东西)所使用的`SSH-Key`。

由于公司所用邮箱于`Gitlib`上邮箱不一致,如果重复生成新的`SSH-Key`只会吧之前的公司的`SSH-Key`覆盖掉,这样并不是我们所想看到的结果。

废话说完了，就开始说说怎么解决这个问题:

### 生成新的的SSH-Key

```python
ssh-keygen -t rsa -C "你的邮箱"
```



第一个内容输入`ompany_id_rsa`(名字随便起，自己能区分的开就行)，余下的一路回车即可

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/101105.png)

到这里,你公司的`SSH-Key`就完成了，查看一下`.ssh`文件夹会多出你新添加`SSH-Key`

```python
cd ~/.ssh
ls

id_rsa
id_rsa.pub
ompany_id_rsa
ompany_id_rsa.pub
known_hosts
```

可以看到目前我们拥有两个不同的SSH-Key，到此为止已经完成一半了，现在还需要关键一步，接着往下看...

### 添加config配置文件

`config`文件作用是给`HOST`单独配置各自的`SSH-Key`，看下具体配置

```python
# 配置GitHub的SSH-Key
Host github.com
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa
    
# 配置公司的SSH-Key
Host gerrit.com
    HostName gerrit.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/ompany_id_rsa
```

保存后配置`**_id_rsa.pub`公钥,以`GitHub`为例：

1. 打开`GitHub`选择右上角头像下拉框选择`Settings`

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/101107.png)

2. 找到`SSH and GPG keys`选项，点击`New SSH key`按钮添加公钥。

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/101108.png)

3. 公钥即为`～/.ssh/**_id_rsa.pub`文件中的内容，我这里使用的是`GitHub`的`id_rsa.pub`

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/101109.png)
好了，没有之后了。祝你在技术的道路上越走越远🤣🤣🤣

