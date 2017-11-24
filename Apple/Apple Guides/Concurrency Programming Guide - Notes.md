# 并发及应用设计
在计算机发展的早期, 计算机单位时间内的工作量是与 CPU 的时钟频率决定的. 但是随着科技的进步和处理器设计得越来越紧凑, 发热以及其他物理限制开始制约处理器的最大时钟频率. 因此, 芯片制造商开始寻求其它的方式来提升他们芯片的性能. 他们的解决方式是增加每片芯片上处理器核心数量. 这样, 单个芯片就可以在不增加 CPU 时钟频率, 改变芯片尺寸, 改变热学特性的情况下, 每秒钟执行更多的指令. 而唯一的问题是, 应该如何最大化利用这些额外的核心.

为了更好地利用多核心, 计算机需要让软件同时做多件事情. 对于像 OS X 或 iOS 这样的现代多任务操作系统, 在每时每刻都可能运行着几百个程序. 因此, 将不同的程序安排在不同核心上应该是可行的. 然而, 这些程序大多数既不是系统守护程序, 也不是实际只消耗极少处理时间的后台应用. 所以我么真正需要的是个人应用程序更加有效地利用额外核心的方式.

应用程序使用多核的传统方式是创建多个线程. 然而, 随着核心数增加, 对线程的处理开始变得麻烦起来. 最大的问题是, 线程代码不能很好地扩展到任意数量的核心. 你不能创建出于核心一样多的线程, 并期望程序运行良好. 你需要知道的是可以高效使用的核心的数量, 但是让应用程序自己计算是一件很困难的事. 即使你正确获得到了核心的数量, 为这么多线程编码并使其高效运行而且还不让它们相互干涉仍然是很难的.

因此, 将这些问题总结一下可以得到, 需要有一个方法可以让应用程序利用电脑的多个核心. 单个程序的工作量也需要能够动态变化, 以适应系统不同的状况. 并且方案要足够简单, 以保证利用多核心的方法不会增加太多的工作量. 好消息是, 苹果的操作系统为所有这些问题提供了解决方案. 这一章, 我们将会看看组成这个解决方案的一些技术以及一些你能够利用并且应用到你代码中的设计调整 ( design tweaks ).

## 离开线程 ( The Move Away from Threads )
虽然线程已经使用了这么多年并且还依然有它发挥的空间, 但是它并不能用一种弹性的方法解决一般性的多任务执行问题. 使用线程, 创建可扩展解决方案的重担完全在于开发人员您. 你必须决定需要使用多少线程, 并且还要随着系统状况动态创建和调整. 另外一个问题是, 您的应用程序承担了创建和维护它所使用的线程的大部分开销.

OS X 和 iOS 使用一种 '异步设计方式' 处理并发的问题, 而不是依赖线程. 异步的函数已经在众多操作系统中存在了很多年, 并用来对那些耗时任务进行初始化, 比如从磁盘读取数据. 当被调用时, 异步函数在幕后做一些工作来启动任务, 但是可能会在任务结束之前就返回. 一般来说, 这种工作会包含获取后台线程, 并在这个线程上启动相应的任务, 然后在任务完成时向调用者发送 ( 通常通过一个回调函数 ) 通知. 过去，如果你想做的事情没有现成的异步函数作为支持的话，那么你就必须自己写异步函数并且创建线程。但是现在， OS X 和 iOS 为你提供的技术可以让你异步执行各种任务，而且你不需要自己管理线程。

异步执行任务的技术之一是 GCD （Grand Central Dispatch）。这个技术把你通常在应用中会写的线程管理的代码下沉到了系统级别。你所需要做的只是定义你想执行的任务然后将它们添加到合适的队列中。GCD 负责创建所需的线程并且安排任务在这些线程上运行。因为线程管理目前变成了系统的一部分，GCD 为任务的管理和执行提供了一个整体的解决方案， 并且效率比传统的线程管理更高。

操作队列（operation queues）是 OBJC 对象，它与派发队列（dispatch queues）非常相似。你定义想要执行的任务并且将它们添加到操作队列，操作队列会负责对这些任务执行的调度工作。像 GCD，操作队列为你管理所有的线程，以确保任务在系统中能够快速并且高效执行。

下面的几个小节提供了更多关于派发队列，操作队列以及其他相关的异步技术的信息。


### 派发队列 ( Dispatch Queues )

派发队列是基于C的用于执行自定义任务的一种机制. 派发队列执行任务既可以是串行的也可以是并行的, 但是总是遵循先进先出的原则. 一个串行派发队列在同一时间只能运行一个任务, 等到这个任务结束之后, 下个任务才能出列, 然后开始执行. 相比之下, 一个并发派发队列会执行尽可能多的任务并且不用等待之前的任务完成.

派发队列还有一些其它的好处:
- 它提供了一个直接且简单的编程接口.
- 它为线程池提供了自动且完整的管理.
- They provide the speed of tuned assembly.
- 内存效率更高 ( 因为线程栈不会在应用内存中保留 ).
- They do not trap to the kernel under load.
- The asynchronous dispatching of tasks to a dispatch queue cannot deadlock the queue.
- They scale gracefully under contention.
- Serial dispatch queues offer a more efficient alternative to locks and other synchronization primitives.

你提交给派发队列的任务必须封装为一个函数或者是一个 block 对象. block 对象是 C 语言的一个特性, 从 OS X v10.6 和 iOS 4.0 开始引进, 从概念上讲, 它很想函数指针, 但它拥有更多的优点. 一般 block 的定义会在函数或者方法内部, 这样 block 就可以从函数或方法中访问其他变量. block 也可以从原始的范围中拷贝到堆上, 当你将任务提交给派发队列时就是这样做的. 这些语法使得仅用相对少的代码实现动态的任务成为可能. 

派发队列是 GCD 的技术中的一部分, 也是 C 运行时的一部分. 更多关于在应用中使用派发队列的信息, 见: [Dispatch Queues](https://developer.apple.com/library/content/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationQueues/OperationQueues.html#//apple_ref/doc/uid/TP40008091-CH102-SW1). 更多关于 block 的信息, 见: [Blocks Programming Topics](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Blocks/Articles/00_Introduction.html#//apple_ref/doc/uid/TP40007502)

### 调度源 ( Dispatch Sources )
调度源是一个基于 C, 为了异步执行特定类型的系统事件的机制. 调度源封装特定类型的系统事件, 并且在事件发生时使用特定的 block 对象或者函数提交到派发队列. 你可以使用调度源去监听以下几种类型的系统事件:
- Timers
- Signal handlers
- Descriptor-related events
- Process-related events
- Mach port events
- Custom events that you trigger

更多关于调度源的信息, 见: [Dispatch Sources](https://developer.apple.com/library/content/documentation/General/Conceptual/ConcurrencyProgrammingGuide/GCDWorkQueues/GCDWorkQueues.html#//apple_ref/doc/uid/TP40008091-CH103-SW1)

### 操作队列 ( Operation Queues )
操作队列是 Cocoa 中与并行派发队列等价的东东, 由 [NSOperationQueue
](https://developer.apple.com/documentation/foundation/operationqueue) 类实现. 派发队列总是以先进先出的顺序执行任务, 而操作队列使用其他的因素决定任务执行的顺序. 这些因素的主要是给定的任务是否取决于其他任务的完成(???). 你可以在定义任务的时候设置依赖关系, 并且使用它来创建复杂的任务执行顺序图.

提交给操作队列的任务必须是 NSOperation 类的实例. 操作对象是一个 OC 对象, 它封装了你想要执行的任务以及执行它所需的数据. 由于 NSOperation 类是一个抽象的基类, 你需要自定义子类去执行任务. 然而, Foundation 框架确实包含了一些具体的子类, 你可以使用它们来执行任务.

操作对象生成 KVO 通知, KVO 是监听你任务进度的一种很有效的方式. 虽然操作队列总是并发执行操作, 不过你可以设置依赖来保证任务按照你想要的顺序执行.

更多关于如何使用操作队列, 以及如何自定义操作对象的信息, 见: [Operation Queues](https://developer.apple.com/library/content/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationObjects/OperationObjects.html#//apple_ref/doc/uid/TP40008091-CH101-SW1).

## 异步设计技术 ( Asynchronous Design Techniques )
在你考虑重新设计代码以支持并发之前, 你应该问问你自己是否有必要自己做. 并发可以通过确保主线程以空闲状态响应用户事件来提升代码的响应速度. 它甚至可以通过在一定时间内平衡各核心的工作来提升代码的执行效率. 然而, 它增加了开销，并增加了代码的整体复杂性, 使编写和调试代码都变得更加困难.

由于它增加了代码复杂度, 并发不是您在产品周期结束时可以移植到应用程序上的特性. 如果要正确使用它, 需要仔细考虑你的 app 需要执行什么样的任务, 以及执行任务时需要的数据结构. 如果错误使用, 你可能会发现你的代码反而运行得更慢了, 而且对用户的响应也不够. 因此, 在设计周期刚开始的时候花些时间来制定一些目标以及思考你需要如何实现.



### 把代码重构为可执行的单元 ( Factor Out Executable Units of Work )
以你对你 app 任务的了解, 你应该已经能够判断哪些代码能够从并发中获益. 如果改变某个任务中的一些步骤的顺序会导致结果的改变, 那么你需要继续按照顺序串行执行这些步骤. 在两种情况下, 你定义任务的执行单元, 它代表需要执行的步骤. 这个工作单元接下来会被你用 block 封装起来, 或者是一个操作对象, 然后被你派发到合适的队列中去.

对于每个你识别出的可执行单元, 最初不要太在意每个单元的工作量. 虽然开启一个线程总是会有一些代价, 但是派发队列和和操作队列的优势之一是: 在很多情况下, 这些代价比使用传统的线程要小得多. 因此, 使用队列执行更小的工作单元会比你自己使用线程更有效率. 当你, 你应该不断去调整实际表现出来的性能, 以及根据需要调整任务的粒度大小. 但是刚开始这样做的时候, 不要把任务分解得太细.

### 确定你需要的队列 ( Identify the Queues You Need )
既然你的任务已经被分解成不同的工作单元, 并且使用 block 对象和操作对象封装好了, 那么你需要确定你想要执行代码的队列. 对于一个给定的任务, 检查你创建的 block 或操作对象, 以及他们正确执行时的顺序.

如果你使用 block 来封装任务, 你可以把 block 加入到串行或者并发的派发队列. 如果要求特定的顺序, 那么你应该将你的 block 加入到串行的派发队列中. 如果没有顺序的要求, 则可以把 block 加入到并发派发队列中或者将他们加入到几个不同的派发队列中去, 依你的需要决定.

如果你是用操作对象封装任务, 队列的选择没有你的对象配置那么重要. 要串行执行操作对象, 你必须配置关联对象之间的依赖. 依赖关系会阻止一个操作执行，直到它所依赖的对象完成工作.

### 提升效率的小Tips
除了把代码重构为小的任务然后加入到队列中，还有一些使用队列多方法来提升你代码整体多效率：
- **如果需要考虑内存使用多话，考虑直接在任务中将结果计算出来**。如果你的应用已经被内存绑定（memory bound），直接计算结果可能比从主存中加载缓存值更快。直接计算结果使用给定多处理器核心的寄存器和缓存，而他们比主存储器速度要快得多。当然，如果测试显示这样做的性能更好的话，你才会这样做。
- **尽早识别出串行的任务，然后尽可能让它们并发执行**。如果一个任务因为依赖某些共享资源必须按照顺序串行执行，那么你应该考虑改变你的架构去除掉那个共享资源。你可以考虑为每个使用这个资源的客户拷贝一份，并且将这些资源一并消除掉。
- **避免使用锁**。派发队列和操作队列提供的支持使得在大部分情况下不需要使用锁。指定一个串行队列（或者使用操作对象的依赖）来顺序执行任务，比使用锁来保护某些共享资源要更好。
- **尽可能使用系统的 Framework** 实现兵法最好的办法是利用系统框架提供的并发方法。很多框架使用线程和一些内部的技术来实现并发的行为。当你定义你的任务时，看看现成的框架是否定义了函数或者方法来实现你想要并发实现的内容。使用这个 API 可以让你少花一些时间并且可以使你的并发更有可能实现。


## （Performance Implications）
操作队列，派发队列，以及派发源是为了使你的代码更方便地执行并发。然而， 这些技术不能保证你的 app 在效率和相应上有提升。您仍然有责任以符合您需求的方式使用队列，并且不会对应用程序的其他资源造成不必要的负担。举例，虽然你可以创建 10000 个操作对象并且将他们提交到操作队列，但是这样做会造成你的 app 分配潜在的重要内存，这可能会导致分页和性能降低（doing so would cause your application to allocate a potentially nontrivial amount of memory, which could lead to paging and decreased performance.）

在对你代码引入并发之前 - 不论是使用队列还是线程 - 你应该收集一些判断你 app 当前性能的基准数据。当引进并发后，你可以收集更多的数据然后将他们和你的基准数据做个对比，看看你的 app 是否的整体性能是否提升。如果并发的引入使你的 app 效率和响应都更低，那你应使用一些性能调试工具来检查问题所在。

想要更多关于性能和性能检测工具的内容，以及进阶的性能相关主题的文章，[Performance Overview](https://developer.apple.com/library/content/documentation/Performance/Conceptual/PerformanceOverview/Introduction/Introduction.html#//apple_ref/doc/uid/TP40001410)。


## 并发和一些其它的技术 （Concurrency and Other Technologies）
把代码重构成为模块化的任务是一种尝试和提升你 app 中并发的一种很好的方式。然而，这种设计思路可能不符合每个 app 的每个场景。取决于你的任务不同，可能有一些其它的方法可以为你的 app 整体的并发提供额外的提升。这一小节描述了一些其它的技术，你可以考虑将它们应用在你的设计中。

### OpenCL 和并发
在 OS X 中，Open Computing Language（OpenCL）是一种基于标准的技术，用于在计算机的图形处理器上执行通用计算。如果您有一套精确定义的计算应用于大型数据集，则OpenCL是一种很好的技术。举例，您可以使用OpenCL对图像的像素执行滤波计算，或者使用它来一次对多个值执行复杂的数学计算。换句话说，OpenCL更适合于数据可以并行运行的问题集。

虽然OpenCL很适合进行大规模数据并行操作，但是它不适合更通用的计算。准备并将数据以及所需的工作内核传输到图形卡上需要大量的工作，以便让 GPU 进行运算。相似地，取出 OpenCL 产生的结果也会耗费大量的工作。结论是，任何与系统交互的工作都不推荐使用 OpenCL。例如，你不应该使用 OpenCL 来处理文件或网络流中的数据。相反，使用OpenCL执行的工作必须更加自包含（self-contained），以便将其传输到图形处理器并进行独立计算。

更多关于 OpenCL 以及该如何使用它的信息，见[OpenCL Programming Guide for Mac](https://developer.apple.com/library/content/documentation/Performance/Conceptual/OpenCL_MacProgGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008312)。

### 何时该使用线程
虽然操作队列和派发队列是并发处理任务的推荐方式，但它们不是万能的。取决于你的 app，你可能还是需要创建自定义线程。如果你确定要创建自定义线程，你应该尽量少地自己创建线程，并且你应该使用这些线程仅仅去处理那些没有别的实现方法的任务。

线程仍然是实现必须实时运行的代码的好方法。派发队列可以尽可能快地运行任务，但是并不能解决实时的限制。如果你需要更多预测在后台运行的代码的行为，线程仍然是个很好的选择。与任何线程编程一样，您应该始终只在有必要的时候使用线程。更多关于线程包的信息，以及如何使用它们，见 [Threading Programming Guide](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Multithreading/Introduction/Introduction.html#//apple_ref/doc/uid/10000057i)

---

# 操作队列

Cocoa 的 operation 以面向对象的方式封装需要异步执行的任务的方法。Operation 被设计成可以与操作队列或者与它们自己结合使用。因为它们是基于 OBJC 的，所以 operation 大多数情况下通常在基于 Cocoa 的 OS X 和 iOS 应用中。

这一章将告诉你如何定义和使用 operation。

## 关于操作对象

操作对象是 *NSOperation* 类的实例，用来封装 app 中需要完成的任务。NSOperation 类本身是一个抽象基类，必须使用它的子类来做一些有用的工作。尽管是抽象类，这个类也确实提供了大量的基础构建以最小化你在子类中需要完成的工作。另外，Foundation 框架提供了两个具体类，你可以在代码中使用它们。下面就列举了这几个类，并且做了一个小结告诉你应该如何使用它们。
- **NSInvocationOperation**：你可以在 app 中根据一个对象以及选择器（selector）来创建一个操作对象。当你现有的方法已经执行所需任务，你可以使用这个类。因为它不需要被继承，你也可以使用这个类以更加动态的方式创建操作对象。
- **NSBlockOperation**：你可以使用这个类来并发执行一个或多个 block 对象。因为它可以执行不止一个 block，block 操作对象会使用‘组’进行操作；仅当所有相关的 blocks 执行结束，则这个 operation 本身才被认为结束了。[Creating an NSBlockOperation Object](https://developer.apple.com/library/content/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationObjects/OperationObjects.html#//apple_ref/doc/uid/TP40008091-CH101-SW2)，[Blocks Programming Topics](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Blocks/Articles/00_Introduction.html#//apple_ref/doc/uid/TP40007502)；
- **NSOpeation**：定义自定义操作对象的基类。继承 *NSOperation* 类可以充分控制你自己实现的 operations，包括更改操作执行的默认方式并报告其状态的能力。

所有的操作对象都支持以下几个核心特性：
- 支持操作对象之间依赖关系图的建立。这些依赖可以保证一个给定的 operation 在它依赖的所有操作结束之后，才会执行。
- 支持可选的 completion block，这个 block 是在 operation 的主要任务完成之后才会执行。
- 支持使用 KVO 通知来监听 operation 执行状态的变化。关于如何观察 KVO 通知，见 [Key-Value Observing Programming Guide](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/KeyValueObserving/KeyValueObserving.html#//apple_ref/doc/uid/10000177i)；
- 支持 operation 优先级的确定，并影响与其相关的 operation 的执行顺序。
- 支持在 operation 执行的过程中使其停止的取消语句。

设计 operations 是为了帮助你在你的 app 中提升并发的级别。Operations 也是一个将 app 中的行为进行管理和封装为简单的具体代码块的好方法。你可以将若干个操作对象提交至操作队列，然后异步地在单独的几个线程中执行相关的任务，而不需要一直在 app 的主线程中运行代码，

## 并发操作 🆚 非并发操作 （Concurrent Versus Non-concurrent Operations）

虽然你通常将 operation 添加到操作队列来执行它，但是这并不是必需的。你也可以通过调用 operation 的 start 方法来手动执行一个操作对象，但是这样做并不能保证 operation 能够并发执行。*NSOperation* 的 *isConcurrent* 方法可以告诉你一个 operation 是同步还是异步执行，这与调用 operation 的 start 方法的线程有关。默认情况下，方法返回 NO，这意味着 operation 在调用线程中同步运行。

如果你想实现一个 *concurrent operation* —— 这说明，这个 operation 在调用线程上异步运行 —— 你必须写一些额外的代码让 operation 异步启动。举例，你创建了一个独立的线程，调用了一个系统的异步函数，或执行其它任何操作，以确保 *start* 方法启动了任务并且立即返回, 并且很可能在任务完成之前返回。

大多数开发人员不应该实现并发操作对象。如果你经常添加 operations 到操作队列，那么你没必要去实现并发 operations。当你提交了一个非并发的 operation 到一个操作队列，这个队列自身会创建了一个线程去运行你的 operation。
因此，向操作队列添加非并发 operation 仍会导致你的操作对象代码的异步执行。在需要异步执行操作而不将其添加到操作队列的情况下，才需要定义并发操作。

## 创建 NSInvocationOperation 对象

*NSInvocationOperation* 类是 *NSOperation* 的一个具体子类，在运行时，可以调用你指定对象的指定 selector。使用这个类可以避免为 app 中的每个任务定义操作对象；特别是你在修改一个现成的 app，它已经有需要的对象和方法用于执行一些必要的任务。你也可以在你想调用的方法依赖于环境时使用它。举例，你可以使用一个 invocation operation 来执行一个 selector，这个 selector 会根据用户的输入而动态改变。

创建一个 invocation operation 的过程很简单。你创建以及初始化一个新的实例，把需要的对象和 selector 传递给它，然后执行初始化方法。下面展示了两个自定义类的方法，以及创建的过程。*taskWithData:* 方法创建了一个新的 invocation 对象并且给了它另外一个方法的名称，这个方法包含了 task 的实现。

```
// Creating an NSInvocationOperation object
@implementation MyCustomClass
- (NSOperation*)taskWithData:(id)data {
    NSInvocationOperation* theOp = [[NSInvocationOperation alloc] initWithTarget:self
                    selector:@selector(myTaskMethod:) object:data];
 
   return theOp;
}
 
// This is the method that does the actual work of the task.
- (void)myTaskMethod:(id)data {
    // Perform the task.
}
@end
```

## 创建一个 NSBlockOperation 对象
NSBlockOperation 类是 NSOperation 的一个子类, 它就像是若干个 block 的封装. 这个类提供了一种面向对象的封装, 它可以为那些使用了操作队列而且不想创建派发队列的 app 进行封装. 你也可以使用 block operation 来利用 operation 之间的依赖, KVO 通知, 以及派发队列没有的一些其它的特性.

当你创建一个 block operation, 在初始化过程中你至少需要添加一个 block; 你可以根据需要在之后添加更多的 blocks. 当需要执行 NSBlockOperation 对象时, NSBlockOperation 对象会按照默认的优先级把所有的 blocks 提交给并发派发队列. NSBlockOperation 对象接下来会等待, 直到所有的 blocks 执行完毕. 当最后一个 block 执行完毕时, operation 对象会将自己标记为已结束. 因此, 你可以使用一个 block operation 来追踪一组正在执行的 blocks, 就像你使用一个 thread join 把多个现成的结果合并到一起一样. 区别是, 由于 block operation 本身是在一个单独的线程上运行的, 你 app 的其他线程可以在等待 block operation 结束时继续执行任务.

下面代码是如何创建一个 NSBlockOperation 对象的简单示例. 这个 block 没有参数以及返回值.

```
// Creating an NSBlockOperation object
NSBlockOperation* theOp = [NSBlockOperation blockOperationWithBlock: ^{
      NSLog(@"Beginning operation.\n");
      // Do some work.
   }];
```

创建完一个 block 操作对象, 你可以使用 *addExecutionBlock:* 方法添加更多的 block. 如果你需要串行执行 blocks, 那么你必须将它们直接提交到期望的派发队列中.

## 定义一个自定义操作对象

如果 block operation 和 invocation operation 对象不是很符合你 app 的需求, 那你可以直接继承 *NSOperation* 类, 并添加你需要的东西. *NSOperation* 类为所有操作对象提供一个通用的子类化点. 这个类还提供了很多基础设施来处理依赖和 KVO 通知. 但是, 你仍然可能需要补充现有的基础设施, 以确保你的 operations 行为正确. 你所需要做的额外工作取决于你想要实现一个非并发操作还是并发操作.

定义一个非并发的 operation 比定义一个并发的 operation 要简单得多. 对于一个非并发的 operation, 你所需要做的所有事情仅仅是执行你的任务, 并且保证它能被取消就行了; 现有类的基础设施会帮你做剩余的所有工作. 对于一个并发的 operation, 你必须用你自定义的代码替换基础设施一些现有的代码. 下面的几个小节将介绍如何实现两种类型的对象.

### 执行主要任务 (Performing the Main Task)
每个操作对象都应该至少实现以下两个方法:
- 一个自定义的初始化方法
- main

你需要一个自定义初始化方法用于把操作对象设置为一个已知的状态, 还有一个自定义的 main 方法用于执行你的任务. 你可以根据需要实现额外的方法, 比如下面列举的:
- 计划从 main 方法中调用的自定义方法
- 访问器, 用于设置数据的值以及访问 operation 的结果
- NSCoding 协议的方法, 可以让你归档和解档操作对象 (operation object)

下面代码展示了一个自定义 *NSOperation* 类的启动模板. (这段代码没有介绍如何处理取消情况, 但是展示了你通常需要的方法.) 这个类的初始化方法只有一个参数, 传递了一个对象, 这个对象包含着一些数据, 然后在操作对象的内部保存了这个对象的引用. 在将结果返回到应用程序之前，main 方法将表面上处理该数据对象.

```
// Defining a simple operation object
@interface MyNonConcurrentOperation : NSOperation
@property id (strong) myData;
-(id)initWithData:(id)data;
@end
 
@implementation MyNonConcurrentOperation
- (id)initWithData:(id)data {
   if (self = [super init])
      myData = data;
   return self;
}
 
-(void)main {
   @try {
      // Do some work on myData and report the results.
   }
   @catch(...) {
      // Do not rethrow exceptions.
   }
}
@end
```

实现 *NSOperation* 子类的更详细的示例代码, 见 [NSOperationSample](https://developer.apple.com/library/content/samplecode/NSOperationSample/Introduction/Intro.html#//apple_ref/doc/uid/DTS10004184)


### 响应取消事件
在一个 operation 开始执行之后, 它会一直执行它的任务, 直到结束或者代码中显式取消了 operation. 取消可以发生在任何事件, 甚至可以发生在 operation 开始执行之前. 虽然 *NSOperation* 类提供了一种取消的方式, 但是识别取消事件是必要的. 如果一个操作被完全中止, 那么可能没有办法回收已分配的资源. 因此, 操作对象应该检查取消事件, 并且如果取消事件在 operation 过程中发生时, 应保证 operation 正常退出.

为了在操作对象中支持取消, 你需要在你自定义的代码中周期性地调用操作对象的 *isCancelled* 方法, 并且在它返回值为 YES 的时候, 立即返回. 支持取消非常重要, 不论你 operation 的持续时间, 不论你是直接继承 *NSOperation* 还是使用它的具体子类. *isCancelled* 方法本身非常轻量, 可以被频繁调用, 而不会造成任何明显的性能损失. 当你在设计操作对象时, 你应该在下面列举出的地方调用 *isCancelled* 方法:
- 在执行任何实际的工作之前立即调用
- 在循环的每次迭代中, 至少调用一次, 或者在每次迭代很长的情况下更加频繁调用
- 在你代码中的任何相对容易中止 operation 的地方调用

下面代码提供了一个非常简单的实例, 关于在操作对象的 main 函数中如何响应取消事件. 在这个例子中, *isCancelled* 方法在 while 循环的每一次迭代中都调用了一次, 允许工作开始之前快速退出, 并且在每个迭代都可以退出.

```
// Responding to a cancellation request

- (void)main {
   @try {
      BOOL isDone = NO;
 
      while (![self isCancelled] && !isDone) {
          // Do some work and set isDone to YES when finished
      }
   }
   @catch(...) {
      // Do not rethrow exceptions.
   }
}
```

虽然刚才的代码中没有包含清除代码 (cleanup code), 但是你自己的代码应该确保能够释放你自己分配的所有资源.

### 为执行并发配置 operation (Configuring Operations for Concurrent Execution)

操作对象在默认情况下会以同步的方式执行 -- 也就是说, 操作对象会在调用 *start*  方法的线程中执行它们的任务. 因为操作队列为非并发 operation 提供线程, 但是大多数的 operations 仍然是异步运行的. 然而, 如果你计划手动执行 operations 且仍然希望它们异步运行, 你必须采取合适的方式来保证这点. 你可以通过将你的操作对象定义为并发的 operation 来保证.

这里列举了实现一个并发 operation 需要重载的方法:
- ```start```: (必须)所有的并发 operation 必须重载这个方法, 用自定义的实现覆盖默认的行为. 因此, 你对这个方法的实现是你的 operation 开始的地方. 你也可以在这建立线程, 或者是执行任务需要的其它环境. 在你的实现中必须禁止调用 super.
- ```main```: (可选)这个方法通常用于实现与操作对象相关联的任务. 虽然你可以在 *start* 方法中执行任务, 但是使用此方法实现任务能使设置和任务代码更加清晰.
- ```isExecuting/isFinished```: (必须)并发 operation 需要建立他们自己的执行环境, 并且需要对使用者报告环境的状态. 因此, 一个并发 operation 必须维护一些状态信息, 知道它什么时候在执行任务, 什么时候已经完成了任务. 它必须能够在使用这些方法之后报告这些信息. 这些方法的实现必须在不同线程中同时调用时是安全的. 当更改这些方法报告的值时，还必须为预期的 key path 生成相应的KVO通知.
- ```isConcurrent```: (必须)为了确定这个 operation 是否是一个并发的 operation, 重载这个方法并且返回 YES.

本节剩余的部分介绍了 ```MyOperation``` 类的示例实现, 它展示了一些实现一个并发 operation 所需的基本代码. ```MyOperation``` 类只是在它自己创建的一个独立的线程上执行了它自己的 ```main``` 方法. ```main``` 方法实际执行的工作其实是无关紧要的. 示例代码的重点在于展示在你定义一个并发 operation 的时候, 你需要提供的基础设施.

下面的代码展示了 ```MyOperation``` 类的 interface 和部分实现. ``` MyOperation``` 类的 ```isConcurrent```, ```isExecuting``` 和 ```isFinished``` 方法非常直接... ```isConcurrent``` 方法应该只是简单返回 YES 用于指示这是一个并发 operation. ```isExecuting``` 和 ```isFinished``` 方法简单返回了存储在该类自身的实例变量中的值.

```
// Defining a concurrent operation

@interface MyOperation : NSOperation {
    BOOL        executing;
    BOOL        finished;
}
- (void)completeOperation;
@end
 
@implementation MyOperation
- (id)init {
    self = [super init];
    if (self) {
        executing = NO;
        finished = NO;
    }
    return self;
}
 
- (BOOL)isConcurrent {
    return YES;
}
 
- (BOOL)isExecuting {
    return executing;
}
 
- (BOOL)isFinished {
    return finished;
}
@end
```

下面代码展示了 ```MyOperation``` 类的 ```start``` 方法. 这个方法的实现至少达到这种程度, 因为这些都是你必须要执行的任务. 例子中, 方法仅仅开启了一个新的线程, 并且让它调用 ```main``` 方法. ```start``` 方法还更新了 ```executing``` 成员变量并且为 ```isExecuting``` key path 生成了 KVO 通知, 以反映这个值的变化. 当它的工作完成时, 方法直接返回, 然后新创建一条独立的线程去执行实际任务.

```
- (void)start {
   // Always check for cancellation before launching the task.
   if ([self isCancelled])
   {
      // Must move the operation to the finished state if it is canceled.
      [self willChangeValueForKey:@"isFinished"];
      finished = YES;
      [self didChangeValueForKey:@"isFinished"];
      return;
   }
 
   // If the operation is not canceled, begin executing the task.
   [self willChangeValueForKey:@"isExecuting"];
   [NSThread detachNewThreadSelector:@selector(main) toTarget:self withObject:nil];
   executing = YES;
   [self didChangeValueForKey:@"isExecuting"];
}
```

下面代码展示了 ```MyOperation``` 类的剩余代码. 就像上面代码展示的 ```main``` 方法是新线程入口点. ```main``` 方法执行与操作对象相关联的工作, 并且在工作最终完成的时候调用自定义的 ```completeOperation``` 方法. ```completeOperation``` 方法接下来会为 ```isExecuting``` 和 ```isFinished``` 这两个 key paths 生成 KVO 通知, 以反馈 operation 的状态变化.

```
- (void)main {
   @try {
 
       // Do the main work of the operation here.
 
       [self completeOperation];
   }
   @catch(...) {
      // Do not rethrow exceptions.
   }
}
 
- (void)completeOperation {
    [self willChangeValueForKey:@"isFinished"];
    [self willChangeValueForKey:@"isExecuting"];
 
    executing = NO;
    finished = YES;
 
    [self didChangeValueForKey:@"isExecuting"];
    [self didChangeValueForKey:@"isFinished"];
}
```

即使 operation 被取消, 你也应该通知 KVO 观察者你的 operation 的工作已经完成. 当一个操作对象依赖于其他操作对象的完成, 那么这个对象会监听其他操作对象的 ```isFinished``` key path. 只有当所有的对象报告它们的任务已经完成后, 依赖 operation 的信号才能运行. 因此, 完成通知生成失败会导致 app 不执行其它的 operation.

### 遵守 KVO 协定
```NSOperation``` 类遵守以下 key paths 的 KVO (翻译有待更加准确)
- ```isCancelled```
- ```isConcurrent```
- ```isExecuting```
- ```isFinished```
- ```isReady```
- ```depedencies```
- ```queuePriority```
- ```completionBlock```

如果你重载了 ```start``` 方法, 或者对 ```NSOperation``` 对象做了任何明显的改变, 除了重载 ```main``` 方法. 那么你必须确保你自定义的对象还保留对这些 key paths 的 KVO 协定. 当重载 ```start``` 方法, 你最应该关心的是 ```isExecuting``` 和 ```isFinished``` 这两个 key paths. 这是重写 ```start``` 方法是最常影响到的两个 key paths.

如果你想实现对除操作对象以外的依赖的支持, 你也可以重载 ```isReady``` 方法, 并且强制它返回 NO, 直到你自定义的依赖条件得到满足. (如果你实现了自定义的依赖, 并且仍然想支持 ```NSOperation``` 类提供的 默认的依赖管理系统, 那么你要确保调用 ```isReady``` 方法的 super 方法.) 当你的操作对象的准备状态发生变更，生成 ``` isReady``` key path 的 KVO 通知, 以报告这些变更. 除非你重载了 ```addDependency:``` 或 ```removeDependency:``` 方法, 不然你不需要担心 ```dependencies``` key path 的 KVO 通知的生成.

虽然你可以为 ```NSOperation``` 类的其他 key paths 生成 KVO 通知, 但是你并不需要这么做. 如果你需要取消一个 operation, 你可以简单地调用现成的 ```cancel``` 方法. 类似, 你也不太需要修改一个操作对象中的队列优先级信息. 最后, 除非你的 operation 有能力动态改变它的并发状态, 不然你也不需要提供 ```isConcurrent``` key path 的 KVO 通知.

## 自定义操作对象的执行行为 (Customizing the Execution Behavior of an Operation Object)
对操作对象的配置发生在你创建他们之后, 但在加入队列之前. 这一节介绍了配置的类型, 并且可以应用到所有的操作对象中, 无论是你继承自 ```NSOperation``` 或是使用现成的子类.

### 配置 operation 之间的依赖
依赖是使不同的操作对象顺序执行的一种方式. 一个 operation 只能等到它依赖的所有 operations 执行完毕之后, 才能开始执行. 因此, 你可以在两个操作对象之间使用依赖创建简单的一对一的依赖, 或构建复杂的对象依赖图.

欲在两个操作对象之间建立依赖, 要使用 ```NSOperation``` 的 ```addDependency:``` 方法. 这个方法创建了从当前操作对象到你指定为参数的目标 operation 的单向依赖. 这种依赖意味着直到目标对象结束执行, 当前对象才能开始执行. 依赖关系不限于在同一队列中的操作. 操作对象管理他们自己的依赖, 所以在 operations 之间创建依赖, 并且将他们全部加入到不同的队列是完全可以接受的. 不过有一件事情是不能接受的, 即在 operations 之间创建循环的依赖关系. 这么做是程序员的失误, 这会使受影响的 operations 一直不会运行. 当所有 operation 的依赖都已经完成执行时, 操作对象通常就可以执行了. (如果你自定义 ```isReady``` 方法的行为, 那么 operation 的准备状态取决于你设置的标准.) 如果操作对象在队列中, 那么这个队列将在任何时间执行那个 operation. 如果你计划手动执行 operation, 那么调用 operation 的 ```start``` 方法取决于你.

> 重要: 应始终在运行 operation 或将其添加到操作队列之前, 配置依赖. 如果在其之后添加依赖, 可能不能防止操作对象的运行.

依赖关系依赖于每个操作对象在状态变化时发出的 KVO 通知. 如果你自定义操作对象的行为, 你应该在自定义的代码中生成适当的 KVO 通知, 以避免依赖关系产生问题. 想要更多关于配置依赖的信息, 参见: [NSOperation Class Reference](https://developer.apple.com/documentation/foundation/operation).

### 改变 operation 的执行优先级
对于添加到队列中的 operations, 执行优先级首先取决于队列中 operation 的准备状态, 然后才取决于他们的相对优先级. 准备状态取决于一个 operation 的依赖, 但是优先级是操作对象本身的属性. 默认情况下, 所有新建的操作对象的优先级都是 'normal', 但是你可以按照需要通过调用操作对象的 ```setQueuePriority:``` 方法来增减优先级.

优先级仅对处于同一操作队列中的 operations 有效. 如果 app 有多个操作队列, 则每个优先级别都与其它队列无关. 因此, 低优先级的 operation 仍然可能比其它队列中的高优先级 operation 先执行.

优先级不能替代依赖. 优先级仅仅能够决定操作队列中那些已经准备好开始执行的 operation. 举例, 如果一个队列中包含有高优先级和低优先级的 operations, 并且两个 operation 都已就绪, 那么队列会先执行高优先级的 operation. 然而, 如果那个高优先级的 operation 还没就绪但是低优先级的 operation 就绪了, 那么队列会先执行低优先级的 operation. 如果你想要阻止一个 operation 在另一个 operation 结束之前执行, 那么你必须使用依赖.

### 更改基础线程优先级
OS X v10.6 及其之后版本, 配置一个 operation 的基础线程的执行优先级已成为可能. 系统中的线程策略由内核管理, 但是通常来说, 更高优先级的线程比更低优先级的线程有更多的机会执行. 在一个操作对象中, 你可以指定线程优先级为一个 0.0 - 1.0 的浮点数, 0.0 是最低的优先级, 1.0 最高. 如果你不指定, 那么 operation 的默认线程优先级为 0.5.

欲设置 operation 的线程优先级, 你必须在将操作对象添加到队列之前(或者手动执行它)调用它的 ```setThreadPriority:``` 方法. 当 operation 开始执行时, 默认的 ```start``` 方法会使用你指定的值去修改当前线程的优先级. 这个心的you'xian

