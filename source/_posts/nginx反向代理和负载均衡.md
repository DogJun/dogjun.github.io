---
title: nginx反向代理和负载均衡
date: 2018-02-01 23:41:55
tags: nginx
categories: 服务器
---
web项目部署一般都少不了nginx做反向代理和负载均衡，最近搞了三台服务器玩了一下，对一些知识点做个总结记录...
<!--more-->
------
# 什么是反向代理
我们有时候，用自己的计算机想访问国外的某个网站B,但是访问不了，此时，又一个中间服务器C可以访问国外的网站B，那么，我们可以用自己的电脑访问服务器C，通过C来访问B这个网站。那么这个时候，服务器C称为代理服务器，这种访问方式叫做正向代理。正向代理有一个特点，就是我们明确知道要访问哪个网站。再如，当我们有一个服务器集群，并且服务器集群中的每台服务器的内容一样的时候，同样我们要直接从个人电脑访问到服务器集群中的服务器的时候无法访问，且此时第三方服务器能访问集群，这个时候，我们通过第三方服务器访问服务器集群的内容，但是此时我们并不知道是哪一台服务器提供的内容，此时的代理方式称为反向代理
# 什么是负载均衡
当一台服务器的单位时间内的访问量越大的时候，服务器的压力会越大。当一台服务器压力大得超过自身的承受能力的时候，服务器会崩溃。为了避免服务器崩溃，让用户有更好的体验，我们通常通过负载均衡的方式来分担服务器的压力。我们可以通过建立很多个服务器，这些服务器组成一个服务器集群，然后，当用户访问我们网站的时候，先访问一个中间服务器，再让这个中间服务器在服务器集群中选择一个压力较小的服务器，然后将该访问请求引入该选择的服务器。这样，用户的每次访问，都会保证服务器集群中的每个服务器的压力趋于平衡，分担了服务器压力，避免了服务器崩溃的情况。
# nginx负载均衡的实现
nginx是一款可以通过反向代理实现负载均衡的服务器，使用nginx服务实现负载均衡的时候，用户的访问首先会访问到nginx服务器，然后nginx服务器再从服务器集群表中选择压力较小的服务器，然后将该访问请求引向该服务器。若服务器集群中的某个服务器崩溃，那么从待选服务器列表中将该服务器删除，也就是说一个服务器假如崩溃了，那么nginx就肯定不会将访问请求引入该服务器了。
# http upstream模块
upstream模块是nginx服务器中的一个重要模块，upstream模块实现在轮询和客户端ip之间实现后端的负载均衡。常用的命令有ip_hash指令，server指令等
- ip_hash
在负载均衡系统中，假如用户在某台服务器上登陆，那么如果该用户第二次请求的时候，因为我们是负载均衡系统，每次请求都会重新定位到服务器集群中的一个服务器，那么此时如果将已经登陆服务器A的用户再定位到其他服务器，显然不妥。故而，我们可以采用ip_hash指令解决这个问题，如果客户端请求已经访问了服务器A并登陆，那么第二次请求的时候，会将该请求通过哈希算法自动定位到该后端服务器中。
- server指令
这个指令主要用于指定服务器的名称和参数
- 安装nginx源
```
rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```
- 安装nginx
```
yum install -y nginx
```
- nginx默认目录
```
whereis nginx
```
输出
```
nginx: /usr/sbin/nginx /usr/lib64/nginx /etc/nginx /usr/share/nginx /usr/share/man/man8/nginx.8.gz
```
- nginx默认路径
    - nginx配置路径：/etc/nginx/
    - PID目录：/var/run/nginx.pid
    - 错误日志：/var/log/nginx/error.log
    - 访问日志：/var/log/nginx/access.log
    - 默认站点目录：/usr/share/nginx/html  
事实上，只需知道Nginx配置路径，其他路径均可在/etc/nginx/nginx.conf 以及/etc/nginx/conf.d/default.conf 中查询到。
- 常用命令
    - 启动
    ```
    nginx
    ```
    - 测试nginx配置是否正确
    ```
    nginx -t
    ```
    - 重启
    ```
    nginx -s reload
    ```
# 一个简单的nginx配置
```
worker_processes 4;
events {
    worker_connections 1024;
}
http {
    upstream firsttest {
        ip_hash;
        server 192.168.0.1 weight=2;
        server 192.168.0.2;
    }
    server {
        listen 8080;
        location / {
            proxy_pass http://firsttest;
        }
    }
}
```
