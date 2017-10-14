---
layout: post
title:  "微信抢票｜在服务器端部署项目的发布版本"
categories: 微信开发
tags: 微信 Django uwsgi nginx
author: thss15fyt
description: 提供了在Django和uwsgi服务器部署方法
---

在服务器端部署发布项目时，除项目本身需要配置外，还需要更根据Web服务器的不同进行配置，
以下给出了使用Django自带服务器和uwsgi服务器的配置方法，均适用nginx进行转发。

## 项目配置
1.修改`config.json`文件

    "DEBUG": false,
    "IGNORE_WECHAT_SIGNATURE": false,

2.修改`WeChatTicket/settings.py`文件，关闭`TEMPLATES`中的`APP_DIRS`

    TEMPLATES = [
      {
        ...
        'APP_DIRS': False,
        ...
        },
    ]

## 使用Django服务器
1.修改`/etc/nginx/conf.d/wct.conf`文件

    location / {
     root /home/ubuntu/WeChatTicket/static;
     }
    location /wechat {
     proxy_pass http://127.0.0.1:8000;
     }
    location /api {
     proxy_pass http://127.0.0.1:8000;
    }
    
2.在8000端口运行Django服务器

    python manage.py runserver 0:8000

也可以使进程在后台运行

    (python manage.py runserver &)

需要关闭服务器进程时

    > lsof -i :8000
    > kill -9 <PID>

3.重启nginx服务

    > touch reload
    > service nginx restart

## 使用uwsgi服务器
1.修改`/etc/nginx/conf.d/wct.conf`文件

    location / {
     root /home/ubuntu/WeChatTicket/static;
     }
    location /wechat {
     uwsgi_pass unix:///home/ubuntu/WeChatTicket/wct.sock;
     include /etc/nginx/uwsgi_params;
     }
    location /api {
     uwsgi_pass unix:///home/ubuntu/WeChatTicket/wct.sock;
     include /etc/nginx/uwsgi_params;
    }

2.重启uwsgi

    > sudo su
    > ps -aux | grep uwsgi
    > kill -9 <PID>
    > exit

    > uwsgi --ini uwsgi.ini

3.重启nginx

    > touch reload
    > service nginx restart
