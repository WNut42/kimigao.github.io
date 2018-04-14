---
layout: post
title: "如何用 Query Objects实现 ActiveRecord scopes"
date: 2018-04-14
comments: true
categories: [Ruby Rails]
---

对于 Rails 开发者来说，scopes 肯定是每天都会面对的，我们会将简单的查询声明为 scope，如果查询需要用到多个 scopes 的话，可以链式调用多个 scopes，查询十分方便。但是如果遇到比较复杂的 scope 时，我们应该重构？怎么让代码更加清晰呢？

我们可以一步步重构下如下的例子，来实现我们的目标：

```ruby
class Video < ActiveRecord::Base
  scope :featured_and_popular,
    	-> { where(featured: true).where('views_count > ?', 100) }
end
```

当然这个例子没那么复杂，不过不影响我们演示重构步骤。

上面👆这个 scope 可以简单的抽取到如下的 query object（query object 简单来说就是封装查询接口的类）。

```ruby
module Videos
  class FeaturedAndPopularQuery
    def initialize(relation = Video.all)
      @relation = relation
    end

    def featured_and_popular
      @relation.where(featured: true).where('views_count > ?', 100)
    end
  end
end
```

在 model 中调用的 scope 可以用如下代码来代替：

```ruby
Videos::FeaturedAndPopularQuery.new.featured_and_popular
```

由于我们可能已经在项目中多处使用 `Video.featured_and_popular` ，因此应该保证对外的接口不变，因此可以重构第一开始的 scope 为如下：

```ruby
class Video < ActiveRecord::Base
  scope :featured_and_popular,
        -> { Videos::FeaturedAndPopularQuery.new.featured_and_popular }
end
```

但是这段代码还是比较丑陋和繁琐的。

不过在继续重构之前，我们可以看一下 scope 方法的源码。

```ruby
def scope(name, body, &block)
  unless body.respond_to?(:call)
    raise ArgumentError, "The scope body needs to be callable."
  end

  if dangerous_class_method?(name)
    raise ArgumentError, "You tried to define a scope named \"#{name}\" " \
      "on the model \"#{self.name}\", but Active Record already defined " \
      "a class method with the same name."
  end

  if method_defined_within?(name, Relation)
    raise ArgumentError, "You tried to define a scope named \"#{name}\" " \
      "on the model \"#{self.name}\", but ActiveRecord::Relation already defined " \
      "an instance method with the same name."
  end

  valid_scope_name?(name)
  extension = Module.new(&block) if block

  if body.respond_to?(:to_proc)
    singleton_class.send(:define_method, name) do |*args|
      scope = all
      scope = scope.instance_exec(*args, &body) || scope
      scope = scope.extending(extension) if extension
      scope
    end
  else
    singleton_class.send(:define_method, name) do |*args|
      scope = all
      scope = scope.scoping { body.call(*args) || scope }
      scope = scope.extending(extension) if extension
      scope
    end
  end
end
```

通过👆代码可以看到，参数 body 可以是任何能响应 call 方法的对象，因此我们可以将上面的 query object 修改为如下代码：

```ruby
module Videos
  class FeaturedAndPopularQuery
    def initialize(relation = Video.all)
      @relation = relation
    end

    def call
      @relation.where(featured: true).where('views_count > ?', 100)
    end
  end
end
```

因此可以将 scope 声明中的 proc 去掉，将 query object 自己作为 scope 的 body 参数。

````ruby
class Video < ActiveRecord::Base
  scope :featured_and_popular, Videos::FeaturedAndPopularQuery.new
end
````

由于 Ruby 中 classes 也是 objects，所以可以给上面的查询类定义个 call 的类方法：

```ruby
module Videos
  class FeaturedAndPopularQuery
    class << self
      delegate :call, to: :new
    end

    def initialize(relation = Video.all)
      @relation = relation
    end

    def call
      @relation.where(featured: true).where('views_count > ?', 100)
    end
  end
end
```

因此最终代码可以简化为：

```ruby
class Video < ActiveRecord::Base
  scope :featured_and_popular, Videos::FeaturedAndPopularQuery
end  
```

经过以上的重构，model 中的代码就比较简洁和清晰了。

#### 参考文章
- [Essential RubyOnRails patterns — part 2: Query Objects](https://medium.com/@blazejkosmowski/essential-rubyonrails-patterns-part-2-query-objects-4b253f4f4539)
- [Delegating to Query Objects through ActiveRecord](http://craftingruby.com/posts/2015/06/29/query-objects-through-scopes.html)
