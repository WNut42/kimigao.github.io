---
layout: post
title: "怎么组织 Go 代码"
date: 2018-03-05
comments: true
categories: [Go]
---

本文翻译自：[How to Write Go Code](https://golang.org/doc/code.html)

### 简述

这篇文章讲述了一个简易 Go 包的开发流程和go tool 的介绍，并描述了获取、编译和安装 Go 包的标准方法和命令。

go tool 需要用特定的方式来组织你的代码。请认真阅读本文档。它用最简单的方式来构建和运行 Go 程序。

### 代码组织

#### 综述

- Go 开发者一般保证他们的 Go 代码在一个单独工作区中
- 工作区包含多个版本控制仓库（例如 Git ）
- 每个仓库包含一个或多个包
- 每个包又由一个或多个在单个目录下 Go 源码文件组成
- 一个包的目录决定了它的 import path

注意：这种方式和其他编程环境不同，其他编程环境一般都是每个项目有单独的工作区，并且工作区和版本控制仓库紧密关联

#### 工作区

工作区有特定的目录结构，包含三个子目录，分别为：

- `src` 包含 Go 源文件
- `pkg` 包含 包对象
- `bin` 包含可执行命令

go tool 会构建源包和安装作为结果的二进制文件到 `pkg` 和 `bin` 目录中。

`src` 子目录一般包含多个版本控制仓库（例如 Git），方便跟踪多个源包的开发。

下面给出一个实际工作区目录结构：

```shell
bin/
    hello								# 可执行命令
	outyet								# 可执行命令
pkg/
    linux_amd64/
    	github.com/golang/example
			stringutil.a				# 包对象
src/
    github.com/golang/example
		.git/							# Git 仓库元数据
    	hello/
    		hello.go					# 命令源码
		outyet/
    		main.go						# 命令源码	
			main_test.go				# 测试源码
		stringutil/
    		reverse.go					# 包源码
			reverse_test.go				# 测试源码
	golang.org/x/image/
    	.git/							# Git 仓库元数据
    	bmp/
    		reader.go					# 包源码
			writer.go					# 包源码
	... (此处省略多个仓库和包)
```

上面的目录树展示了一个包含两个代码仓库的工作区（example 和 image ）。

- example 代码仓库包含两个命令（hello 和 outyet ）和一个包（stringutil）；
- image 目录包含 bmp 包

典型的工作区，包含多个源码仓库，源码仓库又包含多个包和命令。大多数 Go 开发者都将他们的 Go 源码和依赖放在一个工作区中。

命令和库是从多个不同的源包中构建的。

#### GOPATH 环境变量

GOPATH 环境变量指定了工作区的位置。它默认是你 home 目录下的 go 目录，例如 Unix 下的 $HOME/go，Plan 9 下的 \$home/go，Windows 下的 %USERPROFILE%\go 目录（通常是 C:\Users\YourName\go）。

如果你想设置不同的工作区径，你需要设置 GOPATH 为你想设定的目录路径。注意 GOPATH 不能和 Go 的安装路径相同。

`go env GOPATH` 命令可以打印出当前有效的 GOPATH；如果环境变量没有设置，它将会打印出默认路径。

方便起见，可以将工作区下的 bin 子目录添加到 PATH 环境变量中：

```shell
$ export PATH=$PATH:$(go env GOPATH)/bin
```

文档中其他脚本中使用 $GOPATH 代替了 \$(go env GOPATH)。如果没有设置 GOPATH，可以用 \$HOME/go 替换。

学习更新关于 GOPATH 环境变量的知识，请用此命令查看 `go help gopath`。

#### 导入路径

导入路径是唯一标识一个包的字符串。一个包的导入路径相当于它在工作区中的路径或者是远程仓库的路径（下面会解释）。

标准库中的包一般为简短的导入路径，例如 "fmt" 和 "net/http"。如果是自己的包，你需要设置基础路径应该尽量避免和标准库或者其他外部库冲突。

如果你的代码保存在某个代码仓库，那你可以使用这个代码仓库的根目录来作为你的基础路径。举个栗子，如果你在 github.com/user 下有个 GitHub 账号，那应该使用 github.com/user 作为你的基础路径。

注意：在构建代码之前，你不需要发布代码到远程仓库。但是如果你要在某天发布代码，好好组织代码是个好习惯。事实上，你可以选在任意路径名，只要保证它在标准库和庞大的 Go 生态系统中唯一即可。

我们将使用 github.com/user 作为基础路径。在工作区目录下创建如下目录来保存源代码：

```shell
$ mkdir -p $GOPATH/src/github.com/user
```

#### 你的第一个 Go 程序

为了编译和运行一个简单的程序，首先需要选择包路径（我们选择了 github.com/user/hello ），其次在工作区中创建相应的包目录：

```shell
$ mkdir $GOPATH/src/github.com/user/hello
```

下一步，在目录下创建源码文件 hello.go，其中包含如下 Go 源码：

```go
package main

import "fmt"

func main() {
    fmt.Printf("Hello, world!!\n")
}
```

现在可以用 go tool编译和安装这个程序了：

```bash
$ go install github.com/user/hello
```

注意：你可以在系统的任何位置执行这条命令。go 命令会通过 GOPATH 环境变量找到工作区，然后查找到工作区目录下的 github.com/user/hello 包下的源代码。

如果你是在包目录下，可以忽略包路径，直接运行 go install 命令：

```shell
$ cd $GOPATH/src/github.com/user/hello
$ go install
```

上面命令构建了 hello 命令，一个可执行的二进制文件。它被安装到了工作区的 bin 目录下，作为 hello （在 Windows 为 hello.exe）。

如果存在错误时，go 命令会打印出输出，所以这命令没有产生输入，则说明它们执行成功。

你可以在命令行中输入全路径来运行程序：

```shell
$ $GOPATH/bin/hello
Hello, world!!
```

如果你已经将 $GOPATH/bin 添加到 PATH 中，可以执行输入二进制文件名：

```shell
$ hello
Hello, world!!
```

如果你使用版本控制系统（例如 Git），现在正是你初始化仓库，添加文件，提交你的第一个包的好时候。当然，这一步是可选的：你编写 Go 代码不需要使用版本控制系统。

```shell
$ cd $GOPATH/src/github.com/user/hello
$ git init
Initialized empty Git repository in /home/user/work/src/github.com/user/hello/.git/
$ git add hello.go
$ git commit -m "initial commit"
[master (root-commit) 0b4507d] initial commit
 1 file changed, 1 insertion(+)
  create mode 100644 hello.go
```

#### 你的第一个库

让我们写一个库，并且在 hello 程序中使用它。

首先，选择包路径（我们使用 github.com/user/stringutil），其次创建相应路径：

```shell
$ mkdir $GOPATH/src/github.com/user/stringutil
```

下一步，在目录下创建 reverse.go 的源文件，源码如下：

```go
// Package stringutil contains utility functions for working with strings.
package stringutil

// Reverse returns its argument string reversed rune-wise left to right.
func Reverse(s string) string {
	r := []rune(s)
	for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
		r[i], r[j] = r[j], r[i]
	}
	return string(r)
}
```

现在，使用 go build 命令编译库包：

```shell
$ go build github.com/user/stringutil
```

如果你在库源目录下，可以直接执行：

```shell
$ go build
```

这个命令不会产生输出文件。如果要生成输出文件，你必须执行 go install 命令，它会将包对象生成到工作区的 pkg 目录下。

在确认了 stringutil 包构建成功后，修改 hello.go 源码（在 $GOPATH/src/github.com/user/hello目录下）:

```go
package main

import (
	"fmt"
	"github.com/user/stringutil"
)

func main() {
	fmt.Printf(stringutil.Reverse("!oG ,olleH"))
}
```

任何时候，go 命令在安装包或者二进制文件时，都会安装它依赖的任何包。所以当你安装 hello 程序时

```shell
$ go install github.com/user/hello
```

stringutil 包也将被自动安装。

执行新版本的程序，你将会看到新的，被反转的信息：

```shell
$ hello
Hello, Go!
```

经过了上面这些步骤后，你的工作区将会是如下：

```
bin/
    hello                 # command executable
pkg/
    linux_amd64/          # this will reflect your OS and architecture
        github.com/user/
            stringutil.a  # package object
src/
    github.com/user/
        hello/
            hello.go      # command source
        stringutil/
            reverse.go    # package source
```

注意go install 将 stringutil.a 放进了 pkg/darwin_amd64 文件夹下和代码对应的目录中。为了之后 go tool 就可以找到这个 package， 从而判断是否需要重新编译。darwin_amd64 是表示当前使用的系统, 它的目的是为了区分交叉编译出的其他平台的 package.

Go 编译出的二进制文件都是静态链接的， 所以上面的 bin/hello 在执行时并不需要 darwin_amd64/go-files/stringutil.a 文件。

#### Package names

Go 代码的第一行必须为

```go
package name
```

这里的 name 是包的默认名称，为了让其他包 import。（当前包下的所有文件必须使用相同的 name）

Go 的规范是包名是导入路径的最后一个元素：如果被导入的包名为 "crypto/rot13"，那包名将为 "rot13"。

编译为可执行文件的源码中，必须要使用 package main。

一个二进制文件链接的所有包的包名不需要唯一，只需要保证导入路径唯一即可（即他们的全路径）。

Go 的命名规范可以参考：https://golang.org/doc/effective_go.html#names

### 测试

Go 有个轻量级的测试框架，由 go test 命令和 testing 包组成。

测试文件必须要以 _test.go 结尾，其中需要 func TestXXX(t *testing.T) 格式的方法。测试框架执行每一个如此格式的方法；如果方法执行了 t.Error 或者 t.Fail 的错误方法，这个测试将被认为失败。

给 stringutil 包添加测试，创建 $GOPATH/src/github.com/user/stringutil/reverse_test.go 文件，源码如下：

```go
package stringutil

import "testing"

func TestReverse(t *testing.T) {
	cases := []struct {
		in, want string
	}{
		{"Hello, world", "dlrow ,olleH"},
		{"Hello, 世界", "界世 ,olleH"},
		{"", ""},
	}
	for _, c := range cases {
		got := Reverse(c.in)
		if got != c.want {
			t.Errorf("Reverse(%q) == %q, want %q", c.in, got, c.want)
		}
	}
}
```

然后运行测试命令 go test：

```shell
$ go test github.com/user/stringutil
ok  	github.com/user/stringutil 0.165s
```
如果是在包目录下，可以直接执行如下命令：

```shell
$ go test
ok  	github.com/user/stringutil 0.165s
```
执行 `go help test` 或者查看 [testing 包文档](https://golang.org/pkg/testing/) 学习更多细节。

### 远程包

导入路径可以描述怎样通过版本控制系统获取包源码，例如 Git。go tool 利用这个特性自动从远程仓库获取包源码。举个栗子，在这篇文档中的 examples 项目，Git 仓库就是存储在 GitHub 的 github.com/golang/example 地址上。
如果你在包的导入路径中包含了仓库 URL，go get 将获取，编译和安装这个包：

```shell
$ go get github.com/golang/example/hello
$ $GOPATH/bin/hello
Hello, Go examples!
```

如果工作区中不存在指定的包，那 go get 命令将把包下载到 GOPATH 中定义的第一个工作区中。（如果在工作去中已经存在包，go get 将跳过从远程仓库获取，行为和 go install 相同）。
在执行了上面 go get 命令后，工作区目录展示如下：

```shell
bin/
    hello                           # command executable
pkg/
    linux_amd64/
        github.com/golang/example/
            stringutil.a            # package object
        github.com/user/
            stringutil.a            # package object
src/
    github.com/golang/example/
	.git/                           # Git repository metadata
        hello/
            hello.go                # command source
        stringutil/
            reverse.go              # package source
            reverse_test.go         # test source
    github.com/user/
        hello/
            hello.go                # command source
        stringutil/
            reverse.go              # package source
            reverse_test.go         # test source
```

GitHub 上的 hello 命令依赖于同一个仓库的 stringutil 包。hello.go 文件中的导入路径使用了相同的导入路径规范，所以 go get 命令可以定位和安装依赖的包。

```go
import "github.com/golang/example/stringutil"
```

这个特性可以方便的让你的 Go 包被其他人使用。[Go Wiki](https://github.com/golang/go/wiki/Projects) 和 [godoc.org](http://godoc.org/) 上列出了很多第三方 Go 项目。
关于使用 go tool 来使用远程仓库的更多信息, 请参考: [go help importpath](http://golang.org/cmd/go/#hdr-Remote_import_paths)

### 下一步

订阅 [golang-announce](https://groups.google.com/group/golang-announce) 邮件列表来获取 Go 的发布消息。
查看  [Effective Go](https://golang.org/doc/effective_go.html) 来学习怎么书写简洁、地道的 Go 代码。
通过 [A Tour of Go](http://tour.golang.org/welcome/1) 来完成一次 go 的旅行
访问 [documentation page](http://golang.org/doc/#articles) 来了解一系列关于Go语言的有深度的文章, 以及 Go 库和工具.

### 获取帮助

寻求实时帮助, 可以使用 [FreeNode](http://freenode.net/) 的IRC server #go-nuts
Go 语言官方邮件列表 [Go Nuts](https://groups.google.com/forum/#!forum/golang-nuts)
汇报 Go 语言的 bug 请使用 [Go issue tracker](http://golang.org/issue)
