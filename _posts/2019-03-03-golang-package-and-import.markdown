---
layout: post
title:  "Golang Package And Import"
date:   2019-03-03 21:46:00 +0800
categories: [golang]
tags: [go, package]
description: 简单介绍 golang 的 package 与 文件夹的关系
---
本文简单介绍 golang 的 package 与 文件夹的关系

先看如下文件结构(位于 $GOPATH/src 目录下)
{% highlight text %}
├── bar
│   ├── balala
│   │   └── main.go
│   └── bar.go
└── foo
    ├── file-1.go
    ├── file-2.go
    └── foo.go
{% endhighlight %}

两个 package，分别是 `bar`， `foo`

foo/ 目录下的所有文件的 package 也可以写任何符合规范的名字，但必须相同。如 file-1.go 是 `package foo`, 其他文件也必须是 `package foo`

bar/bar.go 中 package 可以写任何名字。bar目录还有另一个 package: `balala`

### 如何 import
---
根据 package 的“导入路径”导入包。

何为导入路径, golang import path，即该包的文件夹相对于 `$GOPATH/src/` 的路径。
如：
{% highlight go %}
import (
    "bar"
    "bar/balala"
    "foo"
)
{% endhighlight %}

那 *.go 文件中的 `package` 是为何用？难道仅仅是为了与同一文件夹内的其他 go 文件的 `package` 保持一致？当然不是，这个是 package declaration, 用于在导入时建立 package 引用变量(用文件夹名不行吗？？？不太懂为什么这样设计)，导入后使用该 package 则是使用该 package name，而不是导入路径。如： foo 中 package 定义为 otherFoo
{% highlight go %}
package otherFoo

func Hello() {
    // xxx
}
// ...
{% endhighlight %}

则应该这样导入和使用
{% highlight go %}
import (
    "foo" // = otherFoo "foo", 默认的引用是 package name
)

func main() {
    otherFoo.Hello()  // package name 是 otherFoo
}
{% endhighlight %}

也可以在导入时定义一个新的引用
{% highlight go %}
import (
    foo "foo" // 建立 package 的引用 foo
)

func main() {
    foo.Hello()
}
{% endhighlight %}
