---
title: iOS弱引用知识总结
---

#### 一、存在的问题

>iOS开发中，页面间的数据交互是非常常见的，其中delegate是比较常用的。因为delegate是一种一对一的数据相应关系，有些时候，单例对象的delegate对象有多个的时候，需要在单例中使用数组来管理，这样导致了，delegate对象被单例的数组强引用，单例不释放，delegate对象也就释放不了。



#### 二、如何解决

**1、手动维护delegate对象的生命周期**：

优点：原始代码，流程自己控制。

缺点：自己开发过程中需要认真的检查delegate对象的生命周期，一旦有误，对象便释放不了。

```objective-c
// HDDelegateTaget.h
@class HDDelegateTaget;

@protocol HDDelegate <NSObject>
- (void)hdDelegateTaget:(HDDelegateTaget *)taget;
@end

@interface HDDelegateTaget : NSObject
@property (nonatomic, strong) NSMutableArray <id <HDDelegate>> *delegates;
+ (instancetype)sharedInstance;
- (void)doSomething;
@end

// HDDelegateTaget.m
@implementation HDDelegateTaget
- (void)doSomething {
    if (_delegates2.count > 0) {
        [_delegates2 enumerateObjectsUsingBlock:^(id<HDDelegate>  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
            if (obj && [obj respondsToSelector:@selector(hdDelegateTaget:)]) {
                [obj hdDelegateTaget:self];
            }
        }];
    }
}
@end
```

```objective-c
// HDWeakArrayVC.m
- (void)viewDidLoad {
    [super viewDidLoad];

    HDTaget *taget1 = [HDTaget new];
    HDTaget *taget2 = [HDTaget new];
    HDTaget *taget3 = [HDTaget new];
    HDTaget *taget4 = [HDTaget new];

    _delegateTaget = [HDDelegateTaget sharedInstance];
    [_delegateTaget.delegates addObject:taget1];
    [_delegateTaget.delegates addObject:taget2];
    [_delegateTaget.delegates addObject:taget3];
    [_delegateTaget.delegates addObject:taget4];
    [_delegateTaget doSomething];
}

// 手动将单例数组的delegate对象释放掉
- (void)dealloc {
    NSLog(@"%s", __func__);
    [_delegateTaget.delegates removeAllObjects];
}
```



**2、使用弱引用方式：**

>这里其实有存在两种方式，使用NSValue和NSPointerArray、CFArrayCreateMutable。
>
>其中NSValue方式其实还是需要自己稍微的处理一下代码，但是delegate对象会在适当的时候执行dealloc方法，开发过程中只需要在dealloc中将单例的数组将该对象删除；
>
>而NSPointerArray方式，delegate对象会在适当的时候执行dealloc方法，并且当delegate对象释放后，该对象自动从数组中删除。
>
>CFArrayCreateMutable方式，和NSValue弱引用相同的处理方式。

**NSValue方式**

```objective-c
// HDDelegateTaget.h
@class HDDelegateTaget;

@protocol HDDelegate <NSObject>
- (void)hdDelegateTaget:(HDDelegateTaget *)taget;
@end

@interface HDDelegateTaget : NSObject
@property (nonatomic, strong) NSMutableArray <NSValue *> *delegates;
+ (instancetype)sharedInstance;
- (void)doSomething;
@end

// HDDelegateTaget.m
@implementation HDDelegateTaget
- (void)doSomething {
	if (_delegates.count > 0) {
        [_delegates enumerateObjectsUsingBlock:^(NSValue *value, NSUInteger idx, BOOL * _Nonnull stop) {
            id<HDDelegate> obj = value.nonretainedObjectValue;
            if (obj && [obj respondsToSelector:@selector(hdDelegateTaget:)]) {
                [obj hdDelegateTaget:self];
            }
        }];
    }
}
@end
```

```objective-c
// HDWeakArrayVC.m
- (void)viewDidLoad {
    [super viewDidLoad];

    HDTaget *taget1 = [HDTaget new];
    HDTaget *taget2 = [HDTaget new];
    HDTaget *taget3 = [HDTaget new];
    HDTaget *taget4 = [HDTaget new];

    _delegateTaget = [HDDelegateTaget sharedInstance];
    [_delegateTaget.delegates addObject:[NSValue valueWithNonretainedObject:taget1]];
    [_delegateTaget.delegates addObject:[NSValue valueWithNonretainedObject:taget2]];
    [_delegateTaget.delegates addObject:[NSValue valueWithNonretainedObject:taget3]];
    [_delegateTaget.delegates addObject:[NSValue valueWithNonretainedObject:taget4]];
    [_delegateTaget doSomething];
}
```

```objective-c
// HDTaget.m
@implementation HDTaget
// 手动将单例数组的delegate对象释放掉
- (void)dealloc {
    NSLog(@"%s", __func__);
    HDDelegateTaget *taget = [HDDelegateTaget sharedInstance];
    [taget.delegates removeObject:[NSValue valueWithNonretainedObject:self]];
}
@end
```



**NSPointerArray方式**

```objective-c
// HDDelegateTaget.h
@class HDDelegateTaget;

@protocol HDDelegate <NSObject>
- (void)hdDelegateTaget:(HDDelegateTaget *)taget;
@end

@interface HDDelegateTaget : NSObject
//不需要手动去删除内部的元素，元素释放后自动删除
@property (nonatomic, strong) NSPointerArray *delegates; 
+ (instancetype)sharedInstance;
- (void)doSomething;
@end

// HDDelegateTaget.m
@implementation HDDelegateTaget
- (void)doSomething {
    if (_delegates.count > 0) {
        [[_delegates allObjects] enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
            if (obj && [obj respondsToSelector:@selector(hdDelegateTaget:)]) {
                [obj hdDelegateTaget:self];
            }
        }];
    }
}
@end
```

```objective-c
// HDWeakArrayVC.m
- (void)viewDidLoad {
    [super viewDidLoad];

    HDTaget *taget1 = [HDTaget new];
    HDTaget *taget2 = [HDTaget new];
    HDTaget *taget3 = [HDTaget new];
    HDTaget *taget4 = [HDTaget new];

    _delegateTaget = [HDDelegateTaget sharedInstance];
    [_delegateTaget.delegates addPointer:(__bridge void *)taget1];
    [_delegateTaget.delegates addPointer:(__bridge void *)taget2];
    [_delegateTaget.delegates addPointer:(__bridge void *)taget3];
    [_delegateTaget.delegates addPointer:(__bridge void *)taget4];
    [_delegateTaget doSomething];
}
```


**CFArrayCreateMutable方式**

```objective-c
@implementation NSMutableArray (WeakReferences)
+ (id)mutableArrayUsingWeakReferences {
    return [self mutableArrayUsingWeakReferencesWithCapacity:0];
}

+ (id)mutableArrayUsingWeakReferencesWithCapacity:(NSUInteger)capacity {
    CFArrayCallBacks callbacks = {0, NULL, NULL, CFCopyDescription, CFEqual};
    // We create a weak reference array
    return (__bridge id)(CFArrayCreateMutable(0, capacity, &callbacks));
}
@end
  
// 初始化
delegates = [NSMutableArray mutableArrayUsingWeakReferences];
```

