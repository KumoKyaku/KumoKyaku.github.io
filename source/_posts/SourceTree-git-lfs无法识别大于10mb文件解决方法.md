---
title: SourceTree-git-lfs无法识别大于10mb文件解决方法
date: 2019-12-16 22:46:37
categories: "软件学习" #文章分类目录 可以省略
tags: #文章标签 可以省略
     - Unity
     - 程序
     - 软件开发
     - 游戏开发
     - git
---





SourceTree 使用lfs时，单个文件大于10mb时会导致无法追踪。虽然使用git lfs 查看会显示为以追踪，但是会推送失败。  
![image-20191216224852819](SourceTree-git-lfs无法识别大于10mb文件解决方法\image-20191216224852819.png)

解决方法：  

<!-- more -->

将选项中比较设置中二进制大小限制调大即可。  
![image-20191216224947326](SourceTree-git-lfs无法识别大于10mb文件解决方法\image-20191216224947326.png)  
![image-20191216225102801](SourceTree-git-lfs无法识别大于10mb文件解决方法\image-20191216225102801.png)


# 其他设置：
![QQ截图20200425152915](SourceTree-git-lfs无法识别大于10mb文件解决方法/QQ截图20200425152915.png)

在全局git配置文件中增加  
```
[lfs]  
  contenttype = 0
```