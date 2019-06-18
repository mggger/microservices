# 服务注册与发现

目前配置中心服务有zookeeper， etcd等等 它们保证了高可用和强一致性， 在cap理论中保证ca， etcd背后的强一致性算法是raft, [我的实现](https://github.com/mggger/raft)

## Apache curator

Apache curator 是zookeeper的基于java写的客户端框架， 它有以下几个功能:
1. 链接管理 (重试机制)
2. 支持异步
3. 配置中心  （服务注册)
4. Watchers,  监听配置的变化， (服务发现)
5. typed models (轻松实现序列化 / 反序列化)
6. 领导选举
7. 分布式锁
8. 分布式计算器


详细的api解释可以看这篇[教程](https://www.baeldung.com/apache-curator)