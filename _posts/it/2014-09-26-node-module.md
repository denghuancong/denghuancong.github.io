---
layout: post
title:  "Node Module"
date:   2014-09-26 11:09:08
categories:  
    - it
author: samueldeng
---

这篇文章介绍如何编写Node模块，node模块遵循commonjs标准，实现起来非常简单，下面简单实现一个maths的模块, 在macosx下编写。

    vim maths.js

然后在maths.js里面写入：
	
	exports.add = function(a, b) {
		return a+b;
	}
	exports.sub = function(a, b) {
		return a-b;
	}

注意到代码里面使用了exports导出两个函数，在node里面使用exports或者module.exports导出模块里面的功能，现在，已经完成了maths模块，下面试着使用下这个模块：

	vim app.js

在相同目录下创建app.js，然后写入：
	
	var maths = require("/maths.js");
	console.log(maths(1,2));

输出：

	3
但注意到我们引入这个模块的形式指明了路径，如何才可以使用npm安装我们自己编写的模块？现在，我们创建一个maths的目录：
	
	mkdir maths
	cd maths
	mv ../maths.js index.js
	vim package.json

在package.json里面写入下面参数：
	
	{
  		"name": "maths",
  		"version": "0.0.1",
  		"private": true
	}

注意version参数必须是三位的，之前我使用了2位的，模块没有包装成功，最后创建README.md文件：
	
	echo maths function add and sub! > README.md

现在，我们使用下面命令包装模块：

	 ~/Project/maths/ npm pack
		maths-0.0.1.tgz
	
模块已经打包完成，下面试着用npm安装这个模块到我们的项目里面（下面创建一个项目目录，进行模块安装）：
	
	mkdir modtest
	cd modtest
	npm install ~/Project/maths/maths-0.0.1.tgz

现在模块已经安装到我们的项目里面，可以查看一下：

	 ~/Project/modtest/ npm install ~/Project/maths/maths-0.0.1.tgz
		maths@0.0.1 node_modules/maths
	 ~/Project/modtest/ ls
		node_modules
	 ~/Project/modtest/ ls node_modules
		maths
	 ~/Project/modtest/ ls node_modules/maths
		README.md    index.js     package.json
	 ~/Project/modtest/

下面我们试着来使用这个模块：

	vim app.js

写入下面代码：

	var maths = require("maths");
	console.log(maths.add(1,2));

输出：

	 ~/Project/modtest/ node app.js
		3

简单的一个模块完成拉 ：），但看见很多别的人模块里面导出功能是使用module.exports的，我们这里使用了exports，到底这两个有什么差别？看下面的例子,编辑文件test.js：
	
	module.exports = "xxxxxxxx";
	exports.add = function(){
		console.log("add");
	}
	
引用这个文件时：
	   
	module.exports = "xxxxxxxx";
    exports.add = function(){
          console.log("add");
    }
                        
	 ~/Project/modtest/ node app.js
		xxxxxxxx
		/Users/apple/Project/modtest/app.js:4
		test.add();
     		 ^
		TypeError: Object xxxxxxxx has no method 'add'

导出的test对象为一个string，没有add方法，再看下面的例子：
	
	module.exports.name = "xxxxxxxx";
	exports.add = function(){
    	console.log("add");
	}

引入这个文件时：
	
	var test = require("./test.js");
	console.log(test.name);
	test.add();

输出：

	 ~/Project/modtest/ node app.js
	xxxxxxxx
	add

因此，我们可以得知，最终导出的是modul.exports上面的对象，attach在exports上面的对象最终会被attach到modul.exports上面，但前提是module.exports是一个对象。
