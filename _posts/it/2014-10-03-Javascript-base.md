---
layout: post
title:  "Base Javascript"
categories: 
	- it
date:   2014-10-03 23:40:33

---


1.访问一个不存在的变量或者没有赋值的变量，得到undefined

2.null变量不是undefined，这个变量有值，是一个特殊的object，值为为null

3.任何不属于number，string，boolean，undefined，null的都为对象object（function也为对象）

4.typeof 操作可以返回变量的类型,用字符串表示，如“string”

5.NaN is a special value that is also a number，Nan也是一个数值

6.下面的情况导致false

* The empty string ""
* null
* undefined
* The number 0
* The number NaN
* The Boolean false

7.函数没有明确返回值，默认返回undefined

8.函数里面包含一个默认的数组arguments（其实是一个object）包含所有的参数

9.if内定义的变量外层可以访问，函数内定义的变量外层不能访问

10.Variable hoisting, 函数内定义的和外层变量同名的变量，定义会被提升到函数开始，因此，一进入函数内，和函数内部同名的外部变量瞬间被秒杀。

	When your JavaScript program execution enters a new function, 
	all the variables declared anywhere in the function are 
	moved (or elevated, or hoisted) to the top of the function. 
	This is an important concept to keep in mind. Further, 
	only the declaration is hoisted, meaning only the presence of 
	the variable is moved to the top. Any assignments stay where they are.

例子：

	{% highlight javascript linenos %}
	var a = 123;
   	function f() {
     	alert(a);
     	var a = 1;
    	alert(a);
	} f();
	{% endhighlight %}
变成:

	{% highlight javascript linenos %}
	var a = 123;
   	function f() {
     	var a; // same as: var a = undefined;
     	alert(a); // undefined
     	a = 1;
     	alert(a); // 1
	}
	{% endhighlight %}

11.即刻执行的函数不能被执行第二次！！！

12.函数可以被作为参数转递

13.闭包说白了就是将变量作用域的范围进行扩大

14.给对象设置属性时，属性的名字可以为下面形式：

	{% highlight javascript linenos %}
  	var hero = {occupation: 1};   	var hero = {"occupation": 1};   	var hero = {'occupation': 1};
	{% endhighlight %}

15.在javascript中如果你想使用类似hash的数据结构，那么可以使用object

16.创建“new 函数名称”的形式可以创建一个对象，并且会调用这个函数，而这个函数也就成为了这个对象的构造函数。

	{% highlight javascript linenos %}
	function dog(name){
		this.name = name;
	}
	var instance_dog = new dog("keke");
	{% endhighlight %}
	
17.每个object实例都有一个constructor属性，指向这个实例的构造函数

18.函数有一个length属性指出期望的参数个数（也就是参数列表的个数）

19.每个函数都有一个prototype属性，这个属性指向一个对象，每个从这个函数构造出来的对象都继续了这个prototype所指向的对象的属性。

	All objects created with this function keep a reference 
	to the prototype property and can use its properties as their own
	
20.function对象的call和apply方法可以用别的对象作为调用实例进行使用，也就是可以使用别的实例作为内部的this引用进行使用，功能一样，但传递的参数不一样。
	
	These methods also allow your objects to "borrow" methods 
	from other objects and invoke them as their own. 
	This is an easy and powerful way to reuse code.
	
	some_obj.someMethod.apply(my_obj, ['a', 'b', 'c']);   	some_obj.someMethod.call(my_obj, 'a', 'b', 'c');21.当对象是new 函数的方式构造出来的，prototype上面的属性才会依附到this上面，单纯使用函数调用，prototype上面的属性不会依附到this对象，因此，通过在prototype上面添加属性的方式，可以给实例进行属性添加。
	Adding methods and properties to the prototype property 	of the constructor function is another way to add functionality 	to the objects this constructor produces.
	
22.属性查找时是按照prototype链往上查找。

23.prototype依附的属性会在new操作之后拷贝到对象上面，对对象属性的修改不会影响到prototype上面的属性。
