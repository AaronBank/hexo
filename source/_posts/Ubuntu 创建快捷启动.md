---
layout: ubuntu
title: Ubuntu 创建快捷启动
date: 2019-07-29 20:41:47
tags: ubuntu
categories: 系统平台
cover: http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/072847.png
---

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/124421.png) 

**以开发工具IDEA编辑器为例，顺便介绍一下如何在Ubuntu 中如何安装和配置快捷启动（因为每次都需要终端中打开很麻烦 (⁎⁍̴̛ᴗ⁍̴̛⁎)）**



### IDEA安装

1. 下载linux版本包
  
	点击前往下载：[https://www.jetbrains.com/idea/download/#section=linux](https://www.jetbrains.com/idea/download/#section=linux)
  
2. cd 进入IDEA文件的目录位置(我的是下载在Downloads文件夹下)

	```
	cd ~/Downloads/
	```
	
3. 减压并移动到/opt文件夹下

	```
	sudo tar -zxvf ideaIU-2018.1.4-no-jdk.tar.gz -C /opt
	```
	
4. 启动
	
	```
	cd /opt/ideaIU-2018.1.4-no-jdk/bin/`
	
	./idea.sh	
	```

- **至此我想你已经看到了启动向导界面了  和往常一样使用就可以了**

- **然而你会发现每次打开都需要执行`./idea.sh`这条命令才能打开，是不是很悲剧😫😫😫，下面说一下如何去配置快捷启动**



### 配置快捷启动



1. 首先你需要`/usr/share/`这个文件夹下

    ```
    cd /usr/share/
    ```

2. 创建一个后缀名为`desktop`的文件（名字你随意,我就叫他idea）

	```
	sudo gedit idea.desktop
	```
	
3. 为文件添加内容

		```desktop
	[Desktop Entry]
	Name = Idea
	Exec = /opt/ideaIU-2018.1.4-no-jdk/bin/idea.sh
	Icon = /opt/ideaIU-2018.1.4-no-jdk/bin/idea.png
	Terminal = false
	Type = Application
	Categories = Developer;
	```

🍎🍎现在可以每次都双击图标就可以启动啦🍎🍎