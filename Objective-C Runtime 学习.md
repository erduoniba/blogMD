### Objective-C Runtime 学习

### 一、类与对象

Objective-C语言是一门动态语言，它将很多静态语言在编译和链接时期做的事放到了运行时来处理。这种动态语言的优势在于：我们写代码时更具灵活性，如我们可以把消息转发给我们想要的对象，或者随意交换一个方法的实现等。

这种特性意味着Objective-C不仅需要一个编译器，还需要一个运行时系统来执行编译的代码。对于Objective-C来说，这个运行时系统就像一个操作系统一样：它让所有的工作可以正常的运行。这个运行时系统即`Objc Runtime`。`Objc Runtime`其实是一个`Runtime`库，它基本上是用C和汇编写的，这个库使得C语言有了面向对象的能力。

`Runtime`库主要做下面几件事：

1. 封装：在这个库中，对象可以用C语言中的结构体表示，而方法可以用C函数来实现，另外再加上了一些额外的特性。这些结构体和函数被runtime函数封装后，我们就可以在程序运行时创建，检查，修改类、对象和它们的方法了。
2. 找出方法的最终执行代码：当程序执行`[object doSomething]`时，会向消息接收者(object)发送一条消息(doSomething)，runtime会根据消息接收者是否能响应该消息而做出不同的反应。这将在后面详细介绍。

Objective-C runtime目前有两个版本：`Modern runtime`和`Legacy runtime`。`Modern Runtime`覆盖了64位的`Mac OS X Apps`，还有`iOS Apps`，`Legacy Runtime`是早期用来给32位 `Mac OS X Apps` 用的，也就是可以不用管就是了。



#### **类与对象基础数据结构**

**Class**

Objective-C类是由`Class`类型来表示的，它实际上是一个指向`objc_class`结构体的指针。它的定义如下：

```
typedef struct objc_class *Class;
```

查看`objc/runtime.h`中`objc_class`结构体的定义如下：

```objective-c
struct objc_class {

    Class isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class super_class                   	OBJC2_UNAVAILABLE;	// 父类
    const char *name                      	OBJC2_UNAVAILABLE;	// 类名
    long version                          	OBJC2_UNAVAILABLE;	// 类的版本信息，默认为0
    long info                            	OBJC2_UNAVAILABLE;	// 类信息，供运行期使用的一些位标识
    long instance_size                   	OBJC2_UNAVAILABLE;	// 该类的实例变量大小
    struct objc_ivar_list *ivars         	OBJC2_UNAVAILABLE;	// 该类的成员变量链表
    struct objc_method_list **methodLists 	OBJC2_UNAVAILABLE;	// 方法定义的链表
    struct objc_cache *cache              	OBJC2_UNAVAILABLE;	// 方法缓存
    struct objc_protocol_list *protocols 	OBJC2_UNAVAILABLE;	// 协议链表

#endif
} OBJC2_UNAVAILABLE;
```

在这个定义中，下面几个字段是我们感兴趣的

1. `isa`：是一个Class 类型的指针，他指向对象的类。需要注意的是在Objective-C中，所有的类自身也是一个对象，这个对象的`Class`里面也有一个`isa`指针，它指向`metaClass`(元类)，我们会在后面介绍它。
2. `super_class`：指向该类的父类，如果该类已经是最顶层的根类(如NSObject或NSProxy)，则`super_class`为NULL。
3. `cache`：用于缓存最近使用的方法。一个接收者对象接收到一个消息时，它会根据`isa`指针去查找能够响应这个消息的对象。在实际使用中，这个对象只有一部分方法是常用的，很多方法其实很少用或者根本用不上。这种情况下，如果每次消息来时，我们都是`methodLists`中遍历一遍，性能势必很差。这时，`cache`就派上用场了。在我们每次调用过一个方法后，这个方法就会被缓存到`cache`列表中，下次调用的时候runtime就会优先去`cache`中查找，如果`cache`没有，才去`methodLists`中查找方法。这样，对于那些经常用到的方法的调用，但提高了调用的效率。
4. `version`：我们可以使用这个字段来提供类的版本信息。这对于对象的序列化非常有用，它可是让我们识别出不同类定义版本中实例变量布局的改变。



针对`cache`，我们用下面例子来说明其执行过程：

```objective-c
NSArray *array = [[NSArray alloc] init];
​```	

其流程是：

1. `[NSArray alloc]`先被执行。因为NSArray没有`+alloc`方法，于是去父类NSObject去查找。
2. 检测NSObject是否响应`+alloc`方法，发现响应，于是检测NSArray类，并根据其所需的内存空间大小开始分配内存空间，然后把`isa`指针指向NSArray类。同时，`+alloc`也被加进cache列表里面。
3. 接着，执行`-init`方法，如果NSArray响应该方法，则直接将其加入`cache`；如果不响应，则去父类查找。
4. 在后期的操作中，如果再以`[[NSArray alloc] init]`这种方式来创建数组，则会直接从cache中取出相应的方法，直接调用。

### objc_object与id

`objc_object`是表示一个类的实例的结构体，它的定义如下(`objc/objc.h`)：

​```objc
struct objc_object {
    Class isa  OBJC_ISA_AVAILABILITY;
};

typedef struct objc_object *id;
```

可以看到，这个结构体只有一个字体，即指向其类的`isa`指针。这样，当我们向一个Objective-C对象发送消息时，运行时库会根据实例对象的`isa`指针找到这个实例对象所属的类。`Runtime`库会在类的方法列表及父类的方法列表中去寻找与消息对应的`selector`指向的方法。找到后即运行这个方法。

当创建一个特定类的实例对象时，分配的内存包含一个`objc_object`数据结构，然后是类的实例变量的数据。NSObject类的`alloc`和`allocWithZone:`方法使用函数`class_createInstance`来创建`objc_object`数据结构。

另外还有我们常见的id，它是一个`objc_object`结构类型的指针。它的存在可以让我们实现类似于C++中泛型的一些操作。该类型的对象可以转换为任何一种对象，有点类似于C语言中`void *`指针类型的作用。



**objc_cache**

上面提到了`objc_class`结构体中的`cache`字段，它用于缓存调用过的方法。这个字段是一个指向`objc_cache`结构体的指针，其定义如下：

```objective-c
struct objc_cache {

    unsigned int mask /* total = mask + 1 */                 OBJC2_UNAVAILABLE;
    unsigned int occupied                                    OBJC2_UNAVAILABLE;
    Method buckets[1]                                        OBJC2_UNAVAILABLE;

};
```

该结构体的字段描述如下：

1. `mask`：一个整数，指定分配的缓存`bucket`的总数。在方法查找过程中，Objective-C runtime使用这个字段来确定开始线性查找数组的索引位置。指向方法`selector`的指针与该字段做一个`AND`位操作`(index = (mask & selector))`。这可以作为一个简单的`hash`散列算法。
2. `occupied`：一个整数，指定实际占用的缓存`bucket`的总数。
3. `buckets`：指向`Method`数据结构指针的数组。这个数组可能包含不超过`mask+1`个元素。需要注意的是，指针可能是NULL，表示这个缓存`bucket`没有被占用，另外被占用的`bucket`可能是不连续的。这个数组可能会随着时间而增长。

### 

**元类(Meta Class)**

在上面我们提到，所有的类自身也是一个对象，我们可以向这个对象发送消息(即调用类方法)。如：

```objective-c
NSArray *array = [NSArray array];
```

这个例子中，`+array`消息发送给了NSArray类，而这个NSArray也是一个对象。既然是对象，那么它也是一个`objc_object`指针，它包含一个指向其类的一个`isa`指针。那么这些就有一个问题了，这个`isa`指针指向什么呢？为了调用`+array`方法，这个类的`isa`指针必须指向一个包含这些类方法的一个`objc_class`结构体。这就引出了`meta-class`的概念

```objective-c
meta-class是一个类对象的类。
```

当我们向一个对象发送消息时，runtime会在这个对象所属的这个类的方法列表中查找方法；而向一个类发送消息时，会在这个类的`meta-class`的方法列表中查找。

`meta-class`之所以重要，是因为它存储着一个类的所有类方法。每个类都会有一个单独的`meta-class`，因为每个类的类方法基本不可能完全相同。

再深入一下，`meta-class`也是一个类，也可以向它发送一个消息，那么它的`isa`又是指向什么呢？为了不让这种结构无限延伸下去，Objective-C的设计者让所有的`meta-class`的`isa`指向基类的`meta-class`，以此作为它们的所属类。即，任何NSObject继承体系下的`meta-class`都使用NSObject的`meta-class`作为自己的所属类，而基类的`meta-class`的`isa`指针是指向它自己。这样就形成了一个完美的闭环。

通过上面的描述，再加上对`objc_class`结构体中`super_class`指针的分析，我们就可以描绘出类及相应`meta-class`类的一个继承体系了，如下图所示：

![](http://upload-images.jianshu.io/upload_images/449095-3e972ec16703c54d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240/q/100) 



讲了这么多，我们还是来写个例子吧：

```objective-c
void TestMetaClass(id self, SEL _cmd) {

    NSLog(@"This objcet is %p", self);
    NSLog(@"Class is %@, super class is %@", [self class], [self superclass]);

    Class currentClass = [self class];
    for (int i = 0; i < 4; i++) {
        NSLog(@"Following the isa pointer %d times gives %p", i, currentClass);
        currentClass = objc_getClass((__bridge void *)currentClass);
    }

    NSLog(@"NSObject's class is %p", [NSObject class]);
    NSLog(@"NSObject's meta class is %p", objc_getClass((__bridge void *)[NSObject class]));
}

#pragma mark -
@implementation Test

- (void)ex_registerClassPair {

    Class newClass = objc_allocateClassPair([NSError class], "TestClass", 0);
    class_addMethod(newClass, @selector(testMetaClass), (IMP)TestMetaClass, "v@:");
    objc_registerClassPair(newClass);

    id instance = [[newClass alloc] initWithDomain:@"some domain" code:0 userInfo:nil];
    [instance performSelector:@selector(testMetaClass)];
}

@end
```

这个例子是在运行时创建了一个`NSError`的子类`TestClass`，然后为这个子类添加一个方法`testMetaClass`，这个方法的实现是`TestMetaClass`函数。

运行后，打印结果是

```objective-c
2014-10-20 22:57:07.352 mountain[1303:41490] This objcet is 0x7a6e22b0
2014-10-20 22:57:07.353 mountain[1303:41490] Class is TestStringClass, super class is NSError
2014-10-20 22:57:07.353 mountain[1303:41490] Following the isa pointer 0 times gives 0x7a6e21b0
2014-10-20 22:57:07.353 mountain[1303:41490] Following the isa pointer 1 times gives 0x0
2014-10-20 22:57:07.353 mountain[1303:41490] Following the isa pointer 2 times gives 0x0
2014-10-20 22:57:07.353 mountain[1303:41490] Following the isa pointer 3 times gives 0x0
2014-10-20 22:57:07.353 mountain[1303:41490] NSObject's class is 0xe10000
2014-10-20 22:57:07.354 mountain[1303:41490] NSObject's meta class is 0x0
```

我们在for循环中，我们通过`objc_getClass`来获取对象的`isa`，并将其打印出来，依此一直回溯到NSObject的`meta-class`。分析打印结果，可以看到最后指针指向的地址是0x0，即NSObject的`meta-class`的类地址。

*这里需要注意的是：我们在一个类对象调用class方法是无法获取meta-class，它只是返回类而已。*



#### **类与对象操作函数**

runtime提供了大量的函数来操作类与对象。类的操作方法大部分是以`class_`为前缀的，而对象的操作方法大部分是以`objc_`或`object_`为前缀。下面我们将根据这些方法的用途来分类讨论这些方法的使用。

**类相关操作函数**

我们可以回过头去看看`objc_class`的定义，runtime提供的操作类的方法主要就是针对这个结构体中的各个字段的。下面我们分别介绍这一些的函数。并在最后以实例来演示这些函数的具体用法。

**类名(name)**

类名操作的函数主要有：

```objective-c
// 获取类的类名
const char * class_getName ( Class cls );
```

- 对于`class_getName`函数，如果传入的`cls`为`Nil`，则返回一个字字符串。

**父类(super_class)和元类(meta-class)**

父类和元类操作的函数主要有：

```objective-c
// 获取类的父类
Class class_getSuperclass ( Class cls );

// 判断给定的Class是否是一个元类
BOOL class_isMetaClass ( Class cls );
```

- `class_getSuperclass`函数，当`cls`为Nil或者`cls`为根类时，返回Nil。不过通常我们可以使用NSObject类的`superclass`方法来达到同样的目的。
- `class_isMetaClass`函数，如果是`cls`是元类，则返回YES；如果否或者传入的`cls`为Nil，则返回NO。



**实例变量大小(instance_size)**

实例变量大小操作的函数有：

```objective-c
// 获取实例大小
size_t class_getInstanceSize ( Class cls );
```



**成员变量(ivars)及属性**

在`objc_class`中，所有的成员变量、属性的信息是放在链表`ivars`中的。`ivars`是一个数组，数组中每个元素是指向`Ivar`(变量信息)的指针。runtime提供了丰富的函数来操作这一字段。大体上可以分为以下几类：

1.成员变量操作函数，主要包含以下函数：

```objective-c
// 获取类中指定名称实例成员变量的信息
Ivar class_getInstanceVariable ( Class cls, const char *name );

// 获取类成员变量的信息
Ivar class_getClassVariable ( Class cls, const char *name );

// 添加成员变量
BOOL class_addIvar ( Class cls, const char *name, size_t size, uint8_t alignment, const char *types );

// 获取整个成员变量列表
Ivar * class_copyIvarList ( Class cls, unsigned int *outCount );
```

- `class_getInstanceVariable`函数，它返回一个指向包含name指定的成员变量信息的`objc_ivar`结构体的指针(`Ivar`)。
- `class_getClassVariable`函数，目前没有找到关于Objective-C中类变量的信息，一般认为Objective-C不支持类变量。注意，返回的列表不包含父类的成员变量和属性。
- Objective-C不支持往已存在的类中添加实例变量，因此不管是系统库提供的类，还是我们自定义的类，都无法动态添加成员变量。但如果我们通过运行时来创建一个类的话，又应该如何给它添加成员变量呢？这时我们就可以使用`class_addIvar`函数了。不过需要注意的是，这个方法只能在`objc_allocateClassPair`函数与`objc_registerClassPair`之间调用。另外，这个类也不能是元类。成员变量的按字节最小对齐量是`1<alignment`。这取决于`ivar`的类型和机器的架构。如果变量的类型是指针类型，则传递`log2(sizeof(pointer_type))`。
- `class_copyIvarList`函数，它返回一个指向成员变量信息的数组，数组中每个元素是指向该成员变量信息的`objc_ivar`结构体的指针。这个数组不包含在父类中声明的变量。`outCount`指针返回数组的大小。需要注意的是，我们必须使用`free()`来释放这个数组。

2.属性操作函数，主要包含以下函数：

```objective-c
// 获取指定的属性
objc_property_t class_getProperty ( Class cls, const char *name );

// 获取属性列表
objc_property_t * class_copyPropertyList ( Class cls, unsigned int *outCount );

// 为类添加属性
BOOL class_addProperty ( Class cls, const char *name, const objc_property_attribute_t *attributes, unsigned int attributeCount );

// 替换类的属性
void class_replaceProperty ( Class cls, const char *name, const objc_property_attribute_t *attributes, unsigned int attributeCount );
```

这一种方法也是针对`ivars`来操作，不过只操作那些是属性的值。我们在后面介绍属性时会再遇到这些函数。

3.在MAC OS X系统中，我们可以使用垃圾回收器。runtime提供了几个函数来确定一个对象的内存区域是否可以被垃圾回收器扫描，以处理`strong/weak`引用。这几个函数定义如下：

```objective-c
const uint8_t * class_getIvarLayout ( Class cls );
void class_setIvarLayout ( Class cls, const uint8_t *layout );
const uint8_t * class_getWeakIvarLayout ( Class cls );
void class_setWeakIvarLayout ( Class cls, const uint8_t *layout );
```

但通常情况下，我们不需要去主动调用这些方法；在调用`objc_registerClassPair`时，会生成合理的布局。在此不详细介绍这些函数。



**方法(methodLists)**

方法操作主要有以下函数：

```objective-c
// 添加方法
BOOL class_addMethod ( Class cls, SEL name, IMP imp, const char *types );

// 获取实例方法
Method class_getInstanceMethod ( Class cls, SEL name );

// 获取类方法
Method class_getClassMethod ( Class cls, SEL name );

// 获取所有方法的数组
Method * class_copyMethodList ( Class cls, unsigned int *outCount );

// 替代方法的实现
IMP class_replaceMethod ( Class cls, SEL name, IMP imp, const char *types );

// 返回方法的具体实现
IMP class_getMethodImplementation ( Class cls, SEL name );
IMP class_getMethodImplementation_stret ( Class cls, SEL name );

// 类实例是否响应指定的selector
BOOL class_respondsToSelector ( Class cls, SEL sel );
```

- `class_addMethod`的实现会覆盖父类的方法实现，但不会取代本类中已存在的实现，如果本类中包含一个同名的实现，则函数会返回NO。如果要修改已存在实现，可以使用`method_setImplementation`。一个Objective-C方法是一个简单的C函数，它至少包含两个参数–`self`和`_cmd`。所以，我们的实现函数(IMP参数指向的函数)至少需要两个参数，如下所示：
- `void myMethodIMP(id self, SEL _cmd){    // implementation ....}`

与成员变量不同的是，我们可以为类动态添加方法，不管这个类是否已存在。

另外，参数`types`是一个描述传递给方法的参数类型的字符数组，这就涉及到类型编码，我们将在后面介绍。

- `class_getInstanceMethod`、`class_getClassMethod`函数，与`class_copyMethodList`不同的是，这两个函数都会去搜索父类的实现。
- `class_copyMethodList`函数，返回包含所有实例方法的数组，如果需要获取类方法，则可以使用`class_copyMethodList(object_getClass(cls), &count)`(一个类的实例方法是定义在元类里面)。该列表不包含父类实现的方法。`outCount`参数返回方法的个数。在获取到列表后，我们需要使用`free()`方法来释放它。
- `class_replaceMethod`函数，该函数的行为可以分为两种：如果类中不存在name指定的方法，则类似于`class_addMethod`函数一样会添加方法；如果类中已存在name指定的方法，则类似于`method_setImplementation`一样替代原方法的实现。
- `class_getMethodImplementation`函数，该函数在向类实例发送消息时会被调用，并返回一个指向方法实现函数的指针。这个函数会比`method_getImplementation(class_getInstanceMethod(cls, name))`更快。返回的函数指针可能是一个指向runtime内部的函数，而不一定是方法的实际实现。例如，如果类实例无法响应`selector`，则返回的函数指针将是运行时消息转发机制的一部分。
- `class_respondsToSelector`函数，我们通常使用NSObject类的`respondsToSelector:`或`instancesRespondToSelector:`方法来达到相同目的。



**实例(Example)**

上面列举了大量类操作的函数，下面我们写个实例，来看看这些函数的实例效果：

```objective-c
//-----------------------------------------------------------
// MyClass.h

@interface MyClass : NSObject <NSCopying, NSCoding>

@property (nonatomic, strong) NSArray *array;
@property (nonatomic, copy) NSString *string;

- (void)method1;
- (void)method2;
+ (void)classMethod1;

@end

//-----------------------------------------------------------
// MyClass.m

#import "MyClass.h"

@interface MyClass () {
    NSInteger       _instance1;
    NSString    *   _instance2;
}

@property (nonatomic, assign) NSUInteger integer;

- (void)method3WithArg1:(NSInteger)arg1 arg2:(NSString *)arg2;

@end

@implementation MyClass

+ (void)classMethod1 {

}

- (void)method1 {
    NSLog(@"call method method1");
}

- (void)method2 {

}

- (void)method3WithArg1:(NSInteger)arg1 arg2:(NSString *)arg2 {
    NSLog(@"arg1 : %ld, arg2 : %@", arg1, arg2);
}
@end

//-----------------------------------------------------------
// main.h
#import "MyClass.h"
#import "MySubClass.h"
#import <objc/runtime.h>

int main(int argc, const char * argv[]) {
    @autoreleasepool {
    
        MyClass *myClass = [[MyClass alloc] init];
        unsigned int outCount = 0;
        Class cls = myClass.class;

        // 类名
        NSLog(@"class name: %s", class_getName(cls));
        NSLog(@"==========================================================");

        // 父类
        NSLog(@"super class name: %s", class_getName(class_getSuperclass(cls)));
        NSLog(@"==========================================================");

        // 是否是元类
        NSLog(@"MyClass is %@ a meta-class", (class_isMetaClass(cls) ? @"" : @"not"));
        NSLog(@"==========================================================");

        Class meta_class = objc_getMetaClass(class_getName(cls));
        NSLog(@"%s's meta-class is %s", class_getName(cls), class_getName(meta_class));
        NSLog(@"==========================================================");

        // 变量实例大小
        NSLog(@"instance size: %zu", class_getInstanceSize(cls));
        NSLog(@"==========================================================");

        // 成员变量
        Ivar *ivars = class_copyIvarList(cls, &outCount);
        for (int i = 0; i < outCount; i++) {
            Ivar ivar = ivars[i];
            NSLog(@"instance variable's name: %s at index: %d", ivar_getName(ivar), i);
        }
        free(ivars);

        Ivar string = class_getInstanceVariable(cls, "_string");
        if (string != NULL) {
            NSLog(@"instace variable %s", ivar_getName(string));
        }

        NSLog(@"==========================================================");

        // 属性操作
        objc_property_t * properties = class_copyPropertyList(cls, &outCount);

        for (int i = 0; i < outCount; i++) {
            objc_property_t property = properties[i];
            NSLog(@"property's name: %s", property_getName(property));
        }
        free(properties);

        objc_property_t array = class_getProperty(cls, "array");
        if (array != NULL) {
            NSLog(@"property %s", property_getName(array));
        }

        NSLog(@"==========================================================");

        // 方法操作
        Method *methods = class_copyMethodList(cls, &outCount);
        for (int i = 0; i < outCount; i++) {
            Method method = methods[i];
            NSLog(@"method's signature: %s", method_getName(method));
        }

        free(methods);

        Method method1 = class_getInstanceMethod(cls, @selector(method1));
        if (method1 != NULL) {
            NSLog(@"method %s", method_getName(method1));
        }

        Method classMethod = class_getClassMethod(cls, @selector(classMethod1));
        if (classMethod != NULL) {
            NSLog(@"class method : %s", method_getName(classMethod));
        }

        NSLog(@"MyClass is%@ responsd to selector: method3WithArg1:arg2:", class_respondsToSelector(cls, @selector(method3WithArg1:arg2:)) ? @"" : @" not");

        IMP imp = class_getMethodImplementation(cls, @selector(method1));
        imp();

        NSLog(@"==========================================================");

        // 协议
        Protocol * __unsafe_unretained * protocols = class_copyProtocolList(cls, &outCount);
        Protocol * protocol;
        for (int i = 0; i < outCount; i++) {
            protocol = protocols[i];
            NSLog(@"protocol name: %s", protocol_getName(protocol));
        }

        NSLog(@"MyClass is%@ responsed to protocol %s", class_conformsToProtocol(cls, protocol) ? @"" : @" not", protocol_getName(protocol));
        NSLog(@"==========================================================");
    }

    return 0;
}
```

这段程序的输出如下：

```objective-c
2014-10-22 19:41:37.452 RuntimeTest[3189:156810] class name: MyClass
2014-10-22 19:41:37.453 RuntimeTest[3189:156810] ==========================================================
2014-10-22 19:41:37.454 RuntimeTest[3189:156810] super class name: NSObject
2014-10-22 19:41:37.454 RuntimeTest[3189:156810] ==========================================================
2014-10-22 19:41:37.454 RuntimeTest[3189:156810] MyClass is not a meta-class
2014-10-22 19:41:37.454 RuntimeTest[3189:156810] ==========================================================
2014-10-22 19:41:37.454 RuntimeTest[3189:156810] MyClass's meta-class is MyClass
2014-10-22 19:41:37.455 RuntimeTest[3189:156810] ==========================================================
2014-10-22 19:41:37.455 RuntimeTest[3189:156810] instance size: 48
2014-10-22 19:41:37.455 RuntimeTest[3189:156810] ==========================================================
2014-10-22 19:41:37.455 RuntimeTest[3189:156810] instance variable's name: _instance1 at index: 0
2014-10-22 19:41:37.455 RuntimeTest[3189:156810] instance variable's name: _instance2 at index: 1
2014-10-22 19:41:37.455 RuntimeTest[3189:156810] instance variable's name: _array at index: 2
2014-10-22 19:41:37.455 RuntimeTest[3189:156810] instance variable's name: _string at index: 3
2014-10-22 19:41:37.463 RuntimeTest[3189:156810] instance variable's name: _integer at index: 4
2014-10-22 19:41:37.463 RuntimeTest[3189:156810] instace variable _string
2014-10-22 19:41:37.463 RuntimeTest[3189:156810] ==========================================================
2014-10-22 19:41:37.463 RuntimeTest[3189:156810] property's name: array
2014-10-22 19:41:37.463 RuntimeTest[3189:156810] property's name: string
2014-10-22 19:41:37.464 RuntimeTest[3189:156810] property's name: integer
2014-10-22 19:41:37.464 RuntimeTest[3189:156810] property array
2014-10-22 19:41:37.464 RuntimeTest[3189:156810] ==========================================================
2014-10-22 19:41:37.464 RuntimeTest[3189:156810] method's signature: method1
2014-10-22 19:41:37.464 RuntimeTest[3189:156810] method's signature: method2
2014-10-22 19:41:37.464 RuntimeTest[3189:156810] method's signature: method3WithArg1:arg2:
2014-10-22 19:41:37.465 RuntimeTest[3189:156810] method's signature: integer
2014-10-22 19:41:37.465 RuntimeTest[3189:156810] method's signature: setInteger:
2014-10-22 19:41:37.465 RuntimeTest[3189:156810] method's signature: array
2014-10-22 19:41:37.465 RuntimeTest[3189:156810] method's signature: string
2014-10-22 19:41:37.465 RuntimeTest[3189:156810] method's signature: setString:
2014-10-22 19:41:37.465 RuntimeTest[3189:156810] method's signature: setArray:
2014-10-22 19:41:37.466 RuntimeTest[3189:156810] method's signature: .cxx_destruct
2014-10-22 19:41:37.466 RuntimeTest[3189:156810] method method1
2014-10-22 19:41:37.466 RuntimeTest[3189:156810] class method : classMethod1
2014-10-22 19:41:37.466 RuntimeTest[3189:156810] MyClass is responsd to selector: method3WithArg1:arg2:
2014-10-22 19:41:37.467 RuntimeTest[3189:156810] call method method1
2014-10-22 19:41:37.467 RuntimeTest[3189:156810] ==========================================================
2014-10-22 19:41:37.467 RuntimeTest[3189:156810] protocol name: NSCopying
2014-10-22 19:41:37.467 RuntimeTest[3189:156810] protocol name: NSCoding
2014-10-22 19:41:37.467 RuntimeTest[3189:156810] MyClass is responsed to protocol NSCoding
2014-10-22 19:41:37.468 RuntimeTest[3189:156810] ==========================================================
```



**动态创建类和对象**

runtime的强大之处在于它能在运行时创建类和对象。



**动态创建类**

动态创建类涉及到以下几个函数：

```objective-c
// 创建一个新类和元类
Class objc_allocateClassPair ( Class superclass, const char *name, size_t extraBytes );

// 销毁一个类及其相关联的类
void objc_disposeClassPair ( Class cls );

// 在应用中注册由objc_allocateClassPair创建的类
void objc_registerClassPair ( Class cls );
```

- `objc_allocateClassPair`函数：如果我们要创建一个根类，则`superclass`指定为Nil。`extraBytes`通常指定为0，该参数是分配给类和元类对象尾部的索引`ivars`的字节数。

为了创建一个新类，我们需要调用`objc_allocateClassPair`。然后使用诸如`class_addMethod`，`class_addIvar`等函数来为新创建的类添加方法、实例变量和属性等。完成这些后，我们需要调用`objc_registerClassPair`函数来注册类，之后这个新类就可以在程序中使用了。

实例方法和实例变量应该添加到类自身上，而类方法应该添加到类的元类上。

- `objc_disposeClassPair`函数用于销毁一个类，不过需要注意的是，如果程序运行中还存在类或其子类的实例，则不能调用针对类调用该方法。

在前面介绍元类时，我们已经有接触到这几个函数了，在此我们再举个实例来看看这几个函数的使用。

```objective-c
Class cls = objc_allocateClassPair(MyClass.class, "MySubClass", 0);

class_addMethod(cls, @selector(submethod1), (IMP)imp_submethod1, "v@:");
class_replaceMethod(cls, @selector(method1), (IMP)imp_submethod1, "v@:");
class_addIvar(cls, "_ivar1", sizeof(NSString *), log(sizeof(NSString *)), "i");

objc_property_attribute_t type = {"T", "@\"NSString\""};
objc_property_attribute_t ownership = { "C", "" };
objc_property_attribute_t backingivar = { "V", "_ivar1"};
objc_property_attribute_t attrs[] = {type, ownership, backingivar};

class_addProperty(cls, "property2", attrs, 3);
objc_registerClassPair(cls);

id instance = [[cls alloc] init];
[instance performSelector:@selector(submethod1)];
[instance performSelector:@selector(method1)];
```

程序的输出如下：

```objective-c
2014-10-23 11:35:31.006 RuntimeTest[3800:66152] run sub method 1
2014-10-23 11:35:31.006 RuntimeTest[3800:66152] run sub method 1
```



**动态创建对象**

动态创建对象的函数如下：

```objective-c
// 创建类实例
id class_createInstance ( Class cls, size_t extraBytes );

// 在指定位置创建类实例
id objc_constructInstance ( Class cls, void *bytes );

// 销毁类实例
void * objc_destructInstance ( id obj );
```

- `class_createInstance`函数：创建实例时，会在默认的内存区域为类分配内存。`extraBytes`参数表示分配的额外字节数。这些额外的字节可用于存储在类定义中所定义的实例变量之外的实例变量。该函数在ARC环境下无法使用。

调用`class_createInstance`的效果与`+alloc`方法类似。不过在使用`class_createInstance`时，我们需要确切的知道我们要用它来做什么。在下面的例子中，我们用NSString来测试一下该函数的实际效果：

```objective-c
id theObject = class_createInstance(NSString.class, sizeof(unsigned));
 
id str1 = [theObject init];
NSLog(@"%@", [str1 class]);

id str2 = [[NSString alloc] initWithString:@"test"];
NSLog(@"%@", [str2 class]);
```

输出结果是：

```objective-c
2014-10-23 12:46:50.781 RuntimeTest[4039:89088] NSString
2014-10-23 12:46:50.781 RuntimeTest[4039:89088] __NSCFConstantString
```

可以看到，使用`class_createInstance`函数获取的是NSString实例，而不是类簇中的默认占位符类`__NSCFConstantString`。

- `objc_constructInstance`函数：在指定的位置(bytes)创建类实例。
- `objc_destructInstance`函数：销毁一个类的实例，但不会释放并移除任何与其相关的引用。



**实例操作函数**

实例操作函数主要是针对我们创建的实例对象的一系列操作函数，我们可以使用这组函数来从实例对象中获取我们想要的一些信息，如实例对象中变量的值。这组函数可以分为三小类：

1.针对整个对象进行操作的函数，这类函数包含

```objective-c
// 返回指定对象的一份拷贝
id object_copy ( id obj, size_t size );

// 释放指定对象占用的内存
id object_dispose ( id obj );
```

有这样一种场景，假设我们有类A和类B，且类B是类A的子类。类B通过添加一些额外的属性来扩展类A。现在我们创建了一个A类的实例对象，并希望在运行时将这个对象转换为B类的实例对象，这样可以添加数据到B类的属性中。这种情况下，我们没有办法直接转换，因为B类的实例会比A类的实例更大，没有足够的空间来放置对象。此时，我们就要以使用以上几个函数来处理这种情况，如下代码所示：

```objective-c
NSObject *a = [[NSObject alloc] init];
id newB = object_copy(a, class_getInstanceSize(MyClass.class));
object_setClass(newB, MyClass.class);
object_dispose(a);
```

2.针对对象实例变量进行操作的函数，这类函数包含：

```objective-c
// 修改类实例的实例变量的值
Ivar object_setInstanceVariable ( id obj, const char *name, void *value );

// 获取对象实例变量的值
Ivar object_getInstanceVariable ( id obj, const char *name, void **outValue );

// 返回指向给定对象分配的任何额外字节的指针
void * object_getIndexedIvars ( id obj );

// 返回对象中实例变量的值
id object_getIvar ( id obj, Ivar ivar );

// 设置对象中实例变量的值
void object_setIvar ( id obj, Ivar ivar, id value );
```

如果实例变量的`Ivar`已经知道，那么调用`object_getIvar`会比`object_getInstanceVariable`函数快，相同情况下，`object_setIvar`也比`object_setInstanceVariable`快。

3.针对对象的类进行操作的函数，这类函数包含：

```objective-c
// 返回给定对象的类名
const char * object_getClassName ( id obj );

// 返回对象的类
Class object_getClass ( id obj );

// 设置对象的类
Class object_setClass ( id obj, Class cls );
```



**获取类定义**

Objective-C动态运行库会自动注册我们代码中定义的所有的类。我们也可以在运行时创建类定义并使用`objc_addClass`函数来注册它们。`runtime`提供了一系列函数来获取类定义相关的信息，这些函数主要包括：

```objective-c
// 获取已注册的类定义的列表
int objc_getClassList ( Class *buffer, int bufferCount );

// 创建并返回一个指向所有已注册类的指针列表
Class * objc_copyClassList ( unsigned int *outCount );

// 返回指定类的类定义
Class objc_lookUpClass ( const char *name );
Class objc_getClass ( const char *name );
Class objc_getRequiredClass ( const char *name );

// 返回指定类的元类
Class objc_getMetaClass ( const char *name );
```

- `objc_getClassList`函数：获取已注册的类定义的列表。我们不能假设从该函数中获取的类对象是继承自NSObject体系的，所以在这些类上调用方法是，都应该先检测一下这个方法是否在这个类中实现。

下面代码演示了该函数的用法：

```objective-c
int numClasses;
Class * classes = NULL;

numClasses = objc_getClassList(NULL, 0);
if (numClasses > 0) {
    classes = malloc(sizeof(Class) * numClasses);
    numClasses = objc_getClassList(classes, numClasses);

    NSLog(@"number of classes: %d", numClasses);

    for (int i = 0; i < numClasses; i++) {

        Class cls = classes[i];
        NSLog(@"class name: %s", class_getName(cls));
    }

    free(classes);
}
```

输出结果如下：

```objective-c
2014-10-23 16:20:52.589 RuntimeTest[8437:188589] number of classes: 1282
2014-10-23 16:20:52.589 RuntimeTest[8437:188589] class name: DDTokenRegexp
2014-10-23 16:20:52.590 RuntimeTest[8437:188589] class name: _NSMostCommonKoreanCharsKeySet
2014-10-23 16:20:52.590 RuntimeTest[8437:188589] class name: OS_xpc_dictionary
2014-10-23 16:20:52.590 RuntimeTest[8437:188589] class name: NSFileCoordinator
2014-10-23 16:20:52.590 RuntimeTest[8437:188589] class name: NSAssertionHandler
2014-10-23 16:20:52.590 RuntimeTest[8437:188589] class name: PFUbiquityTransactionLogMigrator
2014-10-23 16:20:52.591 RuntimeTest[8437:188589] class name: NSNotification
2014-10-23 16:20:52.591 RuntimeTest[8437:188589] class name: NSKeyValueNilSetEnumerator
2014-10-23 16:20:52.591 RuntimeTest[8437:188589] class name: OS_tcp_connection_tls_session
2014-10-23 16:20:52.591 RuntimeTest[8437:188589] class name: _PFRoutines
......还有大量输出
```

- 获取类定义的方法有三个：`objc_lookUpClass`, `objc_getClass`和`objc_getRequiredClass`。如果类在运行时未注册，则`objc_lookUpClass`会返回nil，而`objc_getClass`会调用类处理回调，并再次确认类是否注册，如果确认未注册，再返回nil。而`objc_getRequiredClass`函数的操作与`objc_getClass`相同，只不过如果没有找到类，则会杀死进程。
- `objc_getMetaClass`函数：如果指定的类没有注册，则该函数会调用类处理回调，并再次确认类是否注册，如果确认未注册，再返回nil。不过，每个类定义都必须有一个有效的元类定义，所以这个函数总是会返回一个元类定义，不管它是否有效。



#### 小结

在这一章中我们介绍了`Runtime`运行时中与类和对象相关的数据结构，通过这些数据函数，我们可以管窥Objective-C底层面向对象实现的一些信息。另外，通过丰富的操作函数，可以灵活地对这些数据进行操作。



#### 参考

1. [Objective-C Runtime Reference](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/ObjCRuntimeRef/)
2. [Objective-C Runtime的数据类型](http://www.cnblogs.com/whyandinside/archive/2013/02/26/2933552.html)
3. [详解Objective-C的meta-class](http://blog.csdn.net/windyitian/article/details/19810875)
4. [what are class_setIvarLayout and class_getIvarLayout?](http://stackoverflow.com/questions/16131172/what-are-class-setivarlayout-and-class-getivarlayout)
5. [What’s the difference between doing alloc and class_createInstance](http://stackoverflow.com/questions/3805499/whats-the-difference-between-doing-alloc-and-class-createinstance)









### 二、成员变量与属性

这章开始，我们将讨论类实现细节相关的内容，主要包括类中成员变量，属性，方法，协议与分类的实现。

本章的主要内容将聚集在Runtime对成员变量与属性的处理。在讨论之前，我们先介绍一个重要的概念：类型编码。

#### 类型编码(Type Encoding)

作为对Runtime的补充，编译器将每个方法的返回值和参数类型编码为一个字符串，并将其与方法的`selector`关联在一起。这种编码方案在其它情况下也是非常有用的，因此我们可以使用`@encode`编译器指令来获取它。当给定一个类型时，`@encode`返回这个类型的字符串编码。这些类型可以是诸如`int`、指针这样的基本类型，也可以是结构体、类等类型。事实上，任何可以作为`sizeof()`操作参数的类型都可以用于`@encode()`。

在`Objective-C Runtime Programming Guide`中的[Type Encoding](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html#//apple_ref/doc/uid/TP40008048-CH100-SW1)一节中，列出了Objective-C中所有的类型编码。需要注意的是这些类型很多是与我们用于存档和分发的编码类型是相同的。但有一些不能在存档时使用。

*注：Objective-C不支持long double类型。@encode(long double)返回d，与double是一样的。*

一个数组的类型编码位于方括号中；其中包含数组元素的个数及元素类型。如以下示例：

```objective-c
float a[] = {1.0, 2.0, 3.0};
NSLog(@"array encoding type: %s", @encode(typeof(a)));
```

输出是：

```objective-c
2014-10-28 11:44:54.731 RuntimeTest[942:50791] array encoding type: [3f]
```

其它类型可参考[Type Encoding](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html#//apple_ref/doc/uid/TP40008048-CH100-SW1)，在此不细说。

另外，还有些编码类型，`@encode`虽然不会直接返回它们，但它们可以作为协议中声明的方法的类型限定符。可以参考[Type Encoding](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html#//apple_ref/doc/uid/TP40008048-CH100-SW1)。

对于属性而言，还会有一些特殊的类型编码，以表明属性是只读、拷贝、`retain`等等，详情可以参考[Property Type String](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtPropertyIntrospection.html#//apple_ref/doc/uid/TP40008048-CH101-SW6)。



#### 成员变量、属性

Runtime中关于成员变量和属性的相关数据结构并不多，只有三个，并且都很简单。不过还有个非常实用但可能经常被忽视的特性，即关联对象，我们将在这小节中详细讨论。



**基础数据类型**

**Ivar**

`Ivar`是表示实例变量的类型，其实际是一个指向`objc_ivar`结构体的指针，其定义如下：

```objective-c
typedef struct objc_ivar *Ivar;

struct objc_ivar {

    char *ivar_name               	OBJC2_UNAVAILABLE;	// 变量名
    char *ivar_type             	OBJC2_UNAVAILABLE;	// 变量类型
    int ivar_offset            		OBJC2_UNAVAILABLE;	// 基地址偏移字节

#ifdef __LP64__
    int space                 		OBJC2_UNAVAILABLE;
#endif
}
```

**objc_property_t**

`objc_property_t`是表示Objective-C声明的属性的类型，其实际是指向`objc_property`结构体的指针，其定义如下：

```objective-c
typedef struct objc_property *objc_property_t;
```

**objc_property_attribute_t**

`objc_property_attribute_t`定义了属性的特性(`attribute`)，它是一个结构体，定义如下：

```objective-c
typedef struct {
    const char *name;           // 特性名
    const char *value;          // 特性值
} objc_property_attribute_t;
```



**关联对象(Associated Object)**

关联对象是Runtime中一个非常实用的特性，不过可能很容易被忽视。

关联对象类似于成员变量，不过是在运行时添加的。我们通常会把成员变量(`Ivar`)放在类声明的头文件中，或者放在类实现的`@implementation`后面。但这有一个缺点，我们不能在分类中添加成员变量。如果我们尝试在分类中添加新的成员变量，编译器会报错。

我们可能希望通过使用(甚至是滥用)全局变量来解决这个问题。但这些都不是`Ivar`，因为他们不会连接到一个单独的实例。因此，这种方法很少使用。

Objective-C针对这一问题，提供了一个解决方案：即关联对象(`Associated Object`)。

我们可以把关联对象想象成一个Objective-C对象(如字典)，这个对象通过给定的key连接到类的一个实例上。不过由于使用的是C接口，所以key是一个`void`指针(`const void *`)。我们还需要指定一个内存管理策略，以告诉`Runtime`如何管理这个对象的内存。这个内存管理的策略可以由以下值指定：

```objective-c
OBJC_ASSOCIATION_ASSIGN
OBJC_ASSOCIATION_RETAIN_NONATOMIC
OBJC_ASSOCIATION_COPY_NONATOMIC
OBJC_ASSOCIATION_RETAIN
OBJC_ASSOCIATION_COPY
```

当宿主对象被释放时，会根据指定的内存管理策略来处理关联对象。如果指定的策略是`assign`，则宿主释放时，关联对象不会被释放；而如果指定的是`retain`或者是`copy`，则宿主释放时，关联对象会被释放。我们甚至可以选择是否是自动`retain/copy`。当我们需要在多个线程中处理访问关联对象的多线程代码时，这就非常有用了。

我们将一个对象连接到其它对象所需要做的就是下面两行代码：

```objective-c
static char myKey;
objc_setAssociatedObject(self, &myKey, anObject, OBJC_ASSOCIATION_RETAIN);
```

在这种情况下，`self`对象将获取一个新的关联的对象`anObject`，且内存管理策略是自动`retain`关联对象，当`self`对象释放时，会自动`release`关联对象。另外，如果我们使用同一个key来关联另外一个对象时，也会自动释放之前关联的对象，这种情况下，先前的关联对象会被妥善地处理掉，并且新的对象会使用它的内存。

```objective-c
id anObject = objc_getAssociatedObject(self, &myKey);
```

我们可以使用`objc_removeAssociatedObjects`函数来移除一个关联对象，或者使用`objc_setAssociatedObject`函数将key指定的关联对象设置为nil。

我们下面来用实例演示一下关联对象的使用方法。

假定我们想要动态地将一个Tap手势操作连接到任何`UIView`中，并且根据需要指定点击后的实际操作。这时候我们就可以将一个手势对象及操作的block对象关联到我们的`UIView`对象中。这项任务分两部分。首先，如果需要，我们要创建一个手势识别对象并将它及block做为关联对象。如下代码所示：

```objective-c
- (void)setTapActionWithBlock:(void (^)(void))block
{
	UITapGestureRecognizer *gesture = objc_getAssociatedObject(self, &kDTActionHandlerTapGestureKey);
 
	if (!gesture)
	{
		gesture = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(__handleActionForTapGesture:)];
		[self addGestureRecognizer:gesture];
		objc_setAssociatedObject(self, &kDTActionHandlerTapGestureKey, gesture, OBJC_ASSOCIATION_RETAIN);
	}

	objc_setAssociatedObject(self, &kDTActionHandlerTapBlockKey, block, OBJC_ASSOCIATION_COPY);
}
​```	

这段代码检测了手势识别的关联对象。如果没有，则创建并建立关联关系。同时，将传入的块对象连接到指定的key上。注意`block`对象的关联内存管理策略。

手势识别对象需要一个`target`和`action`，所以接下来我们定义处理方法：


​```objc
- (void)__handleActionForTapGesture:(UITapGestureRecognizer *)gesture
{
	if (gesture.state == UIGestureRecognizerStateRecognized)
	{
		void(^action)(void) = objc_getAssociatedObject(self, &kDTActionHandlerTapBlockKey);

		if (action)
		{
			action();
		}
	}
}
```

我们需要检测手势识别对象的状态，因为我们只需要在点击手势被识别出来时才执行操作。

从上面的例子我们可以看到，关联对象使用起来并不复杂。它让我们可以动态地增强类现有的功能。我们可以在实际编码中灵活地运用这一特性。



**成员变量、属性的操作方法**

**成员变量**

成员变量操作包含以下函数：

```objective-c
// 获取成员变量名
const char * ivar_getName ( Ivar v );

// 获取成员变量类型编码
const char * ivar_getTypeEncoding ( Ivar v );

// 获取成员变量的偏移量
ptrdiff_t ivar_getOffset ( Ivar v );
```

- `ivar_getOffset`函数，对于类型`id`或其它对象类型的实例变量，可以调用`object_getIvar`和`object_setIvar`来直接访问成员变量，而不使用偏移量。

**关联对象**

关联对象操作函数包括以下：

```objective-c
// 设置关联对象
void objc_setAssociatedObject ( id object, const void *key, id value, objc_AssociationPolicy policy );

// 获取关联对象
id objc_getAssociatedObject ( id object, const void *key );

// 移除关联对象
void objc_removeAssociatedObjects ( id object );
```

关联对象及相关实例已经在前面讨论过了，在此不再重复。

**属性**

属性操作相关函数包括以下：

```objective-c
// 获取属性名
const char * property_getName ( objc_property_t property );

// 获取属性特性描述字符串
const char * property_getAttributes ( objc_property_t property );

// 获取属性中指定的特性
char * property_copyAttributeValue ( objc_property_t property, const char *attributeName );

// 获取属性的特性列表
objc_property_attribute_t * property_copyAttributeList ( objc_property_t property, unsigned int *outCount );
```

- `property_copyAttributeValue`函数，返回的`char *`在使用完后需要调用`free()`释放。
- `property_copyAttributeList`函数，返回值在使用完后需要调用`free()`释放。



#### 实例

假定这样一个场景，我们从服务端两个不同的接口获取相同的字典数据，但这两个接口是由两个人写的，相同的信息使用了不同的字段表示。我们在接收到数据时，可将这些数据保存在相同的对象中。对象类如下定义：

```objective-c
@interface MyObject: NSObject

@property (nonatomic, copy) NSString    *   name;                  
@property (nonatomic, copy) NSString    *   status;                 

@end
```

接口A、B返回的字典数据如下所示：

```objective-c
@{@"name1": "张三", @"status1": @"start"}
@{@"name2": "张三", @"status2": @"end"}
```

通常的方法是写两个方法分别做转换，不过如果能灵活地运用Runtime的话，可以只实现一个转换方法，为此，我们需要先定义一个映射字典(全局变量)

```objective-c
static NSMutableDictionary *map = nil;

@implementation MyObject	

+ (void)load
{
    map = [NSMutableDictionary dictionary];
    map[@"name1"]                = @"name";
    map[@"status1"]              = @"status";
    map[@"name2"]                = @"name";
    map[@"status2"]              = @"status";
}

@end
```

上面的代码将两个字典中不同的字段映射到`MyObject`中相同的属性上，这样，转换方法可如下处理：

```objective-c
- (void)setDataWithDic:(NSDictionary *)dic
{
    [dic enumerateKeysAndObjectsUsingBlock:^(NSString *key, id obj, BOOL *stop) {

        NSString *propertyKey = [self propertyForKey:key];
        if (propertyKey)
        {
            objc_property_t property = class_getProperty([self class], [propertyKey UTF8String]);

            // TODO: 针对特殊数据类型做处理
            NSString *attributeString = [NSString stringWithCString:property_getAttributes(property) encoding:NSUTF8StringEncoding];

            ...

            [self setValue:obj forKey:propertyKey];
        }
    }];
}
```

当然，一个属性能否通过上面这种方式来处理的前提是其支持KVC。

#### 小结

本章中我们讨论了Runtime中与成员变量和属性相关的内容。成员变量与属性是类的数据基础，合理地使用Runtime中的相关操作能让我们更加灵活地来处理与类数据相关的工作。

#### 参考

1. [Objective-C Runtime Programming Guide](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html)
2. [Associated Objects](http://www.cocoanetics.com/2012/06/associated-objects/)







### 三、方法与消息

这一章，我们就要开始讨论Runtime中最有意思的一部分：消息处理机制。我们将详细讨论消息的发送及消息的转发。不过在讨论消息之前，我们先来了解一下与方法相关的一些内容。

#### 基础数据类型

**SEL**

SEL又叫选择器，是表示一个方法的`selector`的指针，其定义如下：

```objective-c
typedef struct objc_selector *SEL;
```

`objc_selector`结构体的详细定义没有在 `<objc/runtime.h>` 头文件中找到。方法的`selector`用于表示运行时方法的名字。Objective-C在编译时，会依据每一个方法的名字、参数序列，生成一个唯一的整型标识(`Int`类型的地址)，这个标识就是`SEL`。如下代码所示：

```objective-c
SEL sel1 = @selector(method1);
NSLog(@"sel : %p", sel1);
```

上面的输出为：

```objective-c
2014-10-30 18:40:07.518 RuntimeTest[52734:466626] sel : 0x100002d72
```

两个类之间，不管它们是父类与子类的关系，还是之间没有这种关系，只要方法名相同，那么方法的SEL就是一样的。每一个方法都对应着一个`SEL`。所以在Objective-C同一个类(及类的继承体系)中，不能存在2个同名的方法，即使参数类型不同也不行。相同的方法只能对应一个`SEL`。这也就导致Objective-C在处理相同方法名且参数个数相同但类型不同的方法方面的能力很差。如在某个类中定义以下两个方法：

```objective-c
- (void)setWidth:(int)width;
- (void)setWidth:(double)width;
```

这样的定义被认为是一种编译错误，所以我们不能像C++, C#那样。而是需要像下面这样来声明：

```objective-c
-(void)setWidthIntValue:(int)width;
-(void)setWidthDoubleValue:(double)width;
```

当然，不同的类可以拥有相同的`selector`，这个没有问题。不同类的实例对象执行相同的`selector`时，会在各自的方法列表中去根据`selector`去寻找自己对应的`IMP`。

工程中的所有的`SEL`组成一个`Set`集合，Set的特点就是唯一，因此SEL是唯一的。因此，如果我们想到这个方法集合中查找某个方法时，只需要去找到这个方法对应的SEL就行了，SEL实际上就是根据方法名`hash`化了的一个字符串，而对于字符串的比较仅仅需要比较他们的地址就可以了，可以说速度上无语伦比！！但是，有一个问题，就是数量增多会增大hash冲突而导致的性能下降（或是没有冲突，因为也可能用的是`perfect hash`）。但是不管使用什么样的方法加速，如果能够将总量减少（多个方法可能对应同一个`SEL`），那将是最犀利的方法。那么，我们就不难理解，为什么`SEL`仅仅是函数名了。

本质上，`SEL`只是一个指向方法的指针（准确的说，只是一个根据方法名`hash`化了的`KEY`值，能唯一代表一个方法），它的存在只是为了加快方法的查询速度。这个查找过程我们将在下面讨论。

我们可以在运行时添加新的`selector`，也可以在运行时获取已存在的`selector`，我们可以通过下面三种方法来获取SEL:

1. `sel_registerName`函数
2. Objective-C编译器提供的`@selector()`
3. `NSSelectorFromString()`方法

**IMP**

`IMP`实际上是一个函数指针，指向方法实现的首地址。其定义如下：

```
id (*IMP)(id, SEL, ...)
```

这个函数使用当前`CPU`架构实现的标准的C调用约定。第一个参数是指向`self`的指针(如果是实例方法，则是类实例的内存地址；如果是类方法，则是指向元类的指针（**因为 元类的单例性，但你也不能直接说它是单例，因为它不是你所能创建的，只是表现形式上同单例一样而已,指向元类的指针和这个元类实例类的指针具有等同性，但是不要这么说，这么说直接让人误解了,以为元类可以生成很多实例**）)，第二个参数是方法选择器(`selector`)，接下来是方法的实际参数列表。

前面介绍过的`SEL`就是为了查找方法的最终实现`IMP`的。由于每个方法对应唯一的`SEL`，因此我们可以通过`SEL`方便快速准确地获得它所对应的`IMP`，查找过程将在下面讨论。取得`IMP`后，我们就获得了执行这个方法代码的入口点，此时，我们就可以像调用普通的C语言函数一样来使用这个函数指针了。

通过取得`IMP`，我们可以跳过Runtime的消息传递机制，直接执行`IMP`指向的函数实现，这样省去了Runtime消息传递过程中所做的一系列查找操作，会比直接向对象发送消息高效一些。

**Method**

介绍完`SEL`和`IMP`，我们就可以来讲讲`Method`了。`Method`用于表示类定义中的方法，则定义如下：

```objective-c
typedef struct objc_method *Method;

struct objc_method {
    SEL method_name                	OBJC2_UNAVAILABLE;	// 方法名
    char *method_types                	OBJC2_UNAVAILABLE;
    IMP method_imp             			OBJC2_UNAVAILABLE;	// 方法实现
}
```

我们可以看到该结构体中包含一个`SEL`和`IMP`，实际上相当于在`SEL`和`IMP`之间作了一个映射。有了SEL，我们便可以找到对应的`IMP`，从而调用方法的实现代码。具体操作流程我们将在下面讨论。

**objc_method_description**

`objc_method_description`定义了一个Objective-C方法，其定义如下：

```
struct objc_method_description { SEL name; char *types; };
```





#### 方法相关操作函数

Runtime提供了一系列的方法来处理与方法相关的操作。包括方法本身及SEL。本节我们介绍一下这些函数。

**方法**

方法操作相关函数包括下以：

```objective-c
// 调用指定方法的实现
id method_invoke ( id receiver, Method m, ... );

// 调用返回一个数据结构的方法的实现
void method_invoke_stret ( id receiver, Method m, ... );

// 获取方法名
SEL method_getName ( Method m );

// 返回方法的实现
IMP method_getImplementation ( Method m );

// 获取描述方法参数和返回值类型的字符串
const char * method_getTypeEncoding ( Method m );

// 获取方法的返回值类型的字符串
char * method_copyReturnType ( Method m );

// 获取方法的指定位置参数的类型字符串
char * method_copyArgumentType ( Method m, unsigned int index );

// 通过引用返回方法的返回值类型字符串
void method_getReturnType ( Method m, char *dst, size_t dst_len );

// 返回方法的参数的个数
unsigned int method_getNumberOfArguments ( Method m );

// 通过引用返回方法指定位置参数的类型字符串
void method_getArgumentType ( Method m, unsigned int index, char *dst, size_t dst_len );

// 返回指定方法的方法描述结构体
struct objc_method_description * method_getDescription ( Method m );

// 设置方法的实现
IMP method_setImplementation ( Method m, IMP imp );

// 交换两个方法的实现
void method_exchangeImplementations ( Method m1, Method m2 );
```

- `method_invoke`函数，返回的是实际实现的返回值。参数`receiver`不能为空。这个方法的效率会比`method_getImplementation`和`method_getName`更快。
- `method_getName`函数，返回的是一个`SEL`。如果想获取方法名的C字符串，可以使用`sel_getName(method_getName(method))`。
- `method_getReturnType`函数，类型字符串会被拷贝到`dst`中。
- `method_setImplementation`函数，注意该函数返回值是方法之前的实现。

**方法选择器**

选择器相关的操作函数包括：

```objective-c
// 返回给定选择器指定的方法的名称
const char * sel_getName ( SEL sel );

// 在Objective-C Runtime系统中注册一个方法，将方法名映射到一个选择器，并返回这个选择器
SEL sel_registerName ( const char *str );

// 在Objective-C Runtime系统中注册一个方法
SEL sel_getUid ( const char *str );

// 比较两个选择器
BOOL sel_isEqual ( SEL lhs, SEL rhs );
```

- `sel_registerName`函数：在我们将一个方法添加到类定义时，我们必须在`Objective-C Runtime`系统中注册一个方法名以获取方法的选择器。



#### 方法调用流程

在Objective-C中，消息直到运行时才绑定到方法实现上。编译器会将消息表达式`[receiver message]`转化为一个消息函数的调用，即`objc_msgSend`。这个函数将消息接收者和方法名作为其基础参数，如以下所示：

```objective-c
objc_msgSend(receiver, selector)
```

如果消息中还有其它参数，则该方法的形式如下所示：

```objective-c
objc_msgSend(receiver, selector, arg1, arg2, ...)
```

这个函数完成了动态绑定的所有事情：

1. 首先它找到`selector`对应的方法实现。因为同一个方法可能在不同的类中有不同的实现，所以我们需要依赖于接收者的类来找到的确切的实现。
2. 它调用方法实现，并将接收者对象及方法的所有参数传给它。
3. 最后，它将实现返回的值作为它自己的返回值。

消息的关键在于我们前面章节讨论过的结构体`objc_class`，这个结构体有两个字段是我们在分发消息的关注的：

1. 指向父类的指针
2. 一个类的方法分发表，即`methodLists`。

当我们创建一个新对象时，先为其分配内存，并初始化其成员变量。其中isa指针也会被初始化，让对象可以访问类及类的继承体系。

下图演示了这样一个消息的基本框架：

![](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Art/messaging1.gif) 

当消息发送给一个对象时，`objc_msgSend`通过对象的`isa`指针获取到类的结构体，然后在方法分发表里面查找方法的`selector`。如果没有找到`selector`，则通过`objc_msgSend`结构体中的指向父类的指针找到其父类，并在父类的分发表里面查找方法的`selector`。依此，会一直沿着类的继承体系到达NSObject类。一旦定位到`selector`，函数会就获取到了实现的入口点，并传入相应的参数来执行方法的具体实现。如果最后没有定位到`selector`，则会走消息转发流程，这个我们在后面讨论。

为了加速消息的处理，运行时系统缓存使用过的`selector`及对应的方法的地址。这点我们在[前面](http://southpeak.github.io/blog/2014/10/25/objective-c-runtime-yun-xing-shi-zhi-lei-yu-dui-xiang/)讨论过，不再重复。

**隐藏参数**

`objc_msgSend`有两个隐藏参数：

1. 消息接收对象
2. 方法的selector

这两个参数为方法的实现提供了调用者的信息。之所以说是隐藏的，是因为它们在定义方法的源代码中没有声明。它们是在编译期被插入实现代码的。

虽然这些参数没有显示声明，但在代码中仍然可以引用它们。我们可以使用`self`来引用接收者对象，使用`_cmd`来引用选择器。如下代码所示：

```objective-c
- strange
{
    id  target = getTheReceiver();
    SEL method = getTheMethod();

    if ( target == self || method == _cmd )
        return nil;

    return [target performSelector:method];
}
```

当然，这两个参数我们用得比较多的是`self`，`_cmd`在实际中用得比较少。

**获取方法地址**

Runtime中方法的动态绑定让我们写代码时更具灵活性，如我们可以把消息转发给我们想要的对象，或者随意交换一个方法的实现等。不过灵活性的提升也带来了性能上的一些损耗。毕竟我们需要去查找方法的实现，而不像函数调用来得那么直接。当然，方法的缓存一定程度上解决了这一问题。

我们上面提到过，如果想要避开这种动态绑定方式，我们可以获取方法实现的地址，然后像调用函数一样来直接调用它。特别是当我们需要在一个循环内频繁地调用一个特定的方法时，通过这种方式可以提高程序的性能。

NSObject类提供了`methodForSelector:`方法，让我们可以获取到方法的指针，然后通过这个指针来调用实现代码。我们需要将`methodForSelector:`返回的指针转换为合适的函数类型，函数参数和返回值都需要匹配上。

我们通过以下代码来看看`methodForSelector:`的使用：

```objective-c
void (*setter)(id, SEL, BOOL);
int i;

setter = (void (*)(id, SEL, BOOL))[target methodForSelector:@selector(setFilled:)];

for (i = 0 ; i < 1000 ; i++)
    setter(targetList[i], @selector(setFilled:), YES);
```

这里需要注意的就是函数指针的前两个参数必须是`id`和`SEL`。

当然这种方式只适合于在类似于`for`循环这种情况下频繁调用同一方法，以提高性能的情况。另外，`methodForSelector:`是由Cocoa运行时提供的；它不是Objective-C语言的特性。



#### 消息转发 

当一个对象能接收一个消息时，就会走正常的方法调用流程。但如果一个对象无法接收指定消息时，又会发生什么事呢？默认情况下，如果是以`[object message]`的方式调用方法，如果`object`无法响应`message`消息时，编译器会报错。但如果是以`perform...`的形式来调用，则需要等到运行时才能确定object是否能接收`message`消息。如果不能，则程序崩溃。

通常，当我们不能确定一个对象是否能接收某个消息时，会先调用`respondsToSelector:`来判断一下。如下代码所示：

```objective-c
if ([self respondsToSelector:@selector(method)]) {
    [self performSelector:@selector(method)];
}
```

不过，我们这边想讨论下不使用`respondsToSelector:`判断的情况。这才是我们这一节的重点。

当一个对象无法接收某一消息时，就会启动所谓”**消息转发(message forwarding)**“机制，通过这一机制，我们可以告诉对象如何处理未知的消息。默认情况下，对象接收到未知的消息，会导致程序崩溃，通过控制台，我们可以看到以下异常信息：

```objective-c
-[SUTRuntimeMethod method]: unrecognized selector sent to instance 0x100111940

*** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[SUTRuntimeMethod method]: unrecognized selector sent to instance 0x100111940'
```

这段异常信息实际上是由NSObject的”`doesNotRecognizeSelector`“方法抛出的。不过，我们可以采取一些措施，让我们的程序执行特定的逻辑，而避免程序的崩溃。

消息转发机制基本上分为三个步骤：

1. 动态方法解析
2. 备用接收者
3. 完整转发

下面我们详细讨论一下这三个步骤。



![](https://raw.githubusercontent.com/WiInputMethod/interview/master/img/ios-runtime-method-resolve.png) 



动态方法解析**

对象在接收到未知的消息时，首先会调用所属类的类方法`+resolveInstanceMethod:`(实例方法)或者`+resolveClassMethod:`(类方法)。在这个方法中，我们有机会为该未知消息新增一个”处理方法””。不过使用该方法的前提是我们已经实现了该”处理方法”，只需要在运行时通过`class_addMethod`函数动态添加到类里面就可以了。如下代码所示：

```objective-c
void functionForMethod1(id self, SEL _cmd) {
   NSLog(@"%@, %p", self, _cmd);
}
	

+ (BOOL)resolveInstanceMethod:(SEL)sel {

    NSString *selectorString = NSStringFromSelector(sel);
    if ([selectorString isEqualToString:@"method1"]) {
        class_addMethod(self.class, @selector(method1), (IMP)functionForMethod1, "@:");
    }

    return [super resolveInstanceMethod:sel];
}
```

不过这种方案更多的是为了实现`@dynamic`属性。



**备用接收者**

如果在上一步无法处理消息，则Runtime会继续调以下方法：

```objective-c
- (id)forwardingTargetForSelector:(SEL)aSelector
```

如果一个对象实现了这个方法，并返回一个非nil的结果，则这个对象会作为消息的新接收者，且消息会被分发到这个对象。当然这个对象不能是`self`自身，否则就是出现无限循环。当然，如果我们没有指定相应的对象来处理`aSelector`，则应该调用父类的实现来返回结果。

使用这个方法通常是在对象内部，可能还有一系列其它对象能处理该消息，我们便可借这些对象来处理消息并返回，这样在对象外部看来，还是由该对象亲自处理了这一消息。如下代码所示：

```objective-c
@interface SUTRuntimeMethodHelper : NSObject

- (void)method2;

@end

@implementation SUTRuntimeMethodHelper

- (void)method2 {
    NSLog(@"%@, %p", self, _cmd);
}

@end

#pragma mark -

@interface SUTRuntimeMethod () {
    SUTRuntimeMethodHelper *_helper;
}

@end

@implementation SUTRuntimeMethod

+ (instancetype)object {
    return [[self alloc] init];
}

- (instancetype)init {

    self = [super init];
    if (self != nil) {
        _helper = [[SUTRuntimeMethodHelper alloc] init];
    }
    return self;
}

- (void)test {
    [self performSelector:@selector(method2)];
}

- (id)forwardingTargetForSelector:(SEL)aSelector {

    NSLog(@"forwardingTargetForSelector");

    NSString *selectorString = NSStringFromSelector(aSelector);

    // 将消息转发给_helper来处理
    if ([selectorString isEqualToString:@"method2"]) {
        return _helper;
    }
    return [super forwardingTargetForSelector:aSelector];
}

@end
```

这一步合适于我们只想将消息转发到另一个能处理该消息的对象上。但这一步无法对消息进行处理，如操作消息的参数和返回值。



**完整消息转发**

如果在上一步还不能处理未知消息，则唯一能做的就是启用完整的消息转发机制了。此时会调用以下方法：

```
- (void)forwardInvocation:(NSInvocation *)anInvocation
```

运行时系统会在这一步给消息接收者最后一次机会将消息转发给其它对象。对象会创建一个表示消息的`NSInvocation`对象，把与尚未处理的消息有关的全部细节都封装在`anInvocation`中，包括`selector`，目标(`target`)和参数。我们可以在`forwardInvocation`方法中选择将消息转发给其它对象。

`forwardInvocation:`方法的实现有两个任务：

1. 定位可以响应封装在`anInvocation`中的消息的对象。这个对象不需要能处理所有未知消息。
2. 使用`anInvocation`作为参数，将消息发送到选中的对象。`anInvocation`将会保留调用结果，运行时系统会提取这一结果并将其发送到消息的原始发送者。

不过，在这个方法中我们可以实现一些更复杂的功能，我们可以对消息的内容进行修改，比如追回一个参数等，然后再去触发消息。另外，若发现某个消息不应由本类处理，则应调用父类的同名方法，以便继承体系中的每个类都有机会处理此调用请求。

还有一个很重要的问题，我们必须重写以下方法：

```objective-c
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector
```

消息转发机制使用从这个方法中获取的信息来创建`NSInvocation`对象。因此我们必须重写这个方法，为给定的`selector`提供一个合适的方法签名。

完整的示例如下所示：

```objective-c
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {

    NSMethodSignature *signature = [super methodSignatureForSelector:aSelector];
    if (!signature) {
        if ([SUTRuntimeMethodHelper instancesRespondToSelector:aSelector]) {
            signature = [SUTRuntimeMethodHelper instanceMethodSignatureForSelector:aSelector];
        }
    }

    return signature;
}

- (void)forwardInvocation:(NSInvocation *)anInvocation {
    if ([SUTRuntimeMethodHelper instancesRespondToSelector:anInvocation.selector]) {
        [anInvocation invokeWithTarget:_helper];
    }
}
```

NSObject的`forwardInvocation:`方法实现只是简单调用了`doesNotRecognizeSelector:`方法，它不会转发任何消息。这样，如果不在以上所述的三个步骤中处理未知消息，则会引发一个异常。

从某种意义上来讲，`forwardInvocation:`就像一个未知消息的分发中心，将这些未知的消息转发给其它对象。或者也可以像一个运输站一样将所有未知消息都发送给同一个接收对象。这取决于具体的实现。



**消息转发与多重继承**

回过头来看第二和第三步，通过这两个方法我们可以允许一个对象与其它对象建立关系，以处理某些未知消息，而表面上看仍然是该对象在处理消息。通过这种关系，我们可以模拟“多重继承”的某些特性，让对象可以“继承”其它对象的特性来处理一些事情。不过，这两者间有一个重要的区别：多重继承将不同的功能集成到一个对象中，它会让对象变得过大，涉及的东西过多；而消息转发将功能分解到独立的小的对象中，并通过某种方式将这些对象连接起来，并做相应的消息转发。

不过消息转发虽然类似于继承，但NSObject的一些方法还是能区分两者。如`respondsToSelector:`和`isKindOfClass:`只能用于继承体系，而不能用于转发链。便如果我们想让这种消息转发看起来像是继承，则可以重写这些方法，如以下代码所示：

```objective-c
- (BOOL)respondsToSelector:(SEL)aSelector
{
	if ( [super respondsToSelector:aSelector])
		return YES;
	else {
		/* Here, test whether the aSelector message can     *
		 * be forwarded to another object and whether that  *
		 * object can respond to it. Return YES if it can.  */
	}

	return NO; 	
}
```



#### 小结

在此，我们已经了解了Runtime中消息发送和转发的基本机制。这也是Runtime的强大之处，通过它，我们可以为程序增加很多动态的行为，虽然我们在实际开发中很少直接使用这些机制(如直接调用`objc_msgSend`)，但了解它们有助于我们更多地去了解底层的实现。其实在实际的编码过程中，我们也可以灵活地使用这些机制，去实现一些特殊的功能，如`hook`操作等。



#### 参考

1. [Objective-C Runtime Reference](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/ObjCRuntimeRef)
2. [Objective-C Runtime Programming Guide](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html)
3. [Objective-C runtime之消息（二）](http://www.xcoder.cn/html/2013/objc_0304/1373.html)
4. [深入浅出Cocoa之消息](http://www.cnblogs.com/kesalin/archive/2011/08/15/objc_method_base.html)





### 四、Method Swizzling

理解`Method Swizzling`是学习`runtime`机制的一个很好的机会。在此不多做整理，仅翻译由Mattt Thompson发表于`nshipster`的[Method Swizzling](http://nshipster.com/method-swizzling/)一文。

`Method Swizzling`是改变一个`selector`的实际实现的技术。通过这一技术，我们可以在运行时通过修改类的分发表中`selector`对应的函数，来修改方法的实现。

例如，我们想跟踪在程序中每一个view controller展示给用户的次数：当然，我们可以在每个view controller的viewDidAppear中添加跟踪代码；但是这太过麻烦，需要在每个view controller中写重复的代码。创建一个子类可能是一种实现方式，但需要同时创建UIViewController, UITableViewController, UINavigationController及其它UIKit中view controller的子类，这同样会产生许多重复的代码。

这种情况下，我们就可以使用`Method Swizzling`，如在代码所示：

```objective-c
#import <objc/runtime.h>

@implementation UIViewController (Tracking)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class class = [self class];         
        // When swizzling a class method, use the following:
        // Class class = object_getClass((id)self);
        SEL originalSelector = @selector(viewWillAppear:);
        SEL swizzledSelector = @selector(xxx_viewWillAppear:);

        Method originalMethod = class_getInstanceMethod(class, originalSelector);
        Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);

        BOOL didAddMethod = class_addMethod(class,
                originalSelector,
                method_getImplementation(swizzledMethod),
                method_getTypeEncoding(swizzledMethod));

        if (didAddMethod) {
            class_replaceMethod(class,
                swizzledSelector,
                method_getImplementation(originalMethod),
                method_getTypeEncoding(originalMethod));
        } else {
            method_exchangeImplementations(originalMethod, swizzledMethod);
        }
    });
}

#pragma mark - Method Swizzling

- (void)xxx_viewWillAppear:(BOOL)animated {
    [self xxx_viewWillAppear:animated];
    NSLog(@"viewWillAppear: %@", self);
}

@end
```

在这里，我们通过`method swizzling`修改了UIViewController的`@selector(viewWillAppear:)`对应的函数指针，使其实现指向了我们自定义的`xxx_viewWillAppear`的实现。这样，当UIViewController及其子类的对象调用`viewWillAppear`时，都会打印一条日志信息。

上面的例子很好地展示了使用`method swizzling`来一个类中注入一些我们新的操作。当然，还有许多场景可以使用`method swizzling`，在此不多举例。在此我们说说使用`method swizzling`需要注意的一些问题：

#### Swizzling应该总是在+load中执行

在Objective-C中，运行时会自动调用每个类的两个方法。`+load`会在类初始加载时调用，`+initialize`会在第一次调用类的类方法或实例方法之前被调用。这两个方法是可选的，且只有在实现了它们时才会被调用。由于`method swizzling`会影响到类的全局状态，因此要尽量避免在并发处理中出现竞争的情况。`+load`能保证在类的初始化过程中被加载，并保证这种改变应用级别的行为的一致性。相比之下，`+initialize`在其执行时不提供这种保证–事实上，如果在应用中没为给这个类发送消息，则它可能永远不会被调用。



#### Swizzling应该总是在dispatch_once中执行

与上面相同，因为`swizzling`会改变全局状态，所以我们需要在运行时采取一些预防措施。原子性就是这样一种措施，它确保代码只被执行一次，不管有多少个线程。GCD的`dispatch_once`可以确保这种行为，我们应该将其作为`method swizzling`的最佳实践。



#### 选择器、方法与实现

在Objective-C中，选择器(`selector`)、方法(`method`)和实现(`implementation`)是运行时中一个特殊点，虽然在一般情况下，这些术语更多的是用在消息发送的过程描述中。

以下是`Objective-C Runtime Reference`中的对这几个术语一些描述：

1. `Selector(typedef struct objc_selector *SEL)`：用于在运行时中表示一个方法的名称。一个方法选择器是一个C字符串，它是在Objective-C运行时被注册的。选择器由编译器生成，并且在类被加载时由运行时自动做映射操作。
2. `Method(typedef struct objc_method *Method)`：在类定义中表示方法的类型
3. `Implementation(typedef id (*IMP)(id, SEL, ...))`：这是一个指针类型，指向方法实现函数的开始位置。这个函数使用为当前CPU架构实现的标准C调用规范。每一个参数是指向对象自身的指针(self)，第二个参数是方法选择器。然后是方法的实际参数。

理解这几个术语之间的关系最好的方式是：一个类维护一个运行时可接收的消息分发表；分发表中的每个入口是一个方法(Method)，其中key是一个特定名称，即选择器(SEL)，其对应一个实现(IMP)，即指向底层C函数的指针。

为了swizzle一个方法，我们可以在分发表中将一个方法的现有的选择器映射到不同的实现，而将该选择器对应的原始实现关联到一个新的选择器中。



#### 调用_cmd

我们回过头来看看前面新的方法的实现代码：

```objective-c
- (void)xxx_viewWillAppear:(BOOL)animated {
    [self xxx_viewWillAppear:animated];
    NSLog(@"viewWillAppear: %@", NSStringFromClass([self class]));
}
```

咋看上去是会导致无限循环的。但令人惊奇的是，并没有出现这种情况。在swizzling的过程中，方法中的`[self xxx_viewWillAppear:animated]`已经被重新指定到UIViewController类的-viewWillAppear:中。在这种情况下，不会产生无限循环。不过如果我们调用的是`[self viewWillAppear:animated]`，则会产生无限循环，因为这个方法的实现在运行时已经被重新指定为`xxx_viewWillAppear:`了。



#### 注意事项

Swizzling通常被称作是一种黑魔法，容易产生不可预知的行为和无法预见的后果。虽然它不是最安全的，但如果遵从以下几点预防措施的话，还是比较安全的：

1. 总是调用方法的原始实现(除非有更好的理由不这么做)：API提供了一个输入与输出约定，但其内部实现是一个黑盒。Swizzle一个方法而不调用原始实现可能会打破私有状态底层操作，从而影响到程序的其它部分。
2. 避免冲突：给自定义的分类方法加前缀，从而使其与所依赖的代码库不会存在命名冲突。
3. 明白是怎么回事：简单地拷贝粘贴swizzle代码而不理解它是如何工作的，不仅危险，而且会浪费学习Objective-C运行时的机会。阅读`Objective-C Runtime Reference`和查看`<objc/runtime.h>`头文件以了解事件是如何发生的。
4. 小心操作：无论我们对Foundation, UIKit或其它内建框架执行Swizzle操作抱有多大信心，需要知道在下一版本中许多事可能会不一样。







### 五、协议与分类

Objective-C中的分类允许我们通过给一个类添加方法来扩充它（但是通过`category`不能添加新的实例变量），并且我们不需要访问类中的代码就可以做到。

Objective-C中的协议是普遍存在的接口定义方式，即在一个类中通过`@protocol`定义接口，在另外类中实现接口，这种接口定义方式也成为“`delegation`”模式，`@protocol`声明了可以被其他任何方法类实现的方法，协议仅仅是定义一个接口，而由其他的类去负责实现。

在本章中，我们来看看runtime对分类与协议的支持。



#### 基础数据类型

**Category**

Category是表示一个指向分类的结构体的指针，其定义如下：

```objective-c
typedef struct objc_category *Category;

struct objc_category {
    char *category_name                          OBJC2_UNAVAILABLE;	// 分类名
    char *class_name                             OBJC2_UNAVAILABLE;	// 分类所属的类名
    struct objc_method_list *instance_methods    OBJC2_UNAVAILABLE;	// 实例方法列表
    struct objc_method_list *class_methods       OBJC2_UNAVAILABLE;	// 类方法列表
    struct objc_protocol_list *protocols         OBJC2_UNAVAILABLE;	// 分类所实现的协议列表
}
```

这个结构体主要包含了分类定义的实例方法与类方法，其中`instance_methods`列表是`objc_class`中方法列表的一个子集，而`class_methods`列表是元类方法列表的一个子集。

**Protocol**

Protocol的定义如下：

```objective-c
typedef struct objc_object Protocol;
```

我们可以看到，`Protocol`其中实就是一个对象结构体。

**操作函数**

Runtime并没有在 `<objc/runtime.h>`  头文件中提供针对分类的操作函数。因为这些分类中的信息都包含在`objc_class`中，我们可以通过针对`objc_class`的操作函数来获取分类的信息。如下例所示：

```objective-c
@interface RuntimeCategoryClass : NSObject

- (void)method1;

@end

@interface RuntimeCategoryClass (Category)

- (void)method2;

@end

@implementation RuntimeCategoryClass

- (void)method1 {

}

@end

@implementation RuntimeCategoryClass (Category)

- (void)method2 {

}
@end

#pragma mark -
NSLog(@"测试objc_class中的方法列表是否包含分类中的方法");

unsigned int outCount = 0;
Method *methodList = class_copyMethodList(RuntimeCategoryClass.class, &outCount);

for (int i = 0; i < outCount; i++) {
    Method method = methodList[i];

    const char *name = sel_getName(method_getName(method));

    NSLog(@"RuntimeCategoryClass's method: %s", name);

    if (strcmp(name, sel_getName(@selector(method2)))) {
        NSLog(@"分类方法method2在objc_class的方法列表中");
    }
}
```

其输出是：

```objective-c
2014-11-08 10:36:39.213 [561:151847] 测试objc_class中的方法列表是否包含分类中的方法
2014-11-08 10:36:39.215 [561:151847] RuntimeCategoryClass's method: method2
2014-11-08 10:36:39.215 [561:151847] RuntimeCategoryClass's method: method1
2014-11-08 10:36:39.215 [561:151847] 分类方法method2在objc_class的方法列表中
```

而对于Protocol，runtime提供了一系列函数来对其进行操作，这些函数包括：

```objective-c
// 返回指定的协议
Protocol * objc_getProtocol ( const char *name );

// 获取运行时所知道的所有协议的数组
Protocol ** objc_copyProtocolList ( unsigned int *outCount );

// 创建新的协议实例
Protocol * objc_allocateProtocol ( const char *name );

// 在运行时中注册新创建的协议
void objc_registerProtocol ( Protocol *proto );

// 为协议添加方法
void protocol_addMethodDescription ( Protocol *proto, SEL name, const char *types, BOOL isRequiredMethod, BOOL isInstanceMethod );

// 添加一个已注册的协议到协议中
void protocol_addProtocol ( Protocol *proto, Protocol *addition );

// 为协议添加属性
void protocol_addProperty ( Protocol *proto, const char *name, const objc_property_attribute_t *attributes, unsigned int attributeCount, BOOL isRequiredProperty, BOOL isInstanceProperty );

// 返回协议名
const char * protocol_getName ( Protocol *p );

// 测试两个协议是否相等
BOOL protocol_isEqual ( Protocol *proto, Protocol *other );

// 获取协议中指定条件的方法的方法描述数组
struct objc_method_description * protocol_copyMethodDescriptionList ( Protocol *p, BOOL isRequiredMethod, BOOL isInstanceMethod, unsigned int *outCount );

// 获取协议中指定方法的方法描述
struct objc_method_description protocol_getMethodDescription ( Protocol *p, SEL aSel, BOOL isRequiredMethod, BOOL isInstanceMethod );

// 获取协议中的属性列表
objc_property_t * protocol_copyPropertyList ( Protocol *proto, unsigned int *outCount );

// 获取协议的指定属性
objc_property_t protocol_getProperty ( Protocol *proto, const char *name, BOOL isRequiredProperty, BOOL isInstanceProperty );

// 获取协议采用的协议
Protocol ** protocol_copyProtocolList ( Protocol *proto, unsigned int *outCount );

// 查看协议是否采用了另一个协议
BOOL protocol_conformsToProtocol ( Protocol *proto, Protocol *other );
```

- `objc_getProtocol`函数，需要注意的是如果仅仅是声明了一个协议，而未在任何类中实现这个协议，则该函数返回的是nil。
- `objc_copyProtocolList`函数，获取到的数组需要使用free来释放
- `objc_allocateProtocol`函数，如果同名的协议已经存在，则返回nil
- `objc_registerProtocol`函数，创建一个新的协议后，必须调用该函数以在运行时中注册新的协议。协议注册后便可以使用，但不能再做修改，即注册完后不能再向协议添加方法或协议

需要强调的是，协议一旦注册后就不可再修改，即无法再通过调用`protocol_addMethodDescription`、`protocol_addProtocol`和`protocol_addProperty`往协议中添加方法等。



#### 小结

Runtime并没有提供过多的函数来处理分类。对于协议，我们可以动态地创建协议，并向其添加方法、属性及继承的协议，并在运行时动态地获取这些信息。



#### 参考

1. [Objective-C Runtime Reference](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/ObjCRuntimeRef/)











### 六、拾遗

面几篇基本介绍了runtime中的大部分功能，包括对类与对象、成员变量与属性、方法与消息、分类与协议的处理。runtime大部分的功能都是围绕这几点来实现的。

本章的内容并不算重点，主要针对前文中对`Objective-C Runtime Reference`内容遗漏的地方做些补充。当然这并不能包含所有的内容。runtime还有许多内容，需要读者去研究发现。



#### super

在Objective-C中，如果我们需要在类的方法中调用父类的方法时，通常都会用到`super`，如下所示：

```objective-c
@interface MyViewController: UIViewController

@end

@implementation	MyViewController

- (void)viewDidLoad {

	[super viewDidLoad];

	// do something
	...
}
@end
```

如何使用`super`我们都知道。现在的问题是，它是如何工作的呢？

首先我们需要知道的是`super`与`self`不同。`self`是类的一个隐藏参数，每个方法的实现的第一个参数即为`self`。而`super`并不是隐藏参数，它实际上只是一个”编译器标示符”，它负责告诉编译器，当调用viewDidLoad方法时，去调用父类的方法，而不是本类中的方法。而它实际上与`self`指向的是相同的消息接收者。为了理解这一点，我们先来看看`super`的定义：

```objective-c
struct objc_super { id receiver; Class superClass; };
```

这个结构体有两个成员：

1. receiver：即消息的实际接收者
2. superClass：指针当前类的父类

当我们使用`super`来接收消息时，编译器会生成一个`objc_super`结构体。就上面的例子而言，这个结构体的`receiver`就是MyViewController对象，与`self`相同；`superClass`指向MyViewController的父类UIViewController。

接下来，发送消息时，不是调用`objc_msgSend`函数，而是调用`objc_msgSendSuper`函数，其声明如下：

```objective-c
id objc_msgSendSuper ( struct objc_super *super, SEL op, ... );
```

该函数第一个参数即为前面生成的`objc_super`结构体，第二个参数是方法的`selector`。该函数实际的操作是：从`objc_super`结构体指向的`superClass`的方法列表开始查找viewDidLoad的`selector`，找到后以`objc->receiver`去调用这个`selector`，而此时的操作流程就是如下方式了

```objective-c
objc_msgSend(objc_super->receiver, @selector(viewDidLoad))
```

由于`objc_super->receiver`就是`self`本身，所以该方法实际与下面这个调用是相同的：

```objective-c
objc_msgSend(self, @selector(viewDidLoad))
```

为了便于理解，我们看以下实例：

```objective-c
@interface MyClass : NSObject

@end

@implementation MyClass

- (void)test {
    NSLog(@"self class: %@", self.class);
    NSLog(@"super class: %@", super.class);
}

@end
```

调用MyClass的test方法后，其输出是：

```objective-c
2014-11-08 15:55:03.256 [824:209297] self class: MyClass
2014-11-08 15:55:03.256 [824:209297] super class: MyClass
```

从上例中可以看到，两者的输出都是MyClass。大家可以自行用上面介绍的内容来梳理一下。



#### 库相关操作

库相关的操作主要是用于获取由系统提供的库相关的信息，主要包含以下函数：

```objective-c
// 获取所有加载的Objective-C框架和动态库的名称
const char ** objc_copyImageNames ( unsigned int *outCount );

// 获取指定类所在动态库
const char * class_getImageName ( Class cls );

// 获取指定库或框架中所有类的类名
const char ** objc_copyClassNamesForImage ( const char *image, unsigned int *outCount );
```

通过这几个函数，我们可以了解到某个类所有的库，以及某个库中包含哪些类。如下代码所示：

```objective-c
NSLog(@"获取指定类所在动态库");

NSLog(@"UIView's Framework: %s", class_getImageName(NSClassFromString(@"UIView")));

NSLog(@"获取指定库或框架中所有类的类名");
const char ** classes = objc_copyClassNamesForImage(class_getImageName(NSClassFromString(@"UIView")), &outCount);
for (int i = 0; i < outCount; i++) {
    NSLog(@"class name: %s", classes[i]);
}
```

其输出结果如下：

```objective-c
2014-11-08 12:57:32.689 [747:184013] 获取指定类所在动态库
2014-11-08 12:57:32.690 [747:184013] UIView's Framework: /System/Library/Frameworks/UIKit.framework/UIKit
2014-11-08 12:57:32.690 [747:184013] 获取指定库或框架中所有类的类名
2014-11-08 12:57:32.691 [747:184013] class name: UIKeyboardPredictiveSettings
2014-11-08 12:57:32.691 [747:184013] class name: _UIPickerViewTopFrame
2014-11-08 12:57:32.691 [747:184013] class name: _UIOnePartImageView
2014-11-08 12:57:32.692 [747:184013] class name: _UIPickerViewSelectionBar
2014-11-08 12:57:32.692 [747:184013] class name: _UIPickerWheelView
2014-11-08 12:57:32.692 [747:184013] class name: _UIPickerViewTestParameters
......
```



#### 块操作

我们都知道block给我们带到极大的方便，苹果也不断提供一些使用block的新的API。同时，苹果在runtime中也提供了一些函数来支持针对block的操作，这些函数包括：

```objective-c
// 创建一个指针函数的指针，该函数调用时会调用特定的block
IMP imp_implementationWithBlock ( id block );

// 返回与IMP(使用imp_implementationWithBlock创建的)相关的block
id imp_getBlock ( IMP anImp );

// 解除block与IMP(使用imp_implementationWithBlock创建的)的关联关系，并释放block的拷贝
BOOL imp_removeBlock ( IMP anImp );
```

- `imp_implementationWithBlock`函数：参数block的签名必须是`method_return_type ^(id self, method_args …)`形式的。该方法能让我们使用block作为`IMP`。如下代码所示：

```objective-c
@interface MyRuntimeBlock : NSObject

@end	

@implementation MyRuntimeBlock

@end

// 测试代码
IMP imp = imp_implementationWithBlock(^(id obj, NSString *str) {
    NSLog(@"%@", str);
});

class_addMethod(MyRuntimeBlock.class, @selector(testBlock:), imp, "v@:@");

MyRuntimeBlock *runtime = [[MyRuntimeBlock alloc] init];
[runtime performSelector:@selector(testBlock:) withObject:@"hello world!"];
```

输出结果是

```objective-c
2014-11-09 14:03:19.779 [1172:395446] hello world!
```



#### 弱引用操作

```objective-c
// 加载弱引用指针引用的对象并返回
id objc_loadWeak ( id *location );

// 存储__weak变量的新值
id objc_storeWeak ( id *location, id obj );
```

- `objc_loadWeak`函数：该函数加载一个弱指针引用的对象，并在对其做`retain`和`autoreleasing`操作后返回它。这样，对象就可以在调用者使用它时保持足够长的生命周期。该函数典型的用法是在任何有使用`__weak`变量的表达式中使用。

● `objc_storeWeak`函数：该函数的典型用法是用于`__weak`变量做为赋值对象时。

这两个函数的具体实施在此不举例，有兴趣的小伙伴可以参考`《Objective-C高级编程：iOS与OS X多线程和内存管理》`中对`__weak`实现的介绍。



#### 宏定义

在runtime中，还定义了一些宏定义供我们使用，有些值我们会经常用到，如表示BOOL值的YES/NO；而有些值不常用，如`OBJC_ROOT_CLASS`。在此我们做一个简单的介绍。

**布尔值**

```objective-c
#define YES  (BOOL)1
#define NO   (BOOL)0
```

这两个宏定义定义了表示布尔值的常量，需要注意的是YES的值是1，而不是非0值。

**空值**

```objective-c
#define nil  __DARWIN_NULL
#define Nil  __DARWIN_NULL
```

其中nil用于空的实例对象，而Nil用于空类对象。

**分发函数原型**

```objective-c
#define OBJC_OLD_DISPATCH_PROTOTYPES  1
```

该宏指明分发函数是否必须转换为合适的函数指针类型。当值为0时，必须进行转换

**Objective-C根类**

```objective-c
#define OBJC_ROOT_CLASS
```

如果我们定义了一个Objective-C根类，则编译器会报错，指明我们定义的类没有指定一个基类。这种情况下，我们就可以使用这个宏定义来避过这个编译错误。该宏在iOS 7.0后可用。

其实在NSObject的声明中，我们就可以看到这个宏的身影，如下所示：

```objective-c
__OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0)

OBJC_ROOT_CLASS
OBJC_EXPORT

@interface NSObject <NSObject> {
    Class isa  OBJC_ISA_AVAILABILITY;
}
```

我们可以参考这种方式来定义我们自己的根类。

**局部变量存储时长**

```objective-c
#define NS_VALID_UNTIL_END_OF_SCOPE
```

该宏表明存储在某些局部变量中的值在优化时不应该被编译器强制释放。

我们将局部变量标记为id类型或者是指向`ObjC`对象类型的指针，以便存储在这些局部变量中的值在优化时不会被编译器强制释放。相反，这些值会在变量再次被赋值之前或者局部变量的作用域结束之前都会被保存。

**关联对象行为**

```objective-c
enum {
   OBJC_ASSOCIATION_ASSIGN  = 0,
   OBJC_ASSOCIATION_RETAIN_NONATOMIC  = 1,
   OBJC_ASSOCIATION_COPY_NONATOMIC  = 3,
   OBJC_ASSOCIATION_RETAIN  = 01401,
   OBJC_ASSOCIATION_COPY  = 01403
};
```

这几个值在前面已介绍过，在此不再重复。



#### 总结

至此，本系列对runtime的整理已完结。当然这只是对runtime的一些基础知识的归纳，力图起个抛砖引玉的作用。还有许多关于runtime有意思东西还需要读者自己去探索发现。



#### 参考

1. [Objective-C Runtime Reference](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/ObjCRuntimeRef/)
2. [iOS:Objective-C中Self和Super详解](http://blog.csdn.net/itianyi/article/details/8678452)
3. [Objective-C的动态特性](http://www.cocoachina.com/industry/20130819/6824.html)







