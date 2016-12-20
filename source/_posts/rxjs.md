---
title: RxJS 入门
date: 2016-12-19 20:01:24
tags: RxJS
---

# RxJS 介绍

RxJS 是一个通过使用observable序列来组合异步和时间驱动程序的库。它提供了一个核心类型，**Observable**, 还有其他类型(Observer, Schedulers, Subjects)，和类似Array中的
操作符, 如(map, filtr, resuce, every, etc)，通过这些来像处理集合一样来处理异步时间。

> Think of RxJS as Lodash for events(可以认为RxJS是时间处理中的lodash)

ReactiveX 结合了观察者模式和迭代器模式和集合的函数式编程来满足管理序列化时间的理想方式。

在RxJS中的重要概念主要有：

1. **Observable**: 代表了一种异步的值或事件的可调用的集合
2. **Observer**: 是一个知道如何监听通过Observable传递值的回调函数集合
3. **Subscription**: 代表了Observable和执行, 主要用来取消Observable的执行
4. **Operators**: 是一种纯函数，使得能够用像map，filter，concat，flatmap等函数操作符来处理集合
5. **Subject**: 和EventEmitter相同，而且是唯一的方式能够像多个Observer广播值或事件
6. **Schedulers**: 是用于控制并发的集中调度器， 并允许我们来协调计算比如在使用setTimeout或者requestAnimationFrame时


## 第一个例子

通常我们这样注册事件

```
var button = document.querySelector('button');
button.addEventListener('click', () => console.log('Clicked!'));
```
如果使用RxJS的话我们可以这么写

```
var button = document.querySelector('button');
Rx.Observable.fromEvent(button, 'click')
    .subscribe(() =>console.log('Clicked!'));
```
<!--more-->
### 纯函数

RxJS采用纯函数来产出数据，这意味着有更少的代码错误。

通常你会创造一个不纯的函数来扰乱你的状态。

```
var count = 0;
var button = document.querySelector('button');
button.addEventListener('click', () => console.log(`Clicked! ${++count} times`));
```

在RxJS中你可以隔离状态

```
var button = document.querySelector('button');
Rx.Observable.fromEvent(button, 'click')
    .scan(count => count+1,0)
    .subscribe(count =>console.log(`Clicked ${count} times!`));
```
scan操作符就像是array的reduce操作。

### 事件流
RxJS 有许多操作符来帮助你控制observables中的事件流

如果用纯JS来控制一个按钮在1s内只能按一次
```
var count = 0;
var rate = 1000;
var lastClick = Date.now() - rate;
var button = document.querySelector('button');
button.addEventListener('click', () => {
    if(Date.now() - lastClick >= rate){
        console.log(`Clicked ${++count} times`);
        lastClick = Date.now();
    }
});
```
而是使用RxJS的话
```
var button = document.querySelector('button');
Rx.Observable.fromEvent(button,'click')
    .throttleTime(1000)
    .scan(count => count+1， 0)
    .subscribe(count => console.log(`Clicked ${count} time`));
```
### 流式传值
你能够通过observables来传递值
下面是一个可以通过纯JS来为每一次点击添加鼠标的X坐标
```
var count = 0;
var rate = 1000;
var lastClick = Date.now() - rate;
var button = document.querySelector('button');
button.addEventListener('click', () => {
    if(Date.now() - lastClick >= rate){
        console.log(++count + event.clientX);
        lastClick = Date.now();
    }
});
```
使用RxJS
```
var button = document.querySelector('button');
Rx.Observable.fromEvent(button,'click')
    .throttleTime(1000)
    .map(event => event.clientX)
    .scan((count, clientX) => count+clientX, 0)
    .subscribe(count => console.log(count));
```
# Observable
Observables是一种懒加载能够提供多值的集合
可以看下面的表

|  | Single | Multiple |
|:-----|:---:|:---------:|
|Pull| Function |Itetator|
|Push| Promise |Observable|

下面的这个例子是一个Observable 在被订阅的时候立刻输出1，2，3 然后在订阅后1s后输出4，然后结束。
```
var observable = Rx.Observable.create(function (observer) {
    observer.next(1);
    observer.next(2);
    observer.next(3);
    setTimeout(() => {
        observer.next(4);
        observer.complete();
    },1000);
});
```
为了调用Observable 我们需要订阅它:

```
console.log('just before subscribe');
observable.subscribe({
    next: x => console.log('got value'+ x),
    error: err => console.error('something wrong occurred:' + err),
    complete: () => consoelg.log('done');
});
console.log('just after subscribe');
```
这个执行结果会是
```
just before subscribe
got value 1
got value 2
got value 3
just after subscribe
got value 4
done
```
## Pull vs Push

Pull和Push是两种不同的协议用于在数据生产者和数据消费者之间。
**什么是Pull?** 在Pull系统中由消费者来决定何时来获取数据，每一个jS函数都是一个Pull系统，函数是一个数据生产者，通过调用函数来获取数据。
**什么是Push?** 在Push系统中，由数据生产者来决定什么时候来向消费者提供数据。Promises 是现在JavaScript中最普遍的Push系统。
RxJS引入了Observable，一种新的push系统，一个Observable是一种多个值的生产者，向Observers(消费者)推送数据。

## Observable 和一般化的函数进行比较
和普遍声称的不同，Observables 既不像EventEmitters 也不像多值的Promise。Observables可能在某些情况下像EventEmitters,即当他们使用RxJS Subjects 来广播时，但是通常Observables和EventEmitters不同。

> Observables 就像是没有参数的函数，但是包括了那些允许多个值的函数

```
function foo() {
  console.log('Hello');
  return 42;
}

var x = foo.call(); // same as foo()
console.log(x);
var y = foo.call(); // same as foo()
console.log(y);
```
通过RxJS
```
var foo = Rx.Observable.create(function(observer){
    console.log('Hello');
    observer.next(42);
});
foo.subscribe(function(x){
    console.log(x);
});
foo.subcribe(function(y){
    console.log(y);
});
```
这两个的输出都是一样的:
```
"Hello"
"42"
"Hello"
"42"
```

> 向一个Observable进行订阅就像是调用一个一般的函数

> Observables能够既能同步又能异步传递数据

Observable可以 “return” 多个值。
```
var foo = Rx.Observable.create(function(observer){
    console.log('Hello');
    observer.next(42);
    observer.next(100);
    observer.next(200);
});
console.log('before');
foo.subscribe(function(x){
    console.log(x);
});
console.log('after');
```
可以输出
```
"before"
"Hello"
42
100
200
"after"
```

总结

* func.call() 意味着 “给我一个同步的值”
* observable.subscribe() 意味着“给我多个值不管是同步还是异步”

## Observable的解析
Observable 通过Rx.Observable.create来**创建**，通过Observer来**订阅**，**执行**来传递next/error/complete来通知观察者，而且这些执行操作可以取消，这就是一个Observable实例的4个方面。

### 创建Observables
Rx.Observable.create 是Observable构造器的别名，它有一个参数就是subscribe函数
下面这个例子每隔1s产生一个'hi'给Observer，
```
var observable = Rx.Observable.create(function subscribe(observer){
    var id = setInterval(() => {
        observer.next('hi');
    },1000)
});
```

### 订阅Observables
上个例子中的observable可以这样订阅 `observable.subscribe(x => console.log(x));`

### 执行Observables

### 处理Observable的执行



