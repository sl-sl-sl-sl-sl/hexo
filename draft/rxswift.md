

`RxSwift`是Swift函数响应式编程的一个开源库，由Github的ReactiveX组织开发，维护。

---

## 选择RxSwift

为什么选用`RxSwift`? 由于`ReactiveCocoa(RAC)`近来版本差异过大, 在`RAC4.0`中, `API`与`RxSwift`非常接近, 而`ReactiveCocoa`更新并没有`RxSwift`快, `RAC5.0`也正在正在开发中. 而`RxSwift3.0beta`已放出, 支持`Xcode8`与`Swift3.0`. 

`RxSwift`对`Swift`的兼容很好，利用了很多的`Swift`特性，语法简单，概念清楚. 最重要的一点: Rx是跨平台的. `RxJava`, `RxJS`, `.Net`, 你可以使用相同的操作、步骤和心态来编写代码.在所有的语言当中，Rx 看起来都是十分相似的.


## 主要实体

* Observable
* Observer
* Subscriber
* Subjects

Observables和Subjects是两个"生产"实体, Observers和Subscribers是两个"消费"实体

### Observable
当我们一异步执行复杂操作时, 例如GCD, Operation来处理这些问题是, 代码会变得复杂难以维护. `Observable`被设计用来解决这些问题.















