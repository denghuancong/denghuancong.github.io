---
layout: post
title:  "关于Node Buffer对象"
date:   2014-11-08 22:18:00
categories: 
	- it  
---

最近写一个专门为新项目写一个nodejs的c++模块去解释protobuf的消息（名字叫node_pd哈，后面写文章介绍），为何要重新造轮子？因为npm上面都TM没有合适的，试用了两个，一个不支持extension，一个extension解释出问题，然后，然后就打算自己写了，目前准备完成了，90%吧，可以正常在C++和javascript之间解释和生成消息，呃。。。，说了些和主题木有关系东东，算是作为背景吧。

Node提高的buffer对象，实际上是一个二进制数组，占用的内存使用堆分配，没有使用V8的垃圾回收机制。这个对象所在的模块由node启动的时候加载，不需要required，可以直接使用，一般使用buffer对象来存储二进制数据，多用于网络服务作为缓冲区，好吧，我承认这几句话很废。

node的buffer对象使用的内存不在v8引擎的管理，是存在于v8内存管理之外的内存空间，代码头文件解释是这么说的：

	A buffer is a chunk of memory stored outside the V8 heap, mirrored by an object in javascript.
	
查看下代码：node-v0.10.32/src/node_buffer.cc

	111 Buffer* Buffer::New(char *data, size_t length,
	112                     free_callback callback, void *hint) {
	113   HandleScope scope;
	114
	115   Local<Value> arg = Integer::NewFromUnsigned(0);
	116   Local<Object> obj = constructor_template->GetFunction()->NewInstance(1, &arg);
	117
	118   Buffer *buffer = ObjectWrap::Unwrap<Buffer>(obj);
	119   buffer->Replace(data, length, callback, hint);


看最后一句，以及下面的最后一句，确实是在系统堆中分配，不在v8的管理。


	162 void Buffer::Replace(char *data, size_t length,
	163                      free_callback callback, void *hint) {
	164   HandleScope scope;
	174   length_ = length;
	175   callback_ = callback;
	176   callback_hint_ = hint;
	177
	178   if (callback_) {
	179     data_ = data;
	180   } else if (length_) {
	181     data_ = new char[length_];
	182     if (data)
	183       memcpy(data_, data, length_);
	189   handle_->SetIndexedPropertiesToExternalArrayData(data_,
	190                                                    kExternalUnsignedByteArray,
	191                                                    length_);
	192   handle_->Set(length_symbol, Integer::NewFromUnsigned(length_));
	193 }

并且会将内存指针设置在handle_这个v8对象（这是个持久型对象）上面,说白了就是把地址放在这个对象中，。当需要在引用这个buffer对象的数据时，在v8环境中就可以根据这个handle_去获取数据。


	75   static inline char* Data(v8::Handle<v8::Value> val) {
 	76     assert(val->IsObject());
 	77     void* data = val.As<v8::Object>()->GetIndexedPropertiesExternalArrayData();
 	78     return static_cast<char*>(data);
 	79   }
 	80
 	81   static inline char* Data(Buffer *b) {
 	82     return Buffer::Data(b->handle_);
 	83   }



顺便说下，buffer的对象最大的理论存储空间为0x3fffffff，也就是1G空间。。。

    
    66 class NODE_EXTERN Buffer: public ObjectWrap {
 	67  public:
 	68   // mirrors deps/v8/src/objects.h
 	69   static const unsigned int kMaxLength = 0x3fffffff;

####关于使用
如果想在node c++模块中使用buffer对象，需要引入buffer和node对象wrapper的头文件（node.h，node_buffer.h，node_object_wrap.h）。

#### 后话
详细使用待研究。。。。。
