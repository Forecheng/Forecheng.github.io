---
title: 蓝牙打印机ESC/POS指令学习
tags:
  - 蓝牙打印机
  - ESC/POS指令
toc: true
date: 2017-01-17 23:12:34
categories: 学习总结
---

蓝牙打印机作为一款蓝牙设备，使用手机连接到打印机之后，可以向其发送指令控制其进行打印数据，这里的指令指的是`ESC/POS`指令，这套指令集兼容市面上大多数蓝牙打印机设备，但对于一些比较旧的蓝牙打印机，会存在指令不兼容的情况，这些旧的设备，它们的程序版本没有升级更新，因此打印的时候会出现乱码、格式不对等问题。
使用蓝牙打印机可以打印文字、图片、二维码、条形码等信息。
通过这篇[Android蓝牙：发现、配对、连接][1]，成功连接打印机之后，可以拿到`BluetoothSocket`对象，通过这个socket可以得到`OutputStream`，然后调用`write(byte[] data)`向这个输出流中写入指令数据即可。

<!--more-->

### 获取输出流、写数据
```java
public OutputStream mOutputStream;
public void getOutStream(BluetoothSocket socket){
    try {
        if (socket != null){
            mOutputStream = socket.getOutputStream();
        }else{
            Log.e("eee", "传入的socket为空");
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}

//向连接流中写数据
public void write(byte[] data){
    if (mOutputStream != null){
        try {
            mOutputStream.write(data);
            mOutputStream.flush();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
### 常用的ESC/POS指令
在网上有很多的指令文档，但要注意的是，并不是每个打印文档上的指令都适用你所用的打印机，因此最好的方式是：购买打印机时请商家提供相应的指令集文档和demo程序，这可以让我们避免一些坑，否则出现乱码或者打印格式不对，很可能就是指令造成的。打印指令描述如下图所示：
<div align=center>
![指令截图](/images/ESCPOS截图.PNG)
</div>

下面列举一些常用的指令：
```java
/**
* 初始化打印机：清除打印缓存，各参数恢复默认值
* */
public void initPrinter(){
    write(new byte[]{0x1B, 0x40});
}

/**
 * 进行换行
 * */
public void setLF(){
    write("\n".getBytes());
}

/**
 * 设置行间距为n点
 *
 * 范围：0-255，默认值为：33
 *
 * @param n 间距点数
 * */
public void setLineSpace(byte n){
    write(new byte[]{0x1B,0x33,n});
}

/**
 * 设置左边距
 *
 * 纸宽 58mm：0 ≤ n ≤ 47，且 0 ≤ （左边距 + 右边距） ≤ 47
 * 纸宽 80mm：0 ≤ n ≤ 71，且 0 ≤ （左边距 + 右边距） ≤ 71
 *
 * @param n 间距点数
 *
 * */
public void setLeftMargin(byte n){
    write(new byte[]{0x1B,0x6C,n});
}

/**
 * 设置右边距
 *
 * 纸宽 58mm：0 ≤ n ≤ 47，且 0 ≤ （左边距 + 右边距） ≤ 47
 * 纸宽 80mm：0 ≤ n ≤ 71，且 0 ≤ （左边距 + 右边距） ≤ 71
 * */
public void setRightMargin(byte n){
    write(new byte[]{0x1B, 0x51,n});
}

/**
 * 设置打印字体
 *
 * 须明确打印机是否支持多字体
 *
 * 参数 n 意义如下：
 * n   类型
 * 0 中文：24×24，外文：12×24
 * 1 中文：16×16，外文：8×16
 * 2 中文：12×12，外文：6×12
 * */
public void setPrintFont(byte n){
    write(new byte[]{0x1B, 0x4D, n});
}

/**
 * 设置对其方式
 *
 * 0, 48  居左  1, 49  居中  2, 50  居右
 * */
public void setAlignMethod(byte n){
    write(new byte[]{0x1B, 0x61,n);
}
    
```
更多打印所需的指令都可以在指令文档中找到。

### 打印文字
```java
/**
* 打印文本
* */
public void printText(String text){
    if (text.length() > 0) {
        // Get the message bytes and tell the BluetoothService to write
        byte[] send;
        try {
            send = text.getBytes("GB2312");

            Log.d("dy", "send.length:" + send.length);
        } catch (UnsupportedEncodingException e) {
            send = text.getBytes();
        }

        write(send);
    }
}
```

### 打印图片

在打印图片时，需要对图片做一些特殊处理，主要有对图片大小进行调整，然后对图片色彩进行转换，因为打印机打印出的内容只有黑白两种颜色。
**彩色图片----> 灰度图片-----> 二值图**
#### 调整图片大小

```java
public static Bitmap resizeImage(Bitmap bitmap, int w, int h) {

    // load the origial Bitmap
    Bitmap BitmapOrg = bitmap;

    int width = BitmapOrg.getWidth();
    int height = BitmapOrg.getHeight();
    int newWidth = w;
    int newHeight = h;

    // calculate the scale
    float scaleWidth = ((float) newWidth) / width;
    float scaleHeight = ((float) newHeight) / height;

    // create a matrix for the manipulation
    Matrix matrix = new Matrix();
    // resize the Bitmap
    matrix.postScale(scaleWidth, scaleHeight);
    // if you want to rotate the Bitmap
    // matrix.postRotate(45);

    // recreate the new Bitmap
    Bitmap resizedBitmap = Bitmap.createBitmap(BitmapOrg, 0, 0, width,
    height, matrix, true);

    // make a Drawable from Bitmap to allow to set the Bitmap
    // to the ImageView, ImageButton or what ever
    return resizedBitmap;
}
```

#### 转为灰度图
```java
// 转成灰度图
public static Bitmap toGrayscale(Bitmap bmpOriginal) {
    int width, height;
    height = bmpOriginal.getHeight();
    width = bmpOriginal.getWidth();
    Bitmap bmpGrayscale = Bitmap.createBitmap(width, height,
    Bitmap.Config.ARGB_8888);
    Canvas c = new Canvas(bmpGrayscale);
    Paint paint = new Paint();
    ColorMatrix cm = new ColorMatrix();
    cm.setSaturation(0);
    ColorMatrixColorFilter f = new ColorMatrixColorFilter(cm);
    paint.setColorFilter(f);
    c.drawBitmap(bmpOriginal, 0, 0, paint);
    return bmpGrayscale;
 }
```

#### 转为二值图
```java
/**
 * 将ARGB图转换为二值图，0代表黑，1代表白
 * 
 * @param mBitmap
 * @return
 */
public static byte[] bitmapToBWPix(Bitmap mBitmap) {

    int[] pixels = new int[mBitmap.getWidth() * mBitmap.getHeight()];
    byte[] data = new byte[mBitmap.getWidth() * mBitmap.getHeight()];

    mBitmap.getPixels(pixels, 0, mBitmap.getWidth(), 0, 0, mBitmap.getWidth(), mBitmap.getHeight());

    // for the toGrayscale, we need to select a red or green or blue color
    //进行二值化
    format_K_threshold(pixels, mBitmap.getWidth(), mBitmap.getHeight(), data);

    return data;
}

public static void format_K_threshold(int[] orgpixels, int xsize,int ysize, byte[] despixels) {

    int graytotal = 0;
    int grayave = 128;
    int i, j;
    int gray;

    int k = 0;
    for (i = 0; i < ysize; i++) {

        for (j = 0; j < xsize; j++) {

            gray = orgpixels[k] & 0xff;
            graytotal += gray;
            k++;
        }
    }
    grayave = graytotal / ysize / xsize;

    // 二值化
    k = 0;
    for (i = 0; i < ysize; i++) {

    for (j = 0; j < xsize; j++) {

            gray = orgpixels[k] & 0xff;

            if (gray > grayave){
                despixels[k] = 0;// white
            }else{
                despixels[k] = 1;
            }
			
            k++;
        }
    }
}
```
#### 逐行打印图片
打印图片指令如下图：
<div align=center>
![打印图片指令](/images/打印图片指令.png)
</div>

```java
// 之所以弄成一维数组，是因为一维数组速度会快一点
private static int[] p0 = { 0, 0x80 };
private static int[] p1 = { 0, 0x40 };
private static int[] p2 = { 0, 0x20 };
private static int[] p3 = { 0, 0x10 };
private static int[] p4 = { 0, 0x08 };
private static int[] p5 = { 0, 0x04 };
private static int[] p6 = { 0, 0x02 };

// nWidth必须为8的倍数,这个只需在上层控制即可, nMode = 0 或者 1
private static byte[] eachLinePixToCmd(byte[] src, int nWidth, int nMode) {
    int nHeight = src.length / nWidth;
    int nBytesPerLine = nWidth / 8;
    byte[] data = new byte[nHeight * (8 + nBytesPerLine)];
    int offset = 0;
    int k = 0;
    for (int i = 0; i < nHeight; i++) {
        offset = i * (8 + nBytesPerLine);
        data[offset + 0] = 0x1d;
        data[offset + 1] = 0x76;
        data[offset + 2] = 0x30;
        data[offset + 3] = (byte) (nMode & 0x01);
        data[offset + 4] = (byte) (nBytesPerLine % 0x100);
        data[offset + 5] = (byte) (nBytesPerLine / 0x100);
        data[offset + 6] = 0x01;
        data[offset + 7] = 0x00;
        for (int j = 0; j < nBytesPerLine; j++) {
            data[offset + 8 + j] = (byte) (p0[src[k]] + p1[src[k + 1]]
                    + p2[src[k + 2]] + p3[src[k + 3]] + p4[src[k + 4]]
                    + p5[src[k + 5]] + p6[src[k + 6]] + src[k + 7]);
            k = k + 8;
        }
    }

    return data;
}

//整合以上所有方法，直接将此方法返回的byte[]数据写进流中
public static byte[] bitmapToByte(Bitmap mBitmap, int nWidth, int nMode){

    Log.e("eee", "图片宽度：" + nWidth);
	
    int width = ((nWidth + 7) / 8) * 8;
    int height = mBitmap.getHeight() * width / mBitmap.getWidth();
    height = ((height + 7) / 8) * 8;
    //调整大小
    Bitmap rszBitmap = resizeImage(mBitmap, width, height);
    //灰度化
    Bitmap grayBitmap = toGrayscale(rszBitmap);
    //转为二值图
    byte[] dithered = bitmapToBWPix(grayBitmap);
    //进行每行打印
    byte[] data = eachLinePixToCmd(dithered, nWidth, nMode);

    return data;
}
```
使用指令去控制蓝牙打印机，是挺严格的，每个指令都要正确，不然打印出来的东西不是乱码就是格式错误，这部分也比较偏，与硬件有一定的联系，希望可以帮助到项目中有使用到蓝牙打印机的开发者。

  [1]: http://www.forecheng.com/2017/01/14/Android%E8%93%9D%E7%89%99%EF%BC%9A%E6%89%AB%E6%8F%8F%E3%80%81%E9%85%8D%E5%AF%B9%E3%80%81%E8%BF%9E%E6%8E%A5/
  [2]: ./images/1484663000750.jpg "1484663000750.jpg"