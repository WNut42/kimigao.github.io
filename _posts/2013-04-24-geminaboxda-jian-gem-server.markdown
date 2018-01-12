---
layout: post
title: "geminabox搭建gem server"
date: 2013-04-24 18:55
comments: true
categories: Ruby
---

### 服务器上搭建Geminabox

* 在服务器上建立如下目录结构（建议在用户home目录下）：


* 编辑config.ru，添加如下内容：

```
require "rubygems"
require "geminabox"

Geminabox.data = "./data"
run Geminabox

```

* 如果需要给geminabox添加Http Basic Auth，可参考下面链接
#### [https://github.com/geminabox/geminabox/wiki/Http-Basic-Auth](https://github.com/geminabox/geminabox/wiki/Http-Basic-Auth)

* 执行如下命令启动gem server
```
rackup

```

默认端口为9292，则gem server地址为http://serverip:9292/。可以用rackup -p 80指定端口。

### 上传gem

```
gem inabox -g http://username:password@serverdomain gemname
```

* 例如：
```
安装actionmailer
gem inabox actionmailer-3.2.2.gem -g http://username:password@serverdomain

安装gems目录下所有gems
gem inabox gems/*  -g http://username:password@serverdomain
```
