---
title: react-native 因图片打包太大问题
date: 2019-07-30 15:59:51
tags: ReactNative
categories: 前端
cover: http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/085345.jpg
---

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/085345.jpg) 



如果RN项目中碰到`生产包太大`的问题。我之前采用的是未处理过的png图片，所以打包后一个应用`android`下达到了**`44M`**，而`ios`已经达到了将近**`60M`**，这很明显是有问题（除非你的应用足够庞大那么这么大事正常的😆）。

遇到这种问题首先想到的就是将包进行减压查看。发现问题就出现在图片`png`太多。思索后决定将应用的图片转成了`webp`的格式，折腾之后，重新打包`android`下**`减少将近26M`**，`ios`下**`减少将近38M`**。我使用的是[react-native-webp-support](https://github.com/TGPSKI/react-native-webp-support)第三方库去解决（`ios`端需要永到这个，安卓直接配置即可），因此将配置webp的方案记录下来，以供大家参考。



### 当前环境

*   OS: macOS High Sierra 10.13.5
*   Node: 8.11.3
*   Yarn: 1.7.0
*   npm: 5.6.0
*   Watchman: 4.9.0
*   Xcode: Xcode 9.4.1
*   react: 16.3.1 => 16.3.1
*   react-native: 0.55.4 => 0.55.4
*   react-native-splash-screen: 3.1.0 => 3.1.0



### 兼容性

- `android`平台下只需要修改配置`{project}/android/app/build.gradle`文件即可，不需要第三方模块支持
- `ios`下需要引入`react-native-webp-support`进行配置



### Android配置

- 修改`{project}/android/app/`目录下的`build.gradle`文件


```java
...

dependencies {

  ...

  compile 'com.facebook.fresco:webpsupport:0.11.0'

  compile 'com.facebook.fresco:animated-webp:0.11.0'

}

...

```

- 配置如下图

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/080132.png)



### Ios配置

`Ios`下比较麻烦，需要引入这个第三方库[react-native-webp-support](https://github.com/TGPSKI/react-native-webp-support)

#### 安装第三方库

```shell
  npm install TGPSKI/react-native-webp-support -S 

  # or with yarn 

  yarn add TGPSKI/react-native-webp-support

```



#### Xcode中进行配置

1. 用Xcode打开项目, 找到`Frameworks`目录，右键选择 `Add Files to [your project's name]`

2. 添加`{project}/node_modules/react-native-webp-support/`下的`WebP.framework`和`WebPDemux.framework`目录

3. 或者直接拖动到`Frameworks`下也行，这里我们假装自己很懂的方式使用添加的方法去添加

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/083449.png) 

4. 将`$(SRCROOT)/../node_modules/react-native-webp-support`分别添加到`Build Settings`下的`Framework Search Paths`和`Header Search Paths`选项中


![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/080138.png) 

5. 在Xcode中, 找到`Libraries`目录，右键选择 `Add Files to [your project's name]`添加`ReactNativeWebp.xcodeproj`

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/080142.png) 

6. 找到`$(SRCROOT)/node_modules/react-native-webp-support`添加到`Libraries`中

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/080145.png) 

7. 将`$(SRCROOT)/node_modules/react-native-webp-support/ReactNativeWebp.xcodeproj/Products/`下的`libReactNatveWebp.a`文件拖到`Build Phases`中的`Link Binary with Libraries`选项中

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/080150.png) 



### 使用

使用就比较简单了，将你现在用的png转化为webp的格式，文件引入直接使用`.webp`为后缀的文件即可

关于转化`webp`方法可是使用[智图工具进行转化](https://zhitu.isux.us/index.php/preview/download)



**如果配置出现问题或者文章哪里写的不清楚，欢迎下方留言ヾ(￣▽￣)**