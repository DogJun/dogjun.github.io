---
title: nodejs深入学习笔记
date: 2018-03-08 22:21:14
tags: node
categories: 前端
---
之前对node只是学习了基础的api，知道express和koa框架怎么使用，最近对node进行了较为深入的学习，记录一些比较重要的知识点。
<!--more-->
------

# NodeJS异步IO原理浅析及优化方案
- 异步IO的好处
    - 前端可以通过异步IO可以消除UI阻塞
    - 假设请求资源A的时间为M，请求资源B的时间为N，那么同步的请求耗时为M+N，如果采用异步方式占用时间为Max（M，N）
    - 随着业务的复杂，会引入分布式系统，时间会线性的增加，M+N+...和Max(M,N,...)，这会放大同步和异步之间的差异
    - I/O是昂贵的，分布式I/O是更昂贵的
    - NodeJS适用于IO密集型不适用CPU密集型
- node如何实现一个sleep
    ```js
        async function test () {
            console.log('hello')
            await sleep(1000)
            console.log('world!')
        }
        function sleep (ms) {
            return new Promise(resolve => setTimeout(resolve, ms))
        }
        test()
    ```
- 内存管理与优化
    - V8垃圾回收机制
        - Node使用Javascript在服务端操作大内存对象受到了一定的限制，64位系统下约为1.4GB，32位操作系统下是0.7GB
        - Process.memoryUsage -> rss 进程常驻内存 heapTotal 已申请的堆内存 heapUsed 已使用的内存 （单位都是字节）
        - V8的垃圾回收策略主要基于分代式垃圾回收机制。在自动垃圾回收的演变过程中，人们发现没有一种垃圾回收算法能够胜任所有场景。V8中内存分为新生代和老生代两代。新生代为存活时间较短对象，老生代为存活时间较长的对象
    - Scavenge算法
        - 在分代基础上，新生代的对象主要是通过Scavenge算法进行垃圾回收，再具体实现时主要采用Cheney算法。Cheney算法是一种采用复制的方式实现的垃圾回收算法。它将内存一分为二，每一个空间称为semispace。这两个semispace中一个处于使用，一个处于闲置。处于使用的称之为From，闲置的称之为To，分配对象时先分配到From，当开始进行垃圾回收时，检查From存活对象复制到To。非存活被释放。然后互换位置。再次进行回收，发现被回收过直接晋升，或者发现To空间已经使用了超过25%。它的缺点是只能使用堆内存的一半，这是一个典型的空间换时间的办法，但是新生代声明周期较短，恰恰就适合这个算法。
        - ![image](http://p1cbbowoo.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-04-08%20%E4%B8%8B%E5%8D%883.06.09.png)
    - Mark-Sweep & Mark-compact
        - V8老生代主要采用Mark-Sweep和Mark-compact，在使用Scavenge不合适，一个是对象较多需要复制量太大而且还是没能解决空间问题。Mark-Sweep是标记清除，标记那些死亡的对象，然后清除。但是清除过后出现内存不连续的情况，所以我们要使用Mark-compact,它是基于Mark-Sweep演变而来的，它将活着的对象移到一边，移动完成后，直接清理边界外的内存。当CPU空间不足的时候会非常的高校。V8后续还引入了延迟处理，增量处理，并计划引入并行标记处理。
        - ![image](http://p1cbbowoo.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-04-08%20%E4%B8%8B%E5%8D%883.11.32.png)
    - 常见内存泄露问题
        - 无限制增长的数组
        - 无限制设置属性和值
        - 任何模块内的私有变量和方法均是永驻的内存的（需要手动 a=null）
        - 大循环，无GC机会
# PM2
- pm2是一个带有负载均衡功能的Node应用的进程管理器，可以把你的独立代码利用全部的服务器上的所有cpu，并保证进程永远都是活着，0秒的重载。
- 内建负载均衡（使用Node cluster集群模块）
- 后台运行
- 0秒停机重载
- 具有Ubuntu和CentOS的启动脚本
- 停止不稳定的进程（避免无限循环）
- 控制台检测
- 提供HTTP API
- 远程控制和实时的接口API（Nodejs模块，允许和PM2进程管理器交互）
