---
layout: post
title: "iOS7 new feature"
description: ""
category: 
tags: []
---
{% include JB/setup %}

# Modules
```#import```语句我们已经写了成千上万次了。
	
	#import <UIKit/UIKit.h>
	#import <MapKit/MapKit.h>
	#import <iAd/iAd.h>

这种语法可以追述到Objetive-C的根源：vanilla C。*#import*语句是一个和*#include*语句有着相似功能的预处理指令。他们之间唯一的区别就是，*#import*不会导入已经导入的头文件。

当预处理发现*#import*指令，就会将这一行代码_替换_为导入头文件的所有内容。而且会迭代的将一大推头文件导入。

UIKit的头文件，*UIKit.h*导入了所有包含在UIKit框架下的其他头文件。这就意味着你不用手动的导入包含在这个框架下的头文件，例如*UIViewController.h*，*UIView.h*,*UIButton.h*。

对UIKit框架的大小感到好奇么？你可以去浏览并数一下所有UIKit的头文件，你会发现它有超过#11000#行代码。

一个标准的iOS应用都会导入UIKit，这就意味着，每一个文件都会有超过11000行的代码。这很不理想，毕竟，更长的代码意味这更长的编译时间。

## 原始解决方案：预编译头文件

预编译头文件，或者叫PCH文件，试图通过提供一种机制来解决这一问题，这种机制预计算并缓存编译的预处理阶段的大部分工作。你或许已经见过这个有Xcode模板生成的PCH文件，看起来想这样：

	#import <Availability.h>
	 
	#ifndef __IPHONE_5_0
	#warning "This project uses features only available in iOS SDK 5.0 and later."
	#endif
	
	#ifdef __OBJC__
		#import <UIKit/UIKit.h>
		#import <Foundation/Foundation.h>
	#endif

因为我们应用中的每一个文件都要用到Foundation，大部分都要用到UIKit，所有文件中包含了这两个框架。通常来说，这两行代码加在这个PCH文件里会因为预计算和缓存让你的每一个文件都获益。

“所以，有什么问题？”你可能会问。照现在的样子对一个PCH文件在技术层面来说没什么不对。然后，你可能会错过由一个被良好优化，高度调谐的PCH文件带来的性能收益。例如，如果你应用中的一些部分用到了Map Kit框架，你把导入框架的头文件的语句写入PCH文件，可能会发现编译时间上的提升。

我们都是懒惰的开发者，也没有人有时间来为他们的每一个项目调谐PCH文件。这就是将*modules*开发为LLVM特性的原因。

## 新的解决方案：Modules

Modules将框架封装的更加简洁。预处理指令不再需要将*#import*语句逐字的替换成头文件的内容。相反，module将框架包装成自持有(self-contained)的区块，这些区块使用和PCH文件同样的预处理规则进行处理，同样可以提升编译速度。但是你不需要再在PCH文件中指定你要使用哪个框架，简单的使用modules就能获得编译速度的提升。

modules的优点不止这些。我确定你要在你的应用中引入一个新的框架需要很多步骤，大致像这样：

1. 在要引入框架的文件中添加#imort语句
2. 编写代码
3. 编译
4. 查看连接期的错误
5. 想起你忘了连接框架
6. 在项目的build phase里连接框架
7. 再次编译

忘记连接框架是在平常不过的事情了，modules同样将这个问题灵巧的解决。（有modules做不到的事情么？）
module不仅告诉编译器由那个头文件组成了module，同样还告诉编译器那些需要连接。这样你就不用手动的连接框架了。这是个小事儿，但是任何让开发编程变得简单的事儿都不是小事儿

## 如何使用

要在你的项目中使用module相当容易。对已存在的项目来说，首先要启用module，你可以在项目的构建选项里(build setting)里找到这个选项，然后将选项设置为YES即可
![](http://cdn3.raywenderlich.com/wp-content/uploads/2013/09/RW-Modules.png)

用Xcode5创建的新项目这个选项是默认启用的，但是你真的应该为你已有的项目启用该选项。

*Link Frameworks Automatically* 启用该选项可以像之前描述的那样自动连接框架。但是也有要禁用该选项的小原因。

一旦module被启用，你就可以在你的代码里使用它了。只需在你习惯的语法上稍作变更即可使用。取```#import```而代之的是#@import#:

	@import UIKit;
	@import MapKit;
	@import iAd;

同样可以导入框架的某一部分，例如你只想导入UIView，像这样：
	
	@import.UIView;

是的， 就是这么简单。我们不必把我们所有的```#import```语句替换为@import，因为编译器会隐式的为我们转换。

别高兴的太早，module也有他的不利之处。我们无法将module用我们自己的框架或者第三方框架，以后可能会实现。但是就现在来说，这是mudule的一个缺点。不过，没有完美之物，Objective-C也一样。

