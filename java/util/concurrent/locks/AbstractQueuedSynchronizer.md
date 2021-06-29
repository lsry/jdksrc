## AbstractQueuedSynchronizer  

### O、概述    
- CAS 操作和等待队列组成的抽象同步器框架；  
- 由子类去实现对应的受保护方法，完成具体的同步器；
- 独占和共享模式；
- 条件对象内部类，帮助互斥锁实现条件等待；
- 序列化需要 readObject 方法实现，默认只序列化底层原子整数维护状态；

### 一、基本字段     
1. 双端队列头 head，尾 tail；
2. 状态 status；

### 二、子类实现    
1. 改变状态值：     
    + getState
    + setState
    + compareAndSetState   

2. 受保护方法——获得和释放资源，默认未实现，抛出 UnsupportedOperationException        
独占模式：      
    + tryAcquire
    + tryRelease     

    共享模式：
    + tryAcquireShared
    + tryReleaseShared   
    + isHeldExclusively：判断是否是独占模式持有资源

### 三、等待队列——CLH 变形   
+ 入队：在队尾执行 CAS，成功后设置前驱的 next 字段；    
+ 出队：第一个结点检查 prev 字段，并且设置 head 的值；
+ 锁取消：结点状态改变，由 cleanQueue 遍历整个队列，清除对应结点；  
+ 条件结点：由条件来连接队列中的结点；

#### 队列结点内部类   
**Node**    
1. 字段：   
    + Node prev：前驱  
    + Node next：后继
    + Thread waiter：当前结点的阻塞线程
    + int status：状态；  
    可取值：    
        + 0：结点刚初始化默认值；  
        + WAITING：1，等待
        + CANCELLED：0x80000000，获取资源已取消，为负；
        + COND：2，在使用条件对象等待中

2. 方法：以 CAS 方式改变结点状态及前驱后缀字段；   

**SharedNode** 继承 Node，作一个类型标记，共享式结点    
**ConditionNode** 继承 Node，使用条件对象而构造的结点    

#### 队列操作     
1. 队列头尾字段 head 和 tail，只有在需要入队时才初始化一个哑结点方便队列操作；   
2. cleanQueue：    

### 四、资源申请——acquire      
这是一个多参数的申请资源方法，由独占或者共享模式共用来统一处理相关问题；      
参数：      
+ Node node：一般为空，条件对象时不为空；
+ int arg：获取资源数目；
+ boolean shared：共享模式为 true，独占为 false；
+ boolean interruptible：
+ boolean timed：当限时获取资源时为 true，无限期为 false； 
+ long time：获取资源的最后截止时间；

### 五、独占模式队列   
**ExclusiveNode** 继承 Node，作一个类型标记，独占式结点     
方法：    
以下是子类实现的：    
+ tryAcquire(int arg)：资源获取，参数默认为 1；
+ tryRelease(int arg)：释放资源；
+ isHeldExclusively：判断是否是独占模式；    

以下外部调用者来申请资源：申请到则线程运行，否则进入等待队列或者由于某种条件而放弃   
+ acquire(int arg)：获得对应数量资源；    
+ acquireInterruptibly(int arg)：可在线程被中断时放弃申请资源；  
+ tryAcquireNanos(int arg, long nanosTimeout)：可在线程被中断或者超时时放弃申请资源； 

以下是释放资源：   
+ release：释放资源，并且调用 signalNext 唤醒下一个结点；
+ signalNext：将后继结点的 WAITING 取消，并且从阻塞中唤醒；