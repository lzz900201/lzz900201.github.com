---
layout: post
title: "nil、Nil、NULL、NSNull的区别"
description: ""
category: iOS_Dev
tags: [nil]
---
{% include JB/setup %}


> **原文地址: [nil / Nil / NULL / NSNull][] 转载请注明出处。**


C语言用0表示`基本数据类型`的空（nothing）,NULL表示`指针`类型的空。  

Objective-C 通过为C的`为空表示法`上添加了一个*nil*, nil是一个不指向任何对象的指针。在语义上区别于NULL，但是在技术层面，他两个又等价。   

在框架层上，Foundation定义了NSNull类，这个类定义了一个叫做*+null*的类方法，这个类方法返回了一个NSNull的单例对象。NSNull和nil与NULL不同的是，NSNull是一个真正的类，而不是一个空值（zero value）。   

另外，在[Foundation/NSObjCRuntime.h][]中，Nil被定义为一个指向空的*类指针*。
它不经常出现，但是确实是指向空。

# 关于nil的一些事

新创建的NSObject类的生命周期开始的时候，他的内容是0。也就是说，这个类拥有的所有指向其他对象的指针都是nil，所以没有必要在init初始化方法里为自己设置一个nil，例如：

	self.(association) = nil;
	
或许nil的最值得注意的一个行为就是，他可以接收发送给它的消息。   

在其他语言中，例如C++，这种行为可以引起程序崩溃，但是在Objective-C语言中，在nil上调用一个方法返回空值（zero value）。这大大简化了表达式，因为我们不需要在使用一个对象之前检查它是否是空值。

	// For example, this expression..
	if (name != nil && [name isEqualToString:@"Steve"]) {...}

	// ...can be simplified to:
	if ([name isEqualToString:@"steve"]) {...}

Objective-C使这种nil简便的工作机制成为一种语言特性，可以规避一些潜在的bug。

# NSNull：为空而生

NSNull被大量的使用在*Foundation*和其他的框架中，以用来避免集合类（NSArrar，NSDictionary,etc.）中不能包含空值（nil）的情况。你可以把NSNull看作是一个高效打包NULL、nil值的一个类，这样一来，空值就可以包含在集合类中了。

	NSMutableDictionary *mutableDictionary = [NSMutableDictionary dictionary];
	mutableDictionary[@"someKey"] = [NSNull null]; // Sets value of NSNull singelton for 'someKey'

	NSLog(@"Keys: %@",[mutableDictionary allKeys]);

----

总结一下，以下是四个表示*空值*的值，每个Objective-C程序员都应该知道：   

| *Symbol* | *Value* | *Meaning* |
|:--------:|:-------:|:---------:|   
| NULL | (void \*)0 | Literal null value for C pointers    
| nil | (id)0 | literal null value for Objective-C objects 
| Nil | (Class)0 | literal null value for Objection classes 
| NSNull | \[NSNull null\] | singleton object used to represent null 





[Foundation/NSObjCRuntime.h]: http://c-faq.com/null/nullor0.html
[nil / Nil / NULL / NSNull]: http://nshipster.com/nil/
