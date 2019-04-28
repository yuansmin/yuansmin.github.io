---
layout: post
title:  "docker 挂载单文件的坑"
date:   2019-04-28 10:00:00 +0800
categories: [docker]
tags: [docker, mount]
description: 容器挂载了主机的文件，在主机上修改了容器里却没有同步，怎么回事？
---

容器挂载了主机的文件，在主机上修改了容器里却没有同步，怎么回事？

现场是这样的：
我起了个 nginx，用 -v 挂载了配置文件

``` shell
docker run -v default.conf:/etc/nginx/conf.d/default.conf -p 8080:80 -d nginx:1.14-alpine
```

然后在主机上修改了配置文件 default.conf，保存后重启容器发现没生效。

于是 exec 进入到容器，查看配置文件，咦，我刚改的配置呢？这里还是我刚启动的那一份配置。该配置文件在主机与容器间不同步！！！

检查了一遍自己的操作后，发现并没有错。只好上 google 寻求大神的帮助。

找到一个相关 issue： [https://github.com/moby/moby/issues/6011](https://github.com/moby/moby/issues/6011)

真相大白，原来是我用 vim 修改的锅。这里得说一下 docker -v mount 的机制：

> -v mount 文件(或文件夹)时，docker 记录的是该文件的 inode，并用 inode 追踪。当我用 vim 编辑了文件后，这个文件的 inode 就变了，也就是说这个 default.conf 文件已经不是我运行 docker run 时的哪个 default.conf 文件了，容器中自然也就没了我新的改动。
> 同时该 issue 的 opener 使用的是 sed -i 修改，也会使 inode 发生变化，sed -i 的机制是创建一个新的临时文件，修改完后在重命名。

对此，官方的建议是挂载文件夹，而不是文件。

同时有一个 PR 提交，添加挂载文件的注意事项。[https://github.com/moby/moby/pull/6854](https://github.com/moby/moby/pull/6854)，但现在文档里似乎已经找不到了。