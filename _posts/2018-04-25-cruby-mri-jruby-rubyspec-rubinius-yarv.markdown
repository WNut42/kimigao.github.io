---
layout: post
title: "CRuby, MRI, JRuby, RubySpec, Rubinius, YARV: 这些名称的含义"
date: 2018-04-25
comments: true
categories: [Ruby]
---

本文翻译自：[CRuby, MRI, JRuby, RubySpec, Rubinius, YARV: A Little Bit of Ruby Naming](http://engineering.appfolio.com/appfolio-engineering/2017/12/28/cruby-mri-jruby-rubyspec-rubinius-yarv-a-little-bit-of-ruby-naming)

如果你已经使用 Ruby 一段时间了，那你肯定听说过 “CRuby” 或者 “MRI”。你应该也听说过 JRuby，估计也听说过一些“其他”的 Rubies 例如 Rubinius，TruffleRuby，再或者一些“异国风情”的 Rubies 例如 Opal，IronRuby，MacRuby 或者 MagLev。

那这些名称都是什么呢？

### CRuby（原来的 MRI）+ YARV

如果你使用 Ruby，那你肯定了解 CRuby， 即使你不知道这个名称。默认的 Ruby 就是 CRuby（被认为“只是Ruby”）。我们之前将“Matz 的 Ruby 解释器”成为 MRI。Matz（Ruby 的创造者）是一个谦虚的人并且 Ruby 是一个团队的成就，所以 Matz 要求我们将 MRI 称为 “CRuby”。由于 Ruby 解释器是用 C 语言写的，所以用 “CRuby” 非常合适。

CRuby 底层的实现经过了几代的技术变更。“YARV” 是 “Yet Another Ruby VM” 的简称。YARV 是一个栈基解释器，CRuby 1.9 到 2.5 一直在使用它。很久之前，它替换了 Ruby 1.8 老一代的”抽象语法树“解释器。现在 YARV 将被[新一代基于 JIT 的解释器/编译器技术（MJIT）](https://appfolio-engineering.squarespace.com/appfolio-engineering/2017/12/26/ruby-3-and-jit-where-when-and-how-fast)扩展。

Ruby 1.8，YARV 和 MJIT 都是 CRuby，只是代表了 CRuby 不同时期的技术：老一代的 Ruby 1.8 解释器，接着是 YARV，然后是 MJIT。

### 其他的 Rubies

“CRuby” 代表着用 C 写的 Ruby，为什么我们必须这么指定？所有的 Ruby 都是用 C 写的么？

当然不是！

JRuby 是用 Java 写的 Ruby 解释器。它被其他的团队编写和维护。JRuby 主要[关注高性能](https://github.com/jruby/jruby/wiki/PerformanceTuning)，特别是长期运行的服务例如 web 服务。它支持比较好的[并发性能](https://github.com/jruby/jruby/wiki/Concurrency-in-jruby)，尤其针对多线程。垃圾回收机制也比较先进，但是 JRuby 会占用更多的内存和需要更长的启动时间，所以不要用 JRuby 来编写小的命令行应用！并且 JRuby 要达到高效的速度，需要较长的”热身“。JRuby 对 Java 类库有很好的兼容性，但使用 C 类库经常有问题，而这些 C 类库在 CRuby 中则使用良好。JRuby 整体是个不同的语言项目，只是用来解释相同的源代码。

TruffleRuby 比较像 JRuby，它也使用 Java 编写（基于 Oracle 的编译器组件 [Truffle 和 Graal](https://github.com/graalvm/graal)）。它比较关注长期运行服务的高性能。它会占用更多的内存，花费更长的时间来到达高效速度，但是当它完全“热身”，它将变得更快。TruffleRuby 从 JRuby 中发展而来，不过现在已经是单独的项目了。

另一个主要的非CRuby的 “纯”Ruby 叫做 [Rubinius](https://rubinius.com/)。它是”Ruby in Ruby“，即使用 Ruby 自身编写，并且使用尽可能少的 C 扩展。因此，其他的 Rubies 例如 TruffleRuby 团队已经在使用它的标准库（比 C/Ruby 混和的更容易优化）。Rubinius 曾经使用过基于 LLVM 的 JIT 解释器，不过现在已经抛弃了。

还有其他一些 Ruby 实现，大部分比较老了。[OMR](https://github.com/rubyomr-preview/rubyomr-preview) 基于 IBM 的编译器/解释器工具箱，[IronRuby](http://ironruby.net/) 是基于 .NET 的 Ruby，[MacRuby](http://macruby.org/) 是可以使用 Objective-C 类库的 Ruby，[Opal](https://github.com/opal/opal) 用来将 Ruby 代码转为 JavaScript 代码，[MagLev](https://maglev.github.io/) 是实现跑在 SmallTalk 虚拟机上的 Ruby。但是其中 JRuby，TruffleRuby 和 Rubinius 是其中最大的三个非CRuby的实现。

注：MacRuby 最终废弃了，但是它的精神继承者 RubyMotion 存活了，一个可以用来编写跨平台的 Mac 和 智能手机 apps 的 Ruby 实现。

### RubySpec 和 什么是Ruby

我们怎么知道不同的 Ruby 实现可以“真正”的运行 Ruby 呢？如果两个实现出现不一致了怎么办？

简答回答是：语言规范说明。不只是那些正式公布的 Ruby 行业规范，也包含（对开发者更重要）一个巨大的可执行 Ruby 规范的测试集合，叫做 RubySpec，几乎所有的Ruby实现都会经过这些测试。

对 Ruby 语言的修改，转变为对 RubySpec 的修改。RubySpec 作为中心化的规范，所以其他 Ruby 实现都应该遵循它。Ruby 作为一门独立自主实现的语言是通过 RubySpec 来定义的。

### 命名

现在你知道了这些名称。最重要的是，你现在知道了不同的 Rubies 来做不同的事情，而且知道不同 Rubies 之间的不同之处。

如果你要指出“它是否是真实的 Ruby”，你也知道可以通过限定的规范说明来做到。
