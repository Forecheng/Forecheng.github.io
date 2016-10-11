---
title: Github文件夹为灰色解决办法
tags:
	- Github
toc: false
date: 2016-10-10 08:43:07
categories: 学习总结
---

问题原因：
在灰色的文件中包含其他仓库，也就是含有`.git文件夹`和`.gitignore文件`。
<!--more-->
解决办法：
删除目录下的`.git文件夹`和`.gitignore文件`，重新进行commit和push。

**如果删除之后没有用，可以尝试下面的方法：**
删除该目录的缓存，在`Git Bush`窗口中敲如下命令：
我的项目根目录下`themes/landspace`为灰色，则敲入的命令为：

``` bash
git rm -r --cached themes/landspace
git add .
git commit -m "remove cahce"
git push origin branch-name
```
提交后，刷新github网页查看该目录是否为灰色。



