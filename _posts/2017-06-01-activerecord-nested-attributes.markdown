---
layout: post
title: "ActiveRecord 嵌套属性"
date: 2017-06-01
comments: true
categories: [Ruby Rails]
---

在项目中我们经常遇到在一个表单中同时保存当前对象和对应的关联对象的情形，针对这种情形，Rails 提供了 ActiveRecord 嵌套属性的方式来方便应对。

嵌套属性允许在当前记录保存时，同时保存关联记录的属性。默认情况下，嵌套属性的更新是被关闭的，如果要开启，需要使用 `#accepts_nested_attributes_for` 类方法。当开启嵌套属性时，会在当前 model 中定义属性的写方法。

下面的代码实例会在当前 model 中生成如下两个实例方法：

`author_attributes=(attributes)` 和 `pages_attributes=(attributes)`

```ruby
class Book < ActiveRecord::Base
  has_one :author
  has_many :pages
    
  accepts_nested_attributes_for :author, :pages
end
```

### One-to-one

指定一个 member 存在一个 avatar：

```ruby
class Member < ActiveRecord::Base
  has_one :avatar
  accepts_nested_attributes_for :avatar
end
```

在一对一关联关系上开启嵌套属性后，可以用如下方式方便创建 member 和 avatar：

```ruby
params = { member: { name: 'Jack', avatar_attributes: { icon: 'smiling' } } }
member = Member.create(params[:member])
member.avatar.id # => 2
member.avatar.icon # => 'smiling'
```

也可以用如下方式，通过更新 member 来更新 avatar：

```ruby
params = { member: { avatar_attributes: { id: '2', icon: 'sad' } } }
member.update params[:member]
member.avatar.icon # => 'sad'
```

默认情况下，只能保存和更新关联模型上的属性，如果要通过属性哈希删除关联模型，你需要开启 `:allow_destroy` 设置项。

```ruby
class Member < ActiveRecord::Base
  has_one :avatar
  accepts_nested_attributes_for :avatar, allow_destroy: true
end
```

现在，当在属性哈希中增加 `_destroy` 键，并且值为 `true` （例如 1, '1', true, 'true'） 时，就可以删除关联模型了：

```ruby
member.avatar_attributes = { id: '2', _destroy: '1' }
member.avatar.marked_for_destruction? # => true
member.save
member.reload.avatar # => nil
```

注意：

- 只有在当前模型保存后，关联的模型才会被删除。
- 只有在更新的属性哈希中指定了`id` ，关联模型才会被删除。

### One-to-many

指定一个 member 存在多个 posts：

```ruby
class Member < ActiveRecord::Base
  has_many :posts
  accepts_nested_attributes_for :posts
end
```

现在可以通过更新 member 来更新 post 的信息了，只需要在 member 的属性哈希中增加 `:posts_attributes` 的键，同时值为包含 post 属性哈希的数组。

post 属性哈希中，没有 `id` 键的记录会被创建；存在 `id` 键的记录会被认为是已存在记录而更新；如果属性哈希中存在 `_destroy` 键，并且值等于 `true` （例如 1, '1', true, 'true'），则该记录会被删除。

```ruby
params = { member: {
  name: 'joe', posts_attributes: [
    { title: 'Kari, the awesome Ruby documentation browser!' },
    { title: 'The egalitarian assumption of the modern citizen' },
    { title: '', _destroy: '1' } # this will be ignored
  ]
}}

member = Member.create(params[:member])
member.posts.length # => 2
member.posts.first.title # => 'Kari, the awesome Ruby documentation browser!'
member.posts.second.title # => 'The egalitarian assumption of the modern citizen'
```

我们可以通过设置 `:reject_if` 为代码块，当代码块执行返回为 `false` 时，关联模型新纪录的创建将会被忽略，具体请看如下实例：

```ruby
class Member < ActiveRecord::Base
  has_many :posts
  accepts_nested_attributes_for :posts, reject_if: proc { |attributes| attributes['title'].blank? }
end

params = { member: {
  name: 'joe', posts_attributes: [
    { title: 'Kari, the awesome Ruby documentation browser!' },
    { title: 'The egalitarian assumption of the modern citizen' },
    { title: '' } # this will be ignored because of the :reject_if proc
  ]
}}

member = Member.create(params[:member])
member.posts.length # => 2
member.posts.first.title # => 'Kari, the awesome Ruby documentation browser!'
member.posts.second.title # => 'The egalitarian assumption of the modern citizen'
```

同时，`:reject_if` 可以接受可用方法的 symbol 串，具体如下：

```ruby
class Member < ActiveRecord::Base
  has_many :posts
  accepts_nested_attributes_for :posts, reject_if: :new_record?
end

class Member < ActiveRecord::Base
  has_many :posts
  accepts_nested_attributes_for :posts, reject_if: :reject_posts

  def reject_posts(attributes)
    attributes['title'].blank?
  end
end
```

如果属性哈希中包含 `id` 键时，将会找到对应已经存在的关联记录，并且更新：

```ruby
member.attributes = {
  name: 'Joe',
  posts_attributes: [
    { id: 1, title: '[UPDATED] An, as of yet, undisclosed awesome Ruby documentation browser!' },
    { id: 2, title: '[UPDATED] other post' }
  ]
}

member.posts.first.title # => '[UPDATED] An, as of yet, undisclosed awesome Ruby documentation browser!'
member.posts.second.title # => '[UPDATED] other post'
```

默认情况下，关联记录是不会被删除的。如果要想通过属性哈希来删除关联记录的话，需要开启 `:allow_destroy` 设置项。这样可以通过设置 `_destory` 键来删除已经存在的记录：

```ruby
class Member < ActiveRecord::Base
  has_many :posts
  accepts_nested_attributes_for :posts, allow_destroy: true
end

params = { member: {
  posts_attributes: [{ id: '2', _destroy: '1' }]
}}

member.attributes = params[:member]
member.posts.detect { |p| p.id == 2 }.marked_for_destruction? # => true
member.posts.length # => 2
member.save
member.reload.posts.length # => 1
```

注意：

- 只有在当前模型保存后，关联的模型才会被删除。
- 只有在更新的属性哈希中指定了`id` ，关联模型才会被删除。
