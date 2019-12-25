---
title: Observer模式弱引用的一些想法
date: 2019-12-25 07:48:27
categories: "程序" #文章分类目录 可以省略
tags: #文章标签 可以省略
     - 异步
     - UniRx
     - 程序
description: #你对本页的描述 可以省略
---

观察者模式可以使用弱引用的方法实现自动注销，但是存中风险。

Source ->OnNext ->Observer4WeakedObject->[OnNext -> CheckTarget -> weakreference]-> WarpObserver:IOnNext ->  Observer

<!-- more -->

这种模式中将Observer函数注册到一个WrapObserver的OnNext委托中。
通过返回的IDispose持有WrapObserver引用。

将WrapObserver 注册到 Observer4WeakedObject 中。
Observer4WeakedObject  保存 WrapObserver 的弱引用。

将Observer4WeakedObject注册到Source完成调用链。

Observer和WrapObserver互相持有引用。一旦Observer没有被其他引用。二者形成孤岛，被GC。

`风险：` 因为无法精确控制GC执行时间，所以不能依赖GC来清理注册。









