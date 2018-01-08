---
title: javascript语言精粹
date: 2018-01-08 22:52:36
tags: js
categories: 前端
---
记录一些琐碎的知识点。
<!--more-->
------
# 基本数据类型
- Boolean
- Number
- String
- null
- undefined
- Symbol
# 声明
- 使用关键字var：函数作用域
- 使用关键字let：块作用域
- 直接使用：全局作用域
只声明不赋值，变量默认值是undefined
const关键字可以声明不可变变量，同样为块作用域。对不可变的理解在对象上的理解需要注意。
```javascript
const num =1;
const obj = {
prop: 'valuye'
}
num = 2; // Uncaught TypeError: Assignment to constant variable.
obj['prop'] = 'value2';

obj = []; // Uncaught TypeError: Assignment to constant variable. 
```
# 变量提升
Javascript中可以引用稍后声明的变量，而不会引发异常。
```javascript
console.log(a); // undefined
var a = 2;
```
# 函数
## 定义函数
- 函数声明
- 函数表达式
- Function 构造函数
- 箭头函数
```javascript
function fn  () {}
var fn = function () {}
var fn = new Function(arg1, arg2, ... argN, funcBody)
var fn = (param) => {}
```
## arguments
arguments: 一个包含了传递给当前执行函数参数的类似于数组的对象
arguments.length: 传递函数的参数的数目
```javascript
function foo （） {
return arguments;
}
foo(1, 2, 3); // Arguments[3]
// {"0":1, "1": 2, "2": 3}
```
## rest
```javascript
function foo (...args) {
return args;
}
foo(1, 2, 3); // Array[3]
// [1, 2, 3]
function fn (a, b, ...args) {
return args;
}
fn(1, 2, 3, 4, 5); // Array[3]
// [3, 4, 5]
```
## 默认值
```javascript
function fn (a =2, b = 3) {
return a + b;
}
fn(2, 3);  // 5
fn(2); // 5
fn(); // 5
```
# 对象
javacript 中对象是可变键控集合
## 定义对象
- 字面量
- 构造函数
```js
var obj = {
prop: 'value'
fn: function () {}
};
var date = new Date();
```
## 构造函数
构造函数和普通函数并没有区别，使用new关键字调用就是构造函数，使用构造函数可以实例化一个对象
```js
function People (name, age) {
this.name = name;
this.age = age;
}
var people = new People('Byron', 26);
```
函数的返回值有两种可能
- 显示调用 return 返回 return 后表达式的求值
- 没有调用 return 返回 undefined
构造函数返回值
1. 没有返回值
2. 简单数据类型
3. 对象类型
前两种情况构造函数返回构造对象的实例，实例化对象正式利用的这个特性
第三种构造函数和普通函数一致，返回 return 后表达式的结果
## prototype
每个函数都有一个 prototype 的对象属性，对象内又一个 constructor 属性，默认指向函数本身
每个对象都有一个  __proto__ 的属性，属相指向其父类的 prototype
```js
function Person  (name) {
this.name = name;
}
Person.prototype.print = function () {
console.log(this.name);
}
var p1 = new Person('DogJun');
p1.print();
```
# this和作用域
作用域可以通俗的理解
  1. 我是谁
  2. 我有哪些马仔
其中我是谁的回答就是 this 
马仔就是我的局部变量
this 场景
- 普通函数
- 严格模式： undefined
- 非严格模式： 全局对象 Node: global 浏览器: window
- 构造函数： 对象的实例
- 对象方法：对象本身
# call & apply
```js
fn.call(context, arg1, arg2, ..., argn)
fn.apply(context, args)
function isNumber (obj) {
return Object.prototype.toString.call(obj) === '[object Number]';
}
```
# Function.prototype.bind
bind 返回一个新函数，函数的作用域为 bind 参数
```js
function fn () {
this.j = 0;
setInterval(function () {
console.log(this.i++);
}.bind(this), 500)
}
fn();
```
# ()=>{}
箭头函数是ES6提供的新特性，是简写的函数表达式，拥有词法作用域和 this 值
```js
function fn () {
this.j = 0;
setInterval(() => {
console.log(this.i++);
}, 500)
}
fn();
```
# 继承
在javascript的场景，继承有两个目标，子类需要得到父类的：
- 对象的属性
- 对象的方法
```js
function inherits (child, parent) {
var _prototype = Object.create(parent.prototype);
_prototype.constructor = child.prototype.constructor;
child.prototype= _prototype;
}
function People (name, age) {
this.name = name;
this.age = age;
}
People.prototype.getName = function () {
return this.name;
}
function English (name, age, language) {
People.call(this, name, age);
this.language = language;
}
inherits(English, People);
English.prototype.introduce = function () {
console.log('Hi, I am ' + this.getName());
console.log('I speak ' + this.language);
}
var en = new English('DogJun', 24, 'English');
en.introduce();
```
# ES6 class 与继承
```js
class People {
constructor (name, age) {
this.name = name;
this.age = age;
}
getName () {
return this.name;
}
}
class English extends People {
constructor (name, age, language) {
super(name. age);
this.language = language;
}
introduce () {
console.log('Hi, I am ' + this.getName());
console.log('I speak ' + this.language);
}
}
let en = new English('DogJun', 24, 'English');
en.introduce();
```
# 高阶函数
高阶函数是把函数当做参数或者返回值是函数的函数
## 回调函数
```js
[1,2,3,4].forEach(function(item){
console.log(item);
})
```
## 闭包
闭包由两部分组成
- 函数
- 环境：函数创建时作用域内的局部变量
```js
function makeCounter(init) {
var init = init || 0;
return function () {
return ++init;
} 
}
var counter = makeCounter(10);
console.log(counter());
console.log(counter());
```
典型错误
```js
for (var i=0;i<doms.length;i++) {
doms.eq(i).on('click',function (ev) {
console.log(i);
})
}
```
正确实现
```js
for (var i = 0; i < doms.length; i++ ) {
(function(i){
doms.eq(i).on('click', function (ev) {
console.log(i);
});
})(i);
}
```
## 惰性函数
```js
function eventBinderGenerator () {
if (window.addEventListener) {
return function (element, type, handler) {
element.addEventListener(type, handler, false);
}
} else {
return function (element, type, handler) {
element.attachEvent('on' + type, handler.bind(element, window.event));
}
}
}

var addEvent = eventBinderGenerator();
```
## 柯里化
一种允许使用部分参数生成函数的方式
```js
function isType (type) {
return function (obj) {
return Object.prototype.toString.call(obj) === '[object ' + type + ']';
}
}
var isNumber = isType('Number');
console.log(isNumber(1));
console.log(isNumber('s'));
var isArray = isType('Array');
console.log(isArray(1));
console.log(isArray([1,2,3]));

function f (n) {
return n * n;
}

function g (n) {
return n * 2;
}
console.log(f(g(5)));
function pipe(f, g) {
return function () {
return f.call(null, g.apply(null, arguments));
}
}
var fn = pipe(f, g);
console.log(fn(5));
```
## 反柯里化
```js
Function.prototype.uncurry = function () {
return this.call.bind(this);
}
```
push 通用化
```js
var push = Array.prototype.push.uncurry();
var arr = [];
push(arr, 1);
push(arr, 2);
push(arr, 3);
```