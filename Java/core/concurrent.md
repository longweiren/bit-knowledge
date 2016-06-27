

> concurrent包中提供了大量的同步操作帮助类

**CountDownLatch 计数器门闩**

> 可以让一组线程等待其他线程里的操作完成后再`同时`执行。
> 
> 比如我们的短跑运动中，所有的运动员都需要等待发令枪响后才能同时起步比赛。
> 
> CountDownLatch的计数初始化后不能重新设置(除了调用countDown会减一)。如果希望能重新技术，可以考虑使用CyclicBarrier。
> 
> CountDownLatch使用 AQS(AbstractQueuedSynchronizer)来表示计数以及计数器的操作。

```
main thread:

// 发令枪信号（计数1）
CountDownLatch startSignal = new CountDownLatch(1);     
// 运动员信号（计数8）
CountDownLatch runnerSignal = new CountDownLatch(8);    

// 初始化8个运动员，每个运动员拥有一个发令枪信号和一个运动员信号
for(int i = 0; i < 8; i++){
    // 运动员准备起跑
    new Thread(new Runner(startSignal, runnerSignal)).start();
}

// 发令枪响，运动员起跑
// (startSignal计数器调用countDown方法，计数减一，结果为0，所有监听startSignal信号的计数器(调用startSignal.await方法)继续工作)
startSignal.countDown();

// 阻塞当前线程，等待所有运动员发出到达终点信号（runnerSignal计数器计数减为0时发出信号）
runnerSignal.await(); 

// 比赛结束

Runner.java

class Runner implements Runnable {
    private final CountDownLatch startSignal;
    private final CountDownLatch runnerSignal;

    public void run(){
        // 等待发令枪信号(等待startSignal计数器计数减为0)
        startSignal.await();

        // 发令枪响，运动员开跑
        Thread.sleep(1000); // 用sleep模拟运动员跑步的过程

        // 运动员到达终点，runnerSignal计数器减一(当所有的运动员都到达终点时，计数器计数减为0，唤醒监听runnerSignal信号的线程)
        runnerSignal.countDown();
    }
}
```

**CyclicBarrier 周期屏障**

> 当一组线程需要等待彼此都到达同一个屏障点时，可以使用SyclicBarrier。
> 
> 比如一组射击比赛中，需要所有运动员都完成射击时才能知道自己的名次。
> 

```
main thread:
class Main{
    public static void main(String args[]) {
        final Race race = new Race();

        // 每一轮比赛的射击运动员集合
        final List<Shooter> shooters = new ArrayList<Shooter>();

        // 初始化屏障(10个运动员)
        final CyclicBarrier barrier = new CyclicBarrier(10, new Runnable(){
            public void run(){
                // 所有10个运动员都完成射击后的统一操作，如计算排名
                race.sort(shooters);
            }
        });

        race.beginNew(1, shooters, barrier);
    }
}

Shooter.java

class Shooter implements Runnable{
    private int score; // 成绩
    private int grade; // 名次
    private CyclicBarrier barrier;

    public Shooter(CyclicBarrier barrier) {
        this.barrier = barrier;
    }

    public void run() {
        Thread.sleep(1000); //模拟射击运动员的射击操作

        //完成射击，等待其他运动员完成射击
        barrier.await();  //返回还有多少人未完成任务

        // 所有人员都完成射击任务得到自己排名后，继续执行(CyclicBarrier第二个参数代表的任务执行完成后才会调用)
        otherAction();
    }
}


class Race{
    int round = 0; // 第几轮比赛
    List<Shooter> shooters;
    CyclicBarrier barrier;

    public void beginNew(int round) {
        beginNew(round, this.shooters, this.barrier);
    }

    public void beginNew(int round, List<Shooter> shooters, CyclicBarrier barrier) {
        this.round = round;
        this.shooters = shooters;
        this.barrier = barrier;

        shooters.clear();
        barrier.reset();

        for(int i = 0; i < 10; i++){
            Shooter shooter = new Shooter(barrier);
            shooters.add(shooter);

            // 运动员开始射击比赛
            new Thread(shooter).start();
        }
    }
}
```


**信号量 Semaphore**

> 互斥排他锁仅允许一个线程进入临界资源(同步块)；Semaphore则类似维护一个许可集，应用线程需要先从许可集中请求许可，获得许可后才可以操作临界资源，使用完临界资源后使用release释放许可。Semaphore允许多个线程同时访问临界资源，具体有多少个线程可以同时访问临界资源由Semaphore控制。
> 
>  与wait/notiry和 await/signal不同的是，acquire/release与锁无关，只要获得许可就可以操作临界资源。而被wait/await阻塞的操作，即使被notify/signal了，也需要等待重新获得锁后才能继续操作。

```
SemaphoreTask.java
class SemaphoreTask{
    private Semaphore sema;
    public semaphoreTask(Semaphore sema) {
        this.sema = sema;
    }

    public void action() {
        sema.acquire();

        otherAction();

        sema.release();
    }

    public static void main(String[] args) {
        final Semaphore sema = new Semaphore(5); // 维护5个许可

        final SemaphoreTask task = new SemaphoreTask(sema);

        // 有5个线程先获得许可；后5个线程要等待有线程释放许可后才能调用otherAction();
        for(int i = 0; i< 10; i++) {
            new Thread(new Runnable(){
                public void run(){
                    task.action();
                }
            }).start();
        }
    }
}
```


#####数据结构

**Queue**

> 一般队列为先进先出(插入队尾，从队列头取数据)；
> 
> 优先队列根据`comparator`对队列元素进行排序(优先级低的为队尾；优先级高的为队列头)；
> 
> stack则是后进先出(队列头和队尾相同)。

boolean *add*(E)

把元素加入队列。如果队列有长度限制，则队列没有空间时会抛出异常。

boolean offer(E)

把元素加入队列。如果队列有长度限制，则队列没有空间时不会抛出异常，而是返回false。队列有长度限制时，offer优于add。



E *remove*()

移除队列头元素并返回该元素。如果队列为空，则抛出异常。

E poll()

移除队列头元素并返回该元素。如果队列为空，则返回null。


E `element`()

返回队列头元素，不移除。如果队列为空，则抛出异常

E peek()
返回队列头元素，不移除。如果队列为空，则返回null。

**BlockingQueue**

put(E)

把元素加入队列。如果队列有长度限制，则队列没有空间时会等待队列有空间容纳元素为止。

E take()

返回队列头元素。如果队列为空会等待队列有元素为止。

**Dequeue(双端队列)**

addFirst(E)

addLast(E)

offerFirst(E)

offerLast(E)

E removeFirst()

E removeLast()

E pollFirst()

E pollLast()

E getFirst()

E getLast()

E peekFirst()

E peekLast()


**ArrayBlockingQueue**

> 有界队列。队列容量在初始化时就固定。
> 
> 对列维护两个索引takeIndex, putIndex，以及一个重入锁ReentrantLock。
> 
> 对队列的操作均使用takeIndex和putIndex定位队列元素，并包含在ReentrantLock的控制范围内。
> 
> ReentrantLock生成了两个Condition：notFull, notEmpty.
> 
> 往队列加元素时，如果队列已满，则调用 notFull.await，等待队列弹出元素;  
> 如果队列未满，元素加入成功，则调用notEmpty.notify()，提示提取元素操作可以提取了。
> 
> 从队列提取元素时，如果队列为空，则调用 notEmpty.await，等待队列加元素；  
> 如果队列不为空，提取元素成功，则调用 notFull.notify，提示插入操作可以插入了。

**LinkedBlockingQueue**

> 固定容量的有界队列（初始化大小或Integer.MAX_VALUE）。维护了两个重入锁，读锁takeLock，和写锁putLock；以及两个Condition：notEmpty(takeLock), notFull(putLock)


> 往队列加元素时，如果队列已满，则调用notFull.await，等待队列弹出元素；  
> 如果队列未满，元素加入成功，且还有剩余空间，则调用notFull.notify，提示等待插入操作继续尝试插入。
> 如果插入后队列只有1个元素（插入前为空队列），则调用notEmpty.notify，提示队列已有元素了，被阻塞的提取操作可以继续执行了。
> 
> 从队列提取元素时，如果队列为空，则调用notEmpty.await，等待队列插入数据；  
> 如果队列有元素，且弹出元素后队列仍有元素，则调用notEmpty.signal()，提示等待提取操作继续尝试提取元素。  
> 如果提取元素前队列是满的，则提取成功后调用notFull.signal()，提示队列还没有满，被阻塞的写入操作可以继续执行了。

**PriorityBlockingQueue**

> 无边界队列。与ArrayBlockingQueue类似，有一个重入锁ReentrantLock及生成的Condition(notEmpty)。
> 
> 优先队列只关心队列是否为空，当容量不够时可以自动扩容。
> 
> 插入元素时，需要关心是否需要扩容、找到元素正确的位置（中间会有元素交换），并提示队列非空(notEmpty.signal())
> 
> 提取元素时，不关心是否有阻塞在notFull条件的。


**DelayQueue**

> 无边界队列。内部数据为优先队列。
> 
> 优先队列里的元素为Delayed类型，只有元素的delay到期才能提取出来。

**SynchronousQueue**

> 一个生产者消费者队列。
> 
> 消费者线程先阻塞等待元素，生产者线程才可以插入元素到队列，否则插入操作会一直阻塞。反之亦然。
> 

**CopyOnWriteArrayList**

> 内部为一个数组，使用一个重入锁ReentrantLock操作元素的插入、删除
> 
> 插入操作时，在lock的控制范围内先从已有数组copy出新数组，并把新元素加入新数组末尾，再把新数组替换原数组。

**CopyOnWriteArraySet**

> 内部为一个CopyOnWriteArrayList

**ConcurrentSkipListSet**

> 内部为一个 ConcurrentNavigableMap

**ConcurrentSkipListMap**

**ConcurrentHashMap**
