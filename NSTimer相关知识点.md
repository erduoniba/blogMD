1、`scheduledTimerWithTimeInterval:` 和 `timerWithTimeInterval:` 区别？

```objective-c
+ (NSTimer *)timerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)yesOrNo;
+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)yesOrNo;
```

`scheduledTimerWithTimeInterval：` 得到的timer对象会加入到当前线程的Runloop中，且默认是NSDefaultRunLoopMode模式，当UI滑动的时候，Runloop会将模式改为NSEventTrackingRunLoopMode，这个时候默认的模式将不再执行，也就是说time会阻塞。

`timerWithTimeInterval：` 得到的timer对象不会主动加入到当前线程的Runloop中，需要手动添加Runloop中，可以设置为NSRunLoopCommonModes模式，NSRunLoopCommonModes是NSDefaultRunLoopMode和NSEventTrackingRunLoopMode的集合，也就是说食用该模式在UI滑动的情况下time不会阻塞。（参考[Apple文档](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html#//apple_ref/doc/uid/10000057i-CH16)）



2、出现循环引用怎么办？

[Weak Reference to NSTimer Target To Prevent Retain Cycle](http://stackoverflow.com/questions/16821736/weak-reference-to-nstimer-target-to-prevent-retain-cycle) 

为什么会出现循环引用的问题的，我可以看看方法中的target在官方的说明：

**The object to which to send the message specified by aSelector when the timer fires. The timer maintains a strong reference to this object until it (the timer) is invalidated**

也就是说timer对象会强引用这个target，然后在设置的seconds秒之后，再次使用target来执行代码

```objective-c
 + (NSTimer *)timerWithTimeInterval:(NSTimeInterval)seconds target:(id)target selector:(SEL)aSelector userInfo:(id)userInfo repeats:(BOOL)repeats
```

解决方式：

1、包装新的target，必须使用__weak修饰作为一个引用，当target释放时，自动执行 `invalidate` 

当然也可以参考[YYWeakProxy](https://github.com/ibireme/YYKit/blob/master/YYKit/Utility/YYWeakProxy.h)建立这个对象

```objective-c
[...]
@implementation BTWeakTimerTarget
{
    __weak target;
    SEL selector;
}

[...]

- (void)timerDidFire:(NSTimer *)timer
{
    if(target)
    {
        [target performSelector:selector withObject:timer];
    }
    else
    {
        [timer invalidate];
    }
}
@end
```

2、block方式来解决循环引用：  [NSTimer和实现弱引用的timer的方式](https://yohunl.com/nstimerhe-shi-xian-ruo-yin-yong-de-timerde-fang-shi/) 

```objective-c
@implementation NSTimer (XXBlocksSupport)
+(NSTimer *)xx_scheduledTimerWithTimeInterval:(NSTimeInterval)interval
                                         block:(void(^)())block
                                       repeats:(BOOL)repeats
{
    return [self scheduledTimerWithTimeInterval:interval
                                          target:self
                                        selector:@selector(xx_blockInvoke:)
                                        userInfo:[block copy]
                                         repeats:repeats];
}
+(void)xx_blockInvoke:(NSTimer *)timer {
    void (^block)() = timer.userinfo;
    if(block) {
        block();
    }
}
@end
```

注意以上NSTimer的target是NSTimer类对象,类对象本身是个单利,此处虽然也是循环引用,但是由于类对象不需要回收,所以没有问题.但是这种方式要注意block的间接循环引用,当然了,解决block的间接循环引用很简单,定义一个weak变量,在block中使用weak变量即可。

3、GCD的方式：  [用Block解决NSTimer循环引用](http://www.jianshu.com/p/1dbd7a228a22) 

```objective-c
uint64_t interval = 1 * NSEC_PER_SEC;
//创建一个专门执行timer回调的GCD队列
dispatch_queue_t queue = dispatch_queue_create("timerQueue", 0);
_timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);
//使用dispatch_source_set_timer函数设置timer参数
dispatch_source_set_timer(_timer, dispatch_time(DISPATCH_TIME_NOW, 0), interval, 0);
//设置回调
dispatch_source_set_event_handler(_timer, ^(){
	NSLog(@"Timer %@", [NSThread currentThread]);
});
dispatch_resume(_timer);//dispatch_source默认是Suspended状态，通过dispatch_resume函数开始它
```

其中的**dispatchsourceset_timer**的最后一个参数,是最后一个参数（leeway），他告诉系统我们需要计时器触发的精准程度。所有的计时器都不会保证100%精准，这个参数用来告诉系统你希望系统保证精准的努力程度。如果你希望一个计时器每5秒触发一次，并且越准越好，那么你传递0为参数。另外，如果是一个周期性任务，比如检查email，那么你会希望每10分钟检查一次，但是不用那么精准。所以你可以传入60，告诉系统60秒的误差是可接受的。他的意义在于降低资源消耗。