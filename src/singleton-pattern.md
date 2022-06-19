# 单例模式

单例模式的定义是：保证一个类仅有一个实例，并提供一个访问它的全局访问点。

单例模式是一种常见的模式，有一些对象我们往往只需要一个，比如线程池、全局缓存、浏览器中的 window 对象等。在 JavaScript 开发中，单例模式的用途同样非常广泛。试想一下，当我们点击登录按钮的时候，页面中会出现一个登录浮窗，而这个登录浮窗是唯一的，无论单击多少次登录按钮，这个浮窗都只会被创建一次，那么这个登录浮窗就适合用单例模式来创建。

## 用代理实现单例模式

``` javascript
function CreateDiv (html) {
  this.html = html
  this.init()
}

CreateDiv.prototype.init = function init () {
  const div = document.createElement('div')
  div.innerHTML = this.html
  document.body.appendChild(div)
}

const ProxySingletonCreateDiv = (function () {
  let instance
  return function SingletonCreateDiv(html) {
    if (!instance) {
      instance = new CreateDiv(html)
    }
    return instance
  }
})();

const a = new ProxySingletonCreateDiv('zhangsan')
const b = new ProxySingletonCreateDiv('lisi')
console.log(a === b) // true
```

通过引入代理类的方式，我们完成了一个单例模式的编写。我们把负责管理单例的逻辑放到了代理类 ProxySingletonCreateDiv 中，这样一来，CreateDiv 就变成了一个普通类，它跟 ProxySingletonCreateDiv 组合起来可以达到单例模式的效果。

除了使用代理，也可以将 `instance` 挂载到 CreateDiv 上作为判断条件，实现方式有很多，具体利弊具体分析。

## JavaScript 中的单例模式

JavaScript 其实是一门无类（class-free）语言，正因如此，生搬 Java 等面向对象语言的单例模式概念并无意义。在 JavaScript 中创建对象的方法非常简单，既然我们只需要一个`唯一`的对象，为什么要先为它创建一个`类`呢？传统的单例模式实现在 JavaScript 中并不适用。

单例模式的`核心`是确保只有一个实例，并提供的全局的访问。

Brendan Eich 本人也承认全局变量是设计上的失误。

作为普通开发者，我们有必要尽量减少全局变量的使用，即使需要，也要把它的污染降到最低。

- 使用命名空间
- 使用闭包封装私有变量

## 惰性单例

惰性单例是在需要的时候才创建对象实例，惰性单例是单例模式的重点，这种技术在实际开发中非常有用。`惰性代表不确定的执行时机`。

``` javascript
function getSingle (fn) {
  let result
  return function () {
    return result || (result = fn.apply(this, arguments))
  }
}

function createLoginLayer () {
  const div = document.createElement('div')
  div.innerHTML = '我是登录浮窗'
  div.style.display = 'none'
  document.body.appendChild(div)
  return div
}

const createSingleLoginLayer = getSingle(createLoginLayer)

document.querySelector('#login-btn').onclick = function () {
  const loginLayer = createSingleLoginLayer()
  loginLayer.style.display = 'block'
}

// 再试试创建唯一的 iframe 用于动态加载第三方页面

function createIframe () {
  const iframe = document.createElement('iframe')
  document.body.appendChild(iframe)
  return iframe
}

const createSingleIframe = getSingle(createIframe)

document.querySelector('#login-btn').onclick = function () {
  const loginLayer = createSingleIframe()
  loginLayer.src = 'https://github.com'
}
```

以上代码，我们把创建实例对象的职责和管理单例的职责分别放置在两个方法里，这两个方法可以独立变化而互不影响，当他们组合在一起的时候，就完成了创建唯一实例对象的功能。

这种单例模式的用途远不止创建实例对象，比如我们通常渲染完页面中的一个列表之后，接下来要给这个列表绑定 click 事件，如果是通过 ajax 动态往列表里追加数据，在使用事件代理的前提下，click 事件实际上只需要在第一次渲染完列表的时候被绑定一次，但是我们不想去判断当前是否第一次渲染列表，如果借助于 jQuery，我们通常选择给节点绑定 one 事件。

如果是使用 getSingle 函数，也能达到一样的效果：

``` javascript
const bindEvent = getSingle(function () {
  document.querySelector('div1').addEventListener('click', function () {
    console.log('click')
  }, false)
  // 不要忘了 return 一个真值
  return true
})

function render () {
  console.log('render')
  bindEvent()
}

render()
render()
render()
```

以上代码，render 函数和 bindEvent 函数都分别执行了 3 次，但 div 实际上只绑定了一次事件。
