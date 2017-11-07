---
title: CentOS 7 搭建ngrok服务器
date: 2017-09-25 10:22:57
tags: ngrok
categories: 微信开发
---
ngrok官方服务被墙了，花生壳开始收费了，免费版不稳定且有流量限制，项目中涉及到微信开发，只有自己搭建ngrok服务器做内网穿透。
<!--more-->
------
# 前提条件
需要一台云服务器和一个域名解析到该IP
# 环境安装
1、安装 gcc
```ssh
yum install gcc
```
2、安装 git
```ssh
yum install git
```
3、安装 go 语言环境

到网站https://golang.org/dl/查找最新的版本链接

下载
```ssh
wget https://storage.googleapis.com/golang/go1.8.linux-amd64.tar.gz
```
解压安装：
```ssh
tar -C /usr/local/ -zxvf go1.8.linux-amd64.tar.gz
```
添加环境变量,编辑：vi /etc/profile，在最后添加：
```ssh
#go lang
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin
```
使环境变量生效
```ssh
source /etc/profile
```
检查是否安装成功：
```ssh
go version
```
输出：go version go1.8 linux/amd64表示安装成功
# 在服务器搭建 ngrok 服务
1、下载 ngrok 源码
```ssh
cd /usr/local/src
git clone https://github.com/inconshreveable/ngrok.git
```
2、生成证书
在自生成证书时需要一个解析到服务器上的主域名，现在以"dogjun.com"为例:
```ssh
cd ngrok
openssl genrsa -out rootCA.key 2048
openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=dogjun.com" -days 5000 -out rootCA.pem
openssl genrsa -out device.key 2048
openssl req -new -key device.key -subj "/CN=dogjun.com" -out device.csr
openssl x509 -req -in device.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out device.crt -days 5000
```
将新生成的证书，替换掉assets/client/tls下的证书
```ssh
yes|cp rootCA.pem assets/client/tls/ngrokroot.crt
yes|cp device.crt assets/server/tls/snakeoil.crt
yes|cp device.key assets/server/tls/snakeoil.key
```
3、编译生成ngrokd（服务端）
```ssh
#这里是交叉编译，linux系统GOOS=linux,64位系统GOARCH=amd64，32位系统GOARCH=386
#当前系统可用go env查看
GOOS=linux GOARCH=amd64 make release-server
```
编译成功后在当前目录的bin目录下可找到ngrokd文件

启动服务端(/usr/local/src/ngrok目录下)
```ssh
./bin/ngrokd -tlsKey="assets/server/tls/snakeoil.key" -tlsCrt="assets/server/tls/snakeoil.crt" -domain="uboff.com"  -httpAddr=":80" -httpsAddr=":8081" -tunnelAddr=":4443"
```
后台运行
```ssh
nohup ./bin/ngrokd -tlsKey="assets/server/tls/snakeoil.key" -tlsCrt="assets/server/tls/snakeoil.crt" -domain="ngrok.dogjun.com"  -httpAddr=":80" -httpsAddr=":8082" -tunnelAddr=":4443" &
```
出现下面信息，启动成功
```ssh
[14:52:23 CST 2017/03/18] [INFO] (ngrok/log.(*PrefixLogger).Info:83) [registry] [tun] No affinity cache specified
[14:52:23 CST 2017/03/18] [INFO] (ngrok/log.Info:112) Listening for public http connections on [::]:80
[14:52:23 CST 2017/03/18] [INFO] (ngrok/log.Info:112) Listening for public https connections on [::]:8081
[14:52:23 CST 2017/03/18] [INFO] (ngrok/log.Info:112) Listening for control and proxy connections on [::]:4443
[14:52:23 CST 2017/03/18] [INFO] (ngrok/log.(*PrefixLogger).Info:83) [metrics] Reporting every 30 seconds
```
4、编译生成ngrok（客户端）
这里生成windows下的客户端：
```ssh
GOOS=windows GOARCH=amd64 make release-client
```
成功会在bin目录下看到windows_amd64文件夹，复制到windows电脑上即可启动
在windows_amd64目录下新建一个ngrok.cfg文件，内容如下：
```ssh
server_addr: "ngrok.dogjun.com:4443"
trust_host_root_certs: false
```
然后就可以启动客户端,我已经把windows_amd64文件夹下载到E盘下，打开CMD输入：
```ssh
ngrok -subdomain hwj -config=ngrok.cfg 80
```
看到下面信息则启动成功：
```ssh
Tunnel Status                 online
Version                       1.7/1.7
Forwarding                    http://hwj.ngrok.dogjun.com -> 127.0.0.1:80
Forwarding                    https://hwj.ngrok.dogjun.com -> 127.0.0.1:80
Web Interface                 127.0.0.1:4040
# Conn                        1
Avg Conn Time                 0.00ms
```
