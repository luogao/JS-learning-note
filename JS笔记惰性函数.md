JS 笔记 惰性函数

## 最前

**感谢大佬[冴羽](https://github.com/mqyqingfeng)的[《JavaScript专题之惰性函数》](https://github.com/mqyqingfeng/Blog/issues/44)**

ps:此笔记内容基本于大佬分享的专题一样，仅供个人记录笔记
**侵删**



## 需求&分析

> 编写一个函数，这个函数返回的是**首次**调用的Date对象；或者是，为了保证代码的兼容性，在给Dom绑定事件时，需有对当前浏览器进行一次判断。

理一下思路：

- **首次**
- 为保证性能，只进行一次判断




## 实现

这里先来实现 “*编写一个函数，这个函数返回的是**首次**调用的Date对象*”为例

为了保证首次，需要一个变量来保持首次调用是的结果：

``` javascript
var foo = (function(){
    var t;
    return function(){
        if (t) return t;
        t = new Date();
    }
})()
```

这个解决方案，利用闭包避免全局变量污染，但每次调用时还是判断了一遍是否存在首次调用的结果。

于是便引出主角**惰性函数**：

直接上代码

``` javascript
var foo = function () {
    var t = new Date();
    foo = function(){
        return t;
    }
    return foo()
}
```

这里自我理解一下实现的方法：这里直接将`foo`重写成一个返回`t`的函数，并且在函数中直接调用`foo`方法。

因此，首次调用`foo`函数便会返回`t`并且`foo`被重写



## 应用场景

上文提到“*为了保证代码的兼容性，在给Dom绑定事件时，需有对当前浏览器进行一次判断*”

实现代码：

``` javascript
function addEvent(type, el, fn){
    if(window.addEventListener){
        addEvent = function(type, el, fn){
            el.addEventListener(type,fn,false)
        }
    }else if(window.attachEvent){
        addEvent = function(type, el, fn){
            el.attachEvent('on'+type, fn)
        }
    }
    addEvent(type, el, fn)
}
```



## 最后

至此惰性函数笔记完成。



## 参考资料

[《JavaScript专题之惰性函数》](https://github.com/mqyqingfeng/Blog/issues/44)

《JavaScript设计模式与开发实践》

