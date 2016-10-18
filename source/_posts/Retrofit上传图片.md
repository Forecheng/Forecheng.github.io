---
title: 使用Retrofit进行图片的上传
tags:
  - Android
  - Retrofit
toc: true
date: 2016-10-18 22:31:37
categories: 学习总结
---
**目标**：使用Retrofit将本地图片上传至服务器指定的文件夹中。
**服务端接口**：入参，使用表单的形式。
<!--more-->

|表单字段名|类型|说明|
|--|--|--|
|“file”|File|上传图片的File对象|
|“filename”|String|上传图片的文件名称|

请求地址：http://208.208.91.150:8080/VIID-V5/image/updateimage

### 上传图片Api
```java
public interface UploadPicApi{
	@Multipart
    @POST("image/updateimage")
    Call<ErrorBean> uploadPic
    (
    	@Header("Authorization")String authorization,
        @Part MultipartBody.Part image,
        @Part("filename") RequestBody filename
    );
}
```
接口中的参数可以参考这篇文章：[Retrofit入门介绍 -- 参数注解][2]

### 上传图片Service
```java
public class UploadPicService{
	private static UploadPicApi api;
    
    //上传图片结果的回调
    public interface IUploadPicListener{
    	void uploadSuccess();
        void uploadFailed(String errorMsg);
    }
    
    /**
     * 包装图片上传接口
     * 
     * @param token 登录返回的access_token
     * @param path 图片路径
     * @param fileName 图片名称
     * @param listener 结果回调
     * */
    public static void uploadPic(String token, String path, String fileName, final IUploadPicListener listener){
    	api = RetrofitClient.getClient().create(UploadPicApi.class);
        //生成文件对象
        File file = new File(path + fileName);
        //指定内容类型为"image/png"
        RequestBody photoRequestBody = RequestBody.create(MediaType.parse("image/png"), file);
        //MultipartBody.Part 被用来发送真实的文件名
        MultipartBody.Part photo = MultipartBody.Part.createFormData("file", fileName, photoRequestBody);
        //由于服务端要求，文件名不能包含后缀，因此去掉后缀
        String name = getFileNameNoEx(fileName);
       	//调用上传接口
        api.uploadPic(token,photo,RequestBody.create(MediaType.parse("text"),name)).enqueue(new Callback<ErrorBean>(){
        	 @Override
            public void onResponse(Call<ErrorBean> call, Response<ErrorBean> response) {

                if (null == response){
                    onFailure(null,new Throwable("upload picture failed:响应为空"));
                    return;
                }

                if (null == response.body()){
                    onFailure(null,new Throwable("upload picture failed:响应消息体为空"));
                    return;
                }

                //错误码 == 0 表示上传成功，其他均为失败
                if (response.body().getErrorCode() == 0){
                    listener.uploadSuccess();
                }else{
                    listener.uploadFailed("错误码:" +response.body().getErrorCode() + "\n错误描述:"+response.body().getErrorMsg());
                }
                LogUtil.d("-----> upload picture successful !");
            }

            @Override
            public void onFailure(Call<ErrorBean> call, Throwable t) {
                listener.uploadFailed(t.toString());
            }
        });
    }
    
    //去掉文件后缀
    //参数：完成的文件名称 ， ***.jpg
    public static String getFileNameNoEx(String fileName){
    	if((fileName != null) && (fileName.length() > 0)){
        	int dot = fileName.lastIndexOf(".");
            if((dot > -1) && (dot < (fileName.length()))){
            	return fileName.subString(0, dot);
            }
        }
        return fileName;
    }
}
```

使用这个接口就可以完成单张图片的上传了。
如果需要上传多个图片，就在接口中声明多个`Part`参数，也可以使用`PartMap`。
多张图片上传可以参考这篇文章：[Android Retrofit 实现文字（参数）和多张图片一起上传][1]
在编写接口时，需要与服务端配合，保持一致，主要是请求消息头的构造，构造错误的话，那么上传基本是失败的。
```java
@Multipart
@POST("image/updateimage")
Call<ErrorBean> uploadMultiPic(@Part("data") String text, @PartMap Map<String,RequestBody> params);
```
关键代码：
```java	
 public void uploadMultiPic(){
	String image1 = "/sdcard/myimage/" + "happy.png";
	String image2 = "/sdcard/myimage/" + "xixi.png";
	List<String> pathList = new ArrayList<>();    //文件路径集合
	pathList.add(image1);
	pathList.add(image2);
	Map<String,RequestBody> requestBodyMap = new Hash<>();
	if(pathList.size() > 0){
		for(int i = 0; i < pathList.size();i++){
			File file = new File(pathList.get(i));
			requestBodyMap.put("file"+i + "\";"filename=\""+file.getName(),RequestBody.create(MediaType.parse("image/png"),file);
		}
	}
	//调用上传接口
	api.uploadMultiPic("some pictures",requestBodyMap){
    	//TODO：回调处理
	}
 }
 
 
```


  [1]: http://chuansong.me/n/557773851468
  [2]: http://www.forecheng.com/2016/09/30/Retrofit%E5%85%A5%E9%97%A8%E4%BB%8B%E7%BB%8D-1/
