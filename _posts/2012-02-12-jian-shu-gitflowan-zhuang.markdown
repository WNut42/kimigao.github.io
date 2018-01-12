---
layout: post
title: "简述gitflow安装"
date: 2012-02-12 00:43
comments: true
categories: [git, gitflow]
---

gitflow是非常优秀的git分支模型，算是git的最佳实践了。

### gitflow的安装
```
$ wget --no-check-certificate -q -O - https://github.com/nvie/gitflow/raw/develop/contrib/gitflow-installer.sh | sudo bash
```

### gitflow自动补全

1、下载git-flow-completion
```
$ git clone git://github.com/bobthecow/git-flow-completion.git
```

2、配置git-completion
```
$ sudo apt-get install git-core bash-completion
```

3、配置git-flow-completion 
```
$ cd git-flow-completion
$ cp git-flow-completion.bash ~/.git-flow-completion.sh
```
在~/.bashrc中加入
```
$ source ~/.git-flow-completion.sh
```
然后执行
```
$ source .bashrc
```

以上是在ubuntu下安装。

### 参考地址

#### [https://github.com/nvie/gitflow](https://github.com/nvie/gitflow)
#### [https://github.com/bobthecow/git-flow-completion](https://github.com/bobthecow/git-flow-completion)
