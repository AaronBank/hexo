---
title: 面向对象从ES3到ES6+的继承方法总结以及对比
date: 2019-07-29 20:47:23
tags: Javascript
categories: 前端
cover: http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/081804.jpg
---

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/081804.jpg)

继承是面向对象中一个非常重要的概念，`Javascript`中可以实现继承，但不支持接口的继承。并且我们js中主要是`依赖原型链实现继承`的，下面说说从**es3**到**es5**中常见的几种继承方式。

**首先**我们**假设**我们有两个类一个`Television`(电视)和`Computer`(电脑)

```javascript
function Television() {
    this.name = '电视'
    this.telecontroller = '遥控器'
}

function Computer() {
    this.name = '电脑'
}
```

`Television`(电视)有个功能叫`play`(播放)，`Computer`(电脑)有个`compute`(计算)的功能。

```javascript
function Television() {
    this.name = '电视'
    this.telecontroller = '遥控器'
}
  
Television.prototype.play = function() {
    alert('播放')
}
  
function Computer() {
    this.name = '电脑'
}
  
Computer.prototype.compute = function() {
    alert('计算')
}
```

然而我们知道`Computer`(电脑)也有`play`(播放)这个功能功能。那你会想到再给`Computer`(电脑)也添加一个`play`(播放)的功能不就好了，但是这样无形中浪费内存的资源，而且写起来也不比较繁琐，这不是我们想要的。这时候继承就出场了，而且实现继承的方式有很多种各有优缺点，接下来我们将每一种拿出来并对比一下他们的优缺点：



### 原型链继承

先上代码

```javascript
function Television() {
    this.name = '电视'
    this.telecontroller = '遥控器'
}

Television.prototype.play = function() {
    alert('播放')
}

function Computer() {
    this.name = '电脑'
}

//必须将这句写到上面否则下面的方法会被覆盖掉
Computer.prototype = new Television()

//因为new Television()实例没有constructor方法 所以我们要讲constructor指向Computer
Computer.prototype.constructor = Computer

Computer.prototype.compute = function() {
    alert('计算')
}
```

`[ 注意 ]` 

> 1. `Computer.prototype =  new Television()`必须写在子类定义原型方法(如现在的`compute`方法)之前,否则子类定义的原型方法会丢失（因为`Computer.prototype`指向堆内存地址变了嘛）。
> 2. 因为父类`Television`生成的对象是没有constructor属性，所以给他加上（为啥没有？我就不用再详细说了吧 (〃'▽'〃)）


`[ 优点 ]`

-  实例是子类的实例，实际上也是父类的一个实例
-  父类新增原型方法/原型属性，子类都能访问到

`[ 缺点 ]`

- 子类的实例的原型都共享同一个父类实例的属性和方法
- 无法继承父类构造函数的属性及方法( new Computer().telecontroller //=> undefined )



### 构造继承

这个就比较简单啦，上代码

```javascript
function Television() {
    this.name = '电视'
    this.telecontroller = '遥控器'
}

Television.prototype.play = function() {
    alert('播放')
}

function Computer() {
    //在这里直接调用父类的方法修改父类的this上下文
    Television.call(this)
    this.name =  '电脑'
}

Computer.prototype.compute = function() {
    alert('计算')
}
```

`[ 优点 ]`

-  简单明了，直接继承父类构造函数的属性及方法

`[ 缺点 ]`

- 无法继承父类原型链上的方法（new Computer().play //=> undefined）



### 组合继承

原型链继承与构造继承相结合各取所长，看下实现

```javascript
function Television() {
    this.name = '电视'
    this.telecontroller = '遥控器'
}

Television.prototype.play = function() {
    alert('播放')
}

function Computer() {
    Television.call(this)
    this.name = '电脑'
}

Computer.prototype = new Television()

Computer.prototype.constructor = Computer

Computer.prototype.compute = function() {
    alert('计算')
}
```


`[ 优点 ]`

-  子类既能继承父类构造函数中的属性和方法，又能继承父类原型链上的方法

`[ 缺点 ]`

- 子类会拥有父类的两份属性，只是子类属性将其覆盖了而已

    

### 原型式继承

原型式继承其实就是定义一个方法，传入Object，并不需要定义一个类，传入参数obj返回一个继承obj对象的新对象，实现以下

首先我们创建一个`objectExtends`方法

```javascript
var obj = {
    name: '电视',
    telecontroller: '遥控器'
}

function objectExtends(obj) {
    function F(){}
    F.prototype = obj
    return new F()
}
```

`[ 优点 ]`

-  简单易懂，直接通过对象生成一个继承该对象的新对象

`[ 缺点 ]`

- 不是类式继承，缺少类的概念

    

### 寄生式继承


这个名词很厉害，说白了就是原型式继承的加强版，同样传入Object，并不需要定义一个类，传入参数obj返回一个继承obj对象的新对象，然后在另一方法中以其他方式增强这个obj对象，返回最终的对象， 实现以下

首先我们创建一个`objectExtends`和`objectEnhance`方法

`objectExtends`方法作用依旧是实现原型式继承
`objectEnhance`方法就是在`objectExtends`方法的基础上对新的obj对象扩展他的原型方法而已

```javascript
var obj = {
    name: '电视',
    telecontroller: '遥控器'
}

function  objectExtends(oldObj) {
    function F(){}
    F.prototype = obj

    return new F()
}

function objectEnhance(obj) {
    var newObj = objectCreate(obj)

    newObj.property = function() {
        alert('扩展一个方法出来')
    }

    return newObj
}
```

`[ 优点 ]`

-  相比原型式继承更加强悍，增强了原型式继承的能力

`[ 缺点 ]`

- 和原型式继一样，缺少类的概念



### 寄生组合式继承

看名字就知道`寄生组合式继承`就是结合了`寄生式继承`和`组合式继承`，也是继承的终极解决方案

```javascript
function  Television() {
    this.name = '电视'
    this.telecontroller = '遥控器'
}

Television.prototype.play = function() {
    alert('播放')
}

function Computer() {
    Television.call(this)
    this.name = '电脑'
}

Computer.prototype.compute = function() {
    alert('计算')
}

function inheritPrototype(Father,Son){
    var newPrototype = Object.create(Father.prototype)

    newPrototype.constructor = Son
    Son.prototype = newPrototype
}

inheritPrototype(Television, Computer)
```

`[ 优点 ]`

-  很完美

`[ 缺点 ]`

- 太繁琐

    

### ES6中的继承

ES6的话就比较简单了，如果有不了解的可以去看看[阮一峰的ECMAScript 6入门](http://es6.ruanyifeng.com/)


```javascript
class Television {
    constructor() {
        this.name =  '电视'
        this.telecontroller =  '遥控器'
    }

    play() {
        alert('播放')
    }
}

class Computer extends Television{
    constructor() {
        super()
        this.name = '电脑'
    }

    compute() {
        alert('计算')
    }
}
```

**`[ 注意 ]`**

- 如果你的子类重写了`constructor`方法，记得调用`super`,以确保父类构造逻辑运行

`[ 优点 ]`

-  和寄生组合式继承实现的效果一致



---

### 关于ES5和ES6中继承的图片

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/125157.jpg)
![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/125209.jpg) 



**更多参考链接：**

[JS继承实现的几种方式及其优缺点](https://segmentfault.com/a/1190000011151188)

[两图看懂ES5和ES6中的继承](https://www.jianshu.com/p/d1bd2218b1f2)

[关于 ES6 中的 Class 的几个特性和玩法](https://www.jianshu.com/p/6c18968c317b)



**好了，有问题的可以再下方留言 (๑•ω•๑)**