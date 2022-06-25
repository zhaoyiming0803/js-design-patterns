# 代理模式

代理模式是为一个对象提供一个代用品或占位符，以便控制对它的访问。

代理模式的关键是，当客户`不方便直接`访问一个对象或者`不满足需要`的时候，提供一个替身对象来`控制`对这个对象的访问，客户实际上访问的是替身对象。替身对象对请求做出一些处理后，再把请求`转交`给本体对象。

- 虚拟代理：把一些开销很大的对象，延迟到真正需要它的时候才去创建。
- 保护代理：用于控制不同权限的对象对目标对象的访问，例如 JavaScript API `Object.defineProperty` 和 `Proxy`。

虚拟代理是最常用的一种代理模式。

## 虚拟代理实现图片预加载

``` javascript
const myImage = (function () {
  const imgNode = document.createElement('img')

  document.body.appendChild(imgNode)

  return {
    setSrc (src) {
      imgNode.src = src
    }
  }
})();

const proxyImage = (function () {
  const img = new Image()

  img.onload = function onload () {
    myImage.setSrc(this.src)
  }

  return {
    setSrc (src) {
      myImage.setSrc('cached loading image url')
      img.src = src
    }
  }
})();

proxyImage.setSrc('real newwork image url')
```

## 代理的意义

面向对象设计的原则 - `单一职责`

单一职责原则指的是，就一个类（通常也包括对象和函数等）而言，应该`仅有`一个引起它变化的原因。如果一个对象承担了多项职责，就意味着这个对象将变得巨大，引起它变化的原因可能会有多个。面向对象设计鼓励`将行为分布到细粒度的对象之中`，如果一个对象承担的职责过多，等于把这些职责耦合到了一起，这种耦合会导致脆弱和低内聚的设计，当变化发生时，设计可能会遭到意外的破坏。

职责被定义为`引起变化的原因`，一个对象或函数同时承担多个职责，则有可能因为其强耦合性影响另外一个职责的实现。

另外，在面向对象的程序设计中，大多数情况下，若违反其他任何原则，同时将违反`开放封闭`原则。

实际上，以上代码，我们需要的只是给 img 节点设置 src，预加载图片只是一个锦上添花的功能。如果把这个操作放在另一个对象里，自然是一个非常好的方法。于是代理的作用在这里就体现出来了，代理负责预加载图片，预加载的功能完成之后，把请求重新交给本体 myImage。

纵观整个程序，我们并没有改变或增加 myImage 的接口，但通过代理对象，实际上给系统添加了新的行为，`这是符合开放封闭原则的`。给 img 节点设置 src 和图片预加载这两个功能，被隔离在两个对象里，它们可以各自变化而不影响对方。何况就算有一天我们不再需要预加载，那么只需要改成请求本体而不是请求代理对象即可。

## 代理和本地接口的一致性

代理接手请求的过程对于用户来说是透明的，用户并不清楚代理的本体的区别，这样做有两个好处：

- 用户可以放心的请求代理，他值关心是否能得到想要的结果。
- 在任何使用本体的地方都可以替换成使用代理。

## 虚拟代理合并 HTTP 请求

``` javascript
function synchronousFile (ids) {
  console.log('start to sync file, ids: ', ids)
}

function proxySyncronousFile () {
  const cache = []
  let timer = null

  return function (id) {
    cache.push(id)

    if (timer) {
      return
    }

    timer = setTimeout(function () {
      synchronousFile(cache.join(','))
      clearTimeout(timer)
      timer = null
      cache.length = 0
    }, 1000)
  }
}
```

## 缓存代理

缓存代理可以为一些开销大的运算结果提供暂时的存储，在下次运算时，如果传递进来的参数跟之前一致，则可以直接返回前面存储的运算结果。

``` javascript
function sum (nums) {
  return nums.reduce((a, b) => a + b, 0)
}

function mult (nums) {
  return nums.reduce((a, b) => a * b, 1)
}

// 有点像策略模式
// 但这块代理模式重点是：不方便直接访问原对象（函数），前置加了缓存
// 如果是策略模式，确实需要重复计算，则不需要加缓存，比如表单校验，每次用户提交数据，都必须从头校验所有数据，因为用户可能会改变表单 value
function createProxyFactory (fn) {
  const cache = {}

  return function () {
    const args = [].slice.call(arguments)
    const key = args.join(',')

    return cache[key] || (cache[key] = fn.apply(null, args))
  }
}

const proxyMult = createProxyFactory(multi)
const proxySum = createProxySum(sum)

console.log(proxyMulti(1, 2, 3))
console.log(proxyMulti(1, 2, 3))
console.log(proxySum(1, 2, 3))
console.log(proxySum(1, 2, 3))
```

## 函数重写

``` javascript
const obj = {
  request (options) {
    const _options = proxyOptions(options)

    ajax(_options)
  }
}

function proxyOptions (options) {
  const success = options.success

  options.success = function success () {
    // ... custom handler
    success && success.apply(this, arguments)
  }
}
```

代理模式包括许多小分类，在 JavaScript 开发中最常用的是`虚拟代理`和`缓存搭理`。虽然代理模式非常有用，但我们在编写业务代码时，往往不需要去预先猜测是否需要使用代理模式，当真正发现不方便直接访问某个对象的时候，再编写代理也不迟。
