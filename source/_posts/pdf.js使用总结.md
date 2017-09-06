---
title: pdf.js使用总结
date: 2017-09-06 16:06:40
tags: javascript
categories: 前端
---
由于项目的开发需要，在线展示PDF，而且要兼容IE，所以就选择了pdf.js,网上教程相对较少，官方文档纯英文，摸索一番，总结一下最基本的使用。
<!--more-->
------
## 一个简单的demo
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
  <style>
    #pdf-container{
      width: 1000px;
    }
  </style>
  <script src="//mozilla.github.io/pdf.js/build/pdf.js"></script>
</head>
<body>
  <div id="pdf-container"></div>
<script type="text/javascript">
// pdfUrl 为 pdf 文件地址
PDFJS.getDocument(pdfUrl).then(function(pdf) {
  // 显示所有页数
  for (var pageNum = 1; pageNum <= pdf.numPages; ++pageNum) {
    pdf.getPage(pageNum).then(function(page) {
      // you can now use *page* here

      var scale = 3;
      var viewport = page.getViewport(1.5);

      var canvas = document.createElement('canvas');
      var context = canvas.getContext('2d');
      canvas.height = viewport.height;
      canvas.width = 1000;

      var renderContext = {
        canvasContext: context,
        viewport: viewport
      };
      page.render(renderContext);

      document.getElementById('pdf-container').appendChild(canvas);
    });
  }
})
</script>
</body>
</html>
```
## 遇到的问题
项目开发过程中发现pdf中的电子签章不显示，控制台警告如下：
Warning: Unimplemented widget field type "Sig", falling back to base field type.
## 解决方案
在pdf.worker.js中有这样一段代码
```javascript
// 行号： 40277
// 可自行根据关键字查找到此代码
// Hide unsupported Widget signatures.
if (data.fieldType === 'Sig') {
  //WGZ
  //warn('unimplemented annotation type: Widget signature');
  this.setFlags(AnnotationFlag.HIDDEN);
}
```
将this.setFlags(AnnotationFlag.HIDDEN);这一行注释掉，就可以显示电子签章。
