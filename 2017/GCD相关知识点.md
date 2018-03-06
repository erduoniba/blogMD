### GCD相关知识点

[多线程技术--GCD](http://hanson647.com/2016/07/19/2016/%E5%A4%9A%E7%BA%BF%E7%A8%8B%E6%8A%80%E6%9C%AF-GCD/) 

[深入学习GCD](http://peilinghui.com/2016/10/05/%E6%B7%B1%E5%85%A5%E5%AD%A6%E4%B9%A0GCD/) 

[GCD 深入理解：第一部分](https://github.com/nixzhu/dev-blog/blob/master/2014-04-19-grand-central-dispatch-in-depth-part-1.md) 

[GCD 深入理解：第二部分](https://github.com/nixzhu/dev-blog/blob/master/2014-05-14-grand-central-dispatch-in-depth-part-2.md) 

#### 1、 GCD（Grand Centra Dispatch）中队列分类：串行与并行 

在使用GCD的时候，我们会把需要处理的任务放到Block中，然后将任务追加到相应的队列里面，这个队列，叫做Dispatch Queue。然而，存在于两种Dispatch Queue，一种是要等待上一个执行完，再执行下一个的Serial Dispatch Queue，这叫做串行队列；另一种，则是不需要上一个执行完，就能执行下一个的Concurrent Dispatch Queue，叫做并行队列，并行队列能开多少个线程由系统决定。这两种，均遵循FIFO原则 （并行队列中，执行顺序遵循FIFO原则，但是结果不确定）。

```
举一个简单的例子，在三个任务中输出1、2、3，串行队列输出是有序的1、2、3，但是并行队列的先后顺序就不一定了。
```

##### 1.1 系统标准提供的两个队列：

```objective-c
// 全局队列，也是一个并行队列, 四个优先级
dispatch_get_global_queue 
// 主队列，在主线程中运行，因为主线程只有一个，所以这是一个串行队列
dispatch_get_main_queue
```

##### 1.2 用户自己创建的两个队列：

```objective-c
// 这是串行队列
dispatch_queue_create("com.demo.serialQueue", DISPATCH_QUEUE_SERIAL) 
// 这是一个并行队列
dispatch_queue_create("com.demo.concurrentQueue", DISPATCH_QUEUE_CONCURRENT) 
//第一个参数为名字，一定要设置这个参数，因为发生崩溃的时候，这个将是一个很重要的指引，这个名字会出现在崩溃信息中，第二个参数如果要创建serial queue就设置为NULL，代表空的c指针，如果是另外一种就设置为DISPATCH_QUEUE_CONCURRENT
//另外要注意的是，虽然我们已经步入了ARC时代，但是Dispatch Queue必须由开发人员来释放，接着上班的代码 dispatch_release(myQueueS); 凡是由create创建的对象都要记得手动释放。但是在iOS6之后无需我们自己管理
  
In the iOS 6.0 SDK and the Mac OS X 10.8 SDK, every dispatch object is also a part of objective C. So you don't want to worry about the memory, ARC will manage the memory of dispatch_queue 
```



#### 2、GCD中线程分类：同步与异步

串行与并行针对的是队列，而同步与异步，针对的则是线程。最大的区别在于，同步线程要阻塞当前线程，必须要等待同步线程中的任务执行完，返回以后，才能继续执行下一任务；而异步线程则是不用等待。

##### 同步与异步线程的创建：

```objective-c
dispatch_sync(..., ^(block)) // 同步线程,在当前线程中执行任务, 不具备开启线程的能力。
dispatch_async(..., ^(block)) // 异步线程, 在新的线程中执行任务, 具备开启线程的能力, 如果当前队列是串行的，则不会开启新的线程。
```

![](http://images.cnitblog.com/i/450136/201406/242038164084931.png) 



#### 3、一些方法

##### 3.1 dispatch_set_target_queue

```objective-c
//dispatch_queue_create函数生成的Dispatch Queue不管是Serial Dispatch Queue还是Concurrent Dispatch Queue，都使用与默认优先级Global Dispatch Queue相同执行优先级的线程，变更执行优先级使用dispatch_set_target_queue函数
dispatch_queue_t mySerialQueue = dispatch_queue_create("com.kugou.mySerQueue", NULL);
dispatch_queue_t myGlobalQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND,0);
dispatch_set_target_queue(mySerialQueue, myGlobalQueue);
//指定第一个参数与第二个参数优先级相同
```



##### 3.2 dispatch_after

```objective-c
dispatch_time_t time = dispatch_time(DISPATCH_TIME_NOW,3ULL * NSEC_PER_SEC);
dispatch_after(time, dispatch_get_main_queue(),^{
 	NSLog(@"123");
})
//注意，dispatch_after函数并不是在指定时间后执行，而只是在指定时间追加处理到dispatch_queue，此源码在3秒后用dispatch_async函数追加block到main dispatch queue相同。在严格时间要求下这个函数的使用会出问题。
```



##### 3.3 dispatch_group

```objective-c
//eg1:
dispatch_group_t group = dispatch_group_create();
dispatch_queue_t queue =    dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT,0);
dispatch_group_async(group, queue, ^{
    sleep(3);
    HDLog(@"1");
}); //任务1
dispatch_group_async(group, queue, ^{
    sleep(2);
    HDLog(@"2");
}); //任务2
dispatch_group_async(group, queue, ^{
    sleep(1);
    HDLog(@"3");
}); //任务3
dispatch_group_notify(group, dispatch_get_main_queue(),^{
    HDLog(@"done");
});
// 3 2 1 done
//任务1、2、3完成后会通知到block中，但是如果任务1、2、3是在其他队列中异步处理比如网络请求，则该方法起不到所以接口成功后通知的功能


//eg2:
dispatch_group_t group = dispatch_group_create();
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT,0);
dispatch_group_async(group, queue, ^{
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        sleep(3);
        HDLog(@"1"); //任务1
    });
});
dispatch_group_async(group, queue, ^{
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        sleep(2);
        HDLog(@"2"); //任务2
    });
});
dispatch_group_async(group, queue, ^{
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        sleep(1);
        HDLog(@"3"); //任务3
    });
});
dispatch_group_notify(group, dispatch_get_main_queue(),^{
  	//dispatch_group_notify 以异步的方式工作, 不阻塞当前线程。当 Dispatch Group 中没有任何任务时，它就会执行其代码，那么 completionBlock 便会运行。你还指定了运行 completionBlock 的队列，此处，主队列就是你所需要的。
    HDLog(@"done");
});
// done 3 2 1
// 如果任务1、2、3是在其他队列中异步处理比如网络请求，则该方法起不到所以接口成功后通知的功能，由此可见 dispatch_group 也只能在当前的 queue 进行任务结束监听。

long result = dispatch_group_wait(group, DISPATCH_TIME_FOREVER)];
// 它会阻塞当前线程, 第一个参数表示要等待哪个group，第二个参数是等待的时间，这里也可以设置一个dispatch_time，如果这个函数返回值为0，则表示你全部处理完毕，否则代表还有函数没有返回。等待的含义是，一旦调用wait函数，调用这个函数的线程就会停止，在经过指定的时间或者是group中的全部执行都结束之后，该函数返回。
// 如果有结束操作这种需求推荐 dispatch_group_notify 方案。


//eg3:
dispatch_group_t group = dispatch_group_create();
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT,0);
dispatch_group_async(group, queue, ^{
    dispatch_group_enter(group); //dispatch_group_enter 手动通知 Dispatch Group 任务已经开始。你必须保证 dispatch_group_enter 和 dispatch_group_leave 成对出现，否则你可能会遇到诡异的崩溃问题。
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        sleep(3);
        HDLog(@"1"); //任务1
        dispatch_group_leave(group); //手动通知 Group 它的工作已经完成。再次说明，你必须要确保进入 Group 的次数和离开 Group 的次数相等。
    });
});
dispatch_group_async(group, queue, ^{
    dispatch_group_enter(group);
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        sleep(2);
        HDLog(@"2"); //任务2
        dispatch_group_leave(group);
    });
});
dispatch_group_async(group, queue, ^{
    dispatch_group_enter(group);
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        sleep(1);
        HDLog(@"3"); //任务3
        dispatch_group_leave(group);
    });
});
dispatch_group_notify(group, dispatch_get_main_queue(),^{
    HDLog(@"done");
});
// 3 2 1 done
// 如果任务1、2、3是在其他队列中异步处理比如网络请求，则该方法起不到所以接口成功后通知的功能，由此可见 dispatch_group 也只能在当前的 queue 进行任务结束监听。
```



##### 3.4 dispatch_barrier （**栅栏**） 

```objective-c
dispatch_queue_t queueC = dispatch_queue_create("com.kugou.gcd.myQueue", DISPATCH_QUEUE_CONCURRENT);
dispatch_async(queue, blk0_reading);
dispatch_async(queue, blk1_reading);
dispatch_async(queue, blk2_reading);
dispatch_barrier_async(queue, blk3_writing);
dispatch_async(queue, blk4_reading);
dispatch_async(queue, blk5_reading);
dispatch_async(queue, blk6_reading);
// 系统会等0，1，2并发执行完，执行blk3，等blk3执行完再并发执行4，5，6

    dispatch_queue_t queueC = dispatch_queue_create("com.demo.concurrentQueue", DISPATCH_QUEUE_CONCURRENT);
    dispatch_async(queueC, ^{
        sleep(3);
        NSLog(@"1");
    });
    dispatch_async(queueC, ^{
        sleep(2);
        NSLog(@"2");
    });

    dispatch_barrier_sync(queueC, ^{
        NSLog(@"3");
    });

    dispatch_async(queueC, ^{
        sleep(1);
        NSLog(@"4");
    });

// 打印结果 ： 2 1 3 4 dispatch_barrier_sync只能控制当前queueC的顺序，但是如果在queueC中又有了其他的异步线程，dispatch_barrier_sync 将无法控制, 比如下面👇：

dispatch_queue_t queueC = dispatch_queue_create("com.demo.concurrentQueue", DISPATCH_QUEUE_CONCURRENT);
    dispatch_async(queueC, ^{
        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
            sleep(3);
            NSLog(@"1");
        });
    });

    dispatch_async(queueC, ^{
        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
            sleep(2);
            NSLog(@"2");
        });
    });

    dispatch_barrier_sync(queueC, ^{
        NSLog(@"3");
    });

    dispatch_async(queueC, ^{
        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
            sleep(1);
            NSLog(@"4");
        });
    });
// 打印结果：3 4 2 1

// 在访问数据库或文件时, 为了提高效率, 读取操作放在并行队列中执行. 但是写入操作必须在串行队列中执行(避免资源抢夺问题). 为了避免麻烦, 此时dispatch_barrier_async函数作用就出来了, 在这函数里进行写入操作, 写入操作会等到所有读取操作完毕后, 形成一道栅栏, 然后进行写入操作, 写入完毕后再把栅栏移除, 同时开放读取操作.

dispatch_barrier_sync和dispatch_barrier_async：
共同点：
1、等待在它前面插入队列的任务先执行完；
2、等待他们自己的任务执行完再执行后面的任务。
不同点：
1、dispatch_barrier_sync将自己的任务插入到队列的时候，需要等待自己的任务结束之后才会继续插入被写在它后面的任务，然后执行它们；
2、dispatch_barrier_async将自己的任务插入到队列之后，不会等待自己的任务结束，它会继续把后面的任务插入到队列，然后等待自己的任务结束后才执行后面任务。
  
处理多线程时读写操作造成线程不安全，使用 dispatch_barrier 时，有如下几种队列来加入 dispatch_barrier：
1、dispatch_get_main_queue中加入 dispatch_barrier，明显不合理，这些文件读存操作不应该在主现场执行，并且不应该在穿行队列执行，不然将毫无意义；
2、自定义串行队列：一个很坏的选择；障碍不会有任何帮助，因为不管怎样，一个串行队列一次都只执行一个操作；
3、全局并发队列：要小心；这可能不是最好的主意，因为其它系统可能在使用队列而且你不能垄断它们只为你自己的目的；
4、自定义并发队列：这对于原子或临界区代码来说是极佳的选择。任何你在设置或实例化的需要线程安全的事物都是使用障碍的最佳候选。（OK）

一个例子：
PhotoManager.m：
@interface PhotoManager ()
@property (nonatomic,strong,readonly) NSMutableArray *photosArray;
@property (nonatomic, strong) dispatch_queue_t concurrentPhotoQueue; ///< Add this
@end

+ (instancetype)sharedManager {
    static PhotoManager *sharedPhotoManager = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        sharedPhotoManager = [[PhotoManager alloc] init];
        sharedPhotoManager->_photosArray = [NSMutableArray array];
        sharedPhotoManager->_concurrentPhotoQueue = dispatch_queue_create("com.selander.GooglyPuff.photoQueue",
                                                    DISPATCH_QUEUE_CONCURRENT); 
    });
 
    return sharedPhotoManager;
}
 
- (NSArray *)photos {
    __block NSArray *array; // 1 __block 关键字允许对象在 Block 内可变。没有它，array 在 Block 内部就只是只读的，你的代码甚至不能通过编译。
    dispatch_sync(self.concurrentPhotoQueue, ^{ // 2 在 concurrentPhotoQueue 上同步调度来执行读操作。
        array = [NSArray arrayWithArray:_photosArray]; // 3 将相片数组存储在 array 内并返回它。
    });
    return array;
}

- (void)addPhoto:(Photo *)photo {
    if (photo) { // 1 在执行下面所有的工作前检查是否有合法的相片。
        dispatch_barrier_async(self.concurrentPhotoQueue, ^{ // 2 添加写操作到你的自定义队列。当临界区在稍后执行时，这将是你队列中唯一执行的条目。
            [_photosArray addObject:photo]; // 3 这是添加对象到数组的实际代码。由于它是一个障碍 Block ，这个 Block 永远不会同时和其它 Block 一起在 concurrentPhotoQueue 中执行。
            dispatch_async(dispatch_get_main_queue(), ^{ // 4 最后你发送一个通知说明完成了添加图片。这个通知将在主线程被发送因为它将会做一些 UI 工作，所以在此为了通知，你异步地调度另一个任务到主线程。
                [self postContentAddedNotification]; 
            });
        });
    }
}
```

![](https://koenig-media.raywenderlich.com/uploads/2014/01/Dispatch-Barrier-480x272.png) 





##### 3.5 dispatch_apply

```objective-c
dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
dispatch_apply(10, queue, ^(size_t index){
NSLog(@"%zu", index);
});
NSLog(@"done");
// 0 1 2 3 4 5 6 7 8 9 done （其实大部分测试都是按照顺序执行，但是将10变成100，就有所变动）
dispatch_apply 表现得就像一个 for 循环，但它能并发地执行不同的迭代。这个函数是同步的，所以和普通的 for 循环一样，它只会在所有工作都完成后才会返回。
当在 Block 内计算任何给定数量的工作的最佳迭代数量时，必须要小心，因为过多的迭代和每个迭代只有少量的工作会导致大量开销以致它能抵消任何因并发带来的收益。而被称为跨越式（striding）的技术可以在此帮到你，即通过在每个迭代里多做几个不同的工作。

那何时才适合用 dispatch_apply 呢？
1、自定义串行队列：串行队列会完全抵消 dispatch_apply 的功能；你还不如直接使用普通的 for 循环。
2、主队列（串行）：与上面一样，在串行队列上不适合使用 dispatch_apply 。还是用普通的 for 循环吧。
3、并发队列：对于并发循环来说是很好选择，特别是当你需要追踪任务的进度时。（OK）

注意点：
1、你创建并行运行线程而付出的开销，很可能比直接使用 for 循环要多。若你要以合适的步长迭代非常大的集合，那才应该考虑使用 dispatch_apply。
2、你用于创建应用的时间是有限的——除非实在太糟糕否则不要浪费时间去提前优化代码。如果你要优化什么，那去优化那些明显值得你付出时间的部分。你可以通过在 Instruments 里分析你的应用，找出最长运行时间的方法。看看 如何在 Xcode 中使用 Instruments 可以学到更多相关知识。
3、通常情况下，优化代码会让你的代码更加复杂，不利于你自己和其他开发者阅读。请确保添加的复杂性能换来足够多的好处。
```



##### 3.6 dispatch_semaphore_t （信号量）

```objective-c
dispatch_semaphore是GCD基于计数器的一种多线程同步机制,异步变成同步,解决因为多线程的特性而引发数据出错的问题，与他相关的共有三个函数，分别是
dispatch_semaphore_create，dispatch_semaphore_signal，dispatch_semaphore_wait。
  
dispatch_semaphore_t semaphore = dispatch_semaphore_create(0); //创建一个信号量。参数指定信号量的起始值。这个数字是你可以访问的信号量，不需要有人先去增加它的数量。（注意到增加信号量也被叫做发射信号量）。译者注：这里初始化为0，也就是说，有人想使用信号量必然会被阻塞，直到有人增加信号量。这里的传入的参数value必须大于或等于0，否则dispatch_semaphore_create会返回NULL
[self block:^{
  	NSLog(@"1"); //任务1
  	dispatch_semaphore_signal(semaphore); //在这里你告诉信号量你不再需要资源了。这就会增加信号量的计数并告知其他想使用此资源的线程。这个函数会使传入的信号量dsema的值加1
}];

dispatch_time_t timeout = dispatch_time(DISPATCH_TIME_NOW, 10 * NSEC_PER_SEC);
dispatch_semaphore_wait(semaphore, timeout); //这个函数的作用是这样的，如果dsema信号量的值大于0，该函数所处线程就继续执行下面的语句，并且将信号量的值减1；如果desema的值为0，那么这个函数就阻塞当前线程等待timeout（注意timeout的类型为dispatch_time_t，不能直接传入整形或float型数），如果等待的期间desema的值被dispatch_semaphore_signal函数加1了，且该函数（即dispatch_semaphore_wait）所处线程获得了信号量，那么就继续向下执行并将信号量减1。如果等待期间没有获取到信号量或者信号量的值一直为0，那么等到timeout时，其所处线程自动执行其后语句。
```





##### 测试一：

```objective-c
NSLog(@"111");
dispatch_sync(dispatch_get_main_queue(), ^{
    NSLog(@"222");
});
NSLog(@"333");	

//结果：
// 打印 "111" 后 程序卡死
// 当前是串行队列，当执行到 dispatch_sync 时，因为是同步操作，所以 dispatch_sync 需要等待 block 执行完之后才能向下执行，当执行 NSLog(@"222"); 的时候会加入到主现场中，但是这个时候主现程正是 dispatch_sync 在执行，所以 block 的 NSLog(@"222") 需要等待 dispatch_sync 执行完之后才能向下执行。这样就你等我，我等你，造成了死锁
```



##### 测试二：

```objective-c
NSLog(@"1"); // 任务1
dispatch_sync(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^{
    NSLog(@"2"); // 任务2
});
NSLog(@"3"); // 任务3
// 打印结果 1 2 3
```

![](http://www.brighttj.com/wp-content/uploads/2015/09/gcd-deadlock-2.png) 

 测试三：

```objective-c
dispatch_queue_t queue = dispatch_queue_create("com.demo.serialQueue", DISPATCH_QUEUE_SERIAL);
NSLog(@"1"); // 任务1
dispatch_async(queue, ^{
    NSLog(@"2"); // 任务2
    NSLog(@"4"); // 任务4
});
NSLog(@"5"); // 任务5
NSLog(@"6"); // 任务6

// 打印结果： 1 5 6 2 4
//首先建立了一个串行的自定义队列，执行任务1后，遇到异步线程，不会等待执行任务2便会往下走，从而执行任务5，网上说的 5 2 顺序不确定，但是我用模拟器及真机测试发现任务5确实在任务2之前执行。
```



 测试四：

```objective-c
dispatch_queue_t queue = dispatch_queue_create("com.demo.serialQueue", DISPATCH_QUEUE_SERIAL);
NSLog(@"1"); // 任务1
dispatch_async(queue, ^{
    NSLog(@"2"); // 任务2
    dispatch_sync(queue, ^{  
        NSLog(@"3"); // 任务3
    });
    NSLog(@"4"); // 任务4
});
NSLog(@"5"); // 任务5

// 打印结果： 1 5 2 程序卡死
//首先建立了一个串行的自定义队列，执行任务1后，遇到异步线程dispatch_async，不会等待执行任务2便会往下走，从而执行任务5，网上说的 5 2 顺序不确定，但是我用模拟器及真机测试发现任务5确实在任务2之前执行。执行任务2之后，遇到了同步线程dispatch_sync，需要等待任务3执行完才能继续往下走，但此时任务3也串行队列中，而此时的串行队列正被 dispatch_sync 占用着，这样就你等我，我等你，造成了死锁
```



测试五：

```objective-c
NSLog(@"1"); // 任务1
dispatch_async(dispatch_get_global_queue(0, 0), ^{
    NSLog(@"2"); // 任务2
    dispatch_sync(dispatch_get_main_queue(), ^{
        NSLog(@"3"); // 任务3
    });
    NSLog(@"4"); // 任务4
});
NSLog(@"5"); // 任务5
// 打印结果： 1 5 2 3 4 
```

![](http://www.brighttj.com/wp-content/uploads/2015/09/gcd-deadlock-4.png) 



测试六：

```objective-c
dispatch_async(dispatch_get_global_queue(0, 0), ^{
    NSLog(@"1"); // 任务1
    dispatch_sync(dispatch_get_main_queue(), ^{
        NSLog(@"2"); // 任务2
    });
    NSLog(@"3"); // 任务3
});
NSLog(@"4"); // 任务4
while (1) {
    
}
NSLog(@"5"); // 任务5
// 打印结果： 4 1 程序卡住
// Main Queue：【异步线程、任务4、死循环、任务5、任务2】
// Global Queue异步线程中的任务有：【任务1、同步线程、任务3】
// 因为 dispatch_async 在 并行队列中，并且异步执行，所以任务4无需等待立刻执行，然后程序进入死循环，Main Queue阻塞。但是加入到Global Queue的异步线程不受影响，继续执行任务1后面的同步线程。这个时候执行Main Queue队列中的同步线程，任务3需要在任务2执行完才能执行，但是此时任务2在主线程中无法执行，所以卡住。
```

![](http://www.brighttj.com/wp-content/uploads/2015/09/gcd-deadlock-5.png) 





#### 4、一些问题：

##### 4.1 并发和并行

```objective-c
并行（parallelism）
这个概念很好理解。所谓并行，就是同时执行的意思，无需过度解读。判断程序是否处于并行的状态，就看同一时刻是否有超过一个“工作单位”在运行就好了。所以，单线程永远无法达到并行状态。
要达到并行状态，最简单的就是利用多线程和多进程。

并发（Concurrent）
使多个操作可以在重叠的时间段内进行(two tasks can start, run, and complete in overlapping time periods)。单核单线程能支持并发。
  
例子说明：
你吃饭吃到一半，电话来了，你一直到吃完了以后才去接，这就说明你不支持并发也不支持并行。你吃饭吃到一半，电话来了，你停了下来接了电话，接完后继续吃饭，这说明你支持并发。你吃饭吃到一半，电话来了，你一边打电话一边吃饭，这说明你支持并行。并发的关键是你有处理多个任务的能力，不一定要同时。并行的关键是你有同时处理多个任务的能力。
```



![](https://koenig-media.raywenderlich.com/uploads/2014/01/Concurrency_vs_Parallelism.png) 

