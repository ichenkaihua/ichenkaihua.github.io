---
layout: post
title: "c# webrequest multpart 多文件上传"
description: "c# webrequest multpart multi file upload"
category: c#
tags: [webrequest,file upload]
fullwidth: true
---

c#中通常使用`HttpWebRequest`进行HTTP网络请求，`HttpWebRequest`只对Http请求进行了最简单的封装。如果要利用Http协议实现多文件上传，则必须使用`POST`方法`multipart/form-data`格式。为了重复使用，我封装了几个方法，实现了多参数文件上传。<!-- more -->

## 添加引用

使用WebRequest需要添加引用`System.Web`,否则引入出错。

## 参数封装
方便起见，我把请求参数进行了封装，代码如下:

```c#

namespace EasyHttp.Net.Core
{
    public class KeyValue
    {
        public string Key;
        public string Value;
        public string FilePath;
        public string ContentType="*/*";

        public KeyValue(string key, string value, string filePath, string contentType)
        {
            Key = key;
            Value = value;
            FilePath = filePath;
            ContentType = contentType;
        }

        public KeyValue() { }

        public KeyValue(string key, string value, string filePath)
        {
            Key = key;
            Value = value;
            FilePath = filePath;
        }

        public KeyValue(string key, string value)
        {
            Key = key;
            Value = value;
        }
    }
}


```
`KeyValue`代表了广义的参数，可以是普通的键值对，也可以是文件参数。

## 多文件上传封装


``` c#
        public static void ExecuteMultipartRequest(HttpWebRequest request, List<KeyValue> nvc)
        {
            Console.WriteLine(request.Headers);
            //   log.Debug(string.Format("Uploading {0} to {1}", file, url));
            string boundary = "---------------------------" + DateTime.Now.Ticks.ToString("x");
            byte[] boundarybytes = System.Text.Encoding.ASCII.GetBytes("\r\n--" + boundary + "\r\n");

            HttpWebRequest wr = request;
            wr.ContentType = "multipart/form-data; boundary=" + boundary;
            wr.Method = "POST";
            wr.KeepAlive = true;
            wr.Credentials = System.Net.CredentialCache.DefaultCredentials;



            using (var rs = wr.GetRequestStream())
            {
                // 普通参数模板
                string formdataTemplate = "Content-Disposition: form-data; name=\"{0}\"\r\n\r\n{1}";
                //带文件的参数模板
                string headerTemplate = "Content-Disposition: form-data; name=\"{0}\"; filename=\"{1}\"\r\nContent-Type: {2}\r\n\r\n";
                foreach (KeyValue keyValue in nvc)
                {
                    //如果是普通参数
                    if (keyValue.FilePath == null)
                    {
                        rs.Write(boundarybytes, 0, boundarybytes.Length);
                        string formitem = string.Format(formdataTemplate, keyValue.Key, keyValue.Value);
                        byte[] formitembytes = System.Text.Encoding.UTF8.GetBytes(formitem);
                        rs.Write(formitembytes, 0, formitembytes.Length);
                    }
                    //如果是文件参数,则上传文件
                    else
                    {
                        rs.Write(boundarybytes, 0, boundarybytes.Length);

                        string header = string.Format(headerTemplate, keyValue.Key, keyValue.FilePath, keyValue.ContentType);
                        byte[] headerbytes = System.Text.Encoding.UTF8.GetBytes(header);
                        rs.Write(headerbytes, 0, headerbytes.Length);

                        using (var fileStream = new FileStream(keyValue.FilePath, FileMode.Open, FileAccess.Read))
                        {
                            byte[] buffer = new byte[4096];
                            int bytesRead = 0;
                            long total = 0;
                            while ((bytesRead = fileStream.Read(buffer, 0, buffer.Length)) != 0)
                            {

                                rs.Write(buffer, 0, bytesRead);
                                total += bytesRead;
                            }
                        }
                    }
                    
                }

                byte[] trailer = System.Text.Encoding.ASCII.GetBytes("\r\n--" + boundary + "--\r\n");
                rs.Write(trailer, 0, trailer.Length);

            }

        }
```

## 使用

```c#
 static void Main(string[] args)
        {
		var request = WebRequest.Create("http://localhost:8080/test/upload") as HttpWebRequest;
            List<KeyValue> keyValues = new List<KeyValue>();
            keyValues.Add(new KeyValue("key1","param1"));
            keyValues.Add(new KeyValue("key2", "param2"));
            keyValues.Add(new KeyValue("file","test1.png","image/png"));
            keyValues.Add(new KeyValue("file", "test2.png", "image/png"));

            EasyHttp.ExecuteMultipartRequest(request,keyValues);
        }

```