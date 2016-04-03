##GCD -Grand Central Dispatch 大中枢派发
###两个核心概念 ：
1. 任务
2. 队列

**任务**分为**同步任务**和**异步任务**  
同步任务：
```objc
dispatch_sync(dispatch_queue_t queue, dispatch_block_t block);

```
异步任务
```objc
dispatch_async(dispatch_queue_t queue, dispatch_block_t block);
```
**队列**可分为**串行队列**和**并发队列**。  
串行队列:  
1.手动创建
```objc
dispatch_queue_t queue = dispatch_queue_create("com.yqx.queue", DISPATCH_QUEUE_SERIAL);
```    
2.主队列(GCD自带的一种特殊的串行队列，放在主队列中的任务都会放到主线程中执行)
```objc
dispatch_queue_t queue = dispatch_get_main_queue();
```
并发队列:  
1.手动创建
```objc
dispatch_queue_t queue = dispatch_queue_create("com.yqx.queue", DISPATCH_QUEUE_CONCURRENT);

```
2.全局并发队列（GCD默认提供，提供给整个应用使用，不需要手动创建）
```objc
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0); 

```
###使用
**GCD的使用**简单概括就是 定制任务，然后将任务添加到队列中就可以了。
GCD会自动将队列中的任务取出，放到对应的线程中执行。任务的取出会遵循队列的FIFO原则。

```objc
//异步 并发 可以同时开启多条线程

- (void)asyncConcurrent{
   
    dispatch_queue_t queue = dispatch_queue_create("com.yqx", DISPATCH_QUEUE_CONCURRENT);
    
    dispatch_async(queue, ^{
        NSLog(@"1-----%@", [NSThread currentThread]);
    });
    dispatch_async(queue, ^{
        NSLog(@"2-----%@", [NSThread currentThread]);
    });
    dispatch_async(queue, ^{
        NSLog(@"3-----%@", [NSThread currentThread]);
    });
    
}

//同步 并发 不会开启新线程

- (void)syncConcurrent{
    
    dispatch_queue_t queue = dispatch_queue_create("com.yqx", DISPATCH_QUEUE_CONCURRENT);
    
    dispatch_sync(queue, ^{
        
    });
    
}

//异步 串行 会开启新的线程，但是任务是串行的，执行完一个任务，再执行下一个任务
- (void)asyncSerial{
    
    dispatch_queue_t queue = dispatch_queue_create("come.yqx", DISPATCH_QUEUE_SERIAL);
    
    dispatch_async(queue, ^{
        
    });
}

//同步 串行 不会开启新的线程，在当前线程执行任务。任务是串行的，执行完一个任务，再执行下一个任务
- (void)syncSerial{
    
    dispatch_queue_t queue = dispatch_queue_create("come.yqx", DISPATCH_QUEUE_SERIAL);
    
    dispatch_sync(queue, ^{
        
    });
}

//异步 主队列 不会开启新线程 只在主线程中执行任务
- (void)asyncMain{
    
    dispatch_queue_t queue = dispatch_get_main_queue();
    
    dispatch_async(queue, ^{
        
    });
}
//同步 主队列 （使用不当会发生主线程锁死）
- (void)syncMain{
    
    dispatch_queue_t queue = dispatch_get_main_queue();
    
    dispatch_sync(queue, ^{
        
    });
}

//异步 全局并发队列
- (void)asyncGlobal{
    
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    
    dispatch_async(queue, ^{
        
    });
}
//同步 全局并发队列
- (void)syncGlobal{
    
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    
    dispatch_sync(queue, ^{
        
    });
}
```
其它常用方法
```objc
//barrier界限、屏障 先执行barrier之前的 再执行barrier 再执行barrier之后
- (void)barrier
{
    dispatch_queue_t queue = dispatch_queue_create("yqx", DISPATCH_QUEUE_CONCURRENT);
    
    dispatch_async(queue, ^{
        NSLog(@"----1-----%@", [NSThread currentThread]);
    });
    dispatch_async(queue, ^{
        NSLog(@"----2-----%@", [NSThread currentThread]);
    });
    
    dispatch_barrier_async(queue, ^{
        NSLog(@"----barrier-----%@", [NSThread currentThread]);
    });
    
    dispatch_async(queue, ^{
        NSLog(@"----3-----%@", [NSThread currentThread]);
    });
    dispatch_async(queue, ^{
        NSLog(@"----4-----%@", [NSThread currentThread]);
    });
}

//延迟执行
- (void)delay
{
//    [self performSelector:@selector(sayHi) withObject:nil afterDelay:2.0];

//    [NSTimer scheduledTimerWithTimeInterval:2.0 target:self selector:@selector(sayHi) userInfo:nil repeats:NO];
    
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        NSLog(@"hi");
    });
}

//这段代码在程序运行过程中只会被执行一次
- (void)once
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        NSLog(@"hi");
    });
}

//队列组
- (void)group
{
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

    dispatch_group_t group =  dispatch_group_create();
    dispatch_group_async(group, queue, ^{
        // 耗时异步操作A
    });
    dispatch_group_async(group, queue, ^{
        // 耗时异步操作B
    });
    dispatch_group_notify(group, queue, ^{
        // AB都执行完毕后执行
    });
}
```

####转载请注明出处 https://github.com/EnterYang