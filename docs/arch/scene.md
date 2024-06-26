# 场景分析
## 库存
redis中的库存记录
```java
{
   availableStock: 90,
   totalStock: 100,
   version: 1609298417020,
   detail: '门店标识#商品标识#订单号#序列号#业务标识#批次号（可忽略）#回滚标识#当前时间#总库存#库存版本号#变化前库存#变化后库存',
   '门店标识#商品标识#订单号#序列号#业务标识': '固定值' ,
    ......
}
```
1. 防止超售： redis扣减库存，这里需要使用lua脚本（开启redis事务 -> 防重逻辑->设置缓存，设置超时时间 -> 设置流水 -> 防重 -> 结束redis事务）
2. 防止少买：通过aof日志，和redis扣减后的mq，双重信息，记录db的流水表（mq采用重试机制（渐进式重试））
3. 恢复机制（TODO):
消单恢复库存—正常处理流程，如图3右图，主要处理步骤如下：
  1、校验流水：校验与恢复库存请求相同的order_id + sku_id + sequence 的库存变更流水，校验详细逻辑如下：
      存在流水，且[ sum(流水库存变动数) + 当前恢复库存数 ] <=  0，则恢复库存；如[ sum(流水库存变动数) + 当前恢复库存数 ] >  0，则不恢复。此校验保证同一订单，商品的恢复库存的数量必须小于或等于扣减库存的数量
      不存在流水，因存在AOF日志消息延迟的可能，所以先将该恢复库存请求放入延迟队列，等待后续继续校验。此处无法判定延迟时间，因此，限定重试次数+过期时间的方案，超过限定次数或者过期时间，仍未获取到流水，则认为非法的恢复库存请求
  2、恢复库存：更新库存Squirrel，进行恢复库存
  3、记流水：Squirrel执行恢复库存成功后，异步通过AOF日志消息记录库存变更流水至流水表

消单恢复库存—异常处理流程，如图3右图，各异常场景处理策略详细如下：
1、同一订单重复恢复库存：流水校验时，可识别出非法恢复库存请求
2、库存服务→Squirrel超时：将恢复库存请求放入延迟度队列，等待后续重试



## 抢红包
1.  set化： 每个红包一个id，请求时根据红包id分到不同的server上
2.  单机队列+ 缓存：请求本分配到单机后，进程会形成一个队列，统计器上还不熟了一个memcache的缓存用于控制最大的并发数
3.  双维度分表解决数据量增多的问题： 因为前面红包数据按id分表，而冷热数据需要按天分表  db.xx.t_y_dd  xx/y是红包ID的hash值后三位，dd是01-31，表示一个月最多31天
4.  
5. 然后，为了保证每个用户抢红包的先后顺序，我们把一个红包相关的操作串行起来，放到一个队列里面，依次消费。
   从上述 set 分流我们可以看出，一台服务器可能会同时处理多个红包的操作，所以，为了保证消费者处理 DB 不被高并发打崩，我们还需要在消费队列时用缓存来限制并发消费数量。
6. 预先生成，指的是在红包开抢之前已经完成了红包的金额拆分，抢红包时只是依次取出拆分好的红包金额。
    使用二倍均值法生成的随机数，每次随机金额会在 0.01 ~ 剩余平均值*2 之间。假设当前红包剩余金额为 10 元，剩余个数为 5，10/5 = 2，则当前用户可以抢到的红包金额为：0.01 ~ 4 元之间。


