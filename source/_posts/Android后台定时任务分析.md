---
title: Android后台定时任务分析
tags:
  - Android
toc: true
date: 2016-11-09 23:02:11
categories: 学习总结
---
实现在后台能够稳定的周期性的发送通知，在锁屏状态下收到通知后亮屏。
说到后台定时任务，首先想到的是用`Service + Timer`组合来实现，在Service中启动定时器，在定时器中执行任务，但这种方式存在一个问题，就是定时任务间隔时间较长时，后台Service很容易被Android 系统kill掉，主要是因为Android自带的内存清理，内存不足时就会被kill，这样任务肯定不能定时执行，达不到想要的结果。
<!--more-->
为了解决这种问题，可以使用 **`前台服务`**，这种服务一直保持运行状态，不会由于内存不足被回收，它与Service的最大区别在于，它会一直显示在系统的状态栏，和Notification的效果很相似。
### 使用前台服务进行通知
```java
/**
 * Description:
 * Created by L02465 on 2016.11.09.
 */
public class MyService extends Service {

    private static final int ONGOING_ID = 1;
    private Timer mTimer;
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {

        startTimer();
        return super.onStartCommand(intent, flags, startId);
    }

    public void startTimer(){
        if (mTimer == null) {
            mTimer = new Timer();
        }

        mTimer.schedule(new TimerTask() {
            @Override
            public void run() {
                showNotification();    
            }
        },1000,1000 * 60 * 1);   //1s之后 每隔1分钟运行一次
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        stopForeground(true);        //停止前台服务
    }
	//发送通知
    public void showNotification(){
    	//点击通知进行跳转
        Intent toMain = new Intent(this,MainActivity.class);
        PendingIntent pi = PendingIntent.getActivity(this,0,toMain, PendingIntent.FLAG_UPDATE_CURRENT);
        Notification notification = new Notification.Builder(this)
                .setLargeIcon(BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher))
                .setSmallIcon(R.drawable.ic_cached_black_24dp)  //设置小图标是需要的，否则显示的通知内容不是自定义的内容
                .setContentIntent(pi)       //单击通知后进行跳转
                .setContentTitle("Hello:")   
                .setContentText("快起来走动走动，不然对身体不好...")
                .setDefaults(Notification.DEFAULT_VIBRATE)   //通知来了进行震动
                .build();
		
        startForeground(ONGOING_ID,notification);  //启动前台服务
    }
```
运行结果：
<div align=center>
![运行结果](/images/notification.png)
</div>
代码主要调用了`startForeground()`来启动前台服务，每隔1分钟进行通知，在状态栏显示出来。
调用`stopForeground(true);`停止前台服务，会让状态栏的通知显示取消掉。
使用`Service+Timer`组合看似可以稳定的在后台运行，但也存在问题：
**就是在连接USB进行调试，按下电源键锁屏状态下会运行正常，当拔掉USB线，锁屏下会发现并没有按设定的间隔进行通知，在打开手机后又开始通知的情况**
### 使用AlarmManager

> Android中的一种系统级别的提示服务，在特定的时间广播一个指定的Intent，可以实现从指定时间开始，以一个固定的时间间隔去执行某项操作，同样可以当做一个定时器去使用。
> （1）可以在指定时长后去执行某项操作
> （2）可以周期性的执行某项操作

#### AlarmManager常用的三个方法以及闹钟类型
```java
/**
 * @param type 闹钟类型
 * @param triggerAtMillis 闹钟执行时间
 * @param operation 执行的动作
 * */
public void set(int type, long triggerAtMillis, PendingIntent operation)
```
用于执行一次性闹钟，在指定的时间去执行动作
```java
/**
 * @param type 闹钟类型
 * @param triggerAtMillis 闹钟执行时间
 * @param intervalMillis 间隔时间
 * @param operation 执行的动作
 * */
public void setRepeating(int type, long triggerAtMillis,
            long intervalMillis, PendingIntent operation)
```
用于执行重复闹钟，在指定时间周期性的执行动作
```java
/**
 * @param type 闹钟类型
 * @param triggerAtMillis 闹钟执行时间
 * @param intervalMillis 间隔时间
 * @param operation 执行的动作
 * */
public void setInexactRepeating(int type, long triggerAtMillis,
            long intervalMillis, PendingIntent operation)
```
与第二种类似，在触发时间不精确的情况下需要。

|闹钟类型|状态值|描述|
| :--: | :--: | :--: |
|ELAPSED_REALTIME|3|在睡眠状态下不可用，使用相对时间|
|ELAPSED_REALTIME_WAKEUP|2|在睡眠状态下唤醒系统并执行提示，使用相对时间|
|RTC|1|在睡眠状态下不可用，使用绝对时间|
|RTC_WAKEUP|0|在睡眠状态下唤醒系统并进行提示，使用绝对时间|
|POWER_OFF_WAKEUP|4|在手机关机状态下也能进行提示，使用绝对时间|

相对时间：相对系统开启时间，当前相对时间就是`SystemClock.elapsedRealtime()`
绝对时间：当前时间`System.currentTimeMillis()`
#### Usage
5分钟之后去开启另一个服务：
```java
public void startAlarm(){
        AlarmManager manager = (AlarmManager)getSystemService(ALARM_SERVICE);
        int minutes = 1000 * 60 * 5;  // 5minutes
        Intent toService = new Intent(this,MyService.class);
        PendingIntent pi = PendingIntent.getService(this,0,toReceiver,0);
        long firstTime = SystemClock.elapsedRealtime() + minutes;   //相对时间
        manager.set(AlarmManager.ELAPSED_REALTIME_WAKEUP,firstTime,pi);
    }
```
每隔5分钟开启另一个服务：
```java
public void startAlarm(){
        AlarmManager manager = (AlarmManager)getSystemService(ALARM_SERVICE);
        int minutes = 1000 * 60 * 5;  // 5minutes
        Intent toService = new Intent(this,NoticeService.class);
        PendingIntent pi = PendingIntent.getService(this,0,toService,0);
        long firstTime = SystemClock.elapsedRealtime();
        manager.setRepeating(AlarmManager.ELAPSED_REALTIME_WAKEUP,firstTime,minutes,pi);
    }
```
在新的服务NoticeService开启时，执行通知的代码，可以正常进行通知，拔掉USB线并锁屏后，也会每隔5分钟按时进行通知，基本稳定。AlarmManager的setRepeating()方法和Timer的Schedule(task,delay,peroid)类似。
也可以定时进行广播，在广播接收器中执行操作：
```java
Intent toReceiver = new Intent(this,MyBroadcast.class);
PendingIntent pi = PendingIntent.getBroadcast(this,0,toReceiver,0);
```

 #### AlarmManager取消
需要注意的是取消的Intent必须与启动Intent保持一致才能取消AlarmManager
```java
AlarmManager manager = (AlarmManager) getSystemService(ALARM_SERVICE);
Intent intent = new Intent(this,NoticeService.class);
PendingIntent pi = PendingIntent.getService(this,0,intent,0);
manager.cancel(pi);
```

#### 通知亮屏代码
```java
public void lightScreen(Context context){
    PowerManager powerManager = (PowerManager) context.getSystemService(Context.POWER_SERVICE);
    boolean isScreenOn = powerManager.isScreenOn();

    LogUtil.e("task : screen on >>> " + isScreenOn);

    if (isScreenOn == false){
        PowerManager.WakeLock wakeLock = powerManager.newWakeLock(PowerManager.FULL_WAKE_LOCK | PowerManager.ACQUIRE_CAUSES_WAKEUP
                                | PowerManager.ON_AFTER_RELEASE,"MyLock");
        wakeLock.acquire(1000);
        PowerManager.WakeLock wakeLock_cpu = powerManager.newWakeLock(PowerManager.PARTIAL_WAKE_LOCK,"MyCpuLock");
        wakeLock_cpu.acquire(1000);
    }
}
```
