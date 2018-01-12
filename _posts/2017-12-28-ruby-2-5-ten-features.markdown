---
layout: post
title: "Ruby 2.5 的十个新特性"
date: 2017-12-28
comments: true
categories: [Ruby]
---

### rescue/else/ensure 可以不用 begin/end 即可加入 do/end 代码块中

Ruby 2.5 以后可以在 do/end 代码块中写不带 begin/end 关键词的 rescure/else/ensure 子句：

```ruby
[1,2,3].each do |n|
  n / 0
rescue
  # rescue
else
  # else
ensure
  # ensure
end
```

但如果使用 {} 代码块，会得到如下错误：

```ruby
[1].each { |n|
  n / 0
rescue
  # rescue
else
  # else
ensure
  # ensure
}
#=> SyntaxError: (irb):3: syntax error, unexpected keyword_rescue, expecting '}'
#     rescue
#     ^~~~~~
```

* [Feature #12906: do/end blocks work with ensure/rescue/else](https://bugs.ruby-lang.org/issues/12906)



### 顶层常量查找被移除

Ruby 2.4 中，如下代码可以正常执行，但会有个 warning：

```ruby
class Staff; end
class ItemsController; end
 
Staff::ItemsController
#=> warning: toplevel constant ItemsController referenced by Staff::ItemsController
#=> ItemsController
```

这是因为顶层常量被定义在 Object 中，Ruby 在 Staff 中查不到 ItemsController 时，会在 Staff 的 superclass 中查找，最终会在 Object 查找到 ItemsController（Object 为 Staff 的 superclass），具体细节参考如下文章：

- [Why you Should not use a Class as a Namespace in Rails Applications](https://blog.jetbrains.com/ruby/2017/03/why-you-should-not-use-a-class-as-a-namespace-in-rails-applications/)

但在 Ruby 2.5 中，Ruby 不会再查找 superclass，所以之前的代码将会报错：

```ruby
Staff::ItemsController
#=> NameError: uninitialized constant Staff::ItemsController
#   Did you mean?  ItemsController
```

- [Feature #11547: remove top-level constant lookup](https://bugs.ruby-lang.org/issues/11547)



### Bundler 集成到 Ruby 标准库

Bundler 被集成到了 Ruby 标准库，不需要再单独执行 ```gem install bundler```

- [Feature #12733: Bundle bundler to ruby core](https://bugs.ruby-lang.org/issues/12733)



### 反序打印追踪日志和错误消息

在 Ruby 2.5 中，backtrace 和 error message 会被反序打印出来，举个栗子：

Ruby 2.4:

```ruby
$ ruby ./test/error_example.rb
./test/error_example.rb:7:in `/': divided by 0 (ZeroDivisionError)
    from ./test/error_example.rb:7:in `method_2'
    from ./test/error_example.rb:2:in `method_1'
    from ./test/error_example.rb:10:in `'
```

Ruby 2.5:

```ruby
$ ruby ./test/error_example.rb
Traceback (most recent call last):
        3: from ./test/error_example.rb:10:in `'
        2: from ./test/error_example.rb:2:in `method_1'
        1: from ./test/error_example.rb:7:in `method_2'
./test/error_example.rb:7:in `/': divided by 0 (ZeroDivisionError)
```

- [Feature #8661: Add option to print backstrace in reverse order(stack frames first & error last)](https://bugs.ruby-lang.org/issues/8661)



### Kernel#yield_self

Kernel#yield_self 被引入，这个方法将消息接收方作为代码块参数，并将代码块的值当做返回值：

```ruby
2.yield_self { |n| n * 10 } #=> 20
 
names = ['Alice', 'Bob']
names.join(', ').yield_self { |s| "(#{s})" } #=> "(Alice, Bob)"
```

- [Feature #6721: Object#yield_self](https://bugs.ruby-lang.org/issues/6721)



### String#delete_prefix/delete_suffix

String#delete_prefix/delete_suffix 可以去掉 string 的前缀和后缀：

```ruby
'invisible'.delete_prefix('in') #=> "visible"
'pink'.delete_prefix('in') #=> "pink"
 
'worked'.delete_suffix('ed') #=> "work"
'medical'.delete_suffix('ed') #=> "medical"
```

- [Feature #12694: Want a String method to remove heading substr](https://bugs.ruby-lang.org/issues/12694)
- [Feature #13665: String#delete_suffix](https://bugs.ruby-lang.org/issues/13665)



### Array#prepend/append 作为 unshift/push 别名

添加 Array#prepend/append 作为 unshift/push 的别名：

```ruby
array = [3, 4]
array.prepend(1, 2) #=> [1, 2, 3, 4]
array               #=> [1, 2, 3, 4]
 
array = [1, 2]
array.append(3, 4)  #=> [1, 2, 3, 4]
array               #=> [1, 2, 3, 4]
```

- [Feature #12746: class Array: alias .prepend to .unshift ?](https://bugs.ruby-lang.org/issues/12746)



### Hash#transform_keys/transform_keys!

Hash#transform_keys 根据代码块的值修改 Hash 的 keys：

```ruby
hash = { a: 1, b: 2 }
hash.transform_keys { |k| k.to_s }
#=> { 'a' => 1, 'b' => 2 }

hash
#=> { a: 1, b: 2 }
```

transform_keys! 会改变 hash 本身

```ruby
hash = { a: 1, b: 2 }
hash.transform_keys! { |k| k.to_s }
#=> { 'a' => 1, 'b' => 2 }
 
hash
#=> { 'a' => 1, 'b' => 2 }
```

- [Feature #13583: Adding `Hash#transform_keys` method](https://bugs.ruby-lang.org/issues/13583)



### Dir.children/each_child

大家可能用过 `Dir.entries` 方法：

```ruby
Dir.entries('./test/dir_a')
#=> [".", "..", "code_a.rb", "text_a.txt"]
```

如果想从返回结果中删除 "." 和 ".." ，可以用 `Dir.children` 代替：

```ruby
Dir.children('./test/dir_a')
#=> ['code_a.rb', 'text_a.txt']
```

`Dir.each_child` 返回 `Enumerator` 对象而不是数组：

```ruby
Dir.each_child('./test/dir_a')
#=> #<Enumerator: Dir:each_child(\"./test/dir_a\")>"
 
Dir.each_child('./test/dir_a').to_a
#=> ['code_a.rb', 'text_a.txt']
```

- [Feature #11302: Dir.entries and Dir.foreach without [“.”, “..”\]](https://bugs.ruby-lang.org/issues/11302)



### ERB#result_with_hash

如下代码展示了如何在 Ruby 2.4 中在 ERB 模板中定义局部变量：

```ruby
require 'erb'
require 'ostruct'
 
namespace = OpenStruct.new(a: 2, b: 3)
template = 'Result: <%= a * b %>'
ERB.new(template).result(namespace.instance_eval { binding }) #=> "Result: 6"
```

但是在 Ruby 2.5 中，你可以用 ERB#result_with_hash 重写：

```ruby
require 'erb'
 
template = 'Result: <%= a * b %>'
ERB.new(template).result_with_hash(a: 2, b: 3) #=> "Result: 6"
```

- [Feature #8631: Add a new method to ERB to allow assigning the local variables from a hash](https://bugs.ruby-lang.org/issues/8631)



### 其他修改

大家可以在如下页面找到 Ruby 2.5 的其他修改：[NEWS for Ruby 2.5.0](https://github.com/ruby/ruby/blob/v2_5_0_preview1/NEWS)

