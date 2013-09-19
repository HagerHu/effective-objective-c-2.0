### 条目1：熟悉Objective-C的根

Objective-C和其他面向对象语言很像，像C++和Java，但在有些方面也有不同。如果你在其他面向对象语言方面有很多的经验，你就会明白其中用到的范例和模式。然而，它实用消息机制而不是方法调用，因为语法上显得略有不同。Objective-C从Smalltalk的消息机制演化而来。消息机制和方法调用的不同就像这样：

// Messaging (Objective-C)Object *obj = [Object new];[obj performWith:parameter1 and:parameter2];// Function calling (C++)Object *obj = new Object;obj->perform(parameter1, parameter2);
关键的不同点就是消息机制是运行时决定那一块代码会被执行；对于方法调用，则是由编译器决定。以多态特性被引入到方法调用举例，运行时查找通过一个叫做虚拟表的东西引入。然而对于消息机制，查找则是由运行时来完成的。实际上，编译器甚至都不关心接受消息的对象类型。它也能够通过动态绑定机制在运行时工作的很好，这个机制会在条目11中提到更多。
Objective-C由运行时做了很多繁重的事情，而不是编译器。运行时包含了使Objective-C能够以面向对象的形式工作所需要的的数据结构和方法信息。例如，运行时含有所有内存管理对应的方法。本质上，运行时是一个集合，它是你的代码和你的代码链接的库的集合。因此，当运行时更新的时候，你的应用在性能提高上会因此获益。一门在编译时间做了很多工作的语言如果想得到类似的性能提高就需要重新编译一次。
Objective-C是C的超集，因此所有C语言的特性在Objective-C中都可以使用。因此，为了更有效的写出Objective-C，你需要理解C和Objective-C的核心概念。特别地，理解C语言地内存模块会有助于理解Objective-C地内存模型，并且明白引用计数是以何种方式工作地。这就需要理解Objective-C中地指针如何被用来指向一个对象。当你定义了一个变量用来指向一个对象地引用，语法看起来是这样地：
NSString *someString = @"The string";
这个语法，直接从C移植而来，定义了一个指向NSString类型叫做something地变量。它的意思是这是一个执行一个NSString类型的指针。所有的Objective-C对象都应该以这种方式定义，因为这些对象的内存需要在堆上而不是在栈上分配。下面这样定义一个在栈上初始化的对象是不合法的：
NSString stackString;// error: interface type cannot be statically allocated
something这个变量指向一块在堆上进行初始化后包含NSString对象的内存。这也就意味着创建林一个指向同一个位置的变量不需要复制一份，只需要将两个变量指向同一个对象就可以了。
NSString *someString = @"The string";NSString *anotherString = someString;
####Figure 1.1 显示一个基于堆的字符串类型和两个指向它的基于栈的指针的内存分布
在这里只有一个NSString实例，但有两个指向同一个实例的两个指针变量。这两个指针变量都是指向字符串类型的指针，也就是当前的栈含有2个位指针大小的内存(32位架构对应4个字节，64位架构对应8个字节)。这块内存包含相同的值，那就是字符串类型实例的内存地址。
图表1.1解释了这个内存分布。字符类型的数据包含了真实字符串的字节信息。
在堆上初始化的内存需要直接管理，而栈上的指向变量的内存当他们初始化后被弹出则可以自动被清理掉。
对于堆内存的内存管理在Objective-C被抽象出来。你不需要使用malloc和free装的方法去初始化或者释放对象的内存。Objective-C的运行时通过交通引用计数的内存管理模型将它独立出来。
在Objective-C，有时候你需要面对定义中不存在*并且使用堆空间的变量。这些变量并没有引用Objective-C对象。一个例子就是CoreGraphics框架中的CGRect：CGRect frame;frame.origin.x = 0.0f;frame.origin.y = 10.0f;frame.size.width = 100.0f;frame.size.height = 150.0f;A CGRect is a C structure, defined like so:struct CGRect {  CGPoint origin;  CGSize size;};typedef struct CGRect CGRect;
这种类型的结构在系统框架之外，可能由于过度使用Objective-C对象而影响性能的时候被使用。创建一个这样的对象，会需要创建和释放堆内存，而结构体不需要。当非对象类型(int, float, double, char等等)需要存储数据的时候，例如CGRect, 就会经常被用到。
在使用Objective-C开始谢任何代码的时候，鼓励你阅读一下C语言相关的文档，并且熟悉它的语法。如果你能够继续深入Objective-C，你会发现语法混淆的特性部分。
需要记住的点：
* Objective-C是添加了面向对象特性的C语言的超集。Objective-C通过动态绑定使用消息机制实现面向对象，也就是一个对象的类型在运行时才能够确定。运行时，而不是编译器，决定那一块代码会被给出的消息执行。
* 理解C语言的核心概念能够帮助你写出更高效的Objective-C代码。特别地，你需要理解内存模型和指针。