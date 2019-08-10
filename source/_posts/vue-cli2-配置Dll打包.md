---
title: vue-cli2 配置Dll打包
date: 2019-08-01 14:47:39
tags: Vue
categories: 工程化
cover: http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/073932.jpg
---

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/073932.jpg)



`vue-cli`在项目打包时，对于依赖的第三方库，比如`vue、vuex、vue-router...`等这些不会修改的依赖，我们希望这些依赖单独进行打。这样做的好处是每次更改我本地代码的文件的时候，`webpack`只需要打包我项目本身的文件代码，而不会再去编译第三方库，打包速度会有质的提升。并且也有利于对这些非业务相关代码在客户端缓存的利用。

因此为了实现这个问题，`DllPlugin` 和 `DllReferencePlugin`插件也就诞生了😬。

不多哔哔，还是说说怎么用吧 (♥ω♥)~♪ 

### 项目分析

本文只阐述关于`vue-cli 2.x`版本，`3.x`对一些第三方库有相对路径引入，如果你依旧想使用`DLL`去切分包可以尝试一下[vue-cli-plugin-dll](https://github.com/fingerpan/vue-cli-plugin-dll)这个工具。好了说下我们当前项目目录：

![image-20190801162922915](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/082923.png)



这是一个标准的`cli 2`创建的工程，下面我做个打包分析（因为我这个项目是真实项目可能比较大）。

#### 添加analyzer

修改package.json

```json
{
  "name": "DLL例子",
  "version": "1.0.0",
  "scripts": {
    "start": "npm run dev",
    "dev": "cross-env NODE_ENV=development webpack-dev-server --inline --progress --config build/webpack.dev.conf.js",
    "build": "cross-env NODE_ENV=production node build/build.js",
    "analyzer": "npm run build --report" //=> here
  }
  ...
}
```

执行打包分析命令

```
yarn analyzer
# or
npm run analyzer
```

打完包会自动打开一个窗口，就可以看到打包后每一块包含着什么，占比多少。下面看一下我打包后的分析图

![image-20190801164951855](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/084952.png)

可以看出`vendor.js`打包后原始大小达到`2.17MB`，而这个包你每次迭代发布时后面的`chunkhash`都会变化。意味着客户端的缓存就没了，当再次打开我们的网站时这个`vendor.js`又会重新加载。

说到这我们可以看下这个`vendor.js`到底包含了什么，为什么会这么大？

![image-20190801165511594](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/085512.png)

放大后我觉得不需要多解释了，`element-ui`、`vue`、`core`、`router`等等。。。

下面我们利用`DLL`对他们单独进行打包：



### 配置DLL

#### 在配置中添加dll版块

修改`/config/index.js`

```javascript
module.exports = {
    dev: {
        ...
    },
    dll: {
        entry: {
            'vueVendor': [
                'babel-polyfill',
                'vue/dist/vue.esm.js',
                'vue-router',
                'vuex',
                'axios'
            ],
            vendor: [
                '@dal/element-ui',
                'nprogress',
                'viewerjs',
                'v-viewer',
                'vuex-persistedstate',
                'qs'
            ]
        },
        outputPath: 'cdn'
    },
    build: {
    	...
    }
}

```

解释一下：

- 首先对于我，我觉得有一部分第三方库是很久都不会变的，所以我将他们打包到`vueVendor`
- 对于一些可能发生变化的（这里的`element-ui`采用的事内部源，所以变化可能也很大）将他们打包到`vendor`，当然这两个名字你可以随便起，但是多文件有个坑最后说。
- `outputPath`用作我们将来`DLL`包打包到哪里，我这里起名叫`cdn`
- 把配置写在这里是为了后续方便维护



#### 创建Dll打包配置文件

在`build`文件夹下添加名为`webpack.dll.conf.js`的文件，内容为：

```javascript
const webpack = require('webpack')
const path = require('path')
const config = require('../config/index')

module.exports = {
    entry: config.dll.entry, // 配置打包入口
    output: {
        path: path.join(__dirname, '..', config.dll.outputPath), // 配置打包输出目录
        filename: '[name].[chunkhash].dll.js', // chunkhash是为了日后清理缓存
        library: '[name]_library'
    },
    plugins: [
        new webpack.DllPlugin({
            path: path.join(__dirname, './[name]-manifest.json'),
            name: '[name]_library',
            context: __dirname // 这里的上下文要和我们后面用的DllReferencePlugin的上下文一致。 
        }),
        new webpack.optimize.UglifyJsPlugin({ // 丑化Js
            compress: {
              warnings: false,
              drop_console:true,
              drop_debugger:true
            },
            output:{
              comments: false,
            },
            sourceMap: false
        })
    ]
}

```



#### 创建DLL打包脚本

```javascript
const webpack = require('webpack')
const path = require('path')
const ora = require('ora') // 一个执行动画，为了美观没啥
const rm = require('rimraf')
const chalk = require('chalk')
const config = require('../config/index')
const dllConfig = require('./webpack.dll.conf')

const spinner = ora('Start pre-operation...')
spinner.start()

rm(path.join(__dirname, '..', config.dll.outputPath), err => { // 打包前清空cdn文件夹
    if (err) throw err
    webpack(dllConfig, (err, stats) => { // 执行打包
        spinner.stop()
        if (err) throw err
        process.stdout.write(stats.toString({ // 一些日志
            colors: true,
            modules: false,
            children: false,
            chunks: false,
            chunkModules: false
        }) + '\n\n')

        if (stats.hasErrors()) { // 如果失败输出错误日志并退出进程
            console.log(chalk.red('  Build failed with errors.\n'))
            process.exit(1)
        }

        console.log(chalk.cyan('  Build complete.\n'))
        console.log(chalk.yellow(' Project decomposition is successful, enter startup status ...'))
    })
})

```

到目前为止我们就能为这些第三方库做打包了`DLL`（dll打包和项目打包是分开的）



#### 配置脚本，开始打包

我们修改`package.json`中的`scripts`添加一条`DLL`打包的命令：

```json
{
  "name": "DLL例子",
  "version": "1.0.0",
  "scripts": {
    "start": "npm run dev",
    "dev": "cross-env NODE_ENV=development webpack-dev-server --inline --progress --config build/webpack.dev.conf.js",
    "build": "cross-env NODE_ENV=production node build/build.js",
    "analyzer": "npm run build --report",
    "dll": "node build/dll.js" //=> here
  },
  ...
}
```

完后执行打包命令：

```shell
yarn dll
# or
npm run dll
```

不出意外的话你的根目录会出现一个叫做`cdn`的文件夹（或者是你自己配置的文件夹名），同时`build`文件夹下也会多出两个`***-manifest.json`文件。上图看一下

![image-20190802111926063](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/031926.png)

打包后的文件目录：

![image-20190802112708091](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/032708.png)



好了，接下来我们只要告诉`webpack`遇到这些公共包的时候不进行打包，这也就是`***-manifest.json`文件的作用，下一步我们要配置`webpack.dev.conf.js`以及`webpack.prod.conf.js`文件



#### 修改开发打包配置

再此之前我们需要安装一个`webpack`插件，作用就是自动帮我们把`Dll`这块静态`js`文件自动引入到项目中。

我们这里用到的是[add-asset-html-webpack-plugin](https://github.com/SimenB/add-asset-html-webpack-plugin)

打开`/build/webpack.dev.conf.js`文件

```javascript
const utils = require('./utils')
const webpack = require('webpack')
const config = require('../config')
const merge = require('webpack-merge')
const path = require('path')
const baseWebpackConfig = require('./webpack.base.conf')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin')
const portfinder = require('portfinder')

// 引入插件
const AddAssetHtmlPlugin = require('add-asset-html-webpack-plugin')

const devWebpackConfig = merge(baseWebpackConfig, {
    devServer: {
        ...
    },
    plugins: [
        ...
        new HtmlWebpackPlugin({
            filename: 'index.html',
            template: 'index.html',
            inject: true
        }),
        // 在HtmlWebpackPlugin后面进行添加
        // 因为我们是多个dll文件，所以需要处理一下
        ...Object.keys(config.dll.entry).map(name => {
            return new webpack.DllReferencePlugin({
                context: __dirname,
                manifest: require(`./${name}-manifest.json`),
            })
        }),
        // 使用AddAssetHtmlPlugin插件将我们的dll文件自动引入到html模板中
        new AddAssetHtmlPlugin([{
            // 指定dll存放位置
            filepath: path.resolve(__dirname, '..', `${config.dll.outputPath}/*.dll.js`),
            // 引入后并将dll文件自动移动到dist文件夹下
            outputPath: utils.assetsPath(config.dll.outputPath),
            // 指定引入静态路径前缀
            publicPath: path.posix.join(config.dev.assetsPublicPath, config.dev.assetsSubDirectory, config.dll.outputPath),
            includeSourcemap: false
        }])
    ]
})
```

好了，自此我们开发环境下就配置完成了。截下来我们说一下生产环境的打包修改：



#### 修改生产打包配置

```javascript
const path = require('path')
const utils = require('./utils')
const webpack = require('webpack')
const config = require('../config')
const merge = require('webpack-merge')
const baseWebpackConfig = require('./webpack.base.conf')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const ExtractTextPlugin = require('extract-text-webpack-plugin')
const OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin')
const UglifyJsPlugin = require('uglifyjs-webpack-plugin')

// 引入插件
const AddAssetHtmlPlugin = require('add-asset-html-webpack-plugin')

const webpackConfig = merge(baseWebpackConfig, {
    plugins: [
        ...
        new HtmlWebpackPlugin({
            ...
        }),
        // 在HtmlWebpackPlugin后面进行添加
        // 因为我们是多个dll文件，所以需要处理一下
        ...Object.keys(config.dll.entry).map(name => {
            return new webpack.DllReferencePlugin({
                context: __dirname,
                manifest: require(`./${name}-manifest.json`),
            })
        }),
        // 使用AddAssetHtmlPlugin插件将我们的dll文件自动引入到html模板中
        new AddAssetHtmlPlugin([{
            // 指定dll存放位置
            filepath: path.resolve(__dirname, '..', `${config.dll.outputPath}/*.dll.js`),
            // 引入后并将dll文件自动移动到dist文件夹下
            outputPath: utils.assetsPath(config.dll.outputPath),
            // 指定引入静态路径前缀(这里我们使用了cdn服务器)
            publicPath: `${config.build.assetsPublicPath}${config.build.assetsSubDirectory}/${config.dll.outputPath}`,
            includeSourcemap: false
        }]),
        ...
    ]
})

module.exports = webpackConfig
```

到此为止，我们用`DLL`进行打包分割已经做完了，下面我们可以看下效果。



#### 再次进行打包分析

```
yarn analyzer
# or
npm run analyzer
```

通过`DLL`包拆分后我们的`vendor.js`仅剩不到`80kb`

![image-20190801191405068](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/111405.png)



我们单独放大这块进行查看：

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/111511.png)



到目前为止我们的的包已经完美的分割开了，最后再查看下打包后的目录：

![image-20190802112222035](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/032222.png)

可以看到`cdn`这个目录已经自动迁移到`/dist`目录下了，那这是怎么做到的呢？其实这个是`AddAssetHtmlPlugin`帮我们做了，也就是这句起作用了

![image-20190802105002776](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/025003.png)

追后我们看下最终生成的`html`模板文件是什么样的：

![image-20190802112503021](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/032503.png)



最后说明一个**容易忽略**的点：`DLL`多入口生成没有问题，但`AddAssetHtmlPlugin`引入你文件的顺序是按你打包后文件夹文件的倒叙方式引入的。当然只有在你打包的是多文件才会有这种问题。好了，接下来我们再把使用中其他细节问题拿出来聊一下



### 检测`cdn`目录完整

如果我们后期变动了`DLL`包中的任何一个，进行了重新`Dll`打包，那么可想而知`cdn`文件夹下的两个文件的`chunkhash`会变化，那么上传到仓库后，别的同事合并你的代码如果不注意可能`cdn`会存在新旧两个版本的文件，这样会导致项目存在风险。所以我们需要做一个工具，打包之前提前检测`cdn`目录下文件是否符合预期，如不符合我们应该前置进行`DLL`打包。同理我们还可以对`build`文件夹下的`***-manifest.json`映射文件进行简单的查验，不通过同样进行前置打包处理，下面我们看下具体如何实现。



首先我们再`build`下创建名为`detection.js`的文件，内容如下：

```javascript
const fs = require("fs")
const path = require('path')
const chalk = require('chalk')
const process = require('child_process')
const config = require('../config/index')

// 构建dll
const execScript = () => {
    try {
        console.log(chalk.yellow(' DLL incomplete will be repackaged...'))
        process.execSync('npm run dll')
    } catch (error) {
        console.log(chalk.red('  Build failed with errors.\n'))
        process.exit(1)
    }
}

module.exports = (env) => {
    console.log(chalk.blue(' Check DLL package integrity...'));
    // 为了保证线上环境包完整性，每次发布build之前先进行dll打包操作，开发环境则做动态监测
    if (env === 'production') return execScript()
    // 获取dll配置入口配置对象
    const dlls = Object.keys(config.dll.entry)
    // 获取dll入口文件个数
    const dllLength = dlls.length

    try {
        // 读取cdn目录文件
        const files = fs.readdirSync(path.join(__dirname, '..', 'cdn'))
        // 读取当前目录文件
        const manifests = fs.readdirSync(__dirname)
        // 判断manifest映射文件是否存在
        const isManifests = manifests.filter(manifest => /.json$/.test(manifest)).every(manifest => dlls.some(dll => `${dll}-manifest.json` === manifest))

        // 如映射文件或文件数量不符则重新构建dll包
        if (isManifests && files.length === dllLength) {
            console.log(chalk.green(' Congratulations DLL package available'))
        } else {
            execScript()
        }
    } catch (error) {
        execScript()
    }
}

```



完后我们在`build/webpack.base.conf.js`文件中引入并执行他：

```javascript
const path = require('path')
const utils = require('./utils')
const config = require('../config')
const vueLoaderConfig = require('./vue-loader.conf')
// 引入我们编写的脚本
const detection = require('./detection')

// 检测dll包可用性
detection(process.env.NODE_ENV)

function resolve (dir) {
    ...
}

const createLintingRule = () => ({
    ...
})

module.exports = {
    ...
}

```



完后我们吧`cdn`下的文件删除一个

![image-20190802114406691](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/034407.png)

执行打包命令，查看是否会重新生成我们的静态文件：

```
yarn build
# or
npm run build
```

![image-20190802114735617](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/034736.png)

可以看到，我们的检测脚本正常执行了 ，并且自动进行了`DLL`打包的过程，最后我们再查看下`dist`目录和`cdn`目录的情况：

![image-20190802114922878](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/034923.png)



好了，有什么问题记得的留言😂😂😂