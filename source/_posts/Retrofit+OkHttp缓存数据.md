---
title: Retrofit+OkHttp缓存数据
tags:
  - Retrofit
  - Cache
toc: true
date: 2016-10-12 21:21:43
categories: 学习总结
---
`Retrofit`本身并不能缓存，从版本2.0开始默认支持缓存，底层的网络连接都依赖于`Okhttp` ，因此数据缓存也需要在`Okhttp`上处理。`Okhttp`的版本不同，其缓存方式也不同。
<!--more-->
### 1. 为什么使用缓存？

减少服务器负荷，降低网络延迟提升用户体验，复杂的缓存处理会根据用户当前的网络情况采取不同的策略，在网络好的时候缓存时间短，数据更新快，在网络差的时候，提高缓存使用的时间，缓存策略不仅仅与网络好坏有关，也与应用本身的用途、业务需求、接口有关，比如应用的数据变更不多，则对缓存使用就比较多，有的应用要保证数据的实时性，比如股票信息，这时就不考虑缓存，如果是天气信息，则可以根据情况设定短的缓存时间，因此应用是否需要缓存，要根据不同的情况作分析，给出不同的方案。
在开发App时，在有网络时调用MobileApi接口，请求新的数据，在断网情况下，可以给用户进行断网提示，但有些需求需要在断网时也显示数据，这时就可以使用缓存。由于`Retrofit` 本身没有缓存处理，则需要自定义拦截器来实现缓存功能。

### 2. 定义拦截器
版本：`Retrofit: 2.1.0` +`Okhttp 3.3.0`：
首先，判断是否有网，有网时请求数据，并保存到缓存中，无网时读取缓存。要使用到CacheControl类，该类主要负责缓存策略的控制，主要有以下策略：

 - FORCE_CACHE ：只走缓存
 - FORCE_NETWORK ：只走网路

**定义拦截器：**
```java
	//拦截器：
    Interceptor interceptor = new Interceptor() {
        @Override
        public okhttp3.Response intercept(Chain chain) throws IOException {
        	// 对请求进行拦截：无网络是强制读取缓存
            Request request = chain.request();
            if (!NetworkUtil.hasNetwork(getApplicationContext())){
                request = request.newBuilder().cacheControl(CacheControl.FORCE_CACHE)
                        .build();
            }
			//对响应进行拦截
            //有网络时，移除header，设置缓存超时时间为1小时
            okhttp3.Response response = chain.proceed(request);
            if (NetworkUtil.hasNetwork(getApplicationContext())){
                int maxAge = 60 * 60;  			 //1小时
                response.newBuilder()
                        .removeHeader("Pragma")
                        .header("Cache-Control","public,max-age="+maxAge)
                        .build();
            }else {
            	//无网络时，缓存时间为1周
                int maxScale = 60 * 60 * 24 * 7;  // 1周
                response.newBuilder().removeHeader("Pragma")
                        .header("Cache-Control","public, only-if-cached, max-stale="+maxScale)
                        .build();
            }

            return response;
        }
    }; 
    
    //判断是否连接网络
    public static void hasNetwork(Context context){
    	if(context != null){
            ConnectivityManager cm = (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);
            if (cm != null){
                NetworkInfo info = cm.getActiveNetworkInfo();
                if (info != null){
                    return info.isAvailable();
                }
            }
        }
        return false;
    }
```
**给Okhttp设置拦截器：**
```java
	//缓存目录
    File cacheFile = new File(context.getExternalCacheDir(),"mycache");
    //缓存大小为20M
    Cache cache = new Cache(cacheFile, 1024 * 1024 * 20);
    //创建OkhttpClient，添加拦截器和缓存
    OkHttpClient client = new OkHttpClient().newBuilder()
                .addInterceptor(interceptor).cache(cache).build();
    //生成Retrofit，并将OkHttpClient对象写入
    Retrofit mRetrofit = new Retrofit.Builder()
    				.baseUrl(BaseUrl)
                    .addConverterFactory(GsonConverterFactory.create())
                    .client(client)
                    .build();
    
```
缓存基本实现，运行程序后，会在SdCard中的`/Android/data/应用包名/cache/mycache`目录下看到缓存文件，当应用卸载后，`/Android/data/应用包名/`这个目录所有文件被删除，不会留下垃圾信息。在`设置---->应用 ---->应用详情里面`进行`清除缓存`也会清理缓存文件，对应上面所说目录。
在拦截器中的两个字段`max-age`和`max-stale`不是很理解，资料较少，有以下比较：

|     |  介绍   |
| :---: | :---:|
| max-age|强制响应缓存者，根据该值，校验新鲜性。即使用自身的Age值，与请求时间做比较，如果超出max-age，则强制去服务端校验，以确保返回一个新鲜的响应|
| max-stale |  允许缓存者，发送一个过期不超过指定时间的旧缓存   |

即使看了这个比较，对此理解也不是很深刻，如果有那位看到，可以留言讨论下。







