---
title: sticky footer
date: 2017-11-05 22:11:30
tags: css
categories: 前端
---
在网页设计中，sticky footer 设计是最古老和最经典的效果之一，大致概括如下：如果页面内容不够长的时候，页脚固定在浏览器底部；如果页面内容足够长的时候，页脚会被内容向下推送。
<!--more-->
------
html代码
```html
<div class="page-wrap">
    content!
</div>
<footer class="site-footer">
    I'm the Sticky Footer.
</footer>  
```
css代码
```css
* {
    margin: 0;
}
html, body {
    height: 100%;
}
.page-wrap {
    min-height: 100%;
    /* equal to footer height */
    margin-bottom: -142px;
}
.page-wrap:after{
    content: "",
    display: block;
}
.site-footer, .page-wrap:after{
    height: 142px;
}
.site-footer{
    background: orange;
}
```
