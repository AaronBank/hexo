---
title: react-native 获取设备imei
date: 2019-07-30 16:36:49
tags: ReactNative
categories: 前端
cover: http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/132622.jpg
---

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/132622.jpg)



项目开发中需要获取设备的`IMEI`，所以整理了一下写出这篇文档，项目使用的是[react-native-imei](https://github.com/SimenCodes/react-native-imei)。但需要注意的是，`Ios`做不了的😿



### 当前环境

- OS: macOS High Sierra 10.13.5
- Node: 8.11.3
- Yarn: 1.7.0
- npm: 5.6.0
- Watchman: 4.9.0
- Xcode: Xcode 9.4.1
- react: 16.3.1 => 16.3.1
- react-native: 0.55.4 => 0.55.4
- react-native-imei: 0.1.1 => 0.1.1



### 前置说明

- `IOS7`以后不能获取手机`IMEI`
- `Android`个别情况下无法获得`IMEI`



### 获取android设备IMEI

#### 安装

```shell
npm install react-native-imei --save
# or with yarn
yarn add react-native-imei
```



#### 自动配置

```shell
react-native link react-native-imei
```



#### 手动配置

1. 在项目路径`{project}/android/`下找到`settings.gradle`文件，添加一下内容

```java
...
  
include ':react-native-imei'

project(':react-native-imei').projectDir =  new  File(rootProject.projectDir, '../node_modules/react-native-imei/android')
  
...
```

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/132707.png) 



2. 在项目路径`{project}/android/app/`下找到`build.gradle`文件，添加一下内容

```java
...

dependencies {
  
    compile project(':react-native-imei')    //-> here
    ...
}

...
```

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/132713.png) 



3. 在项目路径`{project}/android/app/src/main/java/com/app/`下找到`MainApplication.java`文件，添加一下内容

```java
...
  
public class MainApplication extends Application implements ReactApplication  {
    private final ReactNativeHost mReactNativeHost = new ReactNativeHost(this) {
        ...

        @Override
        protected List<ReactPackage> getPackages()  {
            return Arrays.<ReactPackage>asList(
                new MainReactPackage(),
                new IMEI() //-> here
            );
        }
    }
};
...
```

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/132721.png) 



### 在项目中使用

1. 在项目中引入`react-native-imei`

```javascript
import  IMEI  from  'react-native-imei'
```

2. 通过`IMEI.getImei()`方法获取当前设备IMEI

```javascript
componentDidMount () {
    const imei = Platform.OS === 'android' ? IMEI.getImei() || 'android' : 'ios'
    console.log(imei)
}
```

3. 由于ios设备不支持IMEI所以我们直接写入`ios`字符串去辨别设备类型，当然你也可以定义其他的方式去做，都可以

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/132726.png) 



**如果配置出现问题或者文章哪里写的不清楚，欢迎下方留言ヾ(￣▽￣)**