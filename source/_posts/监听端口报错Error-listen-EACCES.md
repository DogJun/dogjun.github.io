---
title: '监听端口报错Error: listen EACCES'
date: 2017-09-23 23:21:17
tags: javascript
categories: 前端
---
mac OS操作系统下，browserSync监听端口报错 Error: listen_EACCESS 0.0.0.0:80.
<!--more-->
------
![](../images/error_80.png)

原因： 监听1024以下端口，需要执行sudo命令
