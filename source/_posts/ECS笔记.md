---
title: ECS 笔记 #文章标题
date: 2019/02/06 23:46:55
updated: 2019/02/06 23:46:55
categories: "架构设计" #文章分类目录 可以省略
tags: #文章标签 可以省略
     - Unity
     - 程序
     - 游戏设计
     - ECS
description: #你对本页的描述 可以省略
---

# system

- 任何两个system不应该有重复逻辑
- 任何两个system都应该可以同时运行

+ 系统应该具有依赖关系和执行顺序，应该类似于unityshader的queue，分为几个大的区间。
+ 系统不应该知道entity是什么（player monster stone），否则非常容易写成面向对象。

<!-- more -->

# compnent
- 不应该存在标识性组件，每个组件都应该存在数据

# 例子
- 真实类型 player monster 都可以attack，持有hpitem，player可以使用hpitem回复，   
  那么，
  + 设计组件 hp attackpoint haveitem canuseitem
  + 设计系统   
    - attacksystem  
        +  hp
        +  attackpoint
    - helthsystem
      + hp
      + canuseitem
    - deathdrop
      + hp
      + haveitem
  + 设计实体 实际上不需要明确知道 player monster
    - ~~player~~ enity
      + hp 
      + attackpoint
      + haveitem 
      + canuseitem
    - ~~monster~~ enity
      + hp 
      + attackpoint
      + haveitem 