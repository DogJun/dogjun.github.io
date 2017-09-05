---
title: 常用url参数操作方法
date: 2017-09-05 21:08:27
tags: javascript
categories: 前端
---
记录一下常用的url参数操作方法
<!--more-->
------
# 获取单个参数
```javascript
/**
 * [getParam ]
 * @param  {String} name
 * @param  {String} url   [default:location.href]
 * @return {String|Boolean}  
 */
function getParam(name, url) {
    if(typeof name !== 'string') return false;
    if (!url) url = window.location.href;
    // 当遇到name[xx]时，对方括号做一下转义为 name\[xxx\]，因为下面还需要使用name做正则
    name = name.replace(/[\[\]]/g, '\\$&');
    var regex = new RegExp('[?&]' + name + '(=([^&#]*)|&|#|$)');
    var results = regex.exec(url);
    if (!results) return null;
    if (!results[2]) return '';
    return decodeURIComponent(results[2].replace(/\+/g, " "));
}

getParam('query','https://juejin.im/search?query=hello&time=2017-11-12')
// output:
// "hello"
```
# 设置单个参数
```javascript
/**
 * [setParam 设置单个参数]
 * @param {String} name
 * @param {String|Number} val  
 * @return {String|Boolean}  
 */
function setParam(name, val, url) {
    if(typeof name !== 'string') return false;
    if (!url) url = window.location.href;
    var _name = name.replace(/[\[\]]/g, '\\$&');
    var value = name + '=' + encodeURIComponent(val);
    var regex = new RegExp(_name + '=[^&]*');
    var urlArr = url.split('#');
    var result = '';

    if(regex.exec(url)){
        result =  url.replace(regex, value);
    }else{
        result = urlArr[0]+'&'+value+ (urlArr[1] || '');
    }

    return result
}
setParam('query','world','https://juejin.im/search?query=hello&time=2017-11-12')
// output:
// "https://juejin.im/search?query=world&time=2017-11-12"
```
# 移除单个参数
```javascript
/**
 * [removeParam 移除单个参数]
 * @param  {String} name
 * @param  {String} url   [default:location.href]
 * @return {String|Boolean}      
 */
function removeParam(name, url) {
    if(typeof name !== 'string') return false;
    if (!url) url = window.location.href;
    var urlparts = url.split('?');
    var prefix = encodeURIComponent(name + '=');
    var pars = urlparts[1].split(/[&;]/g);
    var i = 0, len = pars.length;

    for (; i < len; i++) {
        if (encodeURIComponent(pars[i]).lastIndexOf(prefix, 0) !== -1) {
            pars.splice(i, 1);
        }
    }

    url = urlparts[0] + (pars.length > 0 ? '?' + pars.join('&') : '');

    return url;
}
removeParam('query','https://juejin.im/search?query=hello&time=2017-11-12')
// output:
// "https://juejin.im/search?time=2017-11-12"
```
# 获取多个参数
```javascript
/**
 * [getParams 获取多个参数]
 * @param  {String} names [多个用空格分割]
 * @param  {String} url   [default:location.href]
 * @return {[String|Boolean]}       
 */
function getParams(names, url) {
    if(typeof name !== 'string') return false;
    var names = names.split(' ');
    var result = {};
    var i = 0,
        len = names.length;
    if (names.length === 0) return false;
    for (; i < len; i++) {
        result[names[i]] = getParam(names[i], url);
    }
    return result;
}
getParams('query time','https://juejin.im/search?query=hello&time=2017-11-12')
// output:
// {query: "hello", time: "2017-11-12"}
```
# 设置多个参数
```javascript
/**
 * [setParams 设置多个参数]
 * @param {Object} obj
 * @param  {String} url   [default:location.href]
 * @return {[String|Boolean]}       
 */
function setParams(obj, url) {
    var result = url || '';
    if (Object.prototype.toString.call(obj) !== '[object Object]') return false;
    for (var name in obj) {
        result = setParam(name, obj[name], result);
    }
    return result;
}
setParams({a:111,b:222,query:'world'},'https://juejin.im/search?query=hello&time=2017-11-12')
// output:
// "https://juejin.im/search?query=world&time=2017-11-12&a=111&b=222"
```
# 移除多个参数
```javascript
/**
 * [removeParams 移除多个参数]
 * @param  {String} names [多个用空格分割]
 * @param  {String} url   [default:location.href]
 * @return {[String|Boolean]}       
 */
function removeParams(names, url) {
    var result = url || '';
    var names = names.split(' ');
    var i = 0,
        len = names.length;
    if (names.length === 0) return false;

    for (; i < len; i++) {
        result = removeParam(names[i], result);
    }
    return result;
}
removeParams('query time','https://juejin.im/search?query=hello&time=2017-11-12')
// output:
// "https://juejin.im/search"
```
# url hash 操作
```javascript
/**
 * [getHash 方法]
 * @param  {[String]} url [default:location.href]
 * @return {[String]}     
 */
function getHash(url) {
    return decodeURIComponent(url ? url.substring(url.indexOf('#') + 1) : window.location.hash.substr(1));
}

/**
 * [setHash 方法]
 * @param {String} hash
 */
function setHash(hash) {
    window.location.replace('#' + encodeURIComponent(hash));
}

/**
 * [removeHash 方法]
 */
function removeHash() {
    window.location.replace('#', '');
```
