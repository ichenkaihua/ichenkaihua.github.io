---
layout: post
title: "java去除https证书验证"
description: "java去除https证书验证"
category: java
fullwidth: true
tags: [https证书,java]
date: 2015-12-14 17:53:53
permalink: /:year/:month/:day/:title/
---

java进行https协议网络请求时，会要求证书验证。如果证书不合格，则会包错。之前项目中使用过第三方服务，提供的https协议的接口，即通过java访问https网络。为了正常使用服务，有必要去除java对https协议证书验证。<!-- more -->
代码如下:

```java
package com.dreamliner.mhml.controller;

import javax.net.ssl.SSLContext;
import javax.net.ssl.TrustManager;
import javax.net.ssl.X509TrustManager;
import javax.security.cert.CertificateException;
import javax.security.cert.X509Certificate;

public class TestController {
	

	static {
		trustSelfSignedSSL();
	}





	private static void trustSelfSignedSSL() {
		try {
			SSLContext ctx = SSLContext.getInstance("TLS");
			X509TrustManager tm = new X509TrustManager() {
				@SuppressWarnings("unused")
				public void checkClientTrusted(X509Certificate[] xcs,
						String string) throws CertificateException {
				}

				@SuppressWarnings("unused")
				public void checkServerTrusted(X509Certificate[] xcs,
						String string) throws CertificateException {
				}

				public java.security.cert.X509Certificate[] getAcceptedIssuers() {
					return null;
				}

				@Override
				public void checkClientTrusted(
						java.security.cert.X509Certificate[] arg0, String arg1)
						throws java.security.cert.CertificateException {
				}

				@Override
				public void checkServerTrusted(
						java.security.cert.X509Certificate[] arg0, String arg1)
						throws java.security.cert.CertificateException {

				}
			};
			ctx.init(null, new TrustManager[] { tm }, null);
			SSLContext.setDefault(ctx);
		} catch (Exception ex) {
			throw new RuntimeException("Exception occurred ", ex);
		}
	}

}  
```
之后只要`TestController`被载入到Class中，就去除https证书验证。




