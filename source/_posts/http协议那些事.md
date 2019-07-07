---
title: http协议那些事
date: 2017-12-08 21:49:30
tags: http
categories: 前端
---
作为一个合格的前端，不了解http协议是不被允许的，给人感觉就是很low。
<!--more-->
------
# 当我们输入网址后发生了什么
1. 输入网址并回车
2. 解析域名
3. 浏览器发送HTTP请求
4. 服务器处理请求
5. 服务器返回HTML响应
6. 浏览器处理HTML页面
7. 继续请求其他资源
# 什么是HTTP协议
- HTTP是超文本传输协议，从www浏览器传输到本地浏览器的 一种传输协议，网站是基于HTTP协议的，例如网站的图片、 CSS、JS等都是基于HTTP协议进行传输的
- HTTP协议是由从客户机到服务器的请求(Request)和从服务器 到客户机的响应(response)进行约束和规范
# 在TCP/IP协议栈中的位置
![image](http://images.dogjun.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-04-08%20%E4%B8%8B%E5%8D%883.33.55.png)
- 目前应用版本HTTP 1.1
- HTTP默认端口号为80
- HTTPS默认端口号为443
# 请求与响应
- HTTP请求组成:请求行、消息报头、请求正文
- HTTP响应组成:状态行、消息报头、响应正文
- 请求行组成:以一个方法符号开头，后面跟着请求的URI和协 议的版本
- 状态行组成:服务器HTTP协议的版本，服务器发回的响应状态代码和状态代码的文本描述
# 请求报文
![image](http://images.dogjun.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-04-08%20%E4%B8%8B%E5%8D%883.35.52.png)
# 响应报文
![image](http://images.dogjun.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-04-08%20%E4%B8%8B%E5%8D%883.51.44.png)
# 请求方法
- GET: 请求获取Request-URI所标识的资源
- POST: 在Request-URI所标识的资源后附加新的数据
- HEAD: 请求获取由Request-URI所标识的资源的响应消息报头
- PUT: 请求服务器存储一个资源，并用Request-URI作为其标识
- DELETE: 请求服务器删除Request-URI所标识的资源
- TRACE: 请求服务器回送收到的请求信息，主要用于测试或诊断
- CONNECT: 保留将来使用
- OPTIONS: 请求查询服务器的性能，或者查询与资源相关的选项和需求
# HTTP状态码
状态代码有三位数字组成，第一个数字定义了响应的类别，且有五种可能取值：
- 1xx：指示信息--表示请求已接收，继续处理
- 2xx：成功--表示已被成功接收、理解、接受
- 3xx：重定向--要完成请求必须进行进一步的操作
- 4xx：客户端错误--请求有语法错误或请求无法实现
- 5xx：服务器端错误--服务器未能实现合法的请求
# 常用的请求报头
- Accept请求报头域用于指定客户端接受哪些类型的信息。eg:Accept:image/gif，Accept:text/ htmlAccept-Charset请求报头域用于指定客户端接受的字符集。Accept-Encoding:Accept-Encoding请求报头域类似于Accept，但是它是用于指定可接受的内容编码
- Accept-Language请求报头域类似于Accept，但是它是用于指定一种自然语言
- Authorization请求报头域主要用于证明客户端有权查看某个资源。当浏览器访问一个页面时，如果收 到服务器的响应代码为401(未授权)，可以发送一个包含Authorization请求报头域的请求，要求服务 器对其进行验证
- Host请求报头域主要用于指定被请求资源的Internet主机和端又号，它通常从HTTP URL中提取出来 的，发送请求时，该报头域是必需的
- User-Agent请求报头域允许客户端将它的操作系统、浏览器和其它属性告诉服务器
# 常用的响应报文
- Location响应报头域用于重定向接受者到一个新的位置。Location 响应报头域常用在更换域名的时候
- Server响应报头域包含了服务器用来处理请求的软件信息。与User- Agent请求报头域是相对应的
- WWW-Authenticate响应报头域必须被包含在401(未授权的)响应 消息中，客户端收到401响应消息时候，并发送Authorization报头 域请求服务器对其进行验证时，服务端响应报头就包含该报头域
# 实体报头
- 请求和响应消息都可以传送一个实体。一个实体由实体报头域 和实体正文组成，但并不是说实体报头域和实体正文要在一起 发送，可以只发送实体报头域。实体报头定义了关于实体正文 (eg:有无实体正文)和请求所标识的资源的元信息
# 常用的实体报头
- Content-Encoding实体报头域被用作媒体类型的修饰符，它的值指示了已经被应用到实体正文 的附加内容的编码，因而要获得Content-Type报头域中所引用的媒体类型，必须采用相应的解 码机制
- Content-Language实体报头域描述了资源所用的自然语言
- Content-Length实体报头域用于指明实体正文的长度，以字节方式存储的十进制数字来表示
- Content-Type实体报头域用语指明发送给接收者的实体正文的媒体类型
- Last-Modified实体报头域用于指示资源的最后修改日期和时间
- Expires实体报头域给出响应过期的日期和时间
# cookies和session
- Cookies是保存在客户端的小段文本，随客户端点每一个请求发送该url下的所有cookies到服务器端
- Session则保存服务器段，通过唯一的值sessionID来区别每一个用户。SessionID随每个连接请求发送到服务器，服务器根据sessionID来识别客户端，再通过session 的key获取session值
# 浏览器缓存机制
- 浏览器缓存机制 - 浏览器第一次请求
![image](http://images.dogjun.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-04-08%20%E4%B8%8B%E5%8D%883.42.52.png)
- 浏览器缓存机制 - 浏览器再次请求
![image](http://images.dogjun.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-04-08%20%E4%B8%8B%E5%8D%883.43.36.png)
#  Etag/If-None-Match策略
- Etag: web服务器响应请求时，告诉浏览器当前资源在服务器的唯一标识(生成规则由服务器决定)
- If-None-Match:当资源过期时(使用Cache-Control标识的max-age)，发现源具有Etage声明，则再次向web服务器请求时带 上头If-None-Match(Etag的值)。web服务器收到请求后发现有头If-None-Match 则与被请求资源的相应校验串进行比对，决 定返回200或304
# Last-Modified/If-Modified-Since策略
- Last-Modified:标示这个响应资源的最后修改时间。web服务器在响应请求时，告诉浏览器资源的最后修改时间
- If-Modified-Since:当资源过期时(使用Cache-Control标识的max-age)，发 现资源具有Last-Modified声明，则再次向web服务器请求时带上头 If-Modified-Since，表示请求时间。web服务器收到请求后发现有头If-Modifed- Since 则与被请求资源的最后修改时间进行比对。若最后修改时间较新，说明资源又被改动过，则响应整片资源内容(写在响应消息包体内)，HTTP200;若最后修改时间较旧，说明资源无新修改，则响应HTTP 304 (无需包体，节省浏览)，告知浏览器继续使用所保存的cache
# 保证HTTP的安全
- 加密数据
- 安全连接HTTPS协议
- HTTP2.0
