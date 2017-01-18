---
title: Android Studio中的一些快捷键
tags:
  - Android Studio
toc: false
date: 2016-11-02 21:01:40
categories: 学习总结
---
### Log 打印 ###
生成类的TAG：**` logt `**
```java
private static final String TAG = "MainActivity";
```
<!--more-->
在方法中打印日志：**` logd `**
```java
Log.d(TAG, "onCreate: ");
```
在方法中打印方法名和参数信息：**` logm `**
```java
Log.d(TAG, "onCreate() called with: " + "savedInstanceState = [" + savedInstanceState + "]");
```
还有`loge`、`logi`、`logr`、`logw`。
### 代码的移动 ###
代码提示：**` ctrl + alt + 空格 `**
代码行上移/下移：**`ctrl + shift + up/down`**
复制代码行到下一行：**`ctrl + D`**
剪切一行代码：**`ctrl + X`**
删除一行代码：**`ctrl + Y`**
光标在方法上移动：**`alt + up/down`**

### 代码查看 ###
打开一个类：**`ctrl + N`**
打开一个文件：**`ctrl + shift + N`**
查看类的声明：**`ctrl + B`**
查看父类：**`ctrl + U`**
查看方法的调用：**`ctrl + alt + H`**
在类中查看方法的实现：**`ctrl + shift + I`**
查看类的继承结构：**`ctrl + H`**
工程面板的显示与隐藏：**`alt + 数字1`**
查看一个类中的所有方法：**`ctrl + F12`**
查看父类中的方法并选择复写：**`ctrl + O`**

代码块生成：**`ctrl + J`**
查找：**`ctrl + F`**
替换：**`ctrl + R`**
打开最近的模板：**`ctrl + E`**

根据自身使用习惯，也可以在*Android Studio*的 **`File --> Settings ---> keymap`** 中改变默认的快捷键，来提高开发效率。

### 快捷代码块

1. **`fori`** 和 **`foreach`**
```java
 for (int i = 0; i < ; i++) {
            
   }
```
2. **`.null`** 和 **`.nn`**：在变量名后面使用，快捷检测是否为null。
```java
 Bundle bundle = getIntent().getExtras();
 if (bundle == null) {
            
  }

  if (bundle != null) {
            
  }
```
3. **`const`** 随机生成一个符合android style的final int，
```java
private static final int  = 181;
```
4. **`key`** 生成一个以**KEY_** 开头的final String
```java
private static final String KEY_ = "";  
```
5. **`fbc`** 快捷生成**findViewById**
```java
() findViewById(R.id.);
```
6. **`visible`** 和 **`gone`** :
```java
Button myBut =  new Button(this);
.setVisibility(View.GONE);
.setVisibility(View.VISIBLE);
```
7. **`starter`** 和 **`newInstance`** :
```java
public static void start(Context context) {
        Intent starter = new Intent(context, Main2Activity.class);
        starter.putExtra();
        context.startActivity(starter);
    }
    
    //------------------------------//
    public static Fragment newInstance() {
        
        Bundle args = new Bundle();
        
        Fragment fragment = new Fragment();
        fragment.setArguments(args);
        return fragment;
    }
```



