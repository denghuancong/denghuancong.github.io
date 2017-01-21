---
layout: post
title:  "Using fibers in NodeJS"
date:   2015-04-21 16:46:33
categories: 
	- it
---


##### 怎么安装这个不需要写了，下面简单说下最近使用的fibers的几个用法，使用fibers主要是为了更好地去做并发（多任务）和让代码更容易读懂。

##### 1.创建并发任务，在new fibers的时候传入任务例程，这个例程函数就作为一个任务去执行，但这是任务是还没有被触发执行的
    {% highlight js linenos %}
var task = new fibers(function(){
	console.log('task a');
});
    {% endhighlight %}

##### 2.执行并发任务，需要调用fibers对象的run函数
    {% highlight js linenos %}
task.run();//这样，fibers对象的例程函数就会被执行

    {% endhighlight %}
##### 3.除了例程函数执行完成，在例程函数里面可以暂停执行并且让run函数返回，返回的内容为yield函数的参数，例如：
    {% highlight js linenos %}
var task = new fibers(function(){
	console.log('task a');
				var f = Fiber.current;// 返回当前的fibers对象
	var yield_return = f.yield("task a pause");
	console.log(yield_return);
});

var run_return  = task.run();// 这时候，输出task a
console.log(run_return);// 输出task a pause

    {% endhighlight %}
##### 如果再次执行run函数，例程会在刚在被中断的位置继续执行，并且yield函数返回run函数的参数，例如：

    {% highlight js linenos %}
task.run(“keep running”);// 例程输出keep running
    {% endhighlight %}

