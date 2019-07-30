---
title: react-native 生成二维码
date: 2019-07-30 16:36:31
tags: ReactNative
categories: 前端
cover: http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/093827.jpg
---

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/093827.jpg) 



对于二维码我们都并不陌生，`react-native`里面我推荐一个开源库[react-native-qrcode](https://github.com/cssivision/react-native-qrcode)去生成。当然，这里只简单的说一下功能的使用，源代码并不多，有兴趣的可以去看看。

好了接下来我们就简单的看一下如何使用这个模块去实现生成二维码的功能:



### 当前环境

- OS: macOS High Sierra 10.13.5
- Node: 8.11.3
- Yarn: 1.7.0
- npm: 5.6.0
- Watchman: 4.9.0
- Xcode: Xcode 9.4.1
- react: 16.3.1 => 16.3.1
- react-native: 0.55.4 => 0.55.4
- react-native-qrcode: 0.2.6 => 0.2.6





### 兼容性

- 支持`ios`和`android`



### 安装模块

```shell
yarn add react-native-splash-screen 
# or with yarn
npm install react-native-splash-screen -S
```



### 在项目中使用模块

```javascript
import React, { Component } from 'react'
import { View, StyleSheet } from 'react-native'
import QRCode from 'react-native-qrcode'

export default class CreateQRCode extends Component{
    render () {
        return (
            <View style={ styles.container }>
                <QRCode
                    value={ https://www.google.com }
                    size={ 130 }
                    bgColor="#000"
                    fgColor="#FFF"
                />
            </View>
        )
    }
}

const styles = StyleSheet.create({
    container: {
        width:  130,
        height:  130
    }
})
```



### 示例结果

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/092753.png) 

## props参数说明


| prop    | 类型               | 默认值                 |
| ------- | ------------------ | ---------------------- |
| value   | string             | https://www.google.com |
| size    | number             | 128                    |
| bgColor | string (CSS color) | "#000"                 |
| fgColor | string (CSS color) | "#FFF"                 |



**好了实现了，东西很简单，有兴趣的可以去读读源码。透露一下，内部是使用Canvas加WebView实现的，如果遇到什么小问题欢迎下方留言，Thanks♪(･ω･)ﾉ**

