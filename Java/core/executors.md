
##### 线程池

`Executor`

void execute(Runnable command)  执行一个任务

`ExecutorService`

void shutdown() 等待当前线程池中已提交任务执行完成后关闭线程池。新提交的任务不会再执行；已提交的任务继续执行。

void shutdownNow()  立即关闭线程池。未启动的任务不再执行并返回；正运行的任务被interrupt。

awaitTermination(long timeout, TimeUnit unit)   等待线程池关闭，或者指定的时间条件满足，或者当前线程被interrupt后执行后续逻辑

Future submit(Callable)  提交一个任务
> 任务执行成功后 future.get() return Callable's result

Future submit(Runnable, result)  
> 任务执行成功后 future.get() return result


Future submit(Runnable)  
> 任务执行成功后 future.get() return null

List<Future> invokeAll(Collection<Callable>)    提交一组任务

T invokeAny(Collection<Callable)    提交一组任务，返回任意一个完成的任务的结果


`ScheduledExecutorService`

ScheduledFuture schedule(Runnable, delay, TimeUnit) 延迟多长时间后执行一个任务

ScheduledFuture scheduleAtFixedRate(Runnable, initialDelay, period, TimeUnit) 延迟多长时间后周期性的执行某个任务
> 1(initialDelay), 2(initialDelay + period), 3(initialDelay + 2*period)...  
> 到规定时间就执行下一个任务。

ScheduledFuture scheduleWithFixedDelay(Runnable, initialDelay, delay, TimeUnit)
> 1(initialDelay), 2(task1_termination + delay), 3(task2_termination + delay)...  
> 要等前一个任务执行完成后，再延迟一定时间执行下一个任务

##### 线程池工厂Executors

`newFixedThreadPool(n)` 创建最多n个可重用线程的线程池

刚创建线程池时，如果线程池没有任务，则不会创建新线程。

新提交一个任务到线程池，如果线程池的线程没有达到最大数n，则创建一个新线程。

如果提交任务时，所有线程都在工作，则新提交的任务先放在队列中。

`newWorkStealingPool(p)`

创建ForkJoinPool线程池。p表示并行数，如果没有p则使用cpu个数作为并行数。

`newSingleThreadExecutor` 

创建只有一个线程的线程池

`newCachedThreadPool`

创建大小动态变化的线程池。当线程池有很多任务需要处理时，会创建新线程处理任务。

当线程空闲超过60s时，线程会被销毁。

因为线程池没有控制最大容量。所以只适合大量的小任务。

`newSingleThreadScheduledExecutor`

创建只有一个线程的线程池。

周期性的执行任务。如果任务执行时抛出异常，则不再执行后续任务。

`newScheduledThreadPool(size)`

创建一个线程池。该线程池可以周期性的执行任务。size为核心线程数量(最小线程数)

> 线程池最大线程数的确定：  
> 1、考虑任务特征  
> 2、考虑计算机硬件情况  
> 
> 最小线程数 >= CPU 数量  
> 
> 越是CPU密集型任务，线程数应该越少；越是IO密集型任务，线程数可以调大。

### Future

cancel(interrupt)

取消任务执行。如果任务未启动，则cancel成功，返回true；如果任务启动运行中，则抛出异常(interrupt==true时)；如果任务执行完成或已取消等，则返回false。

当interrupt=true时，取消运行中的任务会抛出异常；否则会等到运行中的任务执行结束。

get()

阻塞直至任务执行结果返回。

get(timeout, TimeUnit) 

阻塞直至任务执行结果返回或超时。

###ForkJoin

**RecursiveAction**

coInvoke() 启动所有任务

compute() 任务具体执行的内容

**RecursiveTask(带返回值)**

fork() 异步执行任务

join() 阻塞等待异步任务返回结果

V compute() 任务执行逻辑，返回值
