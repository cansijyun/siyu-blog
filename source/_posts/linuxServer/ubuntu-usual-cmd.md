---
title: Linux常用命令和常用包
date: 19-03-03 20:19:47
tags:
categories: Linux和服务端
---
## Nginx
`sudo apt-get install nginx`

`cd /etc/nginx/sites-available`

可以随意去一个nginx设置文件夹

`vim nginx_conf`

`sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled`

看看有没有错误

`sudo nginx -t`

`sudo systemctl restart nginx`