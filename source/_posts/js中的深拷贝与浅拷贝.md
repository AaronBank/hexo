---
title: js中的深拷贝与浅拷贝
date: 2019-07-31 13:43:49
tags: Javascript
categories: 前端
cover: http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/032418.jpg
---

![未标题-1](http://hexo-shen.oss-cn-beijing.aliyuncs.com/hexo/032418.jpg)



js 数据类型可分为两大类`基本数据类型`和`引用数据类型`。我们都知道`引用类型`的值是存放在`堆`里面的。

如果现在我们有一个对象A他的值想赋值给B，像这样：

```javascript
let A = {
    age: 24,
    name: '漫步者',
    detailed: {
        introduce: 'Do one thing at a time, and do well.  '
    }
}

let B = A

B.name = 'Aaron'

console.log(B.name) //=> 'Aaron'
console.log(A.name) //=> 'Aaron'
```



分别打印`A`和`B`结果，由于`A`的类型是`引用数据类型`，所以导致`B`和`A`指向同一个内存地址，所以你操作了`B`那么`A`也会跟随着改变。

但是问题来了我们并不想因为操作了`B`使得`A`也受到波及，怎么办。。。



对于基本类型，拷贝过程就是对值的复制，这个过程会开辟出一个新的内存空间，将值复制到新的内存空间。而对于引用类型来书，浅拷贝过程就是对指针的复制，这个过程并没有开辟新的堆内存空间，只是将指向该内存的地址进行了复制。然而对引用类型的拷贝会出现一个问题，那就是修改其中一个对象的属性，则另一个对象的属性也会改变。产生了问题那必然有相对解决的方法， 就这样深拷贝就开始入场了，深拷贝会开辟新的栈，两个对象对应两个不同的地址，这样一来，改一个对象的属性，也不会改变另一个对象的属性。

下面我们用代码看一下浅拷贝于深拷贝是如何实现的 :



### 浅拷贝实现方式

#### 1. for in

```javascript
let A = {
    age: 24,
    name: '漫步者',
    detailed: {
        introduce: 'Do one thing at a time, and do well.  '
    }
}

const extend = (initialObj) => {
    let result = {}

    for (let key in initialObj) result[key] = initialObj[key]
    
    return result
}

let B = extend(A)

B.name = 'Aaron'
console.log(B) //=> 'Aaron'
console.log(A) //=> '漫步者'
```



#### 2. Object.assign()

```javascript
let B = Object.assign({}, A)

B.name = 'Aaron'

console.log(B) //=> 'Aaron'
console.log(A) //=> '漫步者'
```



### 对象的深拷贝

如果我们把上面的`A`改成`obj`那浅拷贝就有心无力了 :

```javascript
let obj = {
    nNum : 1,
    oAar : [1,2,3,4]
};
```



我们不妨可以再用上面的方法试一下

```javascript
let newObj = extend(obj);

newObj === obj   //=> false
newObj.oAar === obj.oAar //=> true
```



从上面的结果显示我们可以看出，`newObj.oAar`与`obj.oAar`指针所指的是同一个位置,那我们如何改正这个问题呢？

其实也很容易,我们将`obj.oAar`再看待为一个新的对象,对其再进行一次浅拷贝不就可以了吗？？不说了,上代码

```javascript
function deepCopy (args) {
    let result = {};

    for (let item in args) {
        if (typeof args[item] === 'object' && args[item] !== null) {
            result[item] = deepCopy(args[item]);
        } else {
            result[item] = args[item];
        };
    };
    return result;
};

result(obj);
```



我们来验证一下

```javascript
let newObj = deepCopy(obj);

newObj === obj  //=> false
newObj.oAar === obj.oAar  //=> false
```



结果也显示我们实现了,ok

但是有没有其他的方式呢,比如说更简洁一些的,也是有的:

我们可以使用jQuery的`$.extend(true, {}, obj)`,同样也可以实现,是不是觉得眼熟。前面我们说浅拷贝的时候也说过这个方法,但是多了一个为true的参数,`$.extend()`方法,第一个参数默认为false,当设置它为true时则会开启深拷贝模式。



想想还有没有什么其他的方式=￣ω￣=?

其实还有一个捷径可走

还是上面举的那个obj,我们对它再进行一次尝试

```javascript
let obj = {
    nNum: 1,
    oAar: [1, 2, 3, 4]
};

function deepCopy (args) {
    let result = {};

    try {
        result = JSON.parse(JSON.stringify(args));
    } catch(error){
        return args
    }
    
    return result;
}

let newObj = deepCopy(obj);

newObj === obj //=> false
newObj.oAar === obj.oAar //=> false
```



ヾ(｡｀Д´｡) 什么鬼,这样也行? 那还前面说那么多废话,直接上这个方法岂不是爽歪歪？

但是这里不得不说明一点,投机取巧往往能达到看似相同的结果,但是还是存在一些隐性的问题的

就如上面这种方式和递归的方法相比是有弊端的,比如它会抛弃对象的`constructor`。也就是深拷贝之后，不管这个对象原来的构造函数是什么，在深拷贝之后都会变成`Object`。

这种方法能正确处理的对象只有 `Number、String、Boolean、Array` ，扁平对象，即那些能够被` json` 直接表示的数据结构。RegExp对象是无法通过这种方式深拷贝。

最后再说一种方法,可以使用`Object.create()`方法达到深拷贝的效果

```javascript
function deepCopy (args) {
    let result = {};

    for (let item in args) {
        if (typeof args[item] === 'object' && args[item] !== null) {
            result[item] = args[item].constructor === Array ? [] : Object.create(prop);
        } else {
            result[item] = args[item];
        };
    };

    return result;
}

let newObj = deepCopy(obj);

newObj === obj //=> false
newObj.oAar === obj.oAar //=> false
```



好了，更多的第三方库的方法我们就不一一列举了，原理更重要嘛，对吧。

最后我们来总结一下：



浅拷贝我们说了3种方法 ：

* 通过`for...in`循环来实现

* 通过`Object.assign()`方法来实现

* 通过`jQuery`中的`$.extend()`方法来实现



深拷贝我们同样说了四种方法 :

* 通过递归的手法实现

* 通过`jQuery`中的`$.extend()`添加`true`参数方法来实现

* 通过`JSON`序列化反序列化来实现

* 用过`Object.create()`方法来实现



**除了这几种如果各位还想到了其他什么好的方法欢迎下方留言ヾ(￣▽￣)**