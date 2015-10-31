---
layout: post
title: Hello github pages blog!
tagline: "Hello github pages blog!"
tags : jekyll, github
---
{% include JB/setup %}
# Hello github pages blog
用了差不多一年的`wordpress`，wordpress对于日常写作非常方便，各种插件应有尽有，但比不上markdown的简洁。

	package com.chenkaihua.springebean.service;

	import com.avaje.ebean.EbeanServer;
	import com.chenkaihua.springebean.entity.User;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.stereotype.Service;

	import java.util.List;

	/**
	 * Created by chenkaihua on 15-10-29.
	 */
	@Service
	public class UserService {


		@Autowired
		EbeanServer ebeanServer;

		public void save(User user){
			ebeanServer.save(user);
		}

		public void saveOnThrowException(User user){
			ebeanServer.save(user);
			throw new IllegalArgumentException("非法参数！！");
		}


		public List<User> users(){
			return ebeanServer.find(User.class).findList();
		}

	}


