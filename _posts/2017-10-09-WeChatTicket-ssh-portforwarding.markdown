---
layout: post
title:  "微信抢票｜利用SSH端口转发实现内网穿透"
categories: 微信开发
tags: 微信 nginx SSH
author: thss15fyt
description: 用于在内网本地服务器进行开发和测试
---

由于某些原因，清华校内网不再允许外网访问，给微信项目的本地开发造成了一定困难，为此可以SSH端口转发的方法进行“内网穿透”，即主动向外网服务器发起请求建立SSH通道，在服务器建立socket监听某个端口，将数据通过SSH通道发送给内网的本地服务器，处理后经同样方法进行回传。

服务器端的配置方法见 [How to Set Up My First Server?](https://blog.magichc7.com/?p=5&from=timeline)

## STEP 1
修改服务器端nginx转发，将监听的80端口转发到闲置端口（以7000为例）:

/etc/nginx/conf.d/wct.conf:

    location / {
     proxy_pass http://127.0.0.1:7000;
    }

## STEP 2
本地运行Django项目（假设运行在8000端口），并建立SSH远程转发，命令格式为：

    ssh -R <remote port>:localhost:<local port> <username>@<host ip>

在本例中为：

    ssh -R 7000:localhost:8000 ubuntu@<your ip>

## STEP 3
经过以上两个步骤，即可通过服务器ip对本地运行的Django项目进行访问，但微信消息中的“绑定”、“帮助”等链接依然无法访问，这是因为获取的url仍为通过本地ip进行访问，为此需要对`config.json`中的`"SITE_DOMAIN"`进行修改：

    "SITE_DOMAIN": "http://<your_server_ip>"

## Q&A
**Q:** 建立ssh连接时遇到：

    Warning: remote port forwarding failed for listen port XXXX

**A:** 进入服务器，查询并结束占用端口的进程

    > sudo su
    > netstat -tunlp
    > kill -s 9 <PID>
