---
title: Android中的Menu
tags:
  - Android
  - Menu
toc: true
date: 2016-12-14 22:02:15
categories: 学习总结
---

### Android系统中的三种基本菜单

 - 选项菜单和应用栏：选项菜单是activity的主菜单项，放置一些对application全局产生影响的操作，例如“setting”、“search”等
 - 上下文菜单：当长按某一个元素时出现的菜单，它提供的操作将影响所选内容或上下文框架
 - 弹出菜单：是以垂直列表形式显示一系列项目，这些项目锚定到调用该菜单的视图中，特别适用于提供与特定内容相关的大量操作，或者为命令的另一部分提供选项。

<!--more-->

创建菜单和创建布局一样，可以使用XML进行定义，也可以直接在Activity中进行定义，但一般情况下，都是在XML文件中进行定义的，这也是推荐的做法。

### 在XML文件中定义菜单
在XML菜单资源中定义菜单及其所有项，定义后，可以在Activity中扩充菜单资源。
这样做的好处是：

 - 更易于使用XML可视化菜单结构
 - 将菜单内容与应用的行为代码分离
 - 可以利用应用资源框架，为不同的版本、不同的屏幕尺寸创建菜单配置

在`res\menu`目录下创建`.xml`文件，没有`menu目录`则新建，文件名是随意的，但文件格式必须为`xml`
```xml
<menu>
```
定义Menu，即菜单项的容器，`<menu>`必须为xml文件的根节点，并且包含一个或多个`<item>`和`<group>`元素
```xml
<item>
```
创建MenuItem，表示一个菜单项，可以嵌套`<menu>`，以便创建子菜单
```xml
<group>
```
是`<item>`的不可见容器，支持对菜单项进行分类，使菜单共享活动状态和可见性等属性
定义很简单菜单的例子如下：
```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <item
        android:id="@+id/refresh"
        android:title="刷新"
        android:orderInCategory="100"
        android:icon="@drawable/refresh"
        android:showAsAction="always"
        app:showAsAction="always"/>

</menu>
```
和大多数组件一样，`菜单项<item>`也有如例子中的一些属性，`android:orderInCategory`属性，对每个MenuItem进行排序，数字越小，越显示在前面。
特别地，`android:showAsAction`指定此项应作为操作项目显示在应用栏中的时间和方式，有如下值可以设定：

 - never：不将MenuItem显示在ActionBar上
 - always：总是将MenuItem显示在ActionBar上
 - ifRoom：ActionBar上有空间则显示，没有则放入溢出菜单中
 - withText：显示在ActionBar，并显示文本

在XML定义完成后，在Activity中使用时，调用`MenuInflater.inflate()`来引入菜单文件。

### 创建选项菜单
当`API <= 10`，也就是`Android 2.3.x`版本以下，按下菜单按钮时，选项菜单出现在屏幕底部，包含6个菜单项，多余菜单项则放入溢出菜单中。

当`API >= 11`，也就是`Android 3.0`及更高版本，选项菜单则出现在`ActionBar`中，从`Android5.0(API > 21)`开始被`ToolBar`所代替。
在Activity中指定选项菜单，重写` onCreateOptionsMenu()`:
```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    MenuInflater inflater = getMenuInflater();
    inflater.inflate(R.menu.my_menu, menu);
    return true;
}
```
处理菜单项的单击事件，需要重写`onOptionsItemSelected() `方法，通过`getItemId()`来识别菜单项，返回菜单项目的唯一ID。
```java
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    // Handle item selection
    switch (item.getItemId()) {
        case R.id.refresh:
            startRefresh();
            return true;
        default:
            return super.onOptionsItemSelected(item);
    }
}
```

### 创建上下文菜单
提供上下文操作的方法有两种：

 - 使用`浮动上下文菜单`。用户长按一个声明支持上下文菜单的视图时，菜单显示为菜单项的浮动列表，用户一次可对一个项目执行上下文操作。
 - 使用`上下文操作模式`。它将在屏幕顶部显示上下文操作栏，其中包括影响所选项的操作项目。

#### 创建浮动上下文菜单
1.先通过调用` registerForContextMenu()`，注册与上下文菜单关联的View，并将其传递给View
2.在Activity或Fragment中实现` onCreateContextMenu() `方法

长按注册的View后，将回调`onCreateContextMenu() `方法:
```java
@Override
public void onCreateContextMenu(ContextMenu menu, View v, ContextMenuInfo menuInfo){
    super.onCreateContextMenu(menu, v, menuInfo);
    MenuInflater inflater = getMenuInflater();
    inflater.inflate(R.menu.context_menu, menu);
}
```
3.实现`onContextItemSelected()`，响应菜单单击事件:
选择菜单项时，将调用此方法 >>>
```java
@Override
public boolean onContextItemSelected(MenuItem item) {
    AdapterContextMenuInfo info = (AdapterContextMenuInfo) item.getMenuInfo();
    switch (item.getItemId()) {
        case R.id.edit:
            editNote(info.id);
            return true;
        case R.id.delete:
            deleteNote(info.id);
            return true;
        default:
            return super.onContextItemSelected(item);
    }
}
```
#### 使用上下文操作模式（略）
### 创建弹出菜单
`PopupMenu` 是锚定到 View 的模态菜单，控件足够的话，会显示在定位视图下方，否则显示在其上方。
#### 创建步骤：

 1. 实例化`PopupMenu`及其构造函数，传递当前应用的Context以及锚定到的View
 2. 使用`MenuInflater`将菜单资源扩充到` PopupMenu.getMenu() `返回的Menu对象中.
 3. 调用PopupMenu.show()

例如单击一个按钮，弹出一个弹出菜单：
```java
public void onClick(View view){
    PopupMenu popup = new PopupMenu(this, v);
    MenuInflater inflater = popup.getMenuInflater();
    inflater.inflate(R.menu.actions, popup.getMenu());
    popup.show();
}
```
#### 处理点击事件
单击菜单项执行操作，必须实现` PopupMenu.OnMenuItemClickListener `接口，并通过调用` setOnMenuItemclickListener()`将其注册到PopupMenu。用户选择项目时，系统会在接口中调用` onMenuItemClick()`回调。
```java
public void onClick(View view){
    PopupMenu popup = new PopupMenu(this, v);
    //当前Activity实现了PopupMenu.OnMenuItemClickListener接口，对弹出菜单进行事件注册
    popup.setOnMenuItemClickListener(this);
    MenuInflater inflater = popup.getMenuInflater();
    inflater.inflate(R.menu.actions, popup.getMenu());
    popup.show();
}
```
```java
@Override
public boolean onMenuItemClick(MenuItem item) {
    switch (item.getItemId()) {
        case R.id.archive:
            archive(item);
            return true;
        case R.id.delete:
            delete(item);
            return true;
        default:
            return false;
    }
}
```
