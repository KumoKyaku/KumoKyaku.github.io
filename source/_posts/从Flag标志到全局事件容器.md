---
title: 从Flag标志到全局事件容器 #文章标题
date: 2020/5/6 23:46:25
updated: 2020/5/6 23:46:25
categories: "游戏开发" #文章分类目录 可以省略
tags: #文章标签 可以省略
     - 架构
     - unity
     - 程序
description: #你对本页的描述 可以省略
---

# 全局标志设计

- 有时候我们触发一个全局事件，但是我们并不想立刻执行，否则可能导致调用链过长，所以我们记录一个全局标志，在主循环合适的位置再执行逻辑。
- 同时也能避免同一帧某个逻辑执行多次，所以全局标志设计自带去重功能。
- 全局标志集合可以集中处理全局字段，统一重置，不需要针对每个字段都重写复位代码。


<!-- more -->

```cs
static class Global
{
    public static int[] Flags_G { get; } = new int[256];

    /// <summary>
    /// 主循环
    /// </summary>
    public static void Update()
    {
        //do something
        //执行业务逻辑设置各个标志位的值 
        //比如设置当前帧需要重新计算所有对象HP
        Global.Flags_G[GlobalFlags.ReCalHP] += 1;

        //后续的某个时刻
        foreach (var item in updateDic)
        {
            if(Global.Flags_G[GlobalFlags.ReCalHP] > 0){
                item.ReCalHP();
            }
        } 

        ///更新管道最后一步
        ResetFlags();
    }

    /// <summary>
    /// 根据设计重置标志
    /// </summary>
    static void ResetFlags()
    {
        Span<int> span = Flags_G.AsSpan();
        span.Slice(0,64).Clear();

        for (int i = 64; i < 128; i++)
        {
            var value = span[i];
            if (value > 0)
            {
                value--;
            }
            span[i] = value;
        }
    }
}

/// <summary>
/// 全局标志索引，声明和使用按照以下标准：
/// <para>全局标志容器是长度为 256 的 int[]，<see cref="Global.Flags_G"/></para>
/// <para>000 - 063 每帧末尾重置标志</para>
/// <para>064 - 127 每帧末尾标志值减一，最小为0</para>
/// <para>128 - 191 从不重置，由生成者处改变标志</para>
/// <para>192 - 255 从不重置，由消费者处改变标志</para>
/// </summary>
public static class GlobalFlags
{
    public const int ReCalMP = 1;
    public const int ReCalHP = 2;
    public const int GodEyes = 128;
}

```

# 从全局标志到实例标志

既然全局需要一个事件标志，那么对于每个实例，也是需要的。

## 设计一（此设计被证实失败）
仿照全局设计，为每个实例增加一个byte[]字段，对于每个类型，下标代表的含义可以不同。  
弊端：  
- 由于继承机制，数组的大小不好设计，过大浪费内存，过小某些类不够用。
- 每个类的下标含义不通，然而某些事件的下标再所有类型中又想要相同。比如希望所有类型RecalHP的标志的下标为1。最终会导致混乱，难以记忆。

## 设计二
设计一的改良。

```cs

public interface IFlagsable<T>: IFlagsResetable
{
    T Flags { get; }
}

public interface IFlagsResetable
{
    /// <summary>
    /// 重置所有标志
    /// </summary>
    void FlagsReset();
}

public struct UnitObjFlags
{
    internal bool OwnerChanged;
    internal bool RecalHP;
}

public class Apple : IFlagsable<UnitObjFlags>, IFlagsResetable
{
    internal protected UnitObjFlags Flags = default;
    UnitObjFlags IFlagsable<UnitObjFlags>.Flags => Flags;
    public void FlagsReset()
    {
        Flags = default;
    }
}

```

优点：  
- 内存开销更小
- 每个标志类型可以自由设置。
- 命名明确，不需要记忆下标。

弊端：  
- 开发过程中添加繁琐，每当需要一个新的事件就需要添加一个字段。
- 不利于继承。
- 需要再住循环中添加一个Pass,来处理标志位重置。
- 根据开发需要，业务标志语义的不同，标志重置代码可能需要特殊处理，比如某些标志需要跨帧。

## 设计三 全局事件容器
其实有各种各样异步消息机制的实现，殊途同归。


```cs
///事件容器
public class EventCollection<T>: HashSet<(T Target,object State)>
{
    public void Add(T element)
    {
        Add((element, null));
    }

    public void Add(T element,object state)
    {
        Add((element, state));
    }
}

public interface IOnOwnerChanged
{
    void OnOwnerChanged(object state);
}

public class Apple:IOnOwnerChanged
{
    void DealData()
    {
        if (oldOwner != newOwner)
        {
            ///处理方式1 自己就是事件处理器
            Global.OwnerChanged.Add(this);
            ///处理方式2 自己不处理事件，使用通用事件处理器，将自己当作参数传递。
            Global.OwnerChanged.Add(OwnerChanger.Default,this);
        }
    }

    public void OnOwnerChanged(object state)
    {
        //具体处理所有者变更过程
    }
}

///逻辑与数据分离。
public class OwnerChanger:IOnOwnerChanged
{
    public static readonly OwnerChanger Default = new OwnerChanger();
    public void OnOwnerChanged(object state)
    {
        //具体处理所有者变更过程
        if(state is Apple apple)
        {
            apple.XXX();
        }
    }
}

static class Global
{
    ///根据需要选用不同的数据结构。HashSet自带去重。
    public static readonly EventCollection
        <IOnOwnerChanged> OwnerChanged = new EventCollection<IOnOwnerChanged>();

    /// <summary>
    /// 主循环
    /// </summary>
    public static void Update()
    {
        ///主循环的某个Pass
        foreach (var item in OwnerChanged)
        {
            item.Target.OnOwnerChanged(item.State);
        }
        OwnerChanged.Clear();
        ///-----------------
    }
}


```

优点：  
- 实例不必含有事件标志字段，类设计更合理。
- 事件处理可以插入到主循环任一位置。
- 不必遍历全局对象集合。
- 可以再同一帧中，总是先处理一个事件，再处理另一个事件。事件具有顺序性。
- 事件处理逻辑可以单独抽离。

弊端：  
- 每帧加入容器，清楚容器操作具有性能开销。
- 对象的生命周期需要更严格的设计。
- 是否需要跨帧需要针对每个事件设计。

处理事件函数需要接口化：   
如果一个对象不能处理一个事件，也没有一个全局的处理者，那么也就不应该添加到全局事件容器中去，表现是，如果对象不继承指定接口，则不能调用Add函数。

# 总结
在具体架构设计中，当需要将处理过程拆分时，要尽可能的保持上下文状态，尽可能的少引入中间变量。  
//todo
