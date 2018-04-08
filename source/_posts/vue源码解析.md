---
title: vue源码解析
date: 2018-02-24 23:35:17
tags: vue
categories: 前端
---
最近项目没有那么忙，想着研究下vue源码，下周给团队的成员share一下，帮助大家更好的做项目，把总结的笔记记录一下。
<!--more-->
------

# vue架构概览
![image](http://p1cbbowoo.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-04-01%20%E4%B8%8B%E5%8D%8810.39.47.png)
- /complier目录是编译模板
- /core目录是Vue.js的核心
- /entries目录是生产打包的入口
- /platforms目录是针对核心模块的平台模块
- /server目录是处理服务器端渲染
- /sfc目录是处理单文件.vue
- /shared目录提供全局用到的工具函数

![image](http://p1cbbowoo.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-04-01%20%E4%B8%8B%E5%8D%8810.50.32.png)
- compents 模板编译的代码
- global-api 最上层的文件接口
- instance 生命周期
- observer 数据收集与订阅
- util 常用工具方法类
- vdom 虚拟dom

结论: Vue.js 的组成是由 core + 对应的 ‘平台’ 补充代码构成(独立构建和运行时构建 只是 platforms 下 web 平台的两种选择)。
![image](http://p1cbbowoo.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-04-01%20%E4%B8%8B%E5%8D%8810.49.57.png)

# 双向数据绑定
## 双向绑定（响应式）所涉及到的技术
- Object.defineProperty
- Observer
- Watcher
- Dep
- Directive

### Object.defineProperty
```js
var obj = {}
var a
Object.defineProperty(obj, 'a', {
    get: function () {
        console.log('get val')
        return a
    },
    set: function () {
        console.log('set val:' + newVal)
        a = newVal
    }
})
obj.a // get val
obj.a = '111' // set val: 111
```
注：vue不支持IE9以下，是因为浏览器不支持Object.defineProperty，而这是vue核心利用的技术，如果想要向下兼容，则需要使用VBScript，VBbScript很早就有class的实现，这样就能拥有get和set方法。还有就是，很早的时候，一些浏览器拥有 （__defineGetter__）和 __defineSetter__ 方法，来模拟set和get，来实现双向数据绑定。

### Observer
观察者模式是软件设计模式的一种。在此种模式中，一个目标对象管理所有相依于它的观察者对象，并且在它本身的状态改变时主动发出通知。这通常透过呼叫各观察者所提供的 方法来实现。此种模式通常被用来实时事件处理系统。  订阅者模式涉及三个对象:发布者、主题对象、订阅者，三个对象间的是一对多的关系，每当主题对象状态发生改变时，其相关依赖对象都会得到通知，并被自动更新。看一个简 单的示例:
```js
// jquery经典实现
(function ($) {
    var o = $({})
    $.subscribe = function () {
        o.subscribe = function () {
            o.on.apply(o, arguments)
        }
        o.unsubscribe = function () {
            o.off.apply(o, arguments)
        }
        o.publish = function () {
            o.trigger.apply(o, arguments)
        }
    }
})(jQuery)

// 订阅
$.subscribe('/some/topic', function (e,a,b,c) {
    console.log(a + b + c)
})
// 发布
$.publish('some/topic', ['a', 'b', 'c']) // 输出abc
// 退订
$.unsubscribe('some/topic')
```
```js
// 主题对象
function Dep () {
    this.subs = [] // 订阅列表
}
// 主题对象通知订阅者
Dep.prototype.notify = function () {
    // 遍历所有的订阅者，执行订阅者提供的更新方法
    this.subs.forEach(function (sub) {
        sub.update()
    })
}

// 订阅者
function Sub (x) {
    this.x = x
}
// 订阅者更新
Sub.prototype.update = function () {
    this.x = this.x + 1
    console.log(this.x)
}

// 发布者
var pub = {
    publish: function () {
        dep.notify()
    }
}
 
var dep = new Dep() // 主题对象实例
dep.subs.push(new Sub(1), new Sub(2)) // 新增两个订阅者
pub.publish() // 发布者发布更新
```
Observer会观察两种类型的数据，Object 与 Array
- src/core/observer/index.js
- src/core/observer/array.js

对于Array类型的数据，由于 JavaScript 的限制，Vue 不能检测变化,会先重写操作数组的原型方法，重写后能达到两个目的：
- 当数组发生变化时，触发 notify
- 如果是 push，unshift，splice这些添加新元素的操作，则会使用observer观察新添加的数据  
重写完原型方法后，遍历拿到数组中的每个数据，使用observer观察它

而对于Object类型的数据，则遍历它的每个key，使用 defineProperty 设置 getter 和setter，当触发getter的时候，observer则开始收集依赖，而触发setter的时候，observe 则触发notify。

### Watcher
src/core/observer/watcher.js  
Watcher 是将模板和 Observer对象结合在一起的纽带。Watcher 是订阅者模式中的订阅者。Watcher 的两个参数： expOrFn最终会被转换为 getter 函数， cb是更新时执行的回调。依赖收集的入口就是get函数。
```js
constructor(
    vm: Component,
    expOrFn: string | Function,
    cd: Function,
    options?: Object
){
    this.vm = vm
    vm._watchers.push(this) // 讲当前watcher类推送到对应的Vue实例中
    // parse expression for getter
    if (typeof expOrFn === 'function') {
        // 如果是函数，相当于指定了当前订阅者获取数据的方法，每次订阅者通过这个方法获取数据与之前的值进行对比
        this.getter = expOrFn
    } else {
        // 否则的话将表达式解析为可执行的函数
        this.getter = parsePath(expOrFn)
    }
    // 如果lazy不为true，则执行get函数进行依赖收集
    this.value = this.lazy ? undefined: this.get()
}
```
只有通过watcher 触发的getter会收集依赖，而所谓的被收集的依赖就是当前watcher.初始化时传入的参数 expOrFn中涉及到的每一项数据，然后触发该数据项的 getter函数；getter 函数中就是通过判断 Dep.target的有无来判断是Watcher 初始化时调用的还是普通数据读取，如果有则进行依赖收集。  
getter 函数是用来连接监控属性与 Watcher 的关键。
```js
get () {
    // 设置全局变量Dep.target,将Watcher保存在这个全局变量中
    pushTarget(this)
    // 调用getter函数，进入get方法进行依赖收集操作
    const value = this.getter.call(this.vm, this.vm)
    if (this.deep) {
        traverse(value)
    }
    popTarget() // 将全局变量Dep.target置为null
    this.cleanupDeps()
    return value
}
```

### Dep
src/core/observer/dep.js  
这个方法是在响应式的过程中调用
的，用户修改数据触发 setter 函数，调用 dep.notify 去通
知订阅者更新视图。
```js
constructor () {
    this.id = uid++
    // 存储Watcher实例数组
    this.subs = []
}

notify () {
    const subs = this.subs.slice()
    // 遍历Watcher列表，调用update方法进行更新操作
    for(let i = 0, l = subs.length; i < l; i++) {
        subs[i].update()
    }
}
```

### Directive
![image](http://p1cbbowoo.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20180402153924.png)
关于编译这块vue分了两种类型，一种是文本节点，一种是元素节点.  
vue内置了这么多的指令，这些指令都会抛出两个接口bind 和 update，这两个接口的作用是，编译的最后一步是执行所有用到的指令的bind方法，而update 方法则是当watcher 触发 update 时，Directive会触发指令的update方法
observe -> 触发setter ->watcher -> 触发update ->Directive -> 触发update ->指令
```js
// hello {{name}}
[{
    value: 'hello'
}, {
    value: 'name',
    tag: true,
    html: false,
    oneTime: false,
    descriptor: {
        def: {
            update: function,
            bind: function
        },
        expression: xx,
        filters: xx,
        name: 'text'
    }
    
}]
```

```js
this._directives.push(new Directive(descriptor, this, node, host, scope, frag))
```
1. 所有 tag 为 true的数据中的扩展对象拿出来生成一个Directive实例并添加到_directives中（_directives是当前vm中存储所有directive实例的地方）
2. 调用所有已绑定的指令的 bind 方法
3. 实例化一个Watcher，将指令的update与watcher绑定在一起（这样就实现了watcher
接收到消息后触发的update方法，指令可以做出对应的更新视图操作）
4. 调用指令的update，首次初始化视图
5. 这里有一个点需要注意一下，实例化 Watcher 的时候，Watcher会将自己主动的推
入Dep依赖中

### 总结
![image](http://p1cbbowoo.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20180402153929.png)

# Virtual dom
DOM操作很慢是两个原因，一个是本身操作就不快，第二是我们（还有很多框架）处理dom的方式很慢，Virtual Dom解决了我们这些愚蠢的程序员对Dom的低劣操作，它让我们不需要进行Dom操作，而是将希望展现的最终结果告诉Vue，Vue通过一个简化的Dom即Virtual dom进行render，当你试图改变显示内容时，新生成的Virtual Dom会与现在的Virtual
dom对比，通过diff算法找到区别，这些操作都是在快速的js中完成的，最后对实际Dom进行最小的Dom操作来完成效果，这就是Virtual Dom的概念。
![image](http://p1cbbowoo.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20180402162223.png)
这仅仅是第一层。真正的 DOM元素非常庞大，这是因为标准就是这么设计的。而且操作它们的时候你要小心翼翼，轻微的触碰可能就会导致页面重排，这就是杀死性能的罪魁祸首。  
‘Virtual-dom’是一系列的模块集合，用来提供声明式的DOM渲染。来看一个简单的 DOM 片段.
本质上就是在 JS 和 DOM 之间做了一个缓存。可以类比 CPU 和硬盘，既然硬盘这么慢，我
们就在它们之间加个缓存：既然 DOM 这么慢，我们就在它们 JS 和 DOM 之间加个缓存。CPU
（JS）只操作内存（Virtual DOM），最后的时候再把变更写入硬盘（DOM）。

```html
<div id="parent">
    <span class="child">item1</span>
    <span class="child">item2</span>
    <span class="child">item3</span>
</div>
```
```js
const dom = {
    tagName: 'div',
    props: {
        id: 'parent'
    },
    children: [
        {tagName: 'span', props: {class: 'child'}, children: ['item1']},
        {tagName: 'span', props: {class: 'child'}, children: ['item2']},
        {tagName: 'span', props: {class: 'child'}, children: ['item3']}
    ]
}
```

# Dom diff
比较两棵DOM树的差异是 Virtual DOM算法最核心的部分，这也是所谓的 Virtual DOM 的 diff 算法。两个树的完全的 diff 算法是一个时间复杂度为 O(n^3)的问题。但是在前端当中，你很少会跨越层级地移动DOM元素。所以 Virtual DOM 只会对同一个层级的元素进行对比。下面的div只会和同一层级的div对比，第二层级的只会跟第二层级对比。这样算法复杂度就可以达到 O(n)。
![image](http://p1cbbowoo.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20180402164259.png)
实际的代码中，会对新旧两棵树进行一个深度优先的遍历，这样每个节点都会有一个唯一的标记，在深度优先遍历的时候，每遍历到一个节点就把该节点和新的的树进行对比。如果有差异的话就记录到一个对象里面。p是patches[1]，ul是patches[3]，类推。
![image](http://p1cbbowoo.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20180402164303.png)

节点的差异指的是什么呢？对 DOM 操作可能会：
- 替换掉原来的节点
- 移动、删除、新增子节点
- 修改了节点的属性
- 对于文本节点，文本内容可能会改变

如果我们把全部顺序都换了呢，这就悲剧了全部要删除在加（列表对比算法）：
- a b c d e f g h i
- a b c h d f g i j
- 这个问题抽象出来其实是字符串的最小编辑距离问题（Edition Distance），最常见的解决算法是 Levenshtein Distance，通过动态规划求解，时间复杂度为 O(M * N)。但是我们并不需要真的达到最小的操作，我们只需要优化一些比较常见的移动情况，牺牲一定DOM操作，让算法时间复杂度达到线性的（O(max(M, N))。
- 因为tagName是可重复的，不能用这个来进行对比。所以需要给子节点加上唯一标识key，就迎刃而解了。
- 通过深度优先遍历两棵树，每层的节点进行对比，记录下每个节点的差异了。
- JavaScript 对象树和render出来真正的DOM树的信息、结构是一样的。所以我们
可以对那棵DOM树也进行深度优先的遍历，遍历的时候从步骤二生成的patches对象中找出当前遍历的节点差异，然后进行 DOM 操作。

一些需要注意的点：
- 尽量不要跨层级的修改dom
- 设置key可以最大化的利用节点
- 不要盲目相信diff的效率，在必要时可以手工优化

一些非常有用的库：
- core/vdom/create-element.js
- core/vdom/patch.js
- 实现patch https://github.com/snabbdom/snabbdom
- 实现diff https://github.com/snabbdom/snabbdom
- 简陋版 https://github.com/livoras/simple-virtualdom/blob/master/lib/diff.js
- 简陋版 https://github.com/livoras/simple-virtualdom/blob/master/lib/patch.js

# vue运行时优化
- 对于不变的内容，标记出来，不做dom diff
![image](http://p1cbbowoo.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20180402171141.png)
- 对于不变的数据，直接生成，不做dom diff
![image](http://p1cbbowoo.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20180402171147.png)
![image](http://p1cbbowoo.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20180402171153.png)
- SSR采用直出字符串拼接的方式，根本不需要dom-diff
![image](http://p1cbbowoo.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20180402171157.png)
- ssr异步加载脚本
![image](http://p1cbbowoo.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20180402171637.png)
