---
layout: post
title:  "Set up Goaccess with Docker"
date:   2019-03-03 14:25:00 +0800
categories: [log-analysis]
tags: [docker, goaccess]
---
本文记录如何使用 docker 快速搭建一套 Goaccess 环境。

[Goaccess](https://github.com/allinurl/goaccess) 是一个简单、轻量、好用的 nginx 日志分析工具，日常简单分析还是不错的。[Goaccess 文档](https://goaccess.io)

准备环境：
{% highlight text %}
    1. 任何安装有 docker >= 17.03 的主机
    2. docker-compose >= 1.21.0
{% endhighlight %}

1. 创建如下目录结构
{% highlight text %}
goaccess/
├── data
│   └── access.log
│   └── goaccess.conf
├── docker-compose.yml
└── report
    └── index.html
{% endhighlight %}

/goaccess.conf: goaccess 配置文件 [详细文档](https://goaccess.io/man)

/docker-compose.yml: docker-compose 配置 [详细文档](https://docs.docker.com/compose/compose-file/)

/data/access.log: 待分析的 nginx 日志文件

/report/index.html: 分析出的报告文件，通过 nginx 访问

2. 拷贝如下配置到 goaccess.conf
{% highlight text %}
log-format COMMON
log-file /srv/data/access.log
output /srv/report/index.html
real-time-html true # 实时分析
{% endhighlight %}

3. 拷贝如下配置到 docker-compose.yml，并替换 \<path\> 为 goaccess 目录路径
{% highlight yaml %}
version: "2.2"
services:
  goaccess:
    image: allinurl/goaccess:latest
    command:
    - goaccess
    - --no-global-config
    - --config-file=/srv/data/goaccess.conf
    - --num-tests=0 # 避免因为nginx日志中有其他格式的行导致 goaccess 无法启动
    volumes:
    - /<path>/goaccess/data:/srv/data
    - /<path>/goaccess/report:/srv/report
    ports:
    - 7890:7890
  nginx:
    image: nginx:1.15-alpine
    volumes:
    - /<path>/goaccess/report:/usr/share/nginx/html
    ports:
    - 7891:80
{% endhighlight %}

4. 拷贝带分析的 nginx 日志到 /data/access.log，或直接实时输出日志到该文件。
5. 启动
{% highlight shell %}
cd /<path>/goaccess  # cd 到 goaccess 主目录
docker-compose up
# 启动并后台运行可添加 -d ： docker-compose up -d
{% endhighlight %}

如启动成功，则可通过 7891 端口访问：http://\<host-ip\>:7891
![compose-up](/assets/imgs/goaccess-docker-compose-up.png)

![goaccess-real-time-html](/assets/imgs/goaccess-real-time-html-gh.png)
如启动失败，根据输出错误信息和文档逐步排查。
