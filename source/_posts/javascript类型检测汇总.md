---
title: javascript类型检测汇总
date: 2019-07-30 16:37:39
tags: Javascript
categories: 前端
cover: http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/060055.jpg
---

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/060055.jpg)



汇总一些简单的易懂的东西出来分享一下，今天说下关于`Javascript`中的类型检测的问题。



### Javascript中的数据类型

说到类型检测前我们首先需要确定`Javascript`中一共有多少种数据类型。



* `Javascript`不支持任何自定义类型的机制，而所有的值最终都将是6大类型之一。由于javascript数据类型具有动态性，因此的确没有再定义其他数据类型的必要。

* `Javascript`数据类型总的分为两大类:基本数据类型(**`5`种**),一种引用数据类型,共 **`6`种**,如下:



1. 基本数据类型

- `Undefined` 类型

-  `Null` 类型

- `Boolean` 类型

-  `Number` 类型

-  `String` 类型



2. 引用数据类型



#### 常用检测类型方法

检测数据类型的方法有:`typeof`、`instanceof`、 `constructor`、 `prototype`。当然还有很多关于第三那方库的API方法我们就不一一列举了。



#### 利用typeof进行类型检测

首先我们使用`typeof`来将我们常用的这几种类型分别测试一下看看返回的是什么

```javascript
typeof undefined  

//=> "undefined"


typeof null

//=> "object"


typeof true

//=> "boolean"


typeof 123

//=> "number"


typeof 'aaa'

//=> "string"


typeof {a : 1}

//=> "object"
```



下图是我们在`chrome`开发者工具中打印的结果，做一下对比

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/022814.png)



首先我们看到了第一个问题那就是`null`和引用类型同时都检测到是`"object"`这种类型,其实`null不是一个空引用`(有很多人认为null就是一个空引用)，而是一个原始值。



啰嗦一下: 其实这是一个js原生的`bug`，早在ES6版本的提案中就有人曾提到应该修复这个bug,但是被驳回了,原因是历史遗留的代码太多,喜欢的可以去看一下: [参考地址](https://link.zhihu.com/?target=http%3A//wiki.ecmascript.org/doku.php%3Fid%3Dharmony%253atypeof_null)。



当然了，即使是bug那我们也得避免，其实方法很简单也很多，下面举一个简单的例子:

```javascript
let a = {a : 1};

if(typeof a === 'object') {
   a !== null ? 'object' : 'null';
}
```



如果待检测的类型是数组、时间或者正则类型呢？我们同样只能拿到`object`。所以 `typeof`用于判断基本数据类型没有问题，但是判断引用数据类型时，就心有余而力不足了。

既然这样那我们就去试试第二种方法，接着往下看



#### 利用instanceof进行类型检测

我们用`instanceof`判断一下上面遗留下来的那几个问题`[1,2,3] ,new Date(), new RegExp(\^\W+$\)`，分别是Date，数组，正则:

```javascript
[1,2,3] instanceof Array

//=> true


new Date() instanceof Date

//=> true


new RegExp('\W+') instanceof RegExp

//=> true
```



看到这要是觉得事情仿佛都解决了，那你就高兴的太早了😏



如果我们需要判断一个未知的变量是什么类型用`instanceof`怎么实现？



```javascript
// 假设unknown是一个未知的变量，类型为数组
var unknown = [1,2,3]

// 我们只能通过繁琐的对比最后才能准确的定位到这是个Array
if(unknown  instanceof Array) {
  alert('Array');
}else if(unknown  instanceof Date) {
  alert('Date');
}....
```



上面的代码看着就头疼，总的来说`instanceof`只适用于校验变量是否是某一种类型，对于校验未知类型的变量并不是很适用。

你可能会问那就没有其他的方法了吗？往下看



#### 利用constructor进行类型检测

上面我们留下了`instanceof`对未知类型检验的问题，感觉`instanceof`不够灵活，那我们接着说另一种方法



```javascript
[1,2,3].constructor

//=> ƒ Array() { [native code] }


[1,2,3].constructor === Array

//=> true


(new Date()).constructor

//=> ƒ Date() { [native code] }


(new Date()).constructor === Date

//=> true
```



通过`constructor`方法我们能对未知的类型进行鉴定，然而有时候结果总是出人意料的，假设是这种情况呢？



```javascript
function Fun(){};
Fun.prototype = new Array(); 

let fun = new Fun();

fun.constructor === Fun

//=> false;

fun.constructor === Array

//=> true;
```

看一下在控制台输出的结果：

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/022813.png)

看来利用`constructor`去做类型检测还是有点缺陷，当出现被修改时，`constructor`得到的结果就是错误的。





#### Object.prototype.toString

javascript中`Object`有个`toString`方法，该方法返回一个表示该对象的字符串，但是很多对象(如:Array等)都重写了`toString`方法，所以需要以`call`或`apply`的形式来配合使用才最安全,不说了看栗子:



```javascript
Object.prototype.toString.call(null);

//=> "[object Null]"


Object.prototype.toString.call(undefined);

//=> "[object Undefined]"


Object.prototype.toString.call('aaa');

//=> "[object String]"


Object.prototype.toString.call(123);

//=> "[object Number]"


Object.prototype.toString.call(true);
//=> "[object Boolean]"


Object.prototype.toString.call([1,2,3]);

//=> "[object Array]"


Object.prototype.toString.call({a : 1});

//=> "[object Object]"


Object.prototype.toString.call(new Date());

//=> "[object Date]"


Object.prototype.toString.call(function(){});

//=> "[object Function]"
```

我们将`chrome`控制台输出的截图抬上来看看

![](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/022813-1.png)

上面的例子很容易说明`Object.prototype.toString.call()`是一个不错的基本数据类型和引用数据类型的一个方式。其实像`jQuery`中`$.type()`也是配合这种方式去实现的,最后我们将其源码拿过来瞅两眼



```javascript
jQuery.each(
  'Boolean Number String Function Array Date RegExp Object Error Symbol'.split(
    ''
  ),
  function(i, name) {
    class2type['[object ' + name + ']'] = name.toLowerCase()
  }
)

function toType(obj) {
  if (obj == null) {
    return obj + ''
  }

  // Support: Android <=2.3 only (functionish RegExp)
  return typeof obj === 'object' || typeof obj === 'function'
    ? class2type[toString.call(obj)] || 'object'
    : typeof obj
}
```



最后总结一下了：

* `typeof` 用来判断基本类型,而在引用类型方面显得无力。

* `instanceof` 用来判断引用类型的具体类型,但不够灵活。

* `constructor` 用来判断实例的构造函数,但容易被伪造。

* `Object.prototype.toString` 用来获取对象的描述字符串。



博客最近在搬迁，多多关照。**有什么问题或者建议大家可以通过留言或者邮箱的方式告诉我。**