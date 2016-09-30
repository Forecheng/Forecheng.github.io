---
title: Retrofit入门介绍
date: 2016-09-30 14:13:59
tags: 
	- Android
	- Retrofit
	- Http
---

**在这次的项目中，由于服务端采用Restful原则提供接口，所以在手机客户端调用接口时，考虑使用现在很流行的Retrofit网络请求框架，很多情况下，和Retrofit搭配使用的是RxJava，他们堪称"黄金组合"，功能强大。**
<!--more-->
还有其他使用率也很高的网络框架，包括google官方提供的一个，关于在项目如何选择，可以参考stormzhang大神写的一篇博客。
博客地址为：[ANDROID开源项目推荐之「网络请求哪家强」][1]

---

## 1.0 Restful
百度百科：Restful

> 一种软件架构风格，是一种设计风格而不是标准，只是提供了一组设计原则和约束条件，主要用于客户端和服务器交互的软件或系统。基于这个风格设计的软件可以更加简洁，更有层次，更易于实现缓存等机制。


## 2.0 Retrofit

> **Type-safe HTTP client for Android and Java by Square**
Retrofit官网地址:[http://square.github.io/retrofit/][2]                                          
Retrofit github地址:[https://github.com/square/retrofit][3]



### 2.0.1 创建Retrofit对象
在项目中使用需要在模块下面的`build.gradle`中添加如下依赖：

> compile 'com.squareup.retrofit2:retrofit:2.1.0'<br>
> compile 'com.squareup.retrofit2:converter-gson:2.1.0'

如果需要使用RxJava，也需要添加相应的依赖：
> compile 'com.squareup.retrofit2:adapter-rxjava:2.1.0'

创建Retrofit对象需要使用`Builder`，指定`BASR_URL`和添加`Converter`.


``` java
public class RetrofitClient{
	public static String Base_url = "http://"+server_addr+":"+port+"VIID-V5/";
    public static Retrofit getClient(){
    		Retrofit retrofit = new Retrofit.Builder()
            	.baseUrl(Base_url)
            	.addConverterFactory(GsonConverterFactory.create())
            	.build();
                
             return retrofit;
    }
}
```
server_addr和port：是要请求的服务器地址和端口。在编写请求接口时，是相对base_url而言的。
Retrofit2必须要以`/`结束，不然会抛出`非法参数异常`。

### 2.0.2 请求接口定义

``` java
public interface LoginApi{
	@Headers({"Contant-Type:application/json","Accept:application/json"})
	@POST("login/")
    Call<TokenBean> login(@Body LoginInfo loginInfo);
}
```
登录发送Post请求，因此接口含有注解`@POST`，可以看出登录请求的完整地址为`http://"+server_addr+":"+port+"VIID-V5/login`，发送POST请求，因此含有请求消息体，为`JSON`数据格式，可以在android studio安装插件`Gson format`，将json数据格式转换为实体类。相应也是json数据，也可以转为实体类得到相应数据。

``` java
/**
	登录请求实体类
*/
public class LoginInfo{
	/**
    username:admin
    password:admin
    */
    private String username;
    //当字段与json中的不一致时，可以使用字段@SerializedName
    @SerializedName("passwd")
    private String password;
    
    public void setUsername(String username){
    	this.username = username;
    }
    public void setPassword(String password){
    	this.password = password;
    }
    
    //TODO:get()
    //TODO:toString()
}

/**
	相应实体类：服务端相应一般都会包含错误码和错误信息
*/
public class TokenBean{
	private int errCode;
    private String errMsg;
    private String access_token;   //请求数据的令牌
    
    //TODO:set()
    //TODO:get()
    //TODO:toString()
}
```
### 2.0.3 接口调用

``` java
	public class LoginService{
    	public static LoginApi api;
        //登陆结果回调
        public interface ILoginListener{
        	void loginSuccess();
            void loginFailed(String errInfo);
        }
        public static void login(LoginInfo info,ILoginListener listener){
        	api = Retrofit.getClient().create(LoginApi.class);   //代理对象
            api.login(info).enqueue(new Callback<TokenBean>(){
            		@Override
                    public void onResponse(Call<GitModel> call, Response<GitModel> response) {
                        //服务端响应信息
                        if(response == null || response.body() == null){
                        	listener.loginFailed("响应消息为空，检查网络连接是否正常!");
                            return；
                        }
                        
                        if(response.code() == 200 && response.body().getErrCode() == 0){
                        	//login successful
                            listener.loginSuccess();
                            //对于其他查询的请求，判断请求成功后，拿到服务端返回的数据
                        }else if(response.code() == 404){
                        	listener.loginFailed("404: 页面找不到");
                        }else if(response.code() == 500){
                        	listener.loginFailed("500: 服务器异常");
                        }else{
                        	listener.loginFailed(reponse.code + "：其他异常信息");
                        }
                    }

                    @Override
                    public void onFailure(Call<GitModel> call, Throwable t) {
                        //服务端响应失败的信息
                        listener.loginFailed(t.getMessage());    //回调出响应超时信息
                    }
                    });
            }
        }
    }
```
一般的Retrofit使用就是这样的流程，在大型项目中，可以根据项目需要，进行自定义。

## 3.0 Retrofit注解详情

### 3.0.1 请求方法


|  请求方法   |  方法简单描述 | 
| :---: | :---: | 
|  POST  |  post请求，信息包含在请求体RequestBody中，一般用于添加   |  
| GET | get请求方式，参数包含在Url ，一般用于查询 |
| DELETE    |  删除数据    | 
|   PUT  |  修改数据  | 
|   PATCH  |    | 
|   HEAD  |    | 
|  OPTIONS  |   | 
|  HTTP  | @HTTP(method="get",path="login/",hasBody=true)   | 

### 3.0.2 标记

|          | 标记           | 简单描述                                                                 |
| :--------: | :--------------: | :------------------------------------------------------------------------: |
| 表单请求 | FormUrlEncoded | 表示请求体是一个form表单，Content-Type:application/x-www-form-urlencoded |
| 表单请求 | Multipart      | 请求体是一个支持文件上传的form表单，Content-Type:multipart/form-data     |
|          | Streaming      |   表示响应体的数据用流的形式返回，如果没有使用该注解，默认会把数据全部载入内存，之后通过流获取数据也不过是读取内存中的数据，所以如果返回的数据比较大，就需要使用这个注解                                                                     |

### 3.0.3 参数注解
|位置|参数注解|描述|
|:-----:|:--------:|:--------------------------:|
|作用于方法|Headers|用于添加请求头|
|作用于方法参数|Header|用于添加不固定值得Header，也用于鉴权的目的|
|作用于方法参数|Body|用于非表单请求体|
|作用于方法参数|Field/FieldMap|用于表单字段，与FormUrlEncoded配合使用|
|作用于方法参数|Part/PartMap|用于表单字段，与Multipart配合使用，用于有文件上传的情况，PartMap的接受类型是Map<String,String>，非String会调用toString()|
|作用于方法参数|Path|用于URL，参数是URL的一部分|
|作用于方法参数|Query/QueryMap|用于URL，比如：@Query("id")，http://127.0.0.1:8080/users/id=10 |
|作用于方法参数|Url|完整的URL|

 1. {占位符}和Path尽量只用在URL的path部分，url中的参数用Query和QueryMap代替
 2. Query/Field和Part这三者都支持数组和实现了Iterable接口的类型，比如List/Set。
 3. 添加head信息，动态添加和静态添加

``` java
	Call<String> getData(@Query("item[]") List<Integer> item);
    // -----------------------------------
    @GET("user")
	Call<User> getUser(@Header("Authorization") String authorization)
	
    @Headers("Cahce-Control:max-age=640000")
```


### 3.0.4 数据类型转换器

Retrofit支持的数据类型转换器
 - Gson: com.squareup.retrofit2:converter-gson:2.1.0
 - Jackson: com.squareup.retrofit2:converter-jackson:2.1.0
 - Moshi: com.squareup.retrofit2:converter-moshi:2.1.0
 - Protobuf: com.squareup.retrofit2:converter-protobuf:2.1.0
 - Wire: com.squareup.retrofit2:converter-wire:2.1.0
 - Simple XML: com.squareup.retrofit2:converter-simplexml:2.1.0
 - Scalars (primitives, boxed, and String): com.squareup.retrofit2:converter-scalars:2.1.0

Retrofit提供的`CallAdapter`：

| name   | build.gradle中的依赖                        |
| :------: | :-------------------------------------------: |
| rxjava | com.squareup.retrofit2:adapter-rxjava:2.1.0 |
| java8  | com.squareup.retrofit2:adapter-java8:2.1.0  |
| guava  |   com.squareup.retrofit2:adapter-guava:2.1.0         |

  [1]: http://stormzhang.com/opensource/2016/08/05/android-open-source-project-recommend2/
  [2]: http://square.github.io/retrofit/
  [3]: https://github.com/square/retrofit
