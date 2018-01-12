---
layout: post
title: "简述Rails 4.1的新功能ActiveRecord enums"
date: 2014-04-20 11:11
comments: true
categories: [Rails]
---

本文翻译自如下：

#### [Rails 4.1 ActiveRecord enums](http://dev.mikamai.com/post/82355998967/rails-4-1-activerecord-enums)

原文很简单，但我为什么还要去翻译呢？ 其实以后我想多通过翻译一下blog来实现下面两个目的：

  * 学习英语
  * 通过翻译blog，能对其中涉及到的技术有更好的思考

所以先拿这边简单的blog来练手(给自己埋个大坑)。anyway，请看下面正文:


Rails 4.1最近发布了，新版本带来了很多新功能，其中最吸引人的要属ActiveRecord enums了，他提供了非常简单的方式在模型中去创建和使用属性的状态。

为了下面的说明，我们建立这样的用例： 我们的应用需要用户表(users)，并且每个用户存在一个状态(state)，这个状态可能是registered, active, blocked。

如果在以前，我们会怎么处理这样的任务呢？ 可能的做法是在users表中增加存储状态(state)的字段，然后在User写一堆的``scope method``来查询数据。不过现在enums给我们
提供了一种简单的方式：

首先，你需要写一个迁移文件来添加state字段到users表中：

``` ruby
class AddStatus < ActiveRecord::Migration
  def change
    add_column :users, :state, :integer
  end
end
```

并且在User类中调用enum的方法，如下：

``` ruby
class User
  enum state: [:registered, :active, :blocked]
end
```

通过具体代码来看一下，enums实现了哪些功能：

``` ruby
user = User.new
user.state
# => nil

user.registered?
# => false

user.state = :registered
user.registered?
# => true
```

能非常简单的通过一条命令来更新和保存记录的状态：

``` ruby
user.registered!
user.persisted?
# => true
user.registered?
# => true
```

针对每个state生成相应的``scope method``：

``` ruby
User.registered
# => #<ActiveRecord::Relation []>
User.active
# => #<ActiveRecord::Relation [#<User id: 7, status: 1...]>
```

可以使用enums生成的scope，根据指定的state创建记录：

``` ruby
User.active.create
# => #<User id: 6, status: 1, ...>
```

获取state字段真实的值，需要用``[]``方法：

``` ruby
user.state
# => "registered"

user[:state]
# => 0
```

通过上面的实例，我们可以知道enums是怎么实现的： ActiveRecord实际存储在数据库中的整数是对应了传递给``enum``方法的数组的下标。

可以在users表中增加state字段的默认值：

``` ruby
class ChangeStatus < ActiveRecord::Migration
  def change
    change_column :users, :status, :integer, default: 1
  end
end
```

这样我们在实例化User时，就会有相应的默认值：

``` ruby
user = User.new
user.state
# => 'active'
```

有很多保留的词，不能用来标识enum的标记，例如已经存在的字段名、存在的方法名，如果我们错误的使用了他们，将会有异常抛出：

``` ruby
class User
  enum state: [:logger]
end
# => ArgumentError: You tried to define an enum named "state" on the model "User", but this will generate a class method "logger", which is already defined by Active Record.
```

如果你对enum是怎么实现的感兴趣，可以查看github上的[代码](https://github.com/rails/rails/blob/master/activerecord/lib/active_record/enum.rb)和[测试](https://github.com/rails/rails/blob/master/activerecord/test/cases/enum_test.rb)。

ActiveRecord enums是很棒的功能，但是感觉还是很局限，我自己实现了个gem叫[clever_column](https://github.com/KimiGao/clever_column)，具体功能下一篇文章来介绍。
