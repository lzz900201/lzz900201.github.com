---
layout: post
title: "ARC简明笔记"
description: ""
category: programming
tags: [iOS_Dev, ARC]
---
{% include JB/setup %}

本文是学习整理 [这篇][article_arc] ARC教程所得，如果需要查看原文，请自行移步。&raquo; [手把手教你ARC][article_arc]


# ARC是什么 ARC不是什么

ARC(Automatic Reference Counting)，自动引用计数，`是LLVM3.0编译器的新特性`，高效的自动内存管理机制。   

但是需要注意，ARC是编译器特性，是编译器在编译代码时，在适当的位置，按照MRC内存管理规则，插入合适的内存管理代码。而不是运行时环境或者GC。   

**ARC完全不会比MRC做的弱。**

> “如果不放心编译器生成结果，就应该直接写汇编” ——某大神

而且在最新的Xcode5中，已经默认使用ARC，并且没有勾选使用MRC的选项，说明苹果认为ARC是个相当靠谱的东西。   

# ARC原理

ARC是编译器特性，只是在代码编译时自动在合适的位置插入*release*或*autorelease*代码，在最合适的地方完成引用计数，以及部分优化。

# ARC机制

将指针指向一个对象，只要指针没有被置空，对象就会一直保留在堆上。适用于*实例变量*、*类的属性*、*局部变量*。

* strong关键字：**只要某一对象被任一```strong```指向，那么它将不会被销毁。如果没有被任何strong指针指向，那么将被销毁**。 默认情况下，_实例变量_和_局部变量_都是strong类型的。

* weak关键字：weak类型的指针也可以指向对象，但并不持有该对象。如果weak指针指向的对象被销毁，那么该指针则被置为nil。*weak指针并不常用*，比较常见的用法是用来解决*循环引用问题*
	
	注意这段代码
	
		__weak NSString *str = [[NSString alloc] initWithFormat:…];
		NSLog(@"%@",str); 
		
	以上两行代码中，输入为nil，为什么？因为alloc出来的对象没有被任何strong指针指向，生成的同时便被销毁，str随即便指向了nil。
	
类的属性（@property）可以使用strong、weak来修饰，只需要替换相应的retain和assign即可。

ARC为我们节省了很多代码，减轻了一些负担，但是这不意味着我们就可以完全不考虑内存管理问题，相反，我应该经常地考虑：`到底谁持有这个对象`，我们要在适当的时候将`strong`指针指向`nil`，否则可能会引起__OOM__。

* ARC中的`dealloc`方法：因为使用了ARC，因此所有的`release`代码就都没有必要出现了。但是对象被销毁的时候`dealloc`方法还是会被调用，针对CF以及下级向的几种情况，还是需要在dealloc中处理一下，例如：
	* retain的CF对象要CFRelease( )。
	* malloc( )到堆上的东西要free( )掉。
	* 添加的observer可以在这里remove。
	* schedule的timer要在这里`invalidate`。
	* …   
	
	**但是不再需要发送[super dealloc]消息，ARC会自动搞定。** 
	 

* 对特定文件设置`启用`或`禁用`ARC，在compile source中设置编译标记。
	* 启用 `-fobjc-arc`
	* 禁用 `-fno-objc-arc`
	
![](http://img.onevcat.com/uploads/2012/06/arcpic15.png)

**property详细的类型和用法总结：**

* strong 和原来的retain比较相似，strong的property对应__strong指针，它将持有指向对象。
* weak 不持有所指向对象，而且当所有对象销毁时能将自己置为nil，*基本所有的outlet都应该使用weak*
* unsafe_unretained 对应原来的assign。需要支持iOS4的时候需要使用这个关键字。该关键字对应 __unsafe_unretained指针。
* copy 和原来的基本一样，copy一个对象并且为其创建一个strong指针。
* assign 对于对象来说永远不用assign了，实在需要的话应该用 unsafe_unretained代替。但是对于基本类型，比如 int,float,BOOL这些，还是需要用assign。

[article_arc]: http://onevcat.com/2012/06/arc-hand-by-hand/


