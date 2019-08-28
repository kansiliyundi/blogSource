---
title: iOS Crash Hook
date: 2018-4-19 15:24:55
tags: iOS
---

> crash 是程序中开发人员必不可少要面临的问题
相对后端程序员可以随时看到crash日志来说,移动端程序员面对crash就比较麻烦了,要拿到第三方统计或者用户上报的日志文件,然后符号化,再去处理.
所以我们希望可以用一些手段去捕获它.

#### Crash捕获

iOS崩溃主要是有Mach异常,以及Objective-C异常两种:
Mach异常最后会被转换为Signal信号,所以可以捕获Signal信号来处理它.
Objective-C的异常NSException会由系统抛出,可以通过注册NSUncaughtExceptionHandler来捕获它.

##### 直接上代码 一步一步来
```
//记录之前的异常处理器
static NSUncaughtExceptionHandler *previousUncaughtExceptionHandler = nil;
```
```
//注册未捕获异常处理
    [[self alloc] registerExceptionHandler];
//注册信号异常处理
    [[self alloc] registerSignalHandler];

```
```
//注册捕获异常处理
-(void)registerExceptionHandler{
    //先获取到程序已经注册过的异常处理方法
    previousUncaughtExceptionHandler = NSGetUncaughtExceptionHandler();
    //将自己的异常处理方法set进去
    NSSetUncaughtExceptionHandler(&HandleException);
}
//注册信号异常处理
-(void)registerSignalHandler{
    //利用signal(int, void (*)(int))函数将需要处理的信号注册进去. 第二个参数如果传入SIG_DFL为默认的信号处理程序
    signal(SIGABRT, SignalHandler);
    signal(SIGILL, SignalHandler);
    signal(SIGSEGV, SignalHandler);
    signal(SIGFPE, SignalHandler);
    signal(SIGBUS, SignalHandler);
    signal(SIGPIPE, SignalHandler);
}
```
上面的代码值得一提的是注册NSException异常的方法registerExceptionHandler.
因为全局只允许存在一个异常处理的Handler,项目中不可避免的会集成一些第三方SDK内部或许也会收集异常,比较友好的做法是我们在注册自己的异常处理方法前,先通过NSGetUncaughtExceptionHandler()获取到程序中已经注册过的处理方法并持有它,再运用NSSetUncaughtExceptionHandler()将我们自己的方法注册进去,直到我们的处理机制运行完毕,再手动调用下之前的异常处理器,将异常传递出去.
```
//附上相应的信号解释
SIGABRT
程序中止命令信号
调用abort函数生成的信号。

SIGILL
程序非法指令信号
执行了非法指令. 通常是因为可执行文件本身出现错误, 或者试图执行数据段. 堆栈溢出时也有可能产生这个信号。

SIGSEGV
程序无效内存中止信号
试图访问未分配给自己的内存, 或试图往没有写权限的内存地址写数据.

SIGFPE
程序浮点异常信号
在发生致命的算术运算错误时发出. 不仅包括浮点运算错误, 还包括溢出及除数为0等其它所有的算术的错误。

SIGBUS
程序内存字节未对齐中止信号
非法地址, 包括内存地址对齐(alignment)出错。比如访问一个四个字长的整数, 但其地址不是4的倍数。它与SIGSEGV的区别在于后者是由于对合法存储地址的非法访问触发的(如访问不属于自己存储空间或只读存储空间)。

SIGPIPE
程序Socket发送失败中止信号
管道破裂。这个信号通常在进程间通信产生，比如采用FIFO(管道)通信的两个进程，读管道没打开或者意外终止就往管道写，写进程会收到SIGPIPE信号。此外用Socket通信的两个进程，写进程在写Socket的时候，读进程已经终止。
```

##### 异常处理
```
//OC异常处理
static void HandleException(NSException *exception) {
    //递增的全局计数器
    int32_t exceptionCount = OSAtomicIncrement32(&UncaughtExceptionCount);
    //如果全局计数器大于10则不处理 避免并发量过大
    if (exceptionCount > UncaughtExceptionMaximum) {
        return;
    }
    //获取堆栈
    NSArray *callStackSymbols = [exception callStackSymbols];
    NSMutableDictionary *userInfo = [NSMutableDictionary dictionaryWithDictionary:[exception userInfo]];
    [userInfo setObject:callStackSymbols forKey:UncaughtExceptionHandlerAddressesKey];
    //新建一个NSException对象并回到主线程对它进行最终的处理
    [[[GGCrashHookManager alloc] init]
     performSelectorOnMainThread:@selector(handleException:)
     withObject:[NSException exceptionWithName:[exception name]
                             reason:[exception reason]
                           userInfo:userInfo]
     waitUntilDone:YES];
    //将异常传递
    previousUncaughtExceptionHandler(exception);
}
```

```
//信号异常处理
static void SignalHandler(int signal) {
    int32_t exceptionCount = OSAtomicIncrement32(&UncaughtExceptionCount);
    if (exceptionCount > UncaughtExceptionMaximum) {
        return;
    }


    NSMutableDictionary *userInfo = [NSMutableDictionary dictionary];
    //获取堆栈信息
    NSArray *callStack = [GGCrashHookManager backtrace];
    [userInfo setObject:callStack forKey:UncaughtExceptionHandlerAddressesKey];
    [userInfo setObject:[NSNumber numberWithInt:signal] forKey:UncaughtExceptionHandlerSignalKey];
    //将信号异常信息也写入到NSException对象里 回到主线程统一做最后处理
    [[[GGCrashHookManager alloc] init]
     performSelectorOnMainThread:@selector(handleException:)
     withObject:
     [NSException exceptionWithName:UncaughtExceptionHandlerSignalExceptionName
                             reason: [NSString stringWithFormat:@"Signal %d was raised!",signal];
                           userInfo: userInfo]
     waitUntilDone:YES];
}
```
##### 异常处理最后一步
```
//最终处理方法
-(void)handleException:(NSException *)exception{
    [self exceptionDataHandle:exception];

    /*
      程序执行到这里就可以结束了,我们只是想要拿到堆栈信息,但是如果在这里你想做一些友好化的提示,例如崩溃弹窗,点击继续APP不死的操作还可以做下面的一些事
      */
      //创建一个runLoop runLoop不提供创建方法 下面的写法会判断是否存在,不存在则会创建
      CFRunLoopRef runLoop = CFRunLoopGetCurrent();
      //获取到runLoop的所有Mode
      CFArrayRef allModes = CFRunLoopCopyAllModes(runLoop);
      //写一个死循环
      while (1) {
          for (NSString *mode in (__bridge NSArray *)allModes) {
              //快速切换Mode
              CFRunLoopRunInMode((CFStringRef)mode, 0.001, false);
          }
      }
      /*
      上面的写法,我个人理解是因为主线程本身开启了它的runLoop,
      runLoop在Main函数中做的事情又本身是在做一个走走停停的死循环,
      而signal信号会通知程序终止,main函数中的runLoop无法继续循环
      尚不确定signal信号发送后是否会关闭主线程的runLoop
      如果会杀死,那就是手动为主线程开启了一条runLoop防止崩溃
      如果不会杀死,那就是强行调起runLoop切换他的mode
      杀死的可能更大,这一点有待测试
      */

    //取消信号监听
    signal(SIGABRT, SIG_DFL);
    signal(SIGILL, SIG_DFL);
    signal(SIGSEGV, SIG_DFL);
    signal(SIGFPE, SIG_DFL);
    signal(SIGBUS, SIG_DFL);
    signal(SIGPIPE, SIG_DFL);

    if ([[exception name] isEqual:UncaughtExceptionHandlerSignalExceptionName]) {
        //如果是信号异常 将信号传递出去
        kill(getpid(), [[[exception userInfo] objectForKey:UncaughtExceptionHandlerSignalKey] intValue]);
    }else{
        //如果是未捕获异常 直接将异常传递出去
        [exception raise];
    }
}

-(void)exceptionDataHandle:(NSException *)exception {
    NSString *title = @"\n--------崩溃日志---------\n";
    NSString *appInfo = @"app信息:\n";
    NSString *exceptionName = @"异常名称:\n";
    NSString *exceptionReason = @"异常原因:\n";
    NSString *exceptionUserInfo = @"异常userInfo:\n";
    NSString *exceptionCallStackSymbols = @"异常堆栈:\n";
    NSString *exceptionInfo = [NSString stringWithFormat:@"%@\n%@%@\n%@%@\n%@%@\n%@%@\n%@%@\n",title,appInfo, getAppInfo(),exceptionName,exception.name,exceptionReason, exception.reason,exceptionUserInfo,exception.userInfo ? : @"no user info", exceptionCallStackSymbols, [exception callStackSymbols]];
    /*
     这个exceptionInfo现在就可以用来打印或者写入自己的日志系统,上传服务器了
    */

}
```
