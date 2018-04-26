### Run Loops

* RunLoops 是线程相关的基础框架的一部分。一个RunLoop 就是一个事件处理的循环。用来不停的调度工作以及处理输入事件。使用RunLoop的目的就是让你的线程有工作的时候忙于工作，而没有工作的时候处于休眠状态。

* RunLoop 接收输入事件来自两种不同的来源：输入源(input source)和定时源(timer Source)。输入源 传递异步事件，通常消息来自于其他线程或程序。定时源则传递同步事件，发生特定的时间或者重复的时间间隔。

* 图 3-1 显示了 run loop 的概念结构以及各种源。输入源传递异步消息给相应的 处理例程，并调用 runUntilDate:方法来退出(在线程里面相关的 NSRunLoop 对象调 用)。定时源则直接传递消息给处理例程，但并不会退出 run loop。

* 除了处理输入源，run loops 也会生成关于 run loop 行为的通知 (notifications)。注册的 run loop 观察者(run-loop Observers)可以收到这些通知并在线程上面使用它们来做额外的处理。你可以使用 Core Foundation 在你的线程注 册 run-loop 观察者。

* 在 run loop 运行过程中，只有和模式相关的源才会被监视并允许他们传递事件 消息。(类似的，只有和模式相关的观察者会通知 run loop 的进程)。和其他模式关 联的源只有在 run loop 运行在其模式下才会运行，否则处于暂停状态。 

* 通过指定模式可以使得 run loop 在某一阶段过滤来源于源的事件 通过指定模式可以使得 run loop 在某一阶段过滤来源于源的事件。大多数时候 run loop 都是运行在系统定义的默认模式上。但是模态面板(modal panel)可以运 行在 “modal”模式下。在这种模式下，只有和模式面板相关的源才可以传递消息给 线程。对于辅助线程，你可以使用自定义模式在一个时间周期操作上屏蔽优先级低的 源传递消息。
	* 模式区分基于事件的源而非事件的种类。

* 输入源
	* 输入源异步的发送消息给你的线程。事件来源取决于输入源的种类。基于端口的输入源和自定义的输入源 

* Cocoa 执行 Selector 的源 执行 selector 请求会在目标线程上序列化，减缓 许多在线程上允许多个方法容易引起的同步问题。当在其他线程上面执行 selector 时，目标线程须有一个活动的 run loop。对于 你创建的线程，这意味着线程在你显式的启动 run loop 之前处于等待状态。由于主 线程自己启动它的 run loop，那么在程序通过委托调用 applicationDidFinishlaunching:的时候你会遇到线程调用的问题。因为 Run loop 通过每次循环来处理所有队列的 selector 的调用，而不是通过 loop 的迭代来处理 selector。

* 定时源 定时源在预设的时间点同步方式传递消息 定时器也和你的runloop的特定模式相关,如果定时器所在的模式当前未被runLoop
 监视，那么定时器将不会开始知道runloop运行在相应的模式下。
 
 **Run Loop 的事件队列**
 
   每次运行run loop 你的线程的run loop对会自动处理之前未处理的消息并通知相关的观察者。
   
 **何时使用 Run Loop**
 
   仅当在为你的程序创建辅助线程的时候，你才需要显式运行一个 run loop。Run loop 是程序主线程基础设施的关键部分 对于辅助线程，你需要判断一个 run loop 是否是必须的。如果是必须的，那么 你要自己配置并启动它。你不需要在任何情况下都去启动一个线程的 run loop
   
**使用 Run Loop 对象**

Run loop 对象 供了添加输入源，定时器和 run loop 的观察者以及启动 run loop 的接口。每个线程都有唯一的与之关联的 run loop 对象











RunLoop  

	 使程序一直运行并接受用户输入
	 决定程序在何时应该处理哪些Event
	 调用解耦(message Queue)
	 节省CPU的时间
	 负责渲染屏幕上的UI
	 
	 
-
	RunLoop的管理并非完全自动。你仍然需要设置线程代码在合适的时候启动runLoop来帮助你处理输入事件。在主线程中runloop是自动创建并运行 在子线程开启Runloop需要手动创建且手动开启。
	#【用DefaultMode启动，具体实现查看 CFRunLoopRunSpecific Line2704】
#【RunLoop的主函数，是一个死循环 dowhile】
void CFRunLoopRun(void) {   /* DOES CALLOUT */
    int32_t result;
    do {
        /*
         参数一：CFRunLoopRunSpecific   具体处理runloop的运行情况
         参数二：CFRunLoopGetCurrent()  当前runloop对象
         参数三：kCFRunLoopDefaultMode  runloop的运行模式的名称
         参数四：1.0e10                 runloop默认的运行时间，即超时为10的九次方
         参数五：returnAfterSourceHandled 回调处理
         */
        result = CFRunLoopRunSpecific(CFRunLoopGetCurrent(), kCFRunLoopDefaultMode, 1.0e10, false);
        CHECK_FOR_FORK();
        
        //【判断】：如果runloop没有停止 且 没有结束则继续循环，相反侧退出。
    } while (kCFRunLoopRunStopped != result && kCFRunLoopRunFinished != result);
}

	#【直观表现】
	RunLoop 其实内部就是do-while循环，在这个循环内部不断地处理各种任务（`比如Source、Timer、Observer`），
	通过判断result的值实现的。所以 可以看成是一个死循环。
	如果没有RunLoop，UIApplicationMain 函数执行完毕之后将直接返回，就是说程序一启动然后就结束；
	 
Runloop和线程关系
 
	 1. 每条线程都有唯一的一个与之对应的RunLoop对象。
	 2. 主线程的RunLoop已经自动创建，子线程的RunLoop需要主动创建。
	 3. RunLoop 在第一次获取时创建 在线程结束时销毁。
	 4. 注解: RunLoop 对象是利用字典进行存储，而且key:线程 -- value 线程对应的RunLoop. 
	 
iOS使用NSRunLoop的方法
	  
	  1. run (开始运行)
		  -(void)run; run方法对应上面CFRunLoopRef中的CFRunLoopRun并不会退出，除非调用CFRunLoopStop();通常如果想要永远不会退出RunLoop才会调用此方法。否则可以使用RunUntilDate。
	
	
 如何创建子线程对应的RunLoop?
     
     解决】：开一个子线程创建 runloop ，不是通过 [alloc init]方法创建，而是直接通过调用currentRunLoop 方法来创建。		  
	 原因: currentRunLoop 本身是懒加载 当第一次调用currentRunLoop方法获得该子线程对应的RunLoop的时候,它先去判断(去字典里查找)这个线程的RunLoop是否存在，如果不存在就会自己创建并返回，如果存在直接返回。

RunLoop 源码
 
	 线程执行了这个函数 (__CFRunLoopRun) 后，就会一直处于这个函数内部 "接受消息->等待->处理" 的循环中，
	 直到这个循环结束函数才返回，当然Runloop精华在于在休眠时几乎不会占用系统资源（系统内核负责）	 
	 
	 int32_t __CFRunLoopRun()
	{
	    // 通知即将进入runloop
	    __CFRunLoopDoObservers(KCFRunLoopEntry);
    
    do
    {
        // 通知将要处理timer和source
        __CFRunLoopDoObservers(kCFRunLoopBeforeTimers);
        __CFRunLoopDoObservers(kCFRunLoopBeforeSources);
        
        // 执行被加入的Block（处理非延迟的主线程调用）
        __CFRunLoopDoBlocks();
        // 处理Source0事件
        __CFRunLoopDoSource0();
        
        if (sourceHandledThisLoop) {
            __CFRunLoopDoBlocks();
         }
        // 如果有 Source1 (基于port) 处于 ready 状态，直接处理这个 Source1 然后跳转去处理消息。
        if (__Source0DidDispatchPortLastTime) {
            Boolean hasMsg = __CFRunLoopServiceMachPort();
            if (hasMsg) goto handle_msg;
        }
            
        // 通知 Observers: RunLoop 的线程即将进入休眠(sleep)。
        if (!sourceHandledThisLoop) {
            __CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopBeforeWaiting);
        }
            
        // GCD dispatch main queue
        CheckIfExistMessagesInMainDispatchQueue();
        
        // 即将进入休眠
        __CFRunLoopDoObservers(kCFRunLoopBeforeWaiting);
        
        // 等待内核mach_msg事件
        mach_port_t wakeUpPort = SleepAndWaitForWakingUpPorts();
        
        // 等待。。。
        
        // 从等待中醒来
        __CFRunLoopDoObservers(kCFRunLoopAfterWaiting);
        
        // 处理因timer的唤醒
        if (wakeUpPort == timerPort)
            __CFRunLoopDoTimers();
        
        // 处理异步方法唤醒,如dispatch_async
        else if (wakeUpPort == mainDispatchQueuePort)
            __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__()
            
        // 处理Source1
        else
            __CFRunLoopDoSource1();
        
        // 再次确保是否有同步的方法需要调用
        __CFRunLoopDoBlocks();
        
    } while (!stop && !timeout);
    
    // 通知即将退出runloop
    __CFRunLoopDoObservers(CFRunLoopExit);
	}
	
	 
**RunLoop相关类**
 
 * CFRunLoopRef 【RunLoop 本身】
 * CFRunloopModeRef 【RunLoop的运行模式】
 * CFRunloopSourceRef 【RunLoop要处理的事件源】
 * CFRunloopTimeRef 【Timer事件】
 * CFRunloopObserveRef 【RunLoop的观察者(监听者)】
	 
	 * 讲解: 一条线程 对应一个RunLoop,RunLoop 总运行在某种特定的CFRunLoopRef下(运行模式下)。
	 * 每个RunLoop都可以包含若干个Mode,每个Mode又包含Source源/Timer事件/Observer观察者。
	 * 在RunLoop中有多个运行模式，每次调用RunLoop的主函数[CFRunLoopRun()]时,只能指定其中一个Mode 称CurrentMode 运行 如果需要切换Mode 只能退出CurrentMode切换到指定的Mode进入。目的以保证不同的Mode下的Source/Timer/Observer互不影响。
	 * Runloop 有效，mode 里面 至少 要有一个timer(定时器事件) 或者是source(源)；

**Runloop 相关类（Mode）**
  
  CFRunLoopModeRef 代表RunLoop的运行模式 系统默认提供了5个Mode.
	   
	1.[kCFRunLoopDefaultMode(NSDefaultRunLoopMode)]APP的默认Mode,通常主线程是在这个Mode下运行。
	2.【UITrackingRunLoopMode】界面跟踪Mode,用于ScrollView追踪触摸滑动，保证界面滑动时不受其他Mode影响。
	3. [UIInitialzationRunLoopMode]在刚启动App时进入的第一个Mode 启动完成后就不在使用。
	4. [GESventReveiveRunLoopMode]接受系统事件的内部Mode 通常用不到。
	5. 【KCFRunLoopCommonModes】这个并不是某种具体的Mode可以说是一个占位Mode 一种模式组合。
	CFRunLoop对外暴露的管理mode接口。
	
**RunLoop 相关类(Source)**
  CFRunloopSourceRef 事件源\输入源 有两种分类模式
  按照函数调用栈的分类 source0和source1
  Source0 非基于端口Port的事件 用于用户主动触发的事件 如:点击按钮或点击屏幕
  Source1 基于端口Port事件 通过内核和其他线程相互发送消息与内核相关。
  补充: Source1事件在处理时会分发一些操作给Source0去处理。
  	
**Runloop 相关类（Timer）**   
	
	CFRunLoopTimerRef是基于时间的触发器。
	基本上说的就是NSTimer(CADisplayLink也是加到RunLoop),它受RunLoop的Mode影响。
	而与NSTimer相比，GCD定时器不会受Runloop影响
	
**Runloop 相关类（Observer）**	

相对来说CFRunLoopObserverRef 理解起来并不复杂,它相当于消息循环中的一个监听器，随时通知外部当前RunLoop的运行状态(它包含一个函数指针_callout_将当前状态及时告诉观察者)具体的Observer状态如下。

	/* jianshu:白开水ln Run Loop Observer Activities */
	typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
	    kCFRunLoopEntry = (1UL << 0),           //即将进入Runloop
	    kCFRunLoopBeforeTimers = (1UL << 1),    //即将处理NSTimer
	    kCFRunLoopBeforeSources = (1UL << 2),   //即将处理Sources
	    kCFRunLoopBeforeWaiting = (1UL << 5),   //即将进入休眠
	    kCFRunLoopAfterWaiting = (1UL << 6),    //从休眠装填中唤醒
	    kCFRunLoopExit = (1UL << 7),            //退出runloop
	    kCFRunLoopAllActivities = 0x0FFFFFFFU   //所有状态改变
	};
	
	
	


[视频 孙源](https://juejin.im/entry/599ba638518825242e5c099c)	
**视频总结:**
	![]( /Users/Start/Desktop/runloop.png)
	