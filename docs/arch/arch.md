##  架构作用
1. 管理复杂性
2. 效率最大化
   -- 架构的本质就是对系统进行有序化重构，不断减少系统的“熵”，使系统不断进化；
   -- 架构的本质就是对系统进行有序化重构，以符合当前业务的发展，并可以快速扩展。




## 待整理

### sql优化
强制索引 force index,因为索引有可能不被选中
```java
   /* Use index specified in FORCE INDEX FOR ORDER BY, if any. */ 
   /* 如果有强制索引，会优先根据强制索引进行查询，*/ 
                   if (tab->table()->force_index)
                     usable_keys.intersect(tab->table()->keys_in_use_for_order_by);
   /* Do a cost based search on the indexes that give sort order */
   /* 如果没有强制索引，会对所有索引进行排序成本评估，选择扫描最少行数的索引*/
                   test_if_cheaper_ordering(tab, join->order, tab->table(),
                                            usable_keys, -1, select_limit,
                                            &best_key, &read_direction,
                                            &select_limit);

### JVM案例
1. 分析来看故障根因是因为该集群的 codecache 过高，导致 Push 节点由原先的 JIT 编译退化为解释执行(具体参考： Java Codecache)，所以该台 Push 节点的请求在同一时间均变慢，成为一台慢节点。

当前 PushServer 节点的 Java 版本为 JDK 8，是从 JDK 7 升级上来的。猜测历史上 PushServer 使用 JDK 7，为增大 JDK7 默认的 40MiB 的ReservedCodeCache，在启动脚本中写死了 128MiB 的值覆盖默认值。当前 PushServer 使用 JDK 8，默认 ReservedCodeCache 即为 240MiB，启动脚本的配置反而限制了最大 CodeCache 使用量，也进一步导致了这个问题。
