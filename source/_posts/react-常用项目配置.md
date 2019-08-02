---
title: react 常用项目配置
date: 2019-07-31 13:44:32
tags: React
categories: 工程化
cover: http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/064604.jpg
---

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/064604.jpg)



## 开发环境配置热更新

### 安装插件

```
yarn add react-hot-loader -D
```



### 修改package.json

```json
"babel":  {
    "plugins":  [
        "react-hot-loader/babel"
    ],
    "presets":  [
        "react-app"
    ]
}
```



### 修改webpack.config.dev.js

```javascript
// 打包入口文件处添加
entry: [
    'react-hot-loader/patch',
    ...
]
```



###  配置App.js项目入口

```javascript
import React, { Component } from  'react'
import { hot } from  'react-hot-loader'

class App extends Component{
    render () {
        return (
            <div className="App"></div>
        );
    }
}

export default hot(module)(App)
```



参考链接： [https://www.jianshu.com/p/45ebcea63057](https://www.jianshu.com/p/45ebcea63057)



## 配置ES7修饰器



### 下载babel插件

```shell
yarn add babel-plugin-transform-decorators-legacy @babel/plugin-proposal-decorators -D
```



### 修改package.json

```json
"babel": {
    "plugins": [
        [
            "@babel/plugin-proposal-decorators",
            {
                "legacy": true
            }
        ],
        [
            "@babel/plugin-proposal-class-properties",
            {
                "loose": true
            }
        ],
    ],
    "presets": [
        "react-app"
    ]
}
```



参考链接： [https://blog.csdn.net/ZhangYaBo_Code/article/details/83066844](https://blog.csdn.net/ZhangYaBo_Code/article/details/83066844)



## 配置less

### 下载less及相关loader

```shell
yarn add less-loader less -D
```





### 修改`webpack.config.dev.js`

```javascript
// 1. 添加 less 正则等变量
// style files regexes

const cssRegex = /\.css$/;
const cssModuleRegex = /\.module\.css$/;
const lessRegex = /\.less$/;
const lessModuleRegex = /\.module\.less$/;

// 2. 修改getStyleLoaders方法，添加一个processorOptions选项用来设置新添加loader的options选项
const getStyleLoaders = (cssOptions, preProcessor, processorOptions={}) => {
    const loaders = [
        require.resolve('style-loader'),
        {
            loader: require.resolve('css-loader'),
            options: cssOptions,
        },
        {
            loader: require.resolve('postcss-loader'),
            options: {
                ident: 'postcss',
                plugins: () => [
                    require('postcss-flexbugs-fixes'),
                    require('postcss-preset-env')({
                        autoprefixer: {
                            flexbox: 'no-2009',
                        },
                        stage: 3,
                    }),
                ],
            },
        },
    ];

    if (preProcessor) {
        loaders.push({
            loader: require.resolve(preProcessor),
            options: Object.assign(processorOptions)
        });
    }
    return loaders;
};

// 3. 修改module部分 将之前所有采用use配置loader部分全部替换成loader
module: {
    rules: [{
        ...
        oneOf: [
            ...
            {
                test: cssRegex,
                exclude: cssModuleRegex,
                loader: getStyleLoaders({
                    importLoaders: 1,
                }),
            },
            {
                test: cssModuleRegex,
                loader: getStyleLoaders({
                importLoaders: 1,
                modules: true,
                    getLocalIdent: getCSSModuleLocalIdent,
                }),
            },
            {
                test: sassRegex,
                exclude: sassModuleRegex,
                loader: getStyleLoaders({ importLoaders: 2 }, 'sass-loader'),
            },
            {
                test: sassModuleRegex,
                loader: getStyleLoaders({
                    importLoaders: 2,
                    modules: true,
                    getLocalIdent: getCSSModuleLocalIdent,
                }, 'sass-loader'),
            },
            // less
            {
                test: lessRegex,
                exclude: lessModuleRegex,
                loader: getStyleLoaders({importLoaders: 2},'less-loader',{
                    javascriptEnabled: true
                }),
            },
            ...
        ],
    }],
},
```



### 修改`webpack.config-prod.js`

```javascript
// 1. 添加less正则等变量
// style files regexes
const cssRegex = /\.css$/;
const cssModuleRegex = /\.module\.css$/;
const lessRegex = /\.less$/;
const lessModuleRegex = /\.module\.less$/;


// 2. 修改getStyleLoaders方法，添加一个processorOptions选项用来设置新添加loader的options选项
const getStyleLoaders = (cssOptions, preProcessor, processorOptions = {}) => {
    const loaders = [
        {
            loader: MiniCssExtractPlugin.loader,
            options: Object.assign(
                {},
                shouldUseRelativeAssetPaths ? { publicPath: '../../' } : undefined
            ),
        },
        {
            loader: require.resolve('css-loader'),
            options: cssOptions,
        },
        {
            loader: require.resolve('postcss-loader'),
            options: {
                ident: 'postcss',
                plugins: () => [
                    require('postcss-flexbugs-fixes'),
                    require('postcss-preset-env')({
                        autoprefixer: {
                            flexbox: 'no-2009',
                        },
                        stage: 3,
                    }),
                ],
                sourceMap: shouldUseSourceMap,
            },
        },
    ];

    if (preProcessor) {
        loaders.push({
            loader: require.resolve(preProcessor),
            options: Object.assign({
                sourceMap: shouldUseSourceMap,
            }, processorOptions)
        });
    }

    return loaders;
};


// 3. 修改module部分  将之前所有采用use配置loader部分全部替换成loader
module: {
    rules: [
        {
            ...
            oneOf: [
                ...
                // less
                {
                    test: lessRegex,
                    exclude: lessModuleRegex,
                    loader: getStyleLoaders({ importLoaders: 2, sourceMap: shouldUseSourceMap }, 'less-loader', {
                        javascriptEnabled: true
                    }),
                },
                ...
            ],
        },
    ],
},
```





参考链接： [https://segmentfault.com/a/1190000010162614](https://segmentfault.com/a/1190000010162614)



## 配置antd按需加载



###  安装插件

```shell
yarn add babel-plugin-import -D
```





### 修改package.json

```json
"babel": {
    "plugins": [
         ["import",{"libraryName":  "antd","style":  "css"}]
    ],
    "presets": [
        "react-app"
    ]
}
```



**或者配置**`webpack.config.dev.js`和`webpack.config.prod.js`文件



```javascript
...
{
    test: /\.(js|mjs|jsx|ts|tsx)$/,
    include: paths.appSrc,
    loader: require.resolve('babel-loader'),
    options: {
        customize: require.resolve(
            'babel-preset-react-app/webpack-overrides'
        ),
        plugins: [
            [
                require.resolve('babel-plugin-named-asset-import'),
                {
                    loaderMap: {
                        svg: {
                            ReactComponent: '@svgr/webpack?-prettier,-svgo![path]',
                        },
                    },
                },
                ['import', [{ libraryName: "antd", style: 'css' }]]
            ],
        ],
        cacheDirectory: true,
        cacheCompression: false,
    },
},
...
```

以上两种方法任何一种都可以实现，个人感觉上面方法比较简单一些😺。



参考链接： [https://blog.csdn.net/well2049/article/details/78801228](https://blog.csdn.net/well2049/article/details/78801228)