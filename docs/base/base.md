## 基础
### HashMap 
头插法死循环，使用尾插法，多线程下丢失节点，
        红黑树，算法稳定，链表log(n),链表的长度大于8 且数组的长度大于64时，此时链表会转为红黑树
        容量是2的次幂，因为位运算取模效率高，hash函数更均匀，扩容效率高
        
### Hashtable 
是线程安全的，方法级的锁，效率低

## 多线程
### 锁
Sychronized和ReenterLock：
功能上：ReentrantLock，可以实现公平锁，尝试加锁(trylock),超时，显示加锁释放，支持中断
       都是可重入的，效率上差不多
底层上：Sychronized是JVM级别的，通过对象上的monitor变量监视，有等待队列，自旋
       ReenterLock是AQS实现，由state + 阻塞队列实现
两者都是悲观锁，但ReenterLock提供了乐观锁的机制，通过tryLock()后，再判断版本号实现

### 线程池
参数：核心线程数，最大线程数，阻塞队列，拒绝策略(AbortPolicy,直接拒接)，存活时间（非核心线程空闲等待时间），时间单位
核心线程中的线程，通过take() 阻塞，直到任务到来

## spring
### IOC&AOP
IOC: 管理调用引用对象的权利交给了spring
AOP：带接口的是动态代理，不带接口的是GClib，aspectj是字节码实现


### 自动装配
@EnableAutoConfiguration： 可以扫描第三方未标注注解的jar，通过扫描jar包下的META-INF/spring.factories

## 存储

### mysql
#### redo undolog

#### limit的性能


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




        
