---
title: Android蓝牙：发现、配对、连接
tags:
  - Android
  - Bluetooth
toc: true
date: 2017-01-14 21:55:33
categories: 学习总结
---

> 蓝牙是一种短距离无线通信标准、协议。

Android平台包含蓝牙网络堆栈支持，因此设备能够以无线方式与其他蓝牙设备进行数据交换。连接蓝牙设备流程是获取到蓝牙设备，进行配对，进行连接，然后进行数据传输，蓝牙打印机作为蓝牙设备的一种，其操作流程也是这样。
蓝牙开发相关所有的API都在`android.bluetooth`包下，使用这些API就能使APP以无线方式连接到其他蓝牙设备，从而实现点到点和多点无线功能。一般情况下，我们所说的蓝牙指的是`传统蓝牙`，`传统蓝牙`适用于电池使用强度较大的操作，在Android4.3（API 级别18）中引入了面向低功耗蓝牙的API支持，称为`BLE蓝牙`。

<!--more-->

在开发中主要使用到的两个类：

 - **BluetoothAdapter**：这个类的对象代表了本地的蓝牙适配器，是所有蓝牙交互的入口点，比如app运行在手机上，那么手机上的蓝牙适配器就是本地蓝牙适配器，发现蓝牙设备、配对、连接基本都要使用它来进行，通过判断该类对象是否为NULL，就可以知道设备是否支持蓝牙，因此该类是开发蓝牙功能的核心类。
 - **BluetoothDevice**：该类表示一个远程蓝牙设备，包含蓝牙设备的一些属性，主要有蓝牙名称、MAC地址、绑定状态等，利用它可以通过`BluetoothSocket`请求与某个远程设备建立连接。

### 设置蓝牙
#### 添加蓝牙权限
要操作蓝牙，需要在`AndroidManifest.xml`中添加蓝牙权限：
```java
<!--添加蓝牙权限-->
<uses-permission android:name="android.permission.BLUETOOTH"/>
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
```
`BLUETOOTH`权限：允许应用程序去连接已经配对的设备。请求连接、接收连接、传输数据需要该权限，主要用于配对之后的操作。
`BLUETOOTH_ADMIN`权限：允许应用程序去发现和配对远程蓝牙设备。用来管理蓝牙设备，拥有此权限后，才能使用本机的蓝牙设备，主要用于配对之前的操作。
优先级：使用`BLUETOOTH_ADMIN`权限的前提是必须有`BLUETOOTH`权限。


#### 启动蓝牙
获取` BluetoothAdapter`：要获取` BluetoothAdapter`，只需调用静态` getDefaultAdapter() `方法。将返回一个表示设备自身的蓝牙适配器，整个系统只有一个蓝牙适配器，如果此方法返回null，则说明设备不支持蓝牙，也就不能进行蓝牙相关的开发。
```java
BluetoothAdapter mBluetoothAdapter = BluetoothAdapter.getDefaultAdapter();
if (mBluetoothAdapter == null){
	Toast.makeText(MainActivity.this, "设备不支持蓝牙", Toast.LENGTH_SHORT).show();
}else{
	//进行下一步操作
}
```
启用蓝牙：可以通过调用` isEnabled() `来判断蓝牙是否启用，返回true说明设备蓝牙已经开启，返回false则蓝牙处于停用状态。
```java
if(! mBluetoothAdapter.isEnabled()){
	//请求开启设备蓝牙
    Intent openBT = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
    //REQUEST_OPEN_BT_CODE = 1
    startActivityForResult(openBT, REQUEST_OPEN_BT_CODE);
}
```
以上代码将通过系统设置发出启动蓝牙的请求，将弹出一个Dialog，请求用户开启蓝牙，单击‘OK’后，将开启蓝牙，如果开启成功，将在` onActivityResult() `回调函数中收到`RESULT_OK`结果码，如果没有开启，将会收到`RESULT_CANCELED`结果码。
除此之外，如果检测到蓝牙没有开启，还可以进行强制开启，而不会弹出Dialog，这是比较流氓的做法，不过我喜欢。
```java
if(! mBluetoothAdapter.isEnabled()){
	//强制开启蓝牙
    mBluetoothAdapter.enable();
}
```
#### 扫描蓝牙设备

开启设备的蓝牙之后，就可以扫描附近处于开启状态的蓝牙设备了，这个过程是一个比较耗费资源的过程，因此在找到要连接的设备之后，就停止扫描，或者在连接蓝牙设备之前停止扫描，如果已经与某一台蓝牙设备连接，那么扫描过程可能会大幅减少用于连接使用的带宽，因此也不应该在设备连接之后进行扫描操作。
进行扫描只需调用`startDiscovery()`方法，该方法的返回值是boolean类型，返回true则表示开启扫描操作成功。具体是注册一个针对ACTION_FOUND Intent的BroadcastReceiver，在广播接收器中可以拿到扫描到的设备对象，从而得到设备的相关信息。
```java
public void startScanBT(){
    IntentFilter filter = new IntentFilter();
    filter.addAction(BluetoothDevice.ACTION_FOUND);   //发现设备
    filter.addAction(BluetoothAdapter. ACTION_DISCOVERY_STARTED ); //扫描开始
    filter.addAction(BluetoothAdapter.ACTION_DISCOVERY_FINISHED);  //扫描结束
    registerReceiver(scanBTReceiver, filter);


    if(mBTAdapter.startDiscovery()){
        Log.i("iii" , " 扫描蓝牙进程开启成功... ");
    }else{
        Log.e("eee" , " 扫描蓝牙进程开启失败... ");
    }

}

private BroadcastReceiver scanBTReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();
        if (action.equals(BluetoothDevice.ACTION_FOUND)){
            //扫描到的蓝牙设备
            BluetoothDevice device = intent.getParcelableExtra(BluetoothDevice.EXTRA_DEVICE);

            //将扫描到的设备add到List中
            mListDevices.add(device);

        }else if (action.equals(BluetoothAdapter. ACTION_DISCOVERY_STARTED )){
            //扫描开始时要执行的操作
            Log.i("iii" , "wa, 设备扫描开始啦..." );
        }else if (action.equals(BluetoothAdapter.ACTION_DISCOVERY_FINISHED)){
            //扫描结束时进行的操作
            Log.i("iii", "ok, 设备扫描终于结束了，好累...");
        }
    }
};
```
停止扫描进程，调用` cancelDiscovery()`即可：
```java
if (mBTAdapter != null){
	mBTAdapter.cancelDiscovery();
}
```
#### 配对的设备
除了扫描蓝牙之外，可以通过调用` getBondedDevices()`得到已经配对成功的设备集，这个API返回值为`Set<BluetoothDevice>`：
```java
Set<BluetoothDevice> bonedDevices = mBTAdapter.getBondedDevices();
```
关于配对结果回调：蓝牙配对状态也是通过广播接收器来实现的 >>
```java
private BroadcastReceiver bondedBTReceiver = new BroadcastReceiver() {
	 @Override
    public void onReceive(Context context, Intent intent) {
   		BluetoothDevice device = intent.getParcelableExtra(BluetoothDevice.EXTRA_DEVICE);
  		switch (device.getBondState()) {
        	//设备的配对状态
            case BluetoothDevice.BOND_BONDING:
                Toast.makeText(MainActivity.this,"正在配对...", Toast.LENGTH_SHORT).show();
                break;
            case BluetoothDevice.BOND_NONE:
                Toast.makeText(MainActivity.this, "没有设备进行配对...", Toast.LENGTH_SHORT).show();
                break;
            case BluetoothDevice.BOND_BONDED: 
                Toast.makeText(MainActivity.this ,"配对完成...", Toast.LENGTH_SHORT).show();
                break;
            default:
                break;
        }
    }
};
IntentFilter filter = new IntentFilter(BluetoothDevice.ACTION_BOND_STATE_CHANGED);
registerReceiver(bondedBTReceiver, filter);
```
对远程进行配对，只需调用`BluetoothDevice`的`createBond ()`，此方法为一个异步方法。
```java
public void createBond(BluetoothDevice device){
    if (device.getBondState() == BluetoothDevice.BOND_NONE ){
        //需要API >= 19
        device.createBond();
    }
}
```
记得要在Activity销毁时进行取消注册：
```java
unregisterReceiver(scanBTReceiver);			//取消注册扫描广播 
unregisterReceiver(bondedBTReceiver);    	//取消注册绑定广播
```

> 配对与连接之间的区别：配对意味着两个设备之间知道彼此的存在，通过配对密钥，建立一个加密的连接。而连接意味着设备之间共享一个通信通道，能够彼此传输数据。

> 当前的 Android Bluetooth API 要求对设备进行配对，然后才能建立 RFCOMM 连接。 （在使用 Bluetooth API 发起加密连接时，会自动执行配对）。

### 连接为client
两台设备之间建立连接，一台充当client的角色，另一台充当server的角色，client角色的设备发起连接，主要使用server端设备的Mac地址，server端开放server socket，当client和server在同一个通信通道上拥有已经连接的`BluetoothSocket`时，就视为连接成功，可以通过socket拿到I/O流，进而传输数据。`蓝牙打印机`只能被动地被其他设备连接，其开放了一个服务器套接字并侦听其他设备的连接。
```java
/**
 * Description:
 * Created by L02465 on 2016.12.24.
 */
public class ConnectBTThread extends Thread {

    private BluetoothDevice mBTDevice;
    private Handler mHandler;
    private BluetoothSocket mBTSocket; 
    private BluetoothAdapter mBTAdapter;

    public static UUID STR_UUID = UUID.fromString("00001101-0000-1000-8000-00805F9B34FB");

    public ConnectBTThread (BluetoothDevice device,BluetoothAdapter adapter, Handler handler){
        this.mBTDevice = device;
        this.mBTAdapter = adapter;
        this.mHandler = handler;

        BluetoothSocket socketTemp = null;

        try{
            socketTemp = device.createInsecureRfcommSocketToServiceRecord(STR_UUID);
        }catch (IOException ex){
            ex.printStackTrace();
        }
        Log.e("eee", "进入连接线程的构造方法....");
        mBTSocket = socketTemp;

        //设为全局的socket
        App.getInstance().mSocket = mBTSocket;
    }

    @Override
    public void run() {

        Log.e("eee","进入线程的run()...");
        
        try {
            //连接之前先断掉设备扫描
            mBTAdapter.cancelDiscovery();

            if (!mBTSocket.isConnected()) {
            	//进行连接
                mBTSocket.connect();

                //使用反射进行连接
//                mBTSocket =(BluetoothSocket) mBTDevice.getClass().getMethod("createRfcommSocket", new Class[] {int.class}).invoke(mBTDevice,1);
//                mBTSocket.connect();

            }

            if (mHandler == null) {
                return;
            }
            
			//连接成功发送message
            Message message = new Message();
            message.what = 0x111;
            message.obj = mBTDevice;
            mHandler.sendMessage(message);

        } catch (IOException ex) {
            Log.e("eee", "连接失败...");
            Message message = new Message();
            message.what = 0x222;
            mHandler.sendMessage(message);
            try {
                mBTSocket.close();
            } catch (IOException ex1) {
                ex1.printStackTrace();
            }

            ex.printStackTrace();
        }
        
        Log.e("eee","run()运行结束...");
    }
    
   	//关闭socket
    public void close(){
        try{
            mBTSocket.close();
        }catch (IOException ex){
            //...
        }
    }
}
```
主要是通过蓝牙设备对象，调用` createRfcommSocketToServiceRecord(UUID) `获取到` BluetoothSocket`，此处的`UUID`必须与服务器端一致，调用`connect()`进行连接，此方法为阻塞调用，连接超时时将引发异常，因为是阻塞的，因此连接蓝牙设备应该在主线程之外的线程进行。

**UUID：** 称为通用唯一标识符，是用于唯一标识信息的字符串 ID 的 128 位标准化格式。 UUID 的特点是其足够庞大，因此可以选择任意随机值而不会发生冲突，通过`UUID.fromString(String)`初始化一个UUID。

### 连接为server
以下代码来自于Google官方 -> API指南 -> 蓝牙模块：
```java
private class AcceptThread extends Thread {
    private final BluetoothServerSocket mmServerSocket;

    public AcceptThread() {
        // Use a temporary object that is later assigned to mmServerSocket,
        // because mmServerSocket is final
        BluetoothServerSocket tmp = null;
        try {
            // MY_UUID is the app's UUID string, also used by the client code
            tmp = mBluetoothAdapter.listenUsingRfcommWithServiceRecord(NAME, MY_UUID);
        } catch (IOException e) { }
        mmServerSocket = tmp;
    }

    public void run() {
        BluetoothSocket socket = null;
        // Keep listening until exception occurs or a socket is returned
        while (true) {
            try {
                socket = mmServerSocket.accept();
            } catch (IOException e) {
                break;
            }
            // If a connection was accepted
            if (socket != null) {
                // Do work to manage the connection (in a separate thread)
                manageConnectedSocket(socket);
                mmServerSocket.close();
                break;
            }
        }
    }

    /** Will cancel the listening socket, and cause the thread to finish */
    public void cancel() {
        try {
            mmServerSocket.close();
        } catch (IOException e) { }
    }
}
```
其中，`accept()`也是阻塞调用，因此需在子线程中调用。`listenUsingRfcommWithServiceRecord`API第一个参数字符串可以随意起一个，第二个参数UUID需和客户端使用的一致。

在连接成功后，两个设备都会有一个`BluetoothSocket`，可以通过这个socket得到I/O流，读取数据和写入数据，在两个设备之间传输数据。

 1. 获取 InputStream 和 OutputStream，二者分别通过套接字以及 getInputStream() 和 getOutputStream() 来处理数据传输
 2. 使用 read(byte[]) 和 write(byte[]) 读取数据并写入到流式传输

其中`read(byte[])`和`write(byte[])`都是阻塞调用，应该在子线程中进行读写操作，当然最后，必须调用`mmSocket.close();`来关闭此socket。

基础知识大概就这些吧，在项目中基本上都是在ListView显示扫描到的蓝牙设备，选择其中一项进行连接，之后再进行数据传输，对蓝牙的操作，尽可能在子线程中进行处理。
最近一段时间在搞蓝牙打印机，蓝牙打印机作为服务端，被动地进行连接，手机向其发送它们能够识别的指令，称为`ESC/POS指令集`，现在市面上大多数的蓝牙打印机都是兼容这个指令集的，通过指令控制蓝牙打印机打印文字、图片、二维码等信息。
因此，下一篇将对蓝牙打印机指令方面做简单介绍。

