---
layout: post
title: "retrofit2 multpart多文件上传"
description:  "retrofit2 upload multipart files"
category: [android]
tags: [retrofi2]
---

`retrofit2`是目前很流行的android网络框架，运用注解和动态代理，极大的简化了网络请求的繁琐步骤，非常适合处理`restfull`网络请求。在项目中，经常需要上传文件到服务器，有时候是需要上传多个文件。网上文章基本都是单文件上传教程，这篇文章主要讲`retrofit`的多文件上传实现。<!-- more -->

## 了解 `multipart/form-data`
在最初的http协议中，没有定义上传文件的`Method`，为了实现这个功能，http协议组改造了post请求，添加了一种post规范，这种规范的`Content-Type`就是`multipart/form-data;boundary=${bound}` ,其中`${bound}`是定义的分隔符，post body里需要用到,尽量保证随机唯一。

post格式如下:

```
--${bound}
Content-Disposition: form-data; name="Filename"
 
HTTP.pdf
--${bound}
Content-Disposition: form-data; name="file000"; filename="HTTP协议详解.pdf"
Content-Type: application/octet-stream
 
%PDF-1.5
file content
%%EOF
 
--${bound}
Content-Disposition: form-data; name="Upload"
 
Submit Query
--${bound}--
```

`${bound}`是`Content-Type`里`boundary`的值

## retrofit2 对`multipart/form-data`的封装

* 在**retrofit**中:
    * 注解`retrofit2.http.Multipart`标记一个请求是`multipart/form-data`类型。
    * 注解`retrofit2.http.Part`代表`Multipart`里的一项数据,即用`${bound}`分隔的内容块。
* 在**okhttp3**中:
    * 类`okhttp3.MultipartBody`为`multipart/form-data`的抽象封装,继承`okhttp3.RequestBody`
    * 类`okhttp3.MultipartBody.Part`为`multipart/form-data`里的一项数据。



``` java
package com.chenkaihua.oneschedule.net;

import java.util.List;

import okhttp3.MultipartBody;
import retrofit2.http.Body;
import retrofit2.http.FormUrlEncoded;
import retrofit2.http.Multipart;
import retrofit2.http.POST;
import retrofit2.http.Part;

/**
 * Created by chenkh on 16-4-2.
 */

public interface FileService {


    /**
     * 通过 List<MultipartBody.Part> 传入多个part实现多文件上传
     * @param parts 每个part代表一个
     * @return 状态信息
     */
    @Multipart
    @POST("users/image")
    BaseResponse<String> uploadFilesWithParts(@Part() List<MultipartBody.Part> parts);


    /**
     * 通过 MultipartBody和@body作为参数来上传
     * @param multipartBody MultipartBody包含多个Part
     * @return 状态信息
     */
    @POST("users/image")
    BaseResponse<String> uploadFileWithRequestBody(@Body MultipartBody multipartBody);


}

```



## 参考资料
><http://my.oschina.net/cnlw/blog/168466>

