---
title: CustomEditor生效标准 #文章标题
categories: "Unity" #文章分类目录 可以省略
tags: #文章标签 可以省略
     - unity
     - editor
description: #你对本页的描述 可以省略
---

## Q:

    [CustomEditor(typeof(GameObject))]
    public class A1 : Editor
    {
    }

    [CustomEditor(typeof(GameObject))]
    public class A2 : Editor
    {
    }
    A1,A2哪一个会生效？

<!-- more -->

## A：

    当项目代码重新编译时，返回反射记录所有[CustomEditor]信息。
    保存于CustomEditorAttributes.kSCustomEditors 和CustomEditorAttributes.kSCustomEditors 两个List中。
    当需要绘制Inspector面板时，查找CustomEditorAttributes.kSCustomEditors 和CustomEditorAttributes.kSCustomEditors 。
    查找方式FirstOrDefault，这意味着加载程序集的顺序将决定调用哪一个Typ会被先记录在List中，也就会生效。

    四种文件目录分别对用4个dll。
    [Asset]->Assembly-CSharp.dll
    [Asset Editor]->Assembly-CSharp-Editor.dll
    [Plugins]->Assembly-CSharp-firstpass.dll
    [Plugins Editor]->Assembly-CSharp-Editor-firstpass.dll

    载入顺序为 1.Assembly-CSharp-firstpass 2.Assembly-CSharp 3.Assembly-CSharp-Editor-firstpass 4.Assembly-CSharp-Editor。

    但是，反射记录所有[CustomEditor]信息时 程序集顺序是【倒着】 读取的，
    这意味着[Asset Editor]的[CustomEditor(typeof(GameObject))]优先其他三种文件夹的脚本生效。

    那么，同为[Asset Editor]中的脚本，也就是Assembly-CSharp-Editor.dll中的哪个类型会优先生效呢？
    取决于类型的Token，而Token取决于编译顺序，编译顺序为，
    1.文件名字相同时，例如0.cs 和 0.cs，所处文件夹名字靠前的先编译。
    2.文件名字不同时，文件名字小的先编译，和文件夹名字无关。

    正常情况下：先编译的优先加载，优先生效。
    例如 Asset/0/Editor/0.cs 中的CustomEditor标记的类型总是最优先生效。

    当然，还有少数例外情况，其他文件通过反射修改list中元素顺序。