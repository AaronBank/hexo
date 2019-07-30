---
title: vue调试神器vue-devtools安装
date: 2019-07-29 15:27:48
tags: Vue
categories: 实用工具
cover: http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/095601.jpg
---

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/095601.jpg) 



欲先攻其事必先利其器，就先来弄弄这个工具`vue-devtools`。



### vue-devtools是什么

`vue-devtools`是一款用来调试`Vue`应用的`Chrome`插件,可极大提高开发者调试项目效率,接着我们说一下如何下载安装这个插件。

### 直接下载插件

国内无法访问(你懂得)，有条件的可以访问这个下载 : [国外Chrome插件下载](https://chrome.google.com/webstore/category/extensions)

访问不了外网的也没事，可以访问国内的地址：[国内Chrome插件下载](http://www.cnplugins.com/)



### 自己编译安装

上面两种你都下载不了，那只能本地编译安装了🤣🤣🤣

1. 第一步 : 克隆项目到本地

- 找到github上vue-devtools项目

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/095608-1.png)

- 使用git讲代码克隆到本地 : `git clone https://github.com/vuejs/vue-devtools.git`

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/095602.png)

2. 第二步 :下载相关依赖 : 

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/095603.png)

3. 第三步 : 项目打包  （yarn build 或 npm run build）

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/095604.png)

4. 第四步：

- 打开chrome 更多工具 -> 扩展程序 -> 加载已解压的扩展程序

  ![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/095607.png)

- 选择打包好的文件

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/095608.png)

- 添加插件

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/095605.png)

5. 第五步：F12打开控制台点击最右侧vue选项卡

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/095606.png)

自此搞定O(∩_∩)O哈哈~