---
title: 一些Android Debug Bridge命令
tags:
  - Android
  - adb
toc: true
date: 2016-11-14 21:50:10
categories: 学习总结
---

### 概述
> Android Debug Bridge（ADB）是android sdk中的一个工具，用这个工具可以直接操作管理Android模拟器或者连接到计算机的Android设备。
> ADB是一个客户端-服务器端程序，客户端是操作的电脑，服务端是Android设备

<!--more-->
### 主要功能
它的主要功能有：

 - 运行设备的Shell
 - 管理模拟器或设备的端口映射
 - 计算和设备之间上传/下载文件
 - 将本地apk安装到模拟器/真实的Android设备

### 一些常用命令
#### ADB启动/停止/查看版本
adb启动：
```java
adb start-server
```
adb停止：
```java
adb kill-server
```
查看版本：
``` java
C:\Users\DELL>adb version
Android Debug Bridge version 1.0.36
Revision af05c7354fe1-android
```
#### 查看设备
```java
adb devices
```
 查看当前连接的设备，以列表的形式显示出来。eg：
 ```java
 D:\AndroidStudioProjects\KeepAction>adb devices
List of devices attached
3c4711c84670    device
 ```
#### 安装apk
apk在当前目录：
```java
adb install <apk名称>
```
apk不在当前目录：
```java
adb install <apk的路径>
```
eg：
```java
C:\Users\DELL>adb install C:\Users\DELL\Desktop\版本\3322\运维\com.maintain.apk
[100%] /data/local/tmp/com.mainta
        pkg: /data/local/tmp/com.mainta
Success
```
保留数据和缓存文件，reinstall apk：
```java
adb install -r com.maintain.apk
```
安装apk到SD卡：
```java
adb install -s com.maintain.apk
```

#### 卸载apk
```java
adb uninstall <packagename>
adb uninstall -k <packagename>
```
eg：
```java
C:\Users\DELL>adb uninstall com.maintain
Success
```
加`-k`参数的话，卸载后会保留app的配置和缓存文件
#### 登录设备shell
```java
adb shell
adb shell <command>
```
这个命令将登录设备的shell。
后面加command将直接运行设备命令，相当于执行远程命令
#### 从电脑上发送文件到设备
```java
adb push <本地路径> <远程路径>
```
本地路径：文件在电脑上的路径
远程路径：文件保存到Android设备上的路径
使用push把电脑上的文件或者文件夹复制到手机上
#### 从设备上下载文件到电脑
```java
adb pull <远程路径> <本地路径>
```
使用pull把手机上的文件或者文件夹复制到电脑上
#### 重启手机
```java
adb reboot
```
对手机进行重启
#### 获取序列号
```java
C:\Users\DELL>adb get-serialno
3c4711c84670
```
#### 获取MAC地址
```java
C:\Users\DELL>adb shell  cat /sys/class/net/wlan0/address
3c:47:11:c8:34:db
```
#### 查看设备型号
```java
C:\Users\DELL>adb shell getprop ro.product.model
HUAWEI Y635-CL00
```
#### 显示包信息
(1)列出系统应用的所有包名：
```java
C:\Users\DELL>adb shell pm list packages -s
package:com.qualcomm.timeservice
package:com.android.defcontainer
package:com.example.android.notepad
package:com.android.contacts
package:com.huawei.hwid
package:com.huawei.mmitest2
package:com.android.phone
...
```
(2)列出除了系统应用的第三方应用包名：
```java
C:\Users\DELL>adb shell pm list packages -3
package:com.tencent.lightalk
package:com.uniview.airimos.mle
package:com.uniview.retrofitdemo_git
package:com.uniview.keepaction
package:com.uniview.imos.sdk
...
```
(3)使用`find`或者`findstr`列出要找的包名：
```java
C:\Users\DELL>adb shell pm list packages | find "qq"
package:com.tencent.mobileqq
```
#### 清除应用数据与缓存
```java
adb shell pm clear <packagename>
```
#### 强行停止应用
```java
adb shell am force-stop <packagename>
```
#### 查看手机Android系统版本
```java
adb shell getprop ro.build.version.release
```
#### 查看屏幕分辨率
```java
C:\Users\DELL>adb shell wm size
Physical size: 1080x1920
```
#### 查看屏幕密度
```java
C:\Users\DELL>adb shell wm density
Physical density: 480
```
#### 显示帮助信息
```java
adb help
```

