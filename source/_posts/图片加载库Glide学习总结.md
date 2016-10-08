---
title: 图片加载库Glide学习总结
date: 2016-10-08 11:13:31
tags:
	- Android
	- Glide
	- 图片加载库
toc: true
---

最近的项目需要加载图片，小白就上网寻找这方面的资料，项目的架构为`MVP+Retrofit`，典型的网络数据请求,看了比较多的资料，于是决定使用Glide,因为数据的呈现方式都是使用ListView,用了才知道该库很牛,于是对接触到的知识点稍微归纳一下，整理学习。
<!--more-->
图片加载一直是一个比较棘手的事情，俗话说就是`“坑”`比较多，于是就google了一下这方面的库，有很多图片库，例如Picasso, Universal-Image-Loader,Fresco,Glide等，这些库都是很火的，使用量很大的图片库， 他们各有优点，不能评说那一个不好，根据项目的需求适当选择。

## 1. 几个库的简单比较

 - **Picasso：**是Square出品的图片库，当然是很厉害的，该公司还有响当当的`Okhttp`和`Retrofit`，都是很牛逼的网络请求库,搭配使用效果可能更好一些，之所以这么说是因为Picasso可以将网络请求到的数据缓存交给Okhttp来处理。
 - **Universal-Image-Loader：**一个强大的图片加载库，包含的配置比较多，使用很广泛，很老牌。
 - **Fresco：**是Facebook出品的，也超强大。从知乎问答下看到该库最大的优点是5.0以下（最低2.3）的bitmap加载，在5.0以下，会将图片放到一个特别的内存存储区，称为：`Ashmem区`，在图片不显示的时候也会自动清除内存（这个是必须的），使得App更加流畅，减少图片内存占用引发的内存不足（OOM）。
 - **Glide：**谷歌推荐使用的图片加载库，Google一些图片应用就使用该库进行图片加载，用起来很流畅，可以说是在Picasso上的进一步扩展，比如支持Gif动态图，在使用时可以传入更多的上下文等等，他们两者最大的区别在于默认的设置不同，比如默认的图片格式不同，尺寸大小也不同等，在下面的链接文章中有具体的对比分析。
 - **总结：**Glide是Picasso的进一步扩展，Picasso能做的Glide也可以做，无非就是配置的不同。

网上关于`Picasso`和`Glide`性能进行比较的文章挺多，不过遗憾的是，大多数都是抄来抄去的，比较清晰的是这篇：[Google推荐的图片加载库Glide介绍][1]，`Picasso`和`Glide`的使用极其相似，但也有不少细节上的区别，看了这篇文章可以大致了解一二。

现在整理一下`Glide`的使用：

## 2. 配置Glide
在Module根目录下的build.gradle中进行库依赖：

``` java
def def latestVersion = '3.7.0'
dependencies {
	compile 'com.github.bumptech.glide:glide:$latestVersion'
    compile 'com.android.support:support-v4:19.1.0'
}
```
Glide需要依赖Support Library v4。
然后同步一下代码build successful,就可以使用Glide了。
## 3. 使用Glide
使用下面一段代码就能实现图片加载，看起来极其简单方便：
``` java
//图片的url
private String image_url = "https://www.image.com/myimage?id=1";
private ImageView imageView = (ImageView)super.findViewById(R.id.image);
Glide.with(context)
	.load(image_url)
    .into(imageView);
```
用这段代码就可以将url所表示的图片装到ImageView显示出来，是不是很爽呢？
with里面的参数相比Picasso不仅仅接受上下文Context对象，还可以是`Activity`，`Fragment`，`FragmentActivity`等，这样可以更好的让加载图片的请求与生命周期动态管理起来。Glide不仅仅支持加载网络图片，还支持以下几种加载方式：

 - 加载资源文件：`DrawableTypeRequest<Integer> load(Integer resourceId)`

``` java
Glide.with(context).load(R.drawable.my_image).into(imageview);
```

 - 加载本地文件：`DrawableTypeRequest<File> load(File file)`

``` java
File file = new File("sdcard/myimage/","image.jpg");
Glide.with(context).load(file).into(imageView);
```

 - 加载Uri：`DrawableTypeRequest<Uri> load(Uri uri)`

``` java

File file = new File(Environment.getExternalStorageDirectory() + File.separator +  "image", "image.jpg");
Uri uri = Uri.fromFile(file);
Glide.with(context).load(uri).into(imageView);
```

 - String加载：`DrawableTypeRequest<String> load(String string)`

### 3.1 设置占位图片
#### 1. placehold
图片加载并不是很实时的，加载成功的时间是不确定的，在加载时，可以设置一个图片显示在ImageView上进行一些`正在加载...`提示等等。

``` java
Glide.with(context)
	.load(image_url)
    .placehold(R.drawable.loading)
    .into(imageView);
```
#### 2. error
在加载网络图片时，如果突然网络断掉，肯定加载不到正确的图片，这时可以设置一个错误图片到ImageView,提示用户加载失败。

``` stylus
Glide.with(context)
	.load(image_url)
    .error(R.drawable.loaderror)
    .into(imageView);
```
当然，也可以给Glide设置监听，当图片加载失败时，可以知道为什么加载失败了。

``` java
//设置加载图片错误时的监听
RequestListener<String,GlideDrawable> loadErrorListener = new RequestListener<String,GlideDrawable>(){
	@Override
    public boolean onException(Exception e,String model,Target<GlideDrawable> target,boolean isFirstResource){
    	//加载异常时回调
        Log.e(TAG,"exception:" + e.toString);
    }
    @Override
    public boolean onResourceReady(GlideDrawable resource,String model,Target<GlideDrawable> target,boolean isFromMemoryCache,boolean isFirstResource){
    	//加载成功时的回调
        //...
    }
};

Glide.with(context)
	.load(image_url)
    .listener(loadErrorListener)
    .into(imageView);
```
### 3.2 图片的调整
Glide加载图片大小根据ImageView尺寸自动调整的，在缓存的时候也是按照图片大小进行缓存，每一种尺寸都会保留一份缓存。
并且可以调用override(int width,int height)在图片显示到ImageView之前改变图片大小，width和height的单位都是`px`。

``` java
Glide.with(context)
	.load(image_url)
    .override(64,64)
    .into(imageView);
```
**缩放**：Glide提供了两种图形转换的标准选项：`centerCrop()`和`fitCenter`；

 - centerCrop()

这个可以对图像进行裁剪，当图片比ImageView大的时候，会将超出ImageView的部分裁剪掉，尽可能让ImageView完全填充，但图像可能不会全部显示

``` java
Glide.with(context)
	.load(image_url)
    .centerCrop()
    .into(imageView);
```

 - fitCenter()

它会自适应ImageView的大小，并且会完整的显示在ImageView中，但是ImageView可能不会被完全填充

**设置缩略图支持**

``` java
Glide.with(context)
	.load(image_url)
    .thumbnail(0.1f)
    .into(imageView);
```
**设置动态转换**
在图片显示之前，可以通过`transformation`对其做一些处理，已达到想要的图片效果，为此，需要创建一个类，该类实现了`Transformation接口`，如果只是对图片进行转换，则可以直接使用Glide封装好的`BitmapTransformation抽象类`，图像的转换只需要在`transform`里实现，并重写`getId()方法`，该方法返回此次转换的唯一标识，要确保唯一性。

``` java
public class GlideRoundTransform extends BitmapTransformation{
	@Override
    protected Bitmap transform(BitmapPool pool,Bitmap toTransform,int outWidth,int outHeight){
    	//转换处理
    }
    
    @Override
    public String getId(){
    	return getClass().getName + Math.random();
    }
}

Glide.with(context)
	.load(image_url)
    .transform(new GlideRoundTransform())
    .into(imageView);
```
当图片需要多个转换时，将每种的转换类对象传入到transform()即可。

``` java
Glide.with(context)
	.load(image_url)
    .transform(new GlideRoundTransform(this),new GlideOtherTransform(this))
    .into(imageView);
```

### 3.3 加载的动画
为了使图片平滑的加载到ImageView，可以设置加载的动画效果，最新的api已经默认实现了一个渐入渐出的动画效果，默认时间300ms，也可以使用`crossFade`，它接受无参或者一个int型的参数，用来指定动画执行的时间。

``` java
Glide.with(context)
	.load(image_url)
    .crossFade(2000)    //2s
    .into(imageView);
```
如果渐入渐出的动画效果不满意，可以自定义动画，使用`animate()`即可，它接受动画的资源id和动画类对象。

``` java
Glide.with(context)
	.load(image_url)
	.animate(R.anim.myAnim)
    //.animate(new MyAnimation())
	.into(imageView);
```
如果不想使用任何动画，直接将图片显示出来，则使用`dontAnimate()`方法。

``` java
Glide.with(context)
	.load(image_url)
    .dontAnimate()    
    .into(imageView);

```
### 3.4 加载Gif动态图
加载Gif动态图是Glide的一个亮点，也是将gif动态图的url传入load即可加载，Glide还提供了Gif相关的两个方法：`asBitmap()`和`asGif()`;

 - asBitmap()：将gif图的第一帧显示出来
 - asGif()：严格显示成gif，当传入的Url不是gif的url时，则按错误处理，可以检查load参数是否为gif。

``` java
Glide.with(context)
	.load(imageUrl)
    .asBitmap()
    .into(imageView);
```


### 3.5 加载优先级
设置图片加载的顺序，有以下几种优先级：

 - Priority.LOW
 - Priority.NORMAL
 - Priority.HIGH
 - Priority.IMMEDIATE

``` java
 Glide.with(context)
	.load(image_url)
    .priority(Priority.NORMAL)    
    .into(imageView);
```
### 3.6 加载目标Target
Target就是Glide获取资源后作用的目标，一般的ImageView就是目标。

``` java
SimpleTarget target = new SimpleTarget<Drawable>(){
            @Override
            public void onResourceReady(Drawable resource, GlideAnimation<? super Drawable> glideAnimation) {
                textView.setBackground(resource);
            }
};
 Glide.with(context)
	.load(image_url)
    .priority(Priority.NORMAL)    
    .into(target);
```
这段代码是将TextView作为Target，并将加载到的图片设置为TextView的背景，SimpleTarget接收泛型数据，可以将其更改为其他想要的类型。也可以指定加载的宽度和高度，单位也是`px`。

### 3.7 缓存策略
几种缓存策略：

 - DiskCacheStrategy.ALL：缓存源资源和转换后的资源
 - DiskCacheStrategy.NONE：什么都不缓存
 - DiskCacheStrategy.SOURCE：只缓存源资源
 - DiskCacheStrategy.RESULT：只缓存转换后的资源

``` java
Glide.with(context)
	.load(imageUrl)
    .diskCacheStrategy(DiskCacheStrategy.ALL)
    .into(imageView);
```
跳过内存缓存：

``` java
Glide.with(this)
	.load(imageUrl)
    .skipMemory(true)
    .into(imageView);
```
### 3.8 缓存的动态清理

``` java

Glide.get(this).clearDiskCache();      //清理磁盘缓存，需要在子线程中执行
Glide.get(this).clearMemory();			//清理内存缓存，可以在UI线程中执行
```


## 4. 结合列表视图的使用
Glide在滑动加载图片时表现突出，这也是Glide的优势之一，在项目中很可能是在ListView或者RecyclerView中显示加载的图片：
1. 在使用ListView进行加载时，可以在Adapter的getView中进行使用

``` java
public View getView(int postion,View convertView,ViewGroup parent){
	ViewHolder holder;
	if(convertView == null){
    	holder = new ViewHolder();
        //.....
    }	
    UserInfo infos = (UserInfo)getItem(postion);
    String imageUrl = infos.getImageUrl();
    Glide.with(convertView.getContext()).load(imageUrl).into(holder.imageView);
}

```
2. 在RecyclerView中使用，在 Adapter的onBindViewHolder方法中使用：

``` java
public void onBindViewHolder(final MyHolder holder, int position){
	Glide.with(holder.imageView.getContext())
    	.load(args[position])
        .into(holder.imageView);
}
```

Glide缓存处理进阶可以参考这篇文章：[Android图片缓存之Glide进阶][2]


以上就是学习Glide时的知识点整理，后面再接触到新的知识点时再进行补充。

  [1]: http://blog.csdn.net/sam_zhang1984/article/details/48524893
  [2]: http://www.cnblogs.com/whoislcj/p/5565012.html
