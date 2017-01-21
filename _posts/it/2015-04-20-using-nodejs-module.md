---
layout: post
title:  "Using NodeJS Module"
date:   2015-03-20 22:18:00
categories: 
	- it
---


* Node中可以使用**module.exports**和**exports**导出借口给外部使用，具体根据需要分别使用不同的形式导出接口

##### 如果是希望使用new的形式让外部访问，可以使用直接对module.exports赋值的形式，将function赋值给module.exports。例如：

    {% highlight js linenos %}
// var A = require('a')
// var a = new A()
module.exports = function（）{
	console.log（"module A"）；
} 
    {% endhighlight %}

##### 如果希望直接访问的形式，可以使用对exports赋值属性的方式，将内部接口暴露给外部使用，例如：
    {% highlight js linenos %}
// var a = require('a')
exports.aFn = function（）{
	console.log（"module A aFn"）；
} 
exports.bFn = function(){
	console.log('module A bFn');
}

    {% endhighlight %}
##### 另外，在模块里面定义的var变量，都为模块内部的私有变量。
