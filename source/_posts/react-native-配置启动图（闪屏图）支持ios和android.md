---
title: react-native 配置启动图（闪屏图）支持ios和android
date: 2019-07-30 16:37:05
tags: ReactNative
categories: 前端
cover: http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/120857.jpg
---

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/120857.jpg)



配置启动图我使用的是第三方模块`react-native-splash-screen`,更加详细的请到 [github地址](https://github.com/crazycodeboy/react-native-splash-screen)进一步查看，我们这里提供基本的配置以及个别问题的解决方案



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



### 安装第三方模块

```shell
yarn add react-native-splash-screen
# or
npm install react-native-splash-screen -S
```


### 自动配置

```
react-native link react-native-splash-screen
# or
rnpm link react-native-splash-screen
```



### 手动配置

`如果自动配置不成功，则需要手动一步步配置，配置不难，其实也是很快捷方便的`

#### Ios配置

1. 用Xcode打开项目, 找到`Libraries`目录，右键选择 `Add Files to [your project's name]`

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/125749.png) 

2. 在`{project}/node_modules/react-native-splash-screen/ios`找到`SplashScreen.xcodeproj`文件，并添加

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/125755.png) 

3. 在XCode中，选择你的项目， 选择`bilid Phases`并将`SplashScreen.xcodeproj/Products/libSplashScreen.a`文件添加到`Link Binary With Libraries`中,拖过去即可

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/125758.png) 

4. 添加搜索路径：在项目 → Build Settings → Search Paths → Header Search Paths处添加一项为`$(SRCROOT)/../node_modules/react-native-splash-screen/ios`

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/125804.png) 



#### Android配置

1. 在项目目录`{prject}/android/`下找到`settings.gradle`文件添加一下内容

```java
...

include ':react-native-splash-screen'   
project(':react-native-splash-screen').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-splash-screen/android')

...
```

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/125809.png) 

2. 在项目目录`{prject}/android/app/`下找到`build.gradle`文件添加`compile project(':react-native-splash-screen')`


```java
...
  
dependencies {
    ...
    compile project(':react-native-splash-screen')
}

...
```


![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/125817.png) 

3. 找到项目目录`{project}/android/app/src/main/java/com/app/`下`MainApplication.java`文件，引入`import org.devio.rn.splashscreen.SplashScreenReactPackage;`并添加`new SplashScreenReactPackage()`


```java
...
    
import org.devio.rn.splashscreen.SplashScreenReactPackage;

public class MainApplication extends Application implements ReactApplication {
    private final ReactNativeHost mReactNativeHost = new ReactNativeHost(this) {
        ...
        @Override
        protected List<ReactPackage> getPackages() {
            return Arrays.<ReactPackage>asList(
                new SplashScreenReactPackage()  //here
            );
        }
        ...
    };
}

...
```


![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/125823.png) 



### 启动图相关配置

#### 配置Ios启动图

1. 找到项目目录`{project}/ios/{project}/`下`AppDelegate.m`文件，引入`#import  "SplashScreen.h"`添加内容如下

```c++
...

#import  "SplashScreen.h"    // here

...

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    ...
    [SplashScreen show];    // here
    ...
    return YES;
}

@end
```

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/125830.png) 

2. 在XCode中, 点击`App/Images.xcassrts`通过`LaunchImage`添加启动图片

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/125835.png) 

3. 在XCode中，选择你的项目， 选择`General`找到`App Icons and Launch Images`下的`Launch Images Sourc`选择你设置好的`LaunchImage`

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/125840.png) 

4. 如出现错误`# unknown receiver 'SplashScreen'; did you mean 'RNSplashScreen'`, 则改动源码以及`AppDelegate.m`文件如下

5. 源码改动: 找到`{project}/node_modules/react-native-splash-screen/ios`文件夹下的`RNSplashScreen.m`文件，将其`[SplashScreen show];`修改为`[RNSplashScreen show];`,如下图

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/125846.png) 

6. `AppDelegate.m`文件改动: 找到项目目录`{project}/ios/{project}/`下`AppDelegate.m`文件，将所有`SplashScreen`改为`RNSplashScreen`即可，如下图

​	

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/130203.png) 



7. 重启尝试是否有其他问题，修改源码地址可查看我的[Github](https://github.com/AaronBank/react-native-splash-screen)项目地址，更多问题请查看官方[issues](https://github.com/crazycodeboy/react-native-splash-screen/issues)



#### 配置Android启动图

1. 在项目目录`{prject}/android/app/src/main/res/`下创建`layout`文件夹

2. 在`layout`文件夹下创建`launch_screen.xml`添加一下内容

```XML
<xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@drawable/launch_screen"
>
</LinearLayout>
```

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/125849.png) 

3. 将你的启动图改名为`launch_screen.png`并添加到`{prject}/android/app/src/main/res/`文件夹下相应的`drawable`文件夹中（根据需求配置相应大小的文件夹）

- `drawable-ldpi`
- `drawable-mdpi`
- `drawable-hdpi`
- `drawable-xhdpi`
- `drawable-xxhdpi`
- `drawable-xxxhdpi`

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/125851.png) 

4. 在`{prject}/android/app/src/main/res/values/`文件夹下修改名为添加一个名为`primary_dark`的属性并设置其颜，添加内容如下

```xml
...
<color name="primary_dark">#000000</color>
...
```


![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/125857.png) 

5. 如果您希望启动屏幕透明，打开`{prject}/android/app/src/main/res/values/styles.xml`并添加`<item name="android:windowIsTranslucent">trueitem>`到文件中

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/125900.png) 



### 在项目中关闭启动图


> 配置完毕之后重启你的app发现启动图已经展现出来了但是迟迟没有关闭
> 我们需要在项目中进行控制，将他在合适的时候进行关闭，方法如下：

1. 首先引入进行模块`import SplashScreen from 'react-native-splash-screen'`

2. 使用`SplashScreen.hide()`方法将其关闭

```javascript
import SplashScreen from 'react-native-splash-screen'

export default class App extends Component {
    componentDidMount() {
        // do stuff while splash screen is shown
        // After having done stuff (such as async tasks) hide the splash screen
        SplashScreen.hide();
    }
}
```

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/125904.png) 



**如果配置出现问题或者文章哪里写的不清楚，欢迎下方留言ヾ(￣▽￣)**