---
title: 异步编程分析 #文章标题
date: 2018/7/27 23:46:25
updated: 2019/1/6 23:46:25
categories: "DOTNET" #文章分类目录 可以省略
tags: #文章标签 可以省略
     - 异步
     - async
description: #你对本页的描述 可以省略
---


* 返回值为Task的方法的Task,是何时构造的？如果第一个await没有同步完成，那么会创建新的task.
  * 返回值为Task的方法同步完成，不会new Task,内部使用的单例的已完成的Task,不会有额外开销。
    internal static readonly Task<TResult> s_defaultResultTask = AsyncTaskCache.CreateCacheableTask<TResult>(default(TResult));

<!-- more -->

* 同一个方法中多个await 会被编译在同一个状态机中。
* 每个await关键字只会触发一次GetAwaiter()方法。
* 没有用到await 就不要用async关键字，真的会有性能损失。内部会构造状态机对象和多个辅助对象。请在项目中将警告CS1998视为错误。
* 综上： 每个async Task 方法，只要方法内不是同步完成，都会创建一个Task.
  
# 主要
* `async`的作用是将方法编译为状态机
* `await`的作用是将方法在状态机中分离为不同阶段。
* `异步方法嵌套`Task被封装在completionAction回调中，**释放这个Action,即释放所有后续Task**.