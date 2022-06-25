# 策略模式

策略模式的定义是：定义一系列的算法（业务逻辑、配置），把它们一个个封装起来，并且使它们可以互相替换。

`在 JavaScript 语言的策略模式中，策略类往往被函数所代替。`

## 使用策略模式计算奖金

策略模式指的是定义一系列算法，把它们一个个封装起来，`将不变的部分和变化的部分隔开是每个设计模式的主题`，策略模式也不例外，`策略模式的目的就是将算法的使用与算法的实现分离开来`。

下面计算奖金的策略模式，其算法的使用方式是不变的，都是根据某个算法取得计算后的奖金数额，而算法的实现是各异和变化的，每种绩效对应着不同的计算规则。

``` javascript
function PerformanceS () {}

PerformanceS.prototype.calculate = function calculate (salary) {
  return salary * 4
}

function PerformanceA () {}

PerformanceA.prototype.calculate = function calculate (salary) {
  return salary * 3
}

function PerformaceB () {}

PerformanceB.prototype.calculate = function calculate (salary) {
  return salary * 2
}

function Bonus () {
  this.salary = null
  this.strategy = null
}

Bonus.prototype.setSalary = function setSalary (salary) {
  this.salary = salary
}

Bonus.prototype.setStratety = function setStratety (strategy) {
  this.strategy = strategy
}

Bonus.prototype.getBonus = function getBonus () {
  if (!this.stratety) {
    throw new Error('Please set strategy')
  }
  // 把计算奖金的操作【委托】给对应的策略对象
  return this.strategy.calculate(this.salary)
}
```

策略模式的定义，如果说的更详细一点，就是：`定义一系列的算法，把它们各自封装成策略类，算法是封装在策略类内部的方法里。在客户对 Context 发起请求的时候，Context 总是把请求【委托】给这些策略对象中的某一个进行计算`。

`并且使它们可以互相替换` 这句话很大程度是相对于静态语言类型的。因为静态类型语言中有类型检查机制，所以各个策略类需要实现同样的接口。当它们的真正类型被隐藏在接口后面时，它们才能被相互替换。而在 JavaScript 这种`类型模糊`的语言中没有这种困扰，任何对象都可以被替换使用。因此，JavaScript 中的`可以相互替换使用`表现为`它们具有相同的目标和意图`。

## 使用 JavaScript 版策略模式计算奖金

在 JavaScript 语言中，`函数也是对象`。

``` javascript
const strategies = {
  'S': function S (salary) {
    return salary * 4
  },
  'A': function A (salary) {
    return salary * 3
  },
  'B': function B (salary) {
    return salary * 2
  }
}

// 同样， Context 也没有必要用 Bonus 类来表示
// 我们依然用函数充当 Context 来接受用户的请求
function calculateBonus (level, salary) {
  return strategies[level](salary)
}
```

甚至不用对象，全部用函数实现：

``` javascript
function PerformanceS (salary) {
  return salary * 4
}

function PerformanceA (salary) {
  return salary * 3
}

function PerformanceB (salary) {
  return salary * 2
}

function calculateBonus (strategy, salary) {
  return strategy(salary)
}
```

## `多态`在策略模式中的体现

使用策略模式重构代码，我们消除了原程序中大片的条件分支语句。所有跟计算奖金有关的逻辑不再放在 Context 中，而是`分布在各个策略对象中`。Context 并没有计算奖金的能力，而是把这个职责委托给了某个策略对象。每个策略对象负责的算法已被各自封装在对象内部。当我们对这些策略对象发出`计算奖金`的请求时，它们会返回`各自不同的计算结果`。这正是`对象多态性`的体现，也是`它们可以相互替换`的目的。`替换 Context 中当前保存的策略对象，便能执行不同的算法来得到我们想要的结果`。`策略对象是可变的，Context 一定场景下可以是不变的`。

## 更广义的`算法`

策略模式指的是定义一系列的算法，并且把它们封装起来。从定义上看，策略模式就是用来封装算法的。但如果把策略模式仅仅用来封装算法，难免有点大材小用。在实际开发中，我们通常会把算法的含义扩散开来，使策略模式也可以用来封装一系列的`业务规则`。`只要这些业务规则指向的目标一致，并且可以被替换使用，我们就可以用策略模式来封装它们`。

## 使用策略模式实现表单校验

``` javascript
const strateties = {
  isEmpty (value, errorMessage) {
    if (value) {
      return errorMeesage
    }
  },
  minLength (value, length, errorMessage) {
    if (value.length < length) {
      return errorMessage
    }
  },
  isMobile (value, errorMessage) {
    if (!/^1[3,5,7,8][0-9]{9}$/.test(value)) {
      return errorMessage
    }
  }
}

function Validator (strateties) {
  this.strateties = strateties
  this.caches = []
}

Validator.prototype.add = function add (value, rule, errorMessage) {
  const arr = rule.split(':')
  const _this = this

  this.caches.push(function validateItem () {
    const strategy = arr.shift()

    arr.unshift(value)
    arr.push(errorMessage)

    return strategy.apply(_this.strateties, arr)
  })  
}

Validator.prototype.start = function start () {
  for (let i = 0; i < this.caches.length; i++) {
    const errorMessage = this.caches[i]()

    if (errorMessage) {
      return errorMessage
    }
  }
}
```

以上我们仅仅通过`配置`的方式就完成了一个表单的校验，这些校验规则也可以复用在程序任何地方，还能作为插件的形式，方便地被移植到其他项目中。

## 给表单项添加多种校验规则

strateties 对象不变，重写 `Validator.prototype.add` 函数即可。

``` javascript
Validator.prototype.add = function add (value, rules) {
  const _this = this

  rules.forEach(rule => {
    const arr = rule.strategy.split(':')

    this.caches.push(function validateItem () {
      const strategy = arr.shift()

      arr.unshift(value)
      arr.push(rule.errorMessage)

      return strategy.apply(_this.strateties, arr)
    })
  })
}

const validator = new Validator(strategies)

validator.add('zhangsan', [{
  strategy: 'isEmpty',
  errorMessage: '用户名不能为空'
}, {
  strategy: 'minLength:10',
  errorMessage: '用户名长度不能小于 10 位'
}])

const errorMessage = validator.start()

if (errorMessage) {
  // ...... 数据校验不合格，执行自定义逻辑
}
```

## 使用函数重构用于表单校验的策略模式

``` javascript
// 一系列策略函数
function isEmpty (value, errorMessage) {
  if (value) {
    return errorMeesage
  }
}

function minLength (value, length, errorMessage) {
  if (value.length < length) {
    return errorMessage
  }
}
  
function isMobile (value, errorMessage) {
  if (!/^1[3,5,7,8][0-9]{9}$/.test(value)) {
    return errorMessage
  }
}

// 接收用户请求并转发给对应的策略函数
const validators = []

function addValidator (value, rules) {
  rules.forEach(rule => {
    const arr = rule.strategy.split(':')

    validators.push(function validateItem () {
      const strategy = arr.shift()
      
      arr.unshift(value)
      arr.push(rule.errorMessage)

      // 策略函数都是纯函数，无需指定特定的 this
      return strategy.apply(null, arr)
    })
  })
}

function clearValidators () {
  validators.length = 0
}

function startValidate () {
  for (let i = 0; i < validators.length; i++) {
    const errorMessage = validators[i]()
    
    if (errorMessage) {
      clearValidators()()
      return errorMessage
    }
  }

  clearValidators()
}

function readyGo () {
  addValidator('zhangsan', [{
    strategy: 'isEmpty',
    errorMessage: '用户名不能为空'
  }, {
    strategy: 'minLength:10',
    errorMessage: '用户名长度不能小于 10 位'
  }])

  const errorMessage = startValidate()
  
  if (errorMessage) {
    // ... 自定义逻辑
  }
}
```

## 策略模式的优缺点

- 策略模式利用`组合`、`委托`和`多态性`等`技术和思想`，可以`有效避免多重条件和选择语句`。
- 策略模式提供了对`开放 - 封闭原则`的完美支持，将算法封装在独立的 strategy 中，使得它们易于切换、易于理解、易于扩展。
- 策略模式中的算法也可以复用在系统的其他地方，从而避免许多重复的复制粘贴工作。
- 在策略模式中利用`组合`和`委托`来让 Context 拥有执行算法的能力，这也是对`继承`的一种更轻便的替代方案。

当然，策略模式也有一些缺点，但并不严重：

- 使用策略模式需要在程序中添加许多策略类或策略对象，但实际上这比把它们负责的逻辑堆砌在 Context 中要更好。
- 使用策略模式必须先了解其拥有的 strategy，比较违反`最少知识原则`。
- 对前端而言，所有的策略类和策略对象代码都必须 build 到产物中供运行时使用，对包体积不友好。

## 一等函数对象与策略模式

在以类为中心的传统面向对象语言中，不同的算法或者行为被封装在各个策略类中，Context 将请求委托给这些策略对象，这些策略对象会根据请求返回不同的执行结果，这样便能表现出`对象的多态性`。

`在函数作为一等对象的语言中，策略模式是隐形的。strategy 就是【值作为函数的变量】`。在 JavaScript 中，除了使用类来封装算法和行为之外，使用函数也是一种选择。这些`算法`可以被封装到函数中并且四处传递，也就是我们常说的`高阶函数`。实际上在 JavaScript 这种将函数作为一等对象的语言里，策略模式已经融入到了语言本身当中，我们经常用高阶函数来封装不同的行为，并且把它传递到另一个函数中。当我们对这些函数发出`调用`的消息时，不同的函数会返回不同的执行结果。在 JavaScript 中，`函数对象的多态性`更加简单，策略类也往往被函数所代替，这时策略模式就成为一种`隐形`的模式。
