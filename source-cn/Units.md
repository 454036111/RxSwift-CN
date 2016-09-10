Units
=====

This document will try to describe what units are, why they are a useful concept, and how to use and create them.
这个文档将尝试描述 units 是什么，为什么他们是有用的理念，并且如果使用和创建他们。

* [Why](#why)
* [How they work](#how-they-work)
* [Why they are named Units](#why-they-are-named-units)
* [RxCocoa units](#rxcocoa-units)
* [Driver unit](#driver-unit)
    * [Why it's named Driver](#why-its-named-driver)
    * [Practical usage example](#practical-usage-example)

## Why

Swift has a powerful type system that can be used to improve correctness and stability of applications and make using Rx a more intuitive and straightforward experience.
Swift 拥有一个强大的类型系统，这能用来提高正确性和应用的稳定性，并且使使用 Rx 成为更直观和直接的经历。

**Units are specific only to the [RxCocoa](https://github.com/ReactiveX/RxSwift/tree/master/RxCocoa) project. However, the same principles could easily be implemented in other Rx implementations, if necessary. There is no private API magic needed.**
**Units 仅在 [RxCocoa](https://github.com/ReactiveX/RxSwift/tree/master/RxCocoa) 项目中。然而如果有需要，同样的原理可以很轻松的被实现在其他 Rx 的实现中。这里没有私有API的魔法需要**

**Units are totally optional. You can use raw observable sequences everywhere in your program and all RxCocoa APIs work with observable sequences.**
**Units 是完全可选的。你可以在你程序的任何地方使用原始的 observable 序列，并且所有 RxCocoa 的接口能和 observable 序列兼容使用**

Units also help communicate and ensure observable sequence properties across interface boundaries.
Units 还帮助交流和确认 observable 序列属性穿越接口边界。

Here are some of the properties that are important when writing Cocoa/UIKit applications.
当我们在写 Cocoa/UIKit 时，这有一些重要的属性。

* Can't error out
* Observe on main scheduler
* Subscribe on main scheduler
* Sharing side effects
* 不能错误退出
* 在主调度器上观察
* 在主调度器上订阅
* 分享副作用

## How they work

At its core, it's just a struct with a reference to observable sequence.
在它的核心中，它只是一个引用 observable 序列的结构。

You can think of them as a kind of builder pattern for observable sequences. When a sequence is built, calling `.asObservable()` will transform a unit into a vanilla observable sequence.
你可以把他们想成一种 observable 序列的构造模型。当一个序列被构造，调用 `.asObservable()` 将转换一个 unit 成为一个 observable 序列。

## Why they are named Units

Using a couple analogies will help us reason about unfamiliar concepts. Here are some analogies showing how units in physics and RxCocoa (Rx units) are similar.
使用一对类比将会帮助我们解释不常见的理念。下面是一些类比告诉我们 units 在物理世界和 RxCocoa（Rx units）之间的相似点。

Analogies:

| Physical units                      | Rx units                                                            |
|-------------------------------------|---------------------------------------------------------------------|
| number (one value)                  | observable sequence (sequence of values)                            |
| dimensional unit (m, s, m/s, N ...) | Swift struct (Driver, ControlProperty, ControlEvent, Variable, ...) |

| 物理 units                      | Rx units                                                            |
|-------------------------------------|---------------------------------------------------------------------|
| 数字 (一个值)                  | observable 序列 (序列的值)                            |
| 空间单位 (m, s, m/s, N ...) | Swift 结构体 (Driver, ControlProperty, ControlEvent, Variable, ...) |

A physical unit is a pair of a number and a corresponding dimensional unit.<br/>
An Rx unit is a pair of an observable sequence and a corresponding struct that describes observable sequence properties.
一个物理单位是一对数字和其匹配的空间单位。<br/>
一个Rx单位（unit）是一对一个 observable 序列和其匹配的描述 observable 序列属性的结构体。

Numbers are the basic compositional glue when working with physical units: usually real or complex numbers.<br/>
Observable sequences are the basic compositional glue when working with Rx units.
当使用物理单位时，数字是一个基本的组成胶水：比如实数和复数。<br/>
当我们使用 Rx 单位时，Observable 序列是一个基本的组成胶水。

Physical units and [dimensional analysis](https://en.wikipedia.org/wiki/Dimensional_analysis#Checking_equations_that_involve_dimensions) can alleviate certain classes of errors during complex calculations.<br/>
Type checking Rx units can alleviate certain classes of logic errors when writing reactive programs.
物理单位和 [dimensional analysis](https://en.wikipedia.org/wiki/Dimensional_analysis#Checking_equations_that_involve_dimensions) 可以在复杂计算中减缓一些错误级别。

Numbers have operators: `+`, `-`, `*`, `/`.<br/>
Observable sequences also have operators: `map`, `filter`, `flatMap` ...
数字有一些操作符: `+`, `-`, `*`, `/`.<br/>
Observable 序列也有一些操作符: `map`, `filter`, `flatMap` ...

Physics units define operations by using corresponding number operations. E.g.
物理单位通过使用相应的数字操作定义操作符，比如：

`/` operation on physical units is defined using `/` operation on numbers.
`/` 操作在物理单位被定义为使用 `/` 操作数字。

11 m / 0.5 s = ...
* First, convert the unit to **numbers** and **apply** `/` **operator** `11 / 0.5 = 22`
* Then, calculate the unit (m / s)
* Lastly, combine the result = 22 m / s
* 首先，转换到**数字**单位，并且**应用** `/` **操作符** `11 / 0.5 = 22`
* 然后，计算单位（m / s）
* 最后，组合结果 = 22 m / s

Rx units define operations by using corresponding observable sequence operations (this is how operators work internally). E.g.
Rx 单位通过使用一致的 observable 序列操作符定义操作（这就是内部操作符如何运行的），例如

The `map` operation on `Driver` is defined using the `map` operation on its observable sequence.
`Driver` 上的 `map` 操作符被定义为在其 observable 序列上使用  `map` 操作符。

```swift
let d: Driver<Int> = Drive.just(11)
driver.map { $0 / 0.5 } = ...
```

* First, convert `Driver` to **observable sequence** and **apply** `map` **operator**
* 首先，转换 `Driver` 为 **observable 序列** 并且 **使用** `map` **操作符**
```swift
let mapped = driver.asObservable().map { $0 / 0.5 } // this `map` is defined on observable sequence
```

* Then, combine that to get the unit value
* 然后，组合结果来获得单位值
```swift
let result = Driver(mapped)
```

There is a set of basic units in physics [(`m`, `kg`, `s`, `A`, `K`, `cd`, `mol`)](https://en.wikipedia.org/wiki/SI_base_unit) that is orthogonal.<br/>
There is a set of basic interesting properties for observable sequences in `RxCocoa` that is orthogonal.

    * Can't error out
    * Observe on main scheduler
    * Subscribe on main scheduler
    * Sharing side effects

在物理中有一系列基本单位[(`m`, `kg`, `s`, `A`, `K`, `cd`, `mol`)](https://en.wikipedia.org/wiki/SI_base_unit)是正交的。<br/>
在 `RxCocoa` 中有一系列有趣的 observable 序列基础属性是正交的。
	
	* 不能错误退出
	* 在主调度器上观察
	* 订阅在主调度器上
	* 分享副作用

Derived units in physics sometimes have special names.<br/>
E.g.
```
N (Newton) = kg * m / s / s
C (Coulomb) = A * s
T (Tesla) = kg / A / s / s
```
物理中派生的单位有时有专有的名称。<br/>
例如：
```
N (牛顿) = kg * m / s / s
C (库仑) = A * s
T (特斯拉) = kg / A / s / s
```

Rx derived units also have special names.<br/>
E.g.
```
Driver = (can't error out) * (observe on main scheduler) * (sharing side effects)
ControlProperty = (sharing side effects) * (subscribe on main scheduler)
Variable = (can't error out) * (sharing side effects)
```

Rx 派生的单位也有专有的名称。<br/>
例如：
```
Driver = (不能错误退出) * (在主调度器上观察) * (分享副作用)
ControlProperty = (分享副作用) * (订阅在主调度器上)
Variable = (不能错误退出) * (分享副作用)
```

Conversion between different units in physics is done with the help of operators defined on numbers `*`, `/`.<br/>
Conversion between different Rx units in done with the help of observable sequence operators.

E.g.

```
Can't error out = catchError
Observe on main scheduler = observeOn(MainScheduler.instance)
Subscribe on main scheduler = subscribeOn(MainScheduler.instance)
Sharing side effects = share* (one of the `share` operators)
```
物理中转换不同的单位通过在数学上的操作符定义的 `*`, `/`。<br/>
Rx 中转换不同的单位通过 observable 序列操作符的帮助。

例如：
```
不能错误退出 = catchError
在主调度器上观察 = observeOn(MainScheduler.instance)
订阅在主调度器上 = subscribeOn(MainScheduler.instance)
分享副作用 = share* (one of the `share` operators)
```

## RxCocoa units

### Driver unit 驱动单元

* Can't error out
* Observe on main scheduler
* Sharing side effects (`shareReplayLatestWhileConnected`)
* 不能错误退出
* 在主调度器上观察
* 分享副作用 (`shareReplayLatestWhileConnected`)

### ControlProperty / ControlEvent  控制属性 / 控制事件

* Can't error out
* Subscribe on main scheduler
* Observe on main scheduler
* Sharing side effects
* 不能错误退出
* 在主调度器上订阅
* 在主调度器上观察
* 分享副作用

### Variable  变量

* Can't error out
* Sharing side effects
* 不能错误退出
* 分享副作用

## Driver  驱动

This is the most elaborate unit. Its intention is to provide an intuitive way to write reactive code in the UI layer.
这是非常精心计划的单元。它的目的是提供一个直接的方式去写 UI 层的响应式代码。

### Why it's named Driver  为什么被命名为驱动（Driver）

Its intended use case was to model sequences that drive your application.
它的目标使用场景是模型化驱动你应用的序列。

E.g.
* Drive UI from CoreData model
* Drive UI using values from other UI elements (bindings)
...

例如：
* 从 CoreData 模型驱动 UI
* 使用从其他 UI 元素的值驱动 UI （bindings）
...

Like normal operating system drivers, in case a sequence errors out, your application will stop responding to user input.

类似普通的操作系统驱动，假如一个序列错误退出， 你的应用将停止用户输入的相应。

It is also extremely important that those elements are observed on the main thread because UI elements and application logic are usually not thread safe.

那些被观察在主线程的元素还非常的重要因为 UI 元素和应用逻辑通常是线程不安全的。

Also, `Driver` unit builds an observable sequence that shares side effects.

另外， `驱动(Drive)` 单元构建了一个 observable 分享副作用的序列。

E.g.


### Practical usage example  实际用例

This is a typical beginner example.
这是一个典型的开始例子。

```swift
let results = query.rx_text
    .throttle(0.3, scheduler: MainScheduler.instance)
    .flatMapLatest { query in
        fetchAutoCompleteItems(query)
    }

results
    .map { "\($0.count)" }
    .bindTo(resultCount.rx_text)
    .addDisposableTo(disposeBag)

results
    .bindTo(resultsTableView.rx_itemsWithCellIdentifier("Cell")) { (_, result, cell) in
        cell.textLabel?.text = "\(result)"
    }
    .addDisposableTo(disposeBag)
```

The intended behavior of this code was to:

* Throttle user input
* Contact server and fetch a list of user results (once per query)
* Bind the results to two UI elements: results table view and a label that displays the number of results

这个代码的意图是：

* Throttle 用户输入
* 联系服务器并且获取一个用户列表（每一个请求）
* 绑定结果到两个 UI 元素：显示结果到 tableView 和显示有多少条记录的标签。

So, what are the problems with this code?:

* If the `fetchAutoCompleteItems` observable sequence errors out (connection failed or parsing error), this error would unbind everything and the UI wouldn't respond any more to new queries.
* If `fetchAutoCompleteItems` returns results on some background thread, results would be bound to UI elements from a background thread which could cause non-deterministic crashes.
* Results are bound to two UI elements, which means that for each user query, two HTTP requests would be made, one for each UI element, which is not the intended behavior.

那么，这个代码有什么问题？

* 如果 `fetchAutoCompleteItems` observable 序列错误退出（连接失败或者处理错误），这个错误将会解绑一切并且 UI 不会响应更多的请求。
* 如果 `fetchAutoCompleteItems` 在一些后台线程返回结果，结果会从后台线程被绑定到 UI 元素，而这会引起不确定的奔溃。
* 结果被绑定到两个 UI 元素，这意味着每个用户查询会制造两个HTTP请求对应两个 UI 元素，这并不是预期的行为。

A more appropriate version of the code would look like this:

更合适的版本的代码应该看起来像如下：

```swift
let results = query.rx_text
    .throttle(0.3, scheduler: MainScheduler.instance)
    .flatMapLatest { query in
        fetchAutoCompleteItems(query)
            .observeOn(MainScheduler.instance)  // results are returned on MainScheduler
            .catchErrorJustReturn([])           // in the worst case, errors are handled
    }
    .shareReplay(1)                             // HTTP requests are shared and results replayed
                                                // to all UI elements

results
    .map { "\($0.count)" }
    .bindTo(resultCount.rx_text)
    .addDisposableTo(disposeBag)

results
    .bindTo(resultTableView.rx_itemsWithCellIdentifier("Cell")) { (_, result, cell) in
        cell.textLabel?.text = "\(result)"
    }
    .addDisposableTo(disposeBag)
```

Making sure all of these requirements are properly handled in large systems can be challenging, but there is a simpler way of using the compiler and units to prove these requirements are met.

在大型系统中确认所有这些要求被合适的处理是有挑战的，但是使用编译器和单位去证明这些要求被满足会有更简单方法。

The following code looks almost the same:

下面代码看起来大致一样：

```swift
let results = query.rx_text.asDriver()        // This converts a normal sequence into a `Driver` sequence.
    .throttle(0.3, scheduler: MainScheduler.instance)
    .flatMapLatest { query in
        fetchAutoCompleteItems(query)
            .asDriver(onErrorJustReturn: [])  // Builder just needs info about what to return in case of error.
    }

results
    .map { "\($0.count)" }
    .drive(resultCount.rx_text)               // If there is a `drive` method available instead of `bindTo`,
    .addDisposableTo(disposeBag)              // that means that the compiler has proven that all properties
                                              // are satisfied.
results
    .drive(resultTableView.rx_itemsWithCellIdentifier("Cell")) { (_, result, cell) in
        cell.textLabel?.text = "\(result)"
    }
    .addDisposableTo(disposeBag)
```

So what is happening here?

所以这里发生了什么？

This first `asDriver` method converts the `ControlProperty` unit to a `Driver` unit.

第一个 `asDriver` 方法转换 `ControlProperty` 单元成为一个 `Driver` 单元。

```swift
query.rx_text.asDriver()
```

Notice that there wasn't anything special that needed to be done. `Driver` has all of the properties of the `ControlProperty` unit, plus some more. The underlying observable sequence is just wrapped as a `Driver` unit, and that's it.

注意这不需要任何其他的操作。`驱动（Driver）`有所有 `ControlProperty` 单元的属性，并且添加了其他。下面的 observable 序列只是被包装了作为 `驱动Driver` 单元。

The second change is:
第二个改变是：

```swift
.asDriver(onErrorJustReturn: [])
```

Any observable sequence can be converted to `Driver` unit, as long as it satisfies 3 properties:

* Can't error out
* Observe on main scheduler
* Sharing side effects (`shareReplayLatestWhileConnected`)

So how do you make sure those properties are satisfied? Just use normal Rx operators. `asDriver(onErrorJustReturn: [])` is equivalent to following code.

任何 observable 序列都能被转换成 `Driver驱动` 单元，只要它满足3个属性：

* 不能错误退出
* 在主调度器上观察
* 分享副作用(`shareReplayLatestWhileConnected`)

所以你如何确认这些属性被满足？仅使用 Rx 操作。`asDriver(onErrorJustReturn: [])` 等价于如下代码：

```
let safeSequence = xs
  .observeOn(MainScheduler.instance)       // observe events on main scheduler
  .catchErrorJustReturn(onErrorJustReturn) // can't error out
  .shareReplayLatestWhileConnected         // side effects sharing
return Driver(raw: safeSequence)           // wrap it up
```

The final piece is using `drive` instead of using `bindTo`.

`drive` is defined only on the `Driver` unit. This means that if you see `drive` somewhere in code, that observable sequence can never error out and it observes on the main thread, which is safe for binding to a UI element.

Note however that, theoretically, someone could still define a `drive` method to work on `ObservableType` or some other interface, so to be extra safe, creating a temporary definition with `let results: Driver<[Results]> = ...` before binding to UI elements would be necessary for complete proof. However, we'll leave it up to the reader to decide whether this is a realistic scenario or not.

最终的片段是使用 `drive` 代替使用 `bindTo`。

`dive` 仅在 `Driver驱动` 上被定义。这意味着如果你看见 `drive` 出现在代码中，observable 序列从不错误退出并且它在主线程上观察，这样绑定一个 UI 元素是安全的。

然而，理论上，有人可以定义一个 `drive` 方法工作在 `ObservableType` 上或者另一些接口，所以为了额外的安全，在绑定 UI 元素之前会用 `let results: Driver<[Results]> = ...` 创建一个临时的定义，为了完全验证这是必要的。然而，我们将留给读者去决定这是否是一个现实的情况。
