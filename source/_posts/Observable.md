title: 实现 Rxjs 中的 Observable
date: 2019-03-11 15:27:00
tags: JS
---
平时工作中很多地方用到了Rxjs，最基础的用法是使用Observable.create方法创建一个可订阅的对象，然后使用subscribe和unsubscribe，进行订阅和取消订阅的操作，通知到观察者observer。对这个用法，我简单地实现了一个自己的版本。

<!--more-->

### 1、Rxjs的简单用法。
```js
const Rx = require('rxjs')

const printer = Rx.Observable.create(observer => {
    let count = 0
    const timer = setInterval(() => {
        observer.next(`something-${count}`)
        count++
        if (count === 5) clearInterval(timer)
    }, 2000)
})

printer.subscribe({
    next: x => console.log(x)
})
```

上面这段代码在printer被订阅之后，会每隔2秒打印出1个推送过来的消息，一共5个。

```shell
something-0
something-1
something-2
something-3
something-4
```

### 2、代码实现V1。
根据上面的用法，我的自定义版本需要提供一个create方法和一个对应实例的subscribe方法，并实现观察者
observer的next方法。 

实现如下：
```js
const myObservable = (_ => {
    const create = func => ({
        subscribe: observer => {
            func(observer)
        }
    })
    return { create }
})()
```

利用了闭包的特性来实现，这样我们就可以使用类似Rxjs的方式来调用这段代码：
```js
const myPrinter = myObservable.create(observer => {
    let count = 0
    const timer = setInterval(() => {
        observer.next(`something-${count}`)
        count++
        if (count === 5) clearInterval(timer)
    }, 2000)
})

myPrinter.subscribe({
    next: x => console.log(x)
})
```

结果也是跟Rxjs的调用结果相同

```shell
something-0
something-1
something-2
something-3
something-4
```

### 3、 代码实现V2
如何实现unsubscribe, 首先需要在 myObservable 里面增加一个 unsubscribe 的方法，然后在触发观察者的回调之前判断是否已经被取消了订阅。

myObservable v2:

```js
const myObservable = (_ => {
    let startSub = false
    const create = func => ({
        subscribe: subObject => {
            startSub = true
            const observer = observerIn => ({
                next: x => startSub ? observerIn.next(x) : _
            })
            func(observer(subObject))
        },
        unsubscribe: _ => {
            startSub = false
        }
    })
    return { create }
})()
```
增加了一个 startSub 的变量，在 unsubscribe 之后， startSub 为false。 同时在调用 observer 的 next 前，增加对 startSub 的判断。

使用方式：

```js
myPrinter.subscribe({
    next: x => console.log(x)
})

setTimeout(() => {
    myPrinter.unsubscribe()
}, 7000);
```

结果只会打印出3个推送过来的消息，因为7秒的时候，取消了订阅，此后观察者就收不到推送消息了。
```shell
something-0
something-1
something-2
```

### 4、代码实现V3
增加异常处理：

```js
const myObservable = (_ => {
    let startSub = false
    const create = func => ({
        subscribe: subObject => {
            startSub = true
            const observer = observerIn => ({
                next: x => startSub ? observerIn.next(x) : _,
                error: x => startSub && observerIn.error ? observerIn.error(x): _
            })
            try {
                func(observer(subObject))
            } catch (error) {
                observer(subObject).error(error)
            }
        },
        unsubscribe: _ => {
            startSub = false
        }
    })
    return { create }
})()
```

对应的使用方法是：

```js
myPrinter.subscribe({
    next: x => console.log(x),
    error: x => console.log('print error', x)
})
```

这个时候，如果 create 的方法执行中遇到错误抛出，就可以从观察者这里得到错误信息

比如 create 时抛出一个异常。
```js
const myPrinter = myObservable.create(o => {
    throw new Error('外部方法出现错误')
})
```

然后在对应的 myPrinter.subscribe 方法中就会得到通知：

```
print error Error: 外部方法出现错误
    at myObservable.create.o (/Users/dingfan/xxx/xxx/test.js:71:11)
    at Object.subscribe (/Users/dingfan/xxx/xxx/test.js:58:17)
    at Object.<anonymous> (/Users/dingfan/xxx/xxx/test.js:80:11)
```

### 5、TODO
上面只是一个对 Rxjs Observable 的一个简单实现，而实际上这个框架有太多其他复杂的机制，observable 的实现远比这个复杂。
