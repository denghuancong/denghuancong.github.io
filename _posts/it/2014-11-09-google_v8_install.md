---
layout: post
title:  "Google V8引擎的安装和使用"
date:   2014-11-09 17:21:11
categories: 
	- it
---


最近写的一个模块需要引用Google V8这个库，下面介绍如何安装和使用。（必须翻墙）

#### 安装

根据参考资料1，2，用git下载v8
	
	git clone https://github.com/v8/v8-git-mirror

然后进入这个目录，切到master分支，下周依赖库（必须翻墙）
	
	cd v8-git-mirror
	git checkout master
	make dependencies  // 下载依赖库

如果成功把依赖都编译好了可以跳过下面这不，我是先在makefile里面把依赖库一个个下载回来再放在合适的目录
,makefile 里面列出的依赖库如下：


	493     svn checkout --force http://gyp.googlecode.com/svn/trunk build/gyp \
	494         --revision 1831
	495     if svn info third_party/icu 2>&1 | grep -q icu46 ; then \
	496       svn switch --force \
	497           https://src.chromium.org/chrome/trunk/deps/third_party/icu52 \
	498           third_party/icu --revision 277999 ; \
	499     else \
	500       svn checkout --force \
	501           https://src.chromium.org/chrome/trunk/deps/third_party/icu52 \
	502           third_party/icu --revision 277999 ; \
	503     fi
	504     svn checkout --force http://googletest.googlecode.com/svn/trunk \
	505         testing/gtest --revision 692
	506     svn checkout --force http://googlemock.googlecode.com/svn/trunk \
	507         testing/gmock --revision 485

东西准备好了，下面编译这个库，注意把ibrary=shared 参数加上，把它编译为so库

	make native mode=debug library=shared -j2

指定native会自动判断你当前系统的架构进行编译，-j2为指定使用2个进程进行编译加快编译速度,编译成功后在v8-git-mirror/out/native/lib.target目录下会出现下面几个文件，


	-rwxr-xr-x. 2 root root  2742392 11月  9 11:31 libicui18n.so
	-rwxr-xr-x. 2 root root 12400430 11月  9 11:30 libicuuc.so
	-rwxr-xr-x. 2 root root 12227393 11月  9 11:45 libv8.so

使用方式和一般的so一样（可以将so拷贝到系统lib目录或者指定LD_LIBRARY_PATH环境变量）.

#### 使用

参考资料中的第三项，将提供代码（可能和下面的有所差别）保存为hw.cc
 
 {% highlight c linenos %}

	#include <v8.h>

	using namespace v8;

	int main(int argc, char* argv[]) {
  		// Create a new Isolate and make it the current one.
  		Isolate* isolate = Isolate::New();
  		Isolate::Scope isolate_scope(isolate);

  		// Create a stack-allocated handle scope.
  		HandleScope handle_scope(isolate);

  		// Create a new context.
  		Local<Context> context = Context::New(isolate);

 		// Enter the context for compiling and running the hello world script.
  		Context::Scope context_scope(context);

  		// Create a string containing the JavaScript source code.
  		Local<String> source = String::NewFromUtf8(isolate, "'Hello' + ', World!'");

  		// Compile the source code.
  		Local<Script> script = Script::Compile(source);

  		// Run the script to get the result.
  		Local<Value> result = script->Run();

  		// Convert the result to an UTF8 string and print it.
  		String::Utf8Value utf8(result);
  		printf("%s\n", *utf8);
  		return 0;
	}
{% endhighlight %}

编译这个文件：

	g++ -I/root/sources/v8-git-mirror/include hw.cc -lv8 -lpthread -o hw

然后执行：
	
	./hw

会输出
	
	hello，world！

这样就算是使用了v8这个库了，里面的代码理解需要了解v8的基本数据结构和原理，上面例子虽然就几行代码，但五脏俱全，算是个小小的将js代码进行解释的程序了。

#### 参考资料

	[1] https://code.google.com/p/v8-wiki/wiki/UsingGit
	[2] https://github.com/v8/v8-git-mirror
	[3] https://developers.google.com/v8/get_started
