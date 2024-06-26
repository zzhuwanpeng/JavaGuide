## 基础
### HashMap 
1. 数组+链表（红黑树，阈值为8）的形式
2. 初始16个，超过0.75负载因子后扩容
3. 在 JDK 1.7 中，多线程环境下的扩容可能导致环形链或数据丢失。在 JDK 1.8 中，多线程环境下可能会发生数据覆盖的情况
4. 为什么 HashMap 的底层数组长度总是 2 的幂次方？
   答案：当数组长度为 2 的幂次方时，使用位运算（h & (length - 1)）来计算索引比使用取模（h % length）更高效，并且数据分布更均匀，减少哈希冲突
5. 遍历时entrySet()时较为高效的

头插法死循环，使用尾插法，多线程下丢失节点，
        红黑树，算法稳定，链表log(n),链表的长度大于8 且数组的长度大于64时，此时链表会转为红黑树
        
### ConcurrentHashMap 
//TODO

单例模式
```java
public class Singleton {
    // 使用 volatile 关键字确保多线程环境下的可见性和禁止指令重排
    private static volatile Singleton instance;

    // 私有构造函数，防止外部通过 new 创建实例
    private Singleton() {
    }

    // 提供一个公共的静态方法，返回单例对象
    public static Singleton getInstance() {
        // 第一次检查，避免不必要的同步
        if (instance == null) {
            // 同步块，保证线程安全
            synchronized (Singleton.class) {
                // 第二次检查，确保只有第一个到达此处的线程能够创建实例
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
在 Spring 的上下文中，bean 的生命周期是由 Spring 容器控制的。容器负责同步和线程安全问题，因此在获取单例 bean 时，不需要使用双重检查锁定。

## 多线程

### 锁
Sychronized和ReenterLock：
功能上：ReentrantLock，可以实现公平锁，尝试加锁(trylock),超时，显示加锁释放，支持中断
       都是可重入的，效率上差不多
底层上：Sychronized是JVM级别的，通过对象上的monitor变量监视，有等待队列，自旋
       ReenterLock是AQS实现，由state + 阻塞队列实现
两者都是悲观锁，但ReenterLock提供了乐观锁的机制，通过tryLock()后，再判断版本号实现

乐观锁：CAS的方式都是乐观锁，AtomicStampedReference还可以解决ABA的问题

### 并发类库
https://pdai.tech/md/java/thread/java-thread-x-juc-overview.html#google_vignette

### Threadlocal
每个线程的 ThreadLocalMap 独立管理该线程的 ThreadLocal 变量。不同线程的 ThreadLocalMap 是隔离的，不共享。
内存泄露：

### AQS:
AQS 核心思想:  
同步队列
一个虚拟队列，一个volitile的status变量，队列中每个节点代表一个线程节点，包含线程id，在队列中的状态，pre节点的指针，next节点的指针。
waitstatus：
- CANCELLED (1)：此节点由于超时或中断被取消。节点进入此状态后将不会再变化。
- SIGNAL (-1)：后继节点的线程处于等待状态，当前节点的线程如果释放或取消，将会通知后继节点，使后继节点的线程得以运行。
- CONDITION (-2)：此节点在条件队列中，节点线程等待在条件上。当其他线程对条件发出信号后，该节点将会从条件队列转移到同步队列中，等待获取同步状态。
- PROPAGATE (-3)：释放共享模式下应该传播到其他节点。这表明下一次共享式同步状态的获取将会无条件地传播下去。
- 新的节点在创建时使用的初始化状态。

条件队列：
结构位于同步队列之下，如果被condition.notify后，被唤醒的线程们将被移动到同步队列中，等待重新获取锁。


### 公平锁和非公平锁
线程一般会先经历锁竞争，排队的阶段，这里是在加锁阶段体现是否是公平锁

AQS队列中，已经有线程在排队了：
公平锁：新线程排队
非公平锁：先竞争锁，再排队，这样如果竞争成功，就绕过排队直接获得锁

### 如何避免死锁：
注意加锁顺序，保证每个线程按同样的顺序加锁。
设置超时时间。

### 线程池
参数：核心线程数，最大线程数，阻塞队列，拒绝策略(AbortPolicy,直接拒接)，存活时间（非核心线程空闲等待时间），时间单位
核心线程中的线程，通过take() 阻塞，直到任务到来

为什么先加入队列，再放入最大线程池： 基于性能考虑，先使用核心线程解决，当队列满了，说明生产水平超出了处理水平，需要增加干活的线程进行处理

①ArrayBlockingQueue
基于数组的有界阻塞队列，按FIFO排序。新任务进来后，会放到该队列的队尾，有界的数组可以防止资源耗尽问题。当线程池中线程数量达到corePoolSize后，再有新任务进来，则会将任务放入该队列的队尾，等待被调度。如果队列已经是满的，则创建一个新线程，如果线程数量已经达到maxPoolSize，则会执行拒绝策略。
②LinkedBlockingQuene
基于链表的无界阻塞队列（其实最大容量为Interger.MAX），按照FIFO排序。由于该队列的近似无界性，当线程池中线程数量达到corePoolSize后，再有新任务进来，会一直存入该队列，而基本不会去创建新线程直到maxPoolSize（很难达到Interger.MAX这个数），因此使用该工作队列时，参数maxPoolSize其实是不起作用的。
③SynchronousQuene
一个不缓存任务的阻塞队列，生产者放入一个任务必须等到消费者取出这个任务。也就是说新任务进来时，不会缓存，而是直接被调度执行该任务，如果没有可用线程，则创建新线程，如果线程数量达到maxPoolSize，则执行拒绝策略。
④PriorityBlockingQueue
具有优先级的无界阻塞队列，优先级通过参数Comparator实现。

拒绝策略
AbstractPolicy（默认）：当任务被拒绝时，会抛出“RejectedExecutionException”异常。
DiscardPolicy：当任务被拒绝时，不会抛出异常，而是直接丢弃任务。
DiscardOldestPolicy：当任务被拒绝时，线程池会丢弃队列中最旧的未执行的任务，然后将被拒绝的任务添加到等待队列当中。
CallerRunsPolicy：当任务被拒绝时，调用execute方法的线程将负责执行该任务。


### wait和sleep
都会让出cpu，但sleep不释放锁，wait会释放锁

## JVM
JMM：实际上是定义了一套机制，来编写正确的开发程序。定义8种操作，happen-before机制等
内存屏障：
在Java内存模型（JMM）中，当我们说“插入一个屏障”时，这是指在程序执行过程中，编译器和处理器在特定位置自动加入特定的指令，这些指令被称为“内存屏障”（Memory Barriers）。
这里volitile，synchronized，final都会插入内存屏障。
```java
public class ImmutableValue {
    private final int value;

    public ImmutableValue(int value) {
        this.value = value;
        // 构造函数结束处，隐含一个StoreStore屏障
    }

    public int getValue() {
        return value;
    }
}
//一旦value字段被写入，就会在构造函数结束时隐含地插入一个内存屏障（具体类型为StoreStore屏障）。这个屏障确保了value字段的赋值操作在任何后续的读操作之前完成，且不会与之重排序。
```
指令重排序的问题
编译器和处理器可能会对指令进行重排序以优化性能，但这可能会在没有额外同步措施的情况下，导致其他线程看到未完全构造的对象。
```java
public class Example {
    private int a;
    private int b;

    public Example() {
        a = 1;
        b = 2;
    }

    public int getA() {
        return a;
    }

    public int getB() {
        return b;
    }
}
//假设在构造函数中对a和b的赋值发生了重排序，且b的赋值先于a执行。如果在这种情况下，构造函数还没有完成，另一个线程尝试访问这个部分构造的对象，它可能会看到a的默认值0而不是1，尽管在构造函数中a是在b之前赋值的。

解决方法：（synchronized也可以解决）
public class SafePublish {
    private int a;
    private int b;
    private volatile int ready;

    public SafePublish() {
        a = 1;
        b = 2;
        ready = 1;  // 写入volatile字段，确保a和b的写入对所有线程可见
    }

    public boolean isReady() {
        return ready == 1;
    }

    public int getA() {
        if (isReady()) {
            return a;
        }
        return -1;
    }

    public int getB() {
        if (isReady()) {
            return b;
        }
        return -1;
    }
}

```

### 类加载
类加载器：目的是判断加载顺序，去哪个目录去加载
顺序：AppClassLoader -> ExtClassLoader -> **BootstrapClassLoader(最先调用，最先调用的类加载器)**
都是交给或代理给父加载器进行加载

### JVM
1.7 分为 栈，堆，方法区
1.8 将方法区改到了本地内存中，叫元空间；另外设置了E，S，O，H四个区域

#### 引用
强软弱虚
软，和虚的区别是是否有足够的空间

#### 根节点

- 本地方法栈中的引用：
- Java 方法栈中的活动线程的局部变量：
- 活动线程：
- 类静态字段：
- 常量池引用：
- JNI 引用：
- 手动管理的引用。这些引用可能是从本地代码显式创建的，需要被视为 GC 根，以避免被 GC 收集。
- 已加载类的引用：


#### 垃圾收集器原理
新生代： ParNew，复制算法
CMS：并发收集器，以获取最短回收停顿时间为目标，采用“标记-清除”算法，分 4 大步进行垃圾收集，其中初始标记和重新标记会 STW 
- 初始标记（Initial Mark）
停顿线程（STW）：这个阶段会有短暂的停顿，因为它需要标记所有从 GC 根直接可达的对象。这包括从根集合（如 Java 栈、本地方法栈、系统类等）直接引用的对象。
- 并发标记（Concurrent Mark）
不停顿线程：在这个阶段，GC 会在应用程序线程并发运行的情况下，遍历从初始标记阶段找到的对象出发的对象图，标记所有可达的对象。这个过程不会停止应用程序的线程。
- 重新标记（Remark）
停顿线程（STW）：由于在并发标记阶段应用程序仍在运行，可能会有一些变更，所以需要重新标记确保标记的准确性。这个阶段通常使用多线程来减少停顿时间，并且采用了一种叫做增量更新（Incremental Update）的技术来处理在并发标记阶段由于程序继续运行而引入的变化。
- 并发清除（Concurrent Sweep）
不停顿线程：在这个阶段，GC 会清理掉那些在标记阶段确定为不可达的对象。这个过程同样是并发进行的，不会停顿应用程序的线程。
- 并发重置（Concurrent Reset）
不停顿线程：在完成清除之后，GC 需要做一些清理工作来为下一次垃圾收集准备。这包括重置 GC 结构的状态等。这个阶段也是并发进行的

G1：
一种服务器端的垃圾收集器，应用在多处理器和大容量内存环境中，在实现高吞吐量的同时，尽可能地满足垃圾收集暂停时间的要求。
标记整理算法
- initial marking：需要stw，仅用于标记GC Root直接关联的对象，速度很快，附加一次Young GC
- concurrent marking：并发进行GC Root Tracing
- final marking：修正并发标记期间用户线程产生的对象变动，记录在线程本地的remember set logs里。最后把所有logs数据合并到remember set里，需要停止线程，但可以并行执行
- live data counting and evaluation：对各个region的回收成本进行排序，制定回收计划

当 GC 发生时，通过 RSet 找到引用当前 Region 的 Old Regions 的某个Card进行扫描，避免了扫描全部的 Old Regions，提高扫描效率。

RSet和Card其实解决了不同维度的问题：前者解决回收老年代时需要扫描所有Old Region的问题，后者解决回收新生代出现跨代引用时需要扫描全部老年代的问题。可以把Card Table理解成内存分页的标识位，RSet理解成记录内存页关系的链表


ZGC：JDK11 中推出的一款低延迟垃圾回收器，适用于大内存低延迟服务的内存管理和回收，SPECjbb 2015 基准测试，在 128G 的大堆下，最大停顿时间才 1.68 ms。停顿时间远胜于 G1 和 CMS。

#### MajorGC
触发条件
老年代空间不足：如果老年代空间不足以分配新的对象，而年轻代和元空间还有足够的空间，这种情况下可能只会触发针对老年代的 Major GC。

#### JVM的优化
- [JVM的优化](./JVM.md)

## 网络
IO多路复用
单个线程监视多个链接，当有事件就绪，这个线程就去通知应用程序去处理。
怎么避免服务端阻塞？-- 服务端把请求注册到一个selector的附路器上，然后循环select()就绪的channal（连接）就可以了
常见的select poll epoll；select和poll都是轮询的方式，epoll是事件驱动的方式，来获取就绪的连接

## spring

### springmvc
filter ： 函数回调
inteceptor： 反射

### springBean的初始化过程
1. 通过构造器或工厂方法创建Bean的实例
2. 通过注解或者配置配置Bean的属性  （**此时涉及三级缓存**） 三级缓存：1级：实例化好的bean（成熟bean），2级：尚未完成初始化的bean（临时bean）  3级：代理bean，保存的是代理bean的factory（不创建代理bean注入原始bean会产生错误）
3. Aware接口
4. BeanPostProcessor 前置方法before
5. bean的初始化方法（init-method或者@PostConstruct注解）
6. 如果Bean实现了InitailizingBean，调用其afterPropertiesSet方法
7. BeanPostProcessor的后置方法，after

三级缓存实现原理的核心类是DefaultSingletonBeanRegistry
```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
		Object singletonObject = this.singletonObjects.get(beanName);
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
			synchronized (this.singletonObjects) {
				singletonObject = this.earlySingletonObjects.get(beanName);
				if (singletonObject == null && allowEarlyReference) {
					ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
					if (singletonFactory != null) {
						singletonObject = singletonFactory.getObject();
						this.earlySingletonObjects.put(beanName, singletonObject);
						this.singletonFactories.remove(beanName);
					}
				}
			}
		}
		return (singletonObject != NULL_OBJECT ? singletonObject : null);
	}
```

### 作用域
未指定scope属性，默认为singleton
@Scope 注解在类上，描述spring容器如何创建Bean实例。
（1）singleton： 表示在spring容器中的单例，通过spring容器获得该bean时总是返回唯一的实例；
（2）prototype：表示每次获得bean都会生成一个新的对象；
（3）request：表示在一次http请求内有效（只适用于web应用）；
（4）session：表示在一个用户会话内有效（只适用于web应用）；
（5）globalSession：表示在全局会话内有效（只适用于web应用）；

### IOC&AOP
IOC: 管理调用引用对象的权利交给了spring
AOP：带接口的是动态代理，不带接口的是GClib，aspectj是字节码实现

spring事务的传播机制
1. A方法调用B方法，事务如何传播，默认是：Require：没有就创建，有就加入
   Require_new 如果事务存在，就挂起原有事务
   Nested
   等等
   这个背不下来  TODO

spring事务什么时候会失效：
spring原理是AOP，失效的根本原因是这个AOP不起作用了：
1. 方法调用自己，this.method,此时this是service对象本身，而不是代理对象
2. 方法不是public方法，@Transaction 无法调用非public方法
3. 异常被吃掉
4. Bean没有被spring管理到

## spring boot
启动类标记：@SpringBootApplication注解标记该类。
该注解包含了多个注解的组合，其中包括@EnableAutoConfiguration（自动装配）、@ComponentScan和@Configuration。

### 自动装配
@EnableAutoConfiguration： 可以扫描第三方未标注注解的jar，通过扫描jar包下的META-INF/spring.factories

### Tomcat是怎么启动的
在创建Spring容器过程中，会利用**@CondtionalOnClass技术来判断当前classpath中是否存在Tomcat依赖**，如果存在则会生成一个启动Tomcat的Bean


## 存储

### mysql
#### b树和b+树的区别
B+树是B树的变种，所有的数据记录都存储在叶子节点，而内部节点只存储关键字，不存储数据记录，这样提高了查询效率。
在B+树中，叶子节点通过指针连接成一个链表，便于范围查询和顺序访问，而B树则没有这样的结构。

#### 事务隔离级别
读未提交，读已提交，不可重复读，串行化
第三个RR是默认的
幻读通过(MVCC和间隙锁控制)
MVCC：当前读和快照读


#### 日志
binlog: 默认是语句复制，cannal需要开启行复制，还有一种混合模式
redo log： 的主要目的是在数据库或系统崩溃后，能够重放（redo）事务操作，以确保已提交的事务更改被永久保存到数据库中。
undolog：提供一种方式来撤销（undo）事务所做的更改，这在事务失败或需要回滚时非常有用。

#### 分库分表
1. 在分库分表后，我们不能再使用mysql的自增主键。因为在插入记录的时候，不同的库生成的记录的自增id会出现冲突。因此需要有一个全局的id生成器。关于分布式id的生成，可以使用我司的Leaf组件
2. ShardDataSource在启动时会解析物理库索引(dbIndexes)和表名后缀(tbSuffix)，然后生成一个库和表的链表
3. 扩容：成倍的增加表，比如128张表，库为除以8，表为%16
4. join， 通过冗余字段避免表join，如有join的解决方法：
   小表广播：适用在一些配置表，或者一般不怎么变更的小表上，然后分表需要和这个小表进行Join。对于小表需要在每个分库上复制一个，所有对这张表的Join就会变成单库Join了。另外，对于这张表的任何变更，zebra后端会使用binlog的方式自动同步到每一个分库上去。所以这种方式叫做小表广播
   Binding Table： 适用在若干个分表上进行Join，前提是这些分表的分表逻辑都是一样的，意味着所有的Join都可以在同一个数据库上进行。比如：表a和表b同时都要分表，而且都是使用UserID进行分表，分表的个数和分库的个数都是一样的
   

#### limit的性能

### redis
#### 数据类型和应用场景
String  |    计数器，sessionid等  |  原理  SDS
hash   |  对象的各个属性 |  Dict，ziplist
列表  |  栈，队列，微信公众号的消息流等 | LinkList/ziplist/quicklist
set 关注的人，点赞等  |   dict/intset
zset  排行榜等 | ziplist/skiplist

#### redis为什么选择单线程
Redis的核心操作，包括接收客户端请求、解析请求、数据读写和发送数据，是在单个线程中完成的，使用了IO多路复用技术来有效地监听和处理来自多个客户端的连接。
Redis本身并不是完全的单线程程序。它在启动时会创建一些后台线程来处理特定任务，比如关闭文件、AOF持久化，并且还有lazyfree线程专门用于异步释放内存，这样可以避免在执行某些命令时导致主线程的卡顿。
Redis6.0引入多线程网络IO处理，提高性能表现。（随着网络硬件的性能提升，Redis 的性能瓶颈有时会出现在网络 I/O 的处理上。所以为了提高网络 I/O 的并行度，Redis 6.0 对于网络 I/O 采用多线程来处理。）
RDB是使用子进程进行存盘，不是线程！

#### 缓存穿透：
定义：缓存没有，直接查库
原因：①缓存中的数据和数据库中的数据都被误删除了，导致缓存和数据库中都没有数据。②黑客恶意攻击，故意大量访问某些读取不存在数据的业务。
解决：
每次系统 A 从数据库中只要没查到，就写一个空值到缓存里去，然后设置一个过期时间。这样的话，下次有相同的 key 来访问的时候，在缓存失效之前，都可以直接从缓存中取数据。
根据用户或者 IP 对接口进行限流。
布隆过滤器，先查redis的布隆过滤器，没有直接返回错误，有的话再查缓存
（一次coe：
采用布隆过滤器做二级缓存进行第一层过滤（如未命中过滤器则直接返回false，如命中过滤器则穿透至redis查询），提升查询性能的同时降低redis的QPS，同时提升redis的命中率
将原来的单个key分片，拆分为多个key来降低每个key中存储的数据量）

布隆过滤器原理：对一个key进行多次不同的hash，每个位都为1的环，说明key存在。

#### 缓存雪崩（穿刺）
缓存统一时间失效，缓存服务器宕机，都会造成数据库压力变大
解决：
设置不同的失效时间比如随机设置缓存的失效时间。
缓存永不失效（不太推荐，实用性太差）。
缓存预热，也就是在程序启动后或运行过程中，主动将热点数据加载到缓存中。

#### 缓存击穿
定义：当缓存中某个热点数据过期时，大量的请求访问了该热点数据，无法从缓存中读取，直接访问数据库。（这是缓存雪崩的一个子问题）
原因：①缓存中某个热点数据过期。
解决：
互斥锁方案，等待第一个请求构建完缓存之后，再释放锁，进而其它请求才能通过该 key 访问数据。
不给热点数据设置过期时间，由后台异步更新缓存，或者在热点数据准备要过期前，提前通知后台线程更新缓存以及重新设置过期时间。


#### Redis的热键探测
如果一个 key 的访问次数比较多且明显多于其他 key 的话，那这个 key 就可以看作是热键。
探测方案：
使用redis的命令redis-cli -p6379 --hotkeys 返回所有 key 的被访问次数。
使用MONITOR监控redis的操作
开源项目等

#### redis怎么和数据库保持一致
先操作缓存的问题：
        问题现象：两个线程，A线程修改数据库，先删除redis数据库，准备开始修改数据库；线程B查询，先查redis没有数据库，则加载老数据到redis；此时A线程再写数据库。出现不一致
        解决：
        删两次数据：弱一致性，线程A先删redis，再删数据库，再删redis；这样下次就查到新的数据了；这里需要有一定的延迟，否则还是会被线程B影响覆盖。
        CQRS：databus，cannal

**先操作数据库则可以直接保证最终一致性**
先删库，再删缓存；但是存在删除失败的情况，采用MQ，解决删除失败的情况

#### redis集群
Redis主从和集群的区别？
Redis主从： 
        1个Master，多Slave，Master负责写，Slave负责读，读写分离；
        通过sentinal进行监控Master是否宕机，重新选主；
        无法扩展
redis-Cluster
        slot槽位，先通过key找到slot位置，然后去对应的节点读写
        多组master-slave结构，每组Master-1个或者多个Slave，Slave是冷备，不提供读，组内自动选主
        多组之间去中心化
        容易扩展
主从同步：先RDB全量快照，再增量写入（Redis默认是RDB的）
        



### es
#### 深度分页
使用searchAfter 需要有一个排序字段。
适用于页面分页。游标的方式一次性快照，适合数据导出，量太大。

#### 选举策略
最小id法，从每个节点随机选择其他的节点中的一个开始
角色：leader，follower，协调节点，数据节点


### Hbase
按列存储
如果有多个列族 ，一行数据库中的rowkey会被存储在多个hfile中
rowkey的前缀设计非常重要

用途：社交，好友关系（因为是稀疏矩阵），点赞，收藏数等等

## 分布式
### 微服务设计 ，DDD

拆分原则：按团队拆分，按业务拆分
原则：
不要有重合业务
接口调用，而不是访问对方的数据
高内聚，低耦合，做好防腐层设计


DDD是一种方法论，领域驱动设计
**通过Ones进行说明** tudo


### duboo

六大核心功能：
负载均衡：随机，轮询，最少活跃数，最短响应时间，一致性hash
三个中心：注册中心(zk,nacos)，配置中心(nacos)，元数据中心
支持失败重试，服务降级
底层调用：Netty，Mina；上次传输协议：dubbo3：Trible（grpc）

过程：
1. 服务者和消费者 注册到 注册中心，粉笔注册和订阅服务
2. 消费者获取到注册中心的服务者列表（根据注册中心的订阅关系），缓存，如果订阅关系变更，消费者会收到一个推送，更新生产者列表缓存。
3. 消费者会生成代理对象，根据负责均衡策略，调用到具体的服务提供者，向monitor记录服务的调用次数和信息
4. 发起调用
5. 服务提供者，反序列化，通过代理调用具体的实现

服务治理：
服务发现与注册：每个服务实例在启动时会注册到服务注册中心，服务消费者可以通过服务注册中心来发现可用的服务实例。
服务配置管理：Spring Cloud Config 提供集中化的外部配置支持，允许应用程序的配置信息与应用程序代码分离，并且能够在不重启的情况下刷新配置。
服务链路追踪：Spring Cloud Sleuth 和 Zipkin 提供了服务链路追踪的能力，帮助开发者了解服务调用的详细流程，以及每个服务调用的时间消耗，便于监控和排查问题。
服务熔断与降级：Spring Cloud Hystrix 提供了熔断器的功能，当下游服务不稳定或响应时间过长时，熔断器会自动开启服务降级策略，防止服务雪崩效应。
负载均衡：Spring Cloud Ribbon 是一个客户端负载均衡工具，可以在进行服务调用时提供负载均衡策略，以优化资源的使用和服务调用的性能。
服务调用：Spring Cloud Feign 是一个声明式的Web服务客户端，它使得编写Web服务客户端更加容易，通过接口和注解即可定义服务绑定和请求映射，简化了服务间的同步调用。
API网关：Spring Cloud Gateway 或者 Netflix Zuul 提供了统一的服务入口，对外暴露API，并且可以实现路由转发、权限校验、流量控制等功能。

### MQ
#### MQ的架构（Kafka）：
![](https://github.com/zzhuwanpeng/JavaGuide/blob/main/docs/img/kafka.png)
https://zhuanlan.zhihu.com/p/671827061  
每个topic有多个patition，一个leader和多个follower，broker就是其中的一个leader或者follower

写入消息： topic，key value；生产环境一般是异步调用
comsumer:  获取offset，消费，返回

消息投递方式，kafka：
at most once：至多一次，但不会重复
at least once：至少一次，但可能重
exactly once：正好一次
如何保证exactly once，生产者使用事务消息，配置enable。idempotence为true，这样日志只记录一次，消费端幂等即可



**消息存储**
1. Kafka
- 一个Topic分别存储在不同的partition中
- 一个partitioin对应着多个replica备份
- 一个relica对应着一个Log
- 一个Log对应多个LogSegment
- 而在LogSegment中存储着log文件、索引文件、其它文件
- 
2. RocketMQ 使用一个统一的CommitLog文件存储所有消息，ConsumeQueue存储消息的地址信息。
   由于是顺序写，区别于kafka的每个partition一个文件的随机写，效率更高

#### 怎么保证消息不丢失
**消息丢失的情况**
1. 跨网络的情况
   producer发送丢失  --  回调确认
           kafka：发送通过实现Callback返回成功失败
           rocketmq: 消息发送+回调，**事务消息**
   leader-> follower（同步）
           可以设置刷盘策略实现，先同步再返回或者先返回在通过从机器
           RocketMq中还有Dleger集群，异步同步，master记录uncommit，当slave完成同步，master再设置为commit。
           Kafka：允许消息少量丢失？？ ACK设置不同的策略
   写入消息到内存，到刷盘
           Rocketmq：同步刷盘 || 异步写盘 -- 配置即可
           Kafka：应该是一样的？？
   consumer拉取，consumer异步执行，先提交offset，再执行事务，此时会丢消息；所以要使用同步机制。

**消息的顺序保证**
   RocketMQ：有机制保证消息的顺序性（MessageQueueSelector，选择队列发送，consumer注解MessageListerOrderly）
   Kafka：没有顺序性的保证
   MQ只保证局部有序，不保证全局有序；例如每个订单各个消息有序
   具体实现：
   RocketMQ：自带机制，MessageQueueSelector保证消息在同一个队列，consummer消费整个队列中的消息（锁队列）
   Kafka：按patition进行分隔，消费者端topic下只对应一个消费者。

** 提高性能，一个消费者多线程读取，怎么保证顺序**
1. orderId按hash再取模，比如生产时partition分配了1,3,5；再按2取模，每个线程一个消息。
2. 每个hash后的线程一个队列，统计一个计数器，批量提交
3. 关闭自动提交，一批任务完成后，统一提交offset（mafka基于滑动窗口实现）

**kafka和rocketmq**
零拷贝：高速读写，内存映射，**不需要在用户态内存拷贝到内核态**
如果没有零拷贝，操作实际是：用户写数据到硬件读取text.txt,读到内核空间，读到用户空间，再写入内核空间，再写入硬件
零拷贝有两种方式：mmap,transer;
mmap:直接获取映射，也就是内存地址，长度，大小等，这样文件不经过用户空间，直接在内核空间完成拷贝；
transer：底层直接使用DMA，不同设备共同访问一个设备，cpu无需大量的中断
RockeMQ：commitlog使用零拷贝，每次写1G
Kafka：index日志使用零拷贝，其他无

### zookeeper
提供了
1. 文件系统
2. 通知机制
#### 四种节点
四种类型的数据节点 Znode
（1）PERSISTENT-持久节点： 除非手动删除，否则节点一直存在于 Zookeeper 上

（2）EPHEMERAL-临时节点：临时节点的生命周期与客户端会话绑定，一旦客户端会话失效（客户端与zookeeper 连接断开不一定会话失效），那么这个客户端创建的所有临时节点都会被移除。

（3）PERSISTENT_SEQUENTIAL-持久顺序节点：基本特性同持久节点，只是增加了顺序属性，节点名后边会追加一个由父节点维护的自增整型数字。 ** 分布式锁 ** 

（4）EPHEMERAL_SEQUENTIAL-临时顺序节点：基本特性同临时节点，增加了顺序属性，节点名后边会追加一个由父节点维护的自增整型数字。** 选举领导者 **

#### zk的作用
集群管理：监控节点存活状态、运行请求等；
主节点选举：主节点挂掉了之后可以从备用的节点开始新一轮选主，主节点选举说的就是这个选举的过程，使用 Zookeeper 可以协助完成这个过程；
分布式锁：Zookeeper 提供两种锁：独占锁、共享锁。独占锁即一次只能有一个线程使用资源，共享锁是读锁共享，读写互斥，即可以有多线线程同时读同一个资源，如果要使用写锁也只能有一个线程使用。Zookeeper 可以对分布式锁进行控制。
命名服务：在分布式系统中，通过使用命名服务，客户端应用能够根据指定名字来获取资源或服务的地址，提供者等信息。

#### 说一下 Zookeeper 的通知机制？
client 端会对某个 znode 建立一个 watcher 事件，当该 znode 发生变化时，这些 client 会收到 zk 的通知，然后 client 可以根据 znode 变化来做出业务上的改变等。


### 分布式基础组件
#### 分布式锁
redis，zk：
redis： 高性能，set操作)+lua（(原子性）， requestId + 超时 + finnaly(释放锁)
ZK： 一致性，zk临时节点，多了节点watch会产生惊群效应


#### 分布式ID
雪花算法
美团leaf：每次取一个段，然后双buffer
美团分布式唯一ID的算法核心是Leaf——一种基于Snowflake算法改进的分布式ID生成系统。Leaf通过结合时间戳、工作机器ID和序列号来生成唯一的ID，确保在分布式环境中的唯一性和顺序性。此外，Leaf还提供了两种模式：Leaf-segment数据库号段模式和Leaf-snowflake时钟回拨优化模式，以适应不同的业务场景和性能需求。

时钟回拨问题：
检测时钟回拨：在生成ID时，系统会检查当前时间是否比最后一次生成ID的时间戳小，如果是，说明发生了时钟回拨。
等待时间追赶：如果检测到时钟回拨，系统不会立即生成新的ID，而是等待直到系统当前时间追赶到最后一次记录的时间戳。这个等待过程保证了在时间戳维度上ID的唯一性

#### 分布式事务
**事务消息**： rocketmq 保证执行本地事务和发送消息是原子性的，：
1. 先发送一个half消息，此时consumer看不见这个消息，用于确保MQ是正常工作的
2. MQ返回half消息后，
3. producer执行本地事务，再发送实际消息，同时携带本地事务状态（三种状态：成功，失败，未知）
4. 成功-consumer开始处理，失败-MQ丢弃消息， 未知
5. 如果是未知，本地事务未执行完（half消息超时），定时询问生产者是否执行完成（15次）回查时检查本地事务的状态，返回本地事务给MQ，重复4步骤

#### 分布式协议
最小ID方案
ZAB协议
1. 投票给自己
2. 收集其他的机器的投票，如果有ID比自己小的，接收投票
3. 计票

raft
** 有任期的概念，投票时可以随机选择领导者节点，或者按时间戳也可以 ** 
- 开始选举
当一个节点（称为候选者）没有在预定时间内接收到当前领导者的心跳消息时，它会认为领导者已经失效。候选者会增加其当前的任期号（term），并转换到候选者状态，然后开始一个新的领导者选举。
- 投票请求
候选者向其他所有节点发送请求投票的消息。这个请求包括候选者的任期号和候选者的日志信息，以便其他节点决定是否投票支持该候选者。
- 投票规则
其他节点在决定是否给候选者投票时，会基于以下规则：
- 每个节点每个任期只能投一票。
节点只会投票给日志至少与自己一样新的候选者（即日志中最后条目的任期号和索引都至少要与自己的一样新）。
- 赢得选举
如果候选者从集群中的大多数节点（超过半数）获得了投票，那么它就被选为新的领导者。

#### 分库分表

#### 分布式任务调度
场景：

1. 大数据量的分片执行

2. 只在一台机器上执行，避免多个机器同时执行（相对单机定时任务而言）

elastic-job

···java
public class OrderProcessingJob extends SimpleJob {
    @Override
    public void execute(ShardingContext shardingContext) {
        int shardingItem = shardingContext.getShardingItem(); // 获取当前分片项
        int shardingTotalCount = shardingContext.getShardingTotalCount(); // 获取分片总数
        List<Long> orderIds = fetchOrderIds(shardingItem, shardingTotalCount); // 获取当前分片项要处理的订单ID列表

        // 处理当前分片项的订单
        for (Long orderId : orderIds) {
            processOrder(orderId);
        }
    }

    private List<Long> fetchOrderIds(int shardingItem, int shardingTotalCount) {
        // 假设订单ID是连续的，您可以根据订单ID来分片
        // 这里只是一个简单的示例，实际情况可能需要根据数据库中的数据来进行分片
        long totalOrders = 10000000; // 假设有1000万订单
        long ordersPerShard = totalOrders / shardingTotalCount; // 每个分片处理的订单数

        long startOrderId = shardingItem * ordersPerShard;
        long endOrderId = (shardingItem + 1) * ordersPerShard;
        if (shardingItem == shardingTotalCount - 1) {
            endOrderId = totalOrders;
        }

        // 根据订单ID范围获取订单列表，这里需要连接数据库并执行相应的查询
        return queryOrderIdsFromDatabase(startOrderId, endOrderId);
    }

    private void processOrder(Long orderId) {
        // 实际处理订单的逻辑
    }

    private List<Long> queryOrderIdsFromDatabase(long startOrderId, long endOrderId) {
        // 实际查询数据库的逻辑
        return new ArrayList<>();
    }
}

elastic-job确保每个分片只在一个机器上执行。
1. 任务分片，分片数量应当是2的倍数，便于扩容。
2. 利用zk作为注册中心

crane
1. 也是zk作为注册中心，完成对Manager的选主
2. 通过Manager服务分配不同的slot到调度器，调度器通过对任务名取hash，判断某个任务是否由自己调度

#### 全链路压测
1. 服务建立唯一标识，System.currentTimeMills调用次数过多，会影响性能，采用1ms只调用一次
2. http（nginxlog），rpc流量录制（实时录制）
3. DB（影子表），redis，mq隔离


## 大数据





        
