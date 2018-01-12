---
layout: post
title: "nginx配置MIME"
date: 2012-04-10 16:53
comments: true
categories: nginx
---

#### 问题描述
用nginx配置好weisite后，访问website，css和js都可以加载，但是没有生效，用firebug查看css和js请求的Content-Type为text/plain.

#### 问题解决
在nginx配置文件nginx.conf中加入如下代码：
```
include       /opt/nginx/conf/mime.types;
default_type  application/octet-stream;
```
#### 问题原因
我安装nginx后，nginx的配置文件什么也没有，我只加入了server{}，没有include mime.types，导致nginx没办法识别css和js文件，因此给浏览器发送了默认的mime类型----text/plain， 以致浏览器不能正常解析css和js文件。

#### 名字解释
> MIME英文全称为“Multipurpose Internet Mail Extensions”，多用途互联网邮件扩展，是一种互联网标准使其能够支持非ASCII字符、二进制格式附件等多种格式的邮件消息。服务器响应浏览器时，会将文件的MIME类型发送给浏览器，浏览器通过MIME类型来决定怎样处理文件。
