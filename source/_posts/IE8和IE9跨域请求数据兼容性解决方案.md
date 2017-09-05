---
title: IE8和IE9跨域请求数据兼容性解决方案
date: 2017-09-05 21:49:14
tags: 跨域
categories: 前端
---
IE8和IE9在实现跨域请求的时候使用XDomainRequest自己实现了一套，所以即使是使用jquery1.9.1版本也无法直接兼容IE8，IE9的跨域请求。
<!--more-->
------
# 解决方案

StackOverflow上对此问题的回答说得很清楚

<http://stackoverflow.com/questions/24206782/ajax-call-not-working-in-ie8>
<http://stackoverflow.com/questions/3362474/jquery-ajax-fails-in-ie-on-cross-domain-calls#11267937>

有两种解决方案

## 自己抽象定义一个ajax请求函数
这个方法在上面第二个链接已经有人做好了，但是我嫌有点麻烦。
## 重定义jquery的$. ajaxTransport方法
有人已经做好了，详见这里。<https://github.com/MoonScript/jQuery-ajaxTransport-XDomainRequest>
不过使用它有几点非常需要注意

- 只能是GET或者是POST请求
- POST请求的数据类型必须是text/plain
- 只支持HTPP和HTTPS协议
- 请求主体和被请求接口协议必须相同，也就是大家要么都是HTTP，要么都是HTTPS
- 必须是异步的
