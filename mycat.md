---
layout: post
title: Mycat
date: 2017-05-03 10:10:00
tags:
- docker
categories: Java
---


# Mycat
[Mycat](http://www.mycat.org.cn)是一个开源的数据库分库分表中间件。基于阿里开源的Cobar产品研发。

# 安装Mycat
可以在[https://github.com/MyCATApache/Mycat-download-new](https://github.com/MyCATApache/Mycat-download-new)下载Mycat。




# server.xml
我们比较关心的有user的配置。在连接到mycat时，要使用这里的user和password
```xml
          <user name="root">
                  <property name="password">123456</property>
                  <property name="schemas">TESTDB</property>
          </user>
 
          <user name="user">
                  <property name="password">user</property>
                  <property name="schemas">TESTDB</property>
                  <property name="readOnly">true</property>
          </user>
```

