#### Demo功能

使用Flink streaming统计每天各个页面的访客数(UV)，每10分钟刷新一次

> 可直接在IDE中运行



#### Demo结果

输入

```
"http://www.example.com/index1\tuser1\t2019-08-09 23:40:15",
"http://www.example.com/index1\tuser2\t2019-08-09 23:42:50",
"http://www.example.com/index1\tuser1\t2019-08-09 23:56:15",
"http://www.example.com/index1\tuser3\t2019-08-09 23:57:15",
"http://www.example.com/index1\tuser1\t2019-08-10 00:05:15",
"http://www.example.com/index1\tuser2\t2019-08-10 00:06:15",
"http://www.example.com/index2\tuser1\t2019-08-10 00:07:15",
"http://www.example.com/index2\tuser1\t2019-08-10 00:06:15",
"http://www.example.com/index6\tuser6\t2019-08-10 00:15:15"
```

输出

```
20190809	http://www.example.com/index1	2 #这是时间窗口08-09 23:40:00 ~ 23:50:00的输出
20190809	http://www.example.com/index1	3 #这是时间窗口08-09 23:50:00 ~ 24:00:00的输出
20190810	http://www.example.com/index1	2 #这是时间窗口08-10 00:00:00 ~ 00:10:00的输出
20190810	http://www.example.com/index2	1 #这是时间窗口08-10 00:00:00 ~ 00:10:00的输出
```



#### 实现方案

原理是基于[State](https://ci.apache.org/projects/flink/flink-docs-release-1.8/dev/stream/state/state.html#working-with-state)来维护各种状态

- 用户去重: 使用MapState来存储用户id

- 区分每天不同的State: 通过获取窗口起始或者终止时间来获得当天日志，并作为state名称的一部分，这样每天会独立生成一个State

- 处理过期State：配置State的[TTL( time-to-live)](https://ci.apache.org/projects/flink/flink-docs-stable/dev/stream/state/state.html#state-time-to-live-ttl)



#### 注意点

- 默认情况下，过期的State只有被再次访问时才会被清理，因此最好方案是使用RocksDB作为状态后端，并使能(enbale)相应的[压缩过滤器](https://ci.apache.org/projects/flink/flink-docs-stable/dev/stream/state/state.html#cleanup-during-rocksdb-compaction)

- 因为维护用户id的MapState可能会很大，所以需要使用可以存储大State的[RocksDBStateBackend](https://ci.apache.org/projects/flink/flink-docs-release-1.8/ops/state/state_backends.html#the-rocksdbstatebackend)作为状态后端
- 使用`MapState<Stirng, Boolean>`而不是`ValueState<Set<String>>`，因为对ValueState的写入和访问是整个对象进行序列化和反序列化的，而MapState的写入和访问只需要序列化和反序列化操作某个key对应的值，参考[Stream Processing with Apache Flink](https://book.douban.com/subject/30152777/)书中第7节






