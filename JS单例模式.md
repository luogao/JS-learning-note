## 为何要有单例模式

书中有举出一个实际场景，当我们点击登陆按钮时，页面中可能会出现一个弹框，而这个弹框是唯一的，无论点多少次登陆按钮，弹框只会被创建一次，那么这种情况下就适合用单例模式来创建弹框。

### 实现一个简单的单例模式

以下代码来自书中

```javascript
var CreateDiv = (function(html) {
    var instance
    var CreateDiv = function() {
        if (instance) {
            return instance
        }
        this.html = html
        this.init()
        return instance = this
    }
    CreateDiv.prototype.init = function() {
        var div = document.createElement('div')
        div.innerHTML = this.html
        document.appendChild(div)
    }
    return CreateDiv
})()
```

> 以上代码通过自执行函数和闭包将instance封装起来。并且返回了真正的`Singleton`构造方法。

通过观察上面代码发现`CreateDiv`里执行了两个操作：

- 1.创建对象并且执行`init`方法。
- 2.保证只有一个对象。这里就暴露出一个问题。

如果某天我们需要用这个方法向页面中创建更多的元素。那我们必须要改写`CreateDiv`，如果我们结合“**单一职责原则**”，我们就知道要去把保证只有一个对象这个操作从`CreateDiv`抽离出来。这个目的可以通过代理来实现。

## 用代理实现单例模式

首先我们把上面代码中的`CreateDiv`方法改写成一个只负责创建DIV的类

```javascript
var CreateDiv = function(html) {
    this.html = html
    this.init()
}

CreateDiv.prototype.init = function() {
    var div = document.createElement('div')
    div.innerHTML = this.html
    document.appendChild(div)
}
```

接下来引入代理类

```javascript
var ProxysingletonCreateDiv = (function() {
    var instance
    return function(html) {
        if (!instance) {
            instance = new CreateDiv(html)
        }
        return instance
    }
})()
var a = new ProxysingletonCreateDiv('test1')
var b = new ProxysingletonCreateDiv('test2')

alert(a === b) // true
```

至此利用代理类也实现了一个单例模式。但目前我们讨论的单例模式跟接近传统面向对象语言中的实现。接下来我们来了解一下JavaScript中的单例模式。

## JavaScript中的单例模式——惰性单例

了解了单例模式的一些实现方法之后。我们可以来看看惰性单例的实现，这种实现方式在JavaScript的实际编程中是很实用的。

### 惰性单例

惰性单例是指在需要的时候才创建对象实例，而不是像之前的代码那样，利用自执行函数在代码执行时就把对象实例创建。

比如最开始就提到，当打开一个网站时，需要登录，但登陆的弹窗只会在点击登陆按钮时出现，甚至有的网站不需要登录就能直接浏览。这时我们并不需要在页面加载时就去创建一个弹窗。我们大可在需要用的时候去创建。

```html
<html>
    <body>
        <button id="loginBtn">登录</button>
    </body>
    <script>
        var createLoginLayer = (function() {
            var div
            return function() {
                if (!div) {
                    var div = document.createElement('div')
                    div.innerHTML = '我是登录弹窗'
                    div.style.display = 'none'
                    document.appendChild(div)
                }
                return div
            }
        })
        document.getElementById('loginBtn').onclick = function() {
            var loginLayer = createLoginLayer()
            loginLayer.style.display = 'block'
        }
    </script>
</html>
```

以上我们实现了一个单例模式的弹窗。但是我们还是可以把其中的控制只有一个对象的操作抽离出来，让我们来实现一个通用的惰性单例。

### 通用惰性单例

通用惰性单例的实现就是要抽离所有单例模式都要实现的——控制只有一个对象。那么我们来看看控制只有一个对象的操作抽象出来是个什么样子：

```javascript
var obj 
if (!obj) {
    obj = xxx
}
```

于是就可以把这个操作的逻辑封装到一个`getSingle`函数中，然后把要执行的函数当作参数传入进去：

```javascript
var getSingle = function(fn) {
    var result
    return function() {
        result || (result = fn.apply(this, arguments))
    }
}
```

这样我们上面写的创建弹窗的方法就可以完全抽离出来：

```javascript
var createLoginLayer = function() {
    var div = document.createElement('div')
    div.innerHTML = '我是登录弹窗'
    div.style.display = 'none'
    document.appendChild(div)
    return div
}

var createSingleLoginLayer = getsingle(createLoginLayer)

document.getElementById('loginBtn').onclick = function() {
    var loginLayer = createSingleLoginLayer()
    loginLayer.style.display = 'block'
}
```

至此我们实现了一个`getSingle`函数来帮我们实现只有一个实例对象的目的，并且将实例对象要做的指责独立出来，两个方法互不打扰。

## 最后

感谢阅读。

博客地址：https://www.lglzy.cn/blog

邮箱：luogao_lg@sina.com