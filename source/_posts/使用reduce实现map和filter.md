title: 使用reduce实现map和filter
date: 2017-04-16 04:10:03
tags: swift, 编程, 闭包
---
其实要拍脑袋直接写好像也不难。
但是还是尝试一步步分解地写下来，这样的话问题就更加有普遍性。
因为并不是所有问题都能一步到位地解决，重要的可能不是问题本身，而是解决问题的步骤和方式，这才是需要精深练习的地方。
<!--more-->

### Map

#### 1、先写测试用例
假设我自己实现的map函数名为ptxMap，假如ptxMap的结果和sdk实现的map方法返回结果是相同的，就可以认为成功啦。

testcase1:
```js
let testArray = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

let resultA = testArray.map() { $0 * 2 }
let resultB = testArray.ptxMap() { $0 * 2 }
assert(resultA == resultB, "two results should equal")
//[2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
//testcase1 fail
```

#### 2、根据测试用例编写第一个版本的代码
第一个可能觉得生涩的地方，是闭包作为参数传入。
[关于闭包的语法介绍](http://fuckingclosuresyntax.com)

其实可以认为闭包是一个匿名函数（虽然实际上在swift中Functions are a special case of closures）
然后本身作为高阶函数，map就是接收一个处理函数。

所以在定义ptxMap时，会传入一个dealFunc的闭包。
因为题目是用reduce来实现map，所以在方法体里调用Array的reduce方法，而reduce的处理函数在这里只是简单做一个往原始数组里面添加元素的操作。

第二个生涩的地方应该是swift的具体语法了，如通过self来调用reduce，还有尾部闭包的语法，然后因为Array本身支持泛型，所以append操作的时候需要转类型。

这样，第一个版本的ptxMap就完成了，测试用例1也能满足。

version one: 
```js
extension Array {
    func ptxMap(_ dealFunc: (Int) -> Int) -> [Int] {
        return self.reduce([]) { (result, element) -> [Int] in
            var tempResult = result
            tempResult.append(dealFunc(element as! Int))
            return tempResult
        }
    }
}
//testcase1 succeed
```

#### 3、添加测试用例，测试可能会失败

平时写代码的时候，虽然先写了一些测试场景，并且也通过了。
但是写着写着，可能就想到新的测试用例了。
testcase1，输出为[Int]类型，所以ptxMap的返回类型也被写死了。
所以这个时候会想到假如输出的类型是由处理闭包决定的呢？
比如下面的testcase2，闭包返回String，ptxMap的返回类型应该为[String]，此时testcase2肯定不能通过测试，那就改代码呗。

testcase2:
```js
let resultA = testArray.map() { "\($0)" }
let resultB = testArray.ptxMap() { "\($0)" }
assert(resultA == resultB, "two results should equal")
//["1", "2", "3", "4", "5", "6", "7", "8", "9", "10"]
//testcase2 fail
```

#### 4、改进程序

既然要满足不同类型的返回，那么有两个方法：
1、函数重载。函数名相同的情况下，参数类型不同。
2、泛型。
如果选择第一种方法，那么返回类型就是有限的，而选择第二种方法，则能支持 满足当前泛型的类型约束的 所有类型
所以第二个版本是支持泛型的版本。

在ptxMap函数名后声明一个即将要用到的泛型类型T，代码如下。

version two:
```js
extension Array {
    public func ptxMap<T>(_ dealFunc: (Element) -> T) -> [T] {
        return self.reduce([]) { (result, element) -> [T] in
            var tempResult = result
            tempResult.append(dealFunc(element))
            return tempResult
        }
    }
}
//testcase1 succeed
//testcase2 succeed
```
这样，testcase2也满足了，其实这里的testcase并不完整，还应该添加一些边界的testcase，像空数组调用等。
好了，map的实现大概就这样了。



#### 延伸，闭包的调用方式

在我们这个例子里，因为ptxMap的参数只有一个，并且为闭包，所以就使用下面这样方式调用，这也是比较建议的调用方式。
而后面这几种，则在这个例子中不太推荐。
其他情况参考[raywenderlich swift-style-guide](https://github.com/raywenderlich/swift-style-guide#closure-expressions)

Preferred:
```js
testArray.ptxMap { $0 * 2 } 
```

Not Preferred:
```js
testArray.ptxMap() { $0 * 2 } 
testArray.ptxMap({ (value) -> Int in
    return value * 2
})
testArray.ptxMap({ $0 * 2 })
```

### Filter和完整代码

下面的代码实现了ptxFilter，因为filter并不涉及到输出类型的变化，所以并没有用到泛型。
下面是完整的代码，测试用例使用了do{}的方式，目的是隔离每个测试用例。

```js
import Foundation

let testArray = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

extension Array {
    public func ptxMap<T>(_ dealFunc: (Element) -> T) -> [T] {
        return self.reduce([]) { (result, element) -> [T] in
            var tempResult = result
            tempResult.append(dealFunc(element))
            return tempResult
        }
    }
    
    public func ptxFilter(_ dealFunc: (Element) -> Bool) -> [Element] {
        return self.reduce([]) { (result, element) -> [Element] in
            var tempResult = result
            if dealFunc(element) {
                tempResult.append(element)
            }
            return tempResult
        }
    }
}

do {
    let resultA = testArray.map { $0 * 2 }
    let resultB = testArray.ptxMap { $0 * 2 }
    assert(resultA == resultB, "two results should equal")
    print(resultB)
}

do {
    let resultA = testArray.map { "\($0)" }
    let resultB = testArray.ptxMap { "\($0)" }
    assert(resultA == resultB, "two results should equal")
    print(resultB)
}

do {
    let resultA = testArray.filter { $0 % 2 == 0 }
    let resultB = testArray.ptxFilter { $0 % 2 == 0 }
    assert(resultA == resultB, "two results should equal")
    print(resultB)
}
```

### 总结
额，为啥要蛋疼写总结呢
首先，在这里栽过跟头，然后，以后所有的学习结果都会用总结的方式来梳理一遍。

最近还在刷算法和数据结构，额，有一些主题大学的时候并没有涉及到，这次也算还回来了，后面再梳理下。
其实解题的时候方法也挺重要的，先明确，然后思考当前的所有方案，选择复杂度相对好的方案，实现，然后一步步优化。
这跟写业务的过程区别不大，所以，有时候需要注重解决问题的方法，因为这不仅仅适用于当前的问题。