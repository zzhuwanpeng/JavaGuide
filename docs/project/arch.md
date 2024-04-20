##
业务概念：
栏目，分类 （适用范围）
专辑，直播，电台，广播
碎片
标签，风控标签，付费信息：风控，主播，用户信息

流程：
手机，车载，k-radio，openapi，小程序
主播上传，媒资上传
付费，审核
智能电台（品牌电台）
直播，广播
灵活配置运营位



#  数据同步-cannal
databus: 同步分片(线程)来提高性能，每行一个id用于分片，按行保证顺序性（可以不按binlog顺序）

## cannal
Server端获取binlog，封装成CannalMessage,里面是对应的Entry
Client **订阅**消息，在代码中处理关心的id，并发的进行处理
通过zk完成高可用，消费记录也记录在offset端。

多线程下的顺序性：
先确保每个client只能分配到固定的ID，（或者直接按库表分）
机器内部有一个阻塞队列记录了各个批次号：LinkedBlockingQueue<Long> batchIds，这样批次号必须顺序消费

幂等性：insert重复更新es


##




