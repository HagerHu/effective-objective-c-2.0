## 条目4: 优先使用类型变量，而不是预编译#define


写代码的时候，你经常需要定义一个常量。例如，一个能够通过动画弹出和隐藏自己的视图类。动画时间就是典型的需要独立出来的一个常量。 你已经学过所有关于Objective-C和C的基础， 因此你可能会这么做：

＃define ANIMATION_DURATION 0.3

这就是一个直接的预编译；不管ANIMATION_DURATION在代码的哪个地方，都会被替换成0.3。这感觉就是你想要的，胆识这种方式没有任何类型信息。这就好像是一个等同于持续时间的值，一点也不准确。同样，预编译会和所有出现ANIMATION_DURATION的地方绑定，因此如果是在一个头文件中定义的，那么所有引入这个头文件的地方都会看到这个定义。

为了解决这些问题，你应该充分使用编译器。通常有更好的办法定义一个常量，而不是通过预编译。例如，下面定义了一个NSTimerInterval类型的常量：

static const NSTimerInterval kAnimationDuration = 0.3;

注意这种形式含有了类型信息， 明确标示这个常量是什么通过很有用。 类型是NSTimerInterval, 这样也可以通过文件说明如何使用这个常量。 如果你有很多常量需要定义，这样无疑可以帮你和其他开发者在后面更好的理解代码。

同时注意下常量是如何命名的。对于常量的命名通过在前面加个字母k，用于标示仅针对当前的实现文件有用。对于需要向类外面暴露的常量，通常在前面加上类的名字。条目19将会讲解更多关于名字的定义。

在哪里定义常量也很重要。有时候，会在头文件中定义预编译常量，但那是非常不好的做法，尤其是他们没有以上面的方式命名。例如，ANIMATION_DURATION常量在头文件中定义则是一个非常不好的命字。即使static const也不应该出现在头文件中。由于Objective-C没有命名空间，这样就会定义叫做kAnimationDuration的全局变量。它的名字应该加个表明这个类的前缀，例如EOCViewClassAnimationDuration。条目19将会说明更多关于命名的机制。

一个不被暴露给外部的常量应该在使用它的实现文件中定义。比如，动画持续时间在使用UIKit的iOS应用程序中，在UIView的子类中使用时看起来像这样：

// EOCAnimatedView.h＃import <UIKit/UIKit.h>@interface EOCAnimatedView : UIView 
- (void)animate;@end// EOCAnimatedView.m＃import "EOCAnimatedView.h"static const NSTimeInterval kAnimationDuration = 0.3;@implementation EOCAnimatedView - (void)animate {    [UIView animateWithDuration:kAnimationDuration                     animations:^(){
                        // Perform animation                     }];}@end
常量被定义为static和const也很重要。const修饰符意味着如果你视图改变这个值，编译器会抛出这个错误。这种情况下，这正式我们所需要的，这个值不应该被改变。static修饰符表示常量只在定义它的转换单元有效。一个逻辑单元就是编译器需要的用来产生一个对象文件的输入。在Objective-C中，通常每个类(每个.m结尾的实现文件)对应一个转换单元。因此在前面的例子中，kAnimationDuration应该定义在EOCAnimatedView对应的私有输出单元中。如果变量没有被定义为static,编译器就会为其创建一个外部可见的符号。如果另一个编译单元也定义了一个相同名字的常量，链接器将会抛出类似这样的错误信息：
duplicate symbol _kAnimationDuration in:
    EOCAnimatedView.o
    EOCOtherView.o
    
实际上，当将一个变量定义为static和const的时候，编译器并不会创建一个符号，而是像预编译做的那样进行替换。然后，记住这样的好处就是太所呈现的类型信息。

有时候，你想要对外暴露某个常量。例如，你有可能在某个类中使用NSNotificationCenter通知其他对象。这在一个对象发出通知，其他监听通知的对象收到通知的情况很有用。通知有一个字符串类型的名字，而这个定义的名字就是需要对外暴露的。这么做就是让任何人都不需要直到确切的名字，又能够简单的使用变量来注册通知，以便接受这些通知。

这样的变量需要出现在全局符号表中，以便他们定义以外的外部单元能够使用。因此，这些变量需要以static const以外的方式定义。这些变量应该这么定义：

// In the header file
extern NSString *const EOCStringConstant;

// In the implementation file
NSString *const EOCStringConstant = @"VALUE";

常量需要在头文件中声明，而在实现文件中定义。对于变量的类型，const修饰符是很重要的。这些定义具有可读性，这个情况里，EOCStringConstant是一个指向NSString的常量指针。这就是我们想要的；变量应该不允许改变成指向另一个NSString对象的指针。

头文件中的关键字extern告诉编译器在一个文件中遇到引入并使用这个变量的时候该如何处理。这个关键字告诉编译器在全局符号表中会有一个EOCStringConstant的符号。这也就是说编译器即使看不到这个变量的定义也能够使用它。编译器在文件被链接的时候会直到那个变量的存在。

变量应该被仅且植被定义一次。它应该在和声明它的头文件关联的实现文件中定义。编译器将会在实现文件生成的object文件中的数据区域为这个变量开辟空间进行存储。当这个object文件连同其他object文件链接生成最后的二进制包的时候，链接器将会无论它有没有被使用，都会保留EOCStringConstant对应的全局符号。

事实上，如果符号要出现在全局符号表中，那么你应该非常谨慎命名。例如，一个处理程序登陆的类可能会在完成登陆后的通知。这个通知看起来这样：

// EOCLoginManager.h
＃import <Foundation/Foundation.h>

extern NSString *const EOCLoginManagerDidLoginNotification;

@interface EOCLoginManager : NSObject
- (void)login;
@end


// EOCLoginManager.m
＃import "EOCLoginManager.h"

NSString *const EOCLoginManagerDidNotification = @"EOCLoginManagerDidLoginNotification";

@implementation EOCLoginManager

- (void)login {
    // Perform login asynchronously, then call 'p_didLogin'.
}

- (void)p_didLogin {
    [[NSNotificationCenter defaultCenter] postNotificationName:EOCLoginDidLoginNotification object:nil];
}

@end


注意这个变量使用的名字。加上变量对应的类型名字作为前缀是明智的，能够帮助减少不必要的冲突。在系统框架中使用的尤为普通。例如，UIKit中以同样的方式定义了全局使用的通知名字。这些名字包括UIApplicationDidEnterBackgroundNotification和UIApplicationWillEnterForegroundNotification。

同理也适用于其他类型的变量定义。如果前面的例子中，动画持续时间需要暴露给EOCAnimatedView以外的类，你可以这么声明：

// EOCAniamtedView.h
extern const NSTimeInterval EOCAnimatedViewAnimationDuration;

// EOCAnimatedView.m
const NSTimeInterval EOCAnimatedViewAnimationDuration = 0.3;

以这种方式定义的变量比使用预编译要好的多，因为编译器可以确保那个值不会被改变。一旦在EOCAnimatedView.m文件种定义了，那么这个值就可以在任何地方使用。预编译可能会被错误的重新定义，也就是说程序的不同部分有可能使用不同的值。

总结一下，避免使用预编译去定义一个变量。相反，要使用对编译器可见的变量，比如在实现文件种使用static const的声明。

#### 需要记住的

* 避免使用预编译。他们在编译之前能够很容易查找并替换，却没有包含任何类型信息。他们可能被重新定义而不会有任何警告，导致整个程序中在使用不统一的值。
* 在实现文件中使用static const去定义编译单元对应的变量。这些变量将不会出现在全局符号表中，因此他们的名字也不需要命名空间。
* 在头文件中声明全局变量，而在相应的实现文件中定义。这些常量将出现在全局符号表中，因此他们的名字需要命名空间，通常会以相关的类型名字作为前缀。

