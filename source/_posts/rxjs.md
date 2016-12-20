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
## 纯净

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

## 流
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








