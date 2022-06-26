# 迭代器模式

迭代器模式是指提供一种方法`顺序访问`一个`聚合对象`中的各个元素，而又不需要暴露该对象的内部表示。迭代器模式可以把`迭代的过程`从业务逻辑中分离出来，在使用迭代器模式之后，即使不关心对象的内部构造，也可以按顺序访问其中的每个元素。

## 实现自己的迭代器

``` javascript
function each (arr, callback) {
  for (let i = 0; i < arr.length; i++) {
    callback(i, arr[i])
  }
}

each([0, 1, 2, 3, 4], function callback (i, item) {
  console.log(i, item)
})
```

## 内部迭代器和外部迭代器

### 内部迭代器

以上 each 函数是一个简单的内部迭代器，each 函数的内部已经定义好了迭代规则，它完全接手整个迭代过程，外部只需要一次初始调用。

内部迭代器在调用的时候非常方便，外界不用关心迭代器的内部实现，跟迭代器的交互也仅仅是一次初始调用，但这也刚好是内部迭代器的`缺点`。由于其内部迭代规则已经被提前规定，以上 each 函数就无法同时迭代两个数组。

比如要实现一个需求：比较两个数组是否相同，只能写下面的 compare 函数

``` javascript
function compare (arr1, arr2) {
  if (arr1.length !== arr2.length) {
    return console.log('arr1 == arr2')
  }

  each(arr1, function callback (i, item) {
    if (item !== arr2[i]) {
      throw new Error('arr1 != arr2')
    }
  })

  console.log('arr1 == arr2')
}
```

以上实现非常不优雅。

### 外部迭代器

外部迭代器必须`显示的`请求迭代下一个元素

外部迭代器增加了一些调用的复杂度，但相对也增强了迭代器的灵活性，我们可以`手动控制`迭代的过程或者顺序。

``` javascript
function iterator (obj) {
  let current = 0

  const next = () => {
    current += 1
  }

  const isDone = () => current >= obj.length

  const getCurrentItem = () => obj[current]

  return {
    next,
    isDone,
    getCurrentItem,
    length: obj.length
  }
}
```

重写 compare 函数

``` javascript
function compare (arr1, arr2) {
  if (arr1.length !== arr2.length) {
    return false
  }

  while (!arr1.isDone() && !arr2.isDone()) {
    if (arr1.getCurrentItem() !== arr2.getCurrentItem()) {
      return false
    }
    arr1.next()
    arr2.next()
  }

  return true
}
```

外部迭代器的调用虽然相对复杂，但它的适用面更广，也能满足更多变的需求。`内部迭代器和外部迭代器在实际生产中没有优劣之分`，究竟使用哪个要根据需求场景而定。

## 迭代类数组对象和字面量对象

无论内部迭代器还是外部迭代器，只要迭代的`聚合对象`拥有 `length 属性`而且可以用`下标访问`，那它就可以被迭代。`for in` 语句可以用来迭代普通字面量对象的属性。

## 倒序迭代器

迭代器模式提供了循环访问一个聚合对象中每个元素的方法，但它没有规定我们以`顺序`、`倒序`还是`中序`来循环遍历聚合对象。

- Webpack 的 loader 迭代顺序就是倒序。

- 数组 reduceRight 迭代顺序也是倒序。

## 迭代器模式的应用

``` javascript
function addEventListener () {
  if (!window.addEventListener) {
    return false
  }

  return function (el, type, handler) {
    el.addEventListener(type, handler, false)
  }
}

function attachEvent () {
  if (!window.attachEvent) {
    return false
  }

  return function (el, type, handler) {
    el.attachEvent(`on${type}`, handler)
  }
}

function iterateEvents (events) {
  for (let i = 0; i < events.length; i++) {
    const event = events[i]()

    if (event) {
      return event
    }
  }
}

const addEvent = iterateEvents([addEventListener, attachEvent])
```

如果还有其他的事件类型，可以渐进增加，而不需要修改任何代码。

当然，迭代器模式在这里主要作用是消除了 if else 条件分支，让代码可维护、可扩展性更强。对于 addEvent 来说，使用`惰性载入函数方案`可能更合适。

``` javascript
function addEvent (el, type, handler) {
  if (window.addEventListener) {
    addEvent = (el, type, handler) => el.addEventListener(type, handler, false)
  } else if (window.attachEvent) {
    addEvent = (el, type, handler) => el.attachEvent(`on${type}`, handler)
  }

  // 仅首次使用
  addEvent(el, type, handler)
}
```
