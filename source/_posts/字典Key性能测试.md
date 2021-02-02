---
title: 字典Key性能测试 #文章标题
date: 2021/2/2 23:46:25
categories: "测试" #文章分类目录 可以省略
tags: #文章标签 可以省略
     - 性能
     - 软件开发
     - C# 
description: #你对本页的描述 可以省略
---



```csharp
const int time = 1000000;
using (new ProfilerScope("int"))
{
    stopwatch.Start();
    for (int i = 0; i < time; i++)
    {
        1.GetHashCode();
    }
    stopwatch.Stop();
    inttime.text = stopwatch.ElapsedMilliseconds.ToString();
}

using (new ProfilerScope("string"))
{
    stopwatch.Restart();
    for (int i = 0; i < time; i++)
    {
        "test".GetHashCode();
    }
    stopwatch.Stop();
    stringtime.text = stopwatch.ElapsedMilliseconds.ToString();
}
```

# GetHashCode

| 次数1000000 | IL2CPP/Windows/Realease |      |
| ----------- | ----------------------- | ---- |
| int         | 1ms                     |      |
| string      | 3ms                     |      |
|             |                         |      |

# Dictionary.Get

```csharp
Dictionary<int, int> intpairs = new Dictionary<int, int>();
intpairs.Add(1, 1);
Dictionary<string, int> strpairs = new Dictionary<string, int>();
strpairs.Add("test", 1);
const int time = 1000000;
using (new ProfilerScope("int"))
{
    stopwatch.Start();
    for (int i = 0; i < time; i++)
    {
        var res = intpairs[1];
    }
    stopwatch.Stop();
    inttime.text = stopwatch.ElapsedMilliseconds.ToString();
}

using (new ProfilerScope("string"))
{
    stopwatch.Restart();
    for (int i = 0; i < time; i++)
    {
        var res = strpairs["test"];
    }
    stopwatch.Stop();
    stringtime.text = stopwatch.ElapsedMilliseconds.ToString();
}
```



| 次数1000000 | IL2CPP/Windows/Realease |      |
| ----------- | ----------------------- | ---- |
| int         | 18-19ms                 |      |
| string      | 21-22ms                 |      |
|             |                         |      |



# Dictionary.Add

```csharp
Dictionary<int, int> intpairs = new Dictionary<int, int>();
intpairs.Add(1, 1);
Dictionary<string, int> strpairs = new Dictionary<string, int>();
strpairs.Add("test", 1);
const int time = 1000000;
using (new ProfilerScope("int"))
{
    stopwatch.Start();
    for (int i = 0; i < time; i++)
    {
        intpairs[1] = 1;
    }
    stopwatch.Stop();
    inttime.text = stopwatch.ElapsedMilliseconds.ToString();
}

using (new ProfilerScope("string"))
{
    stopwatch.Restart();
    for (int i = 0; i < time; i++)
    {
        strpairs["test"] = 1;
    }
    stopwatch.Stop();
    stringtime.text = stopwatch.ElapsedMilliseconds.ToString();
}
```



| 次数1000000 | IL2CPP/Windows/Realease |      |
| ----------- | ----------------------- | ---- |
| int         | 17-18ms                 |      |
| string      | 20-21ms                 |      |
|             |                         |      |

---
key的类型对字典性能影响不大。
