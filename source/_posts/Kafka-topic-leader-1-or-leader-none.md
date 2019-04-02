title: Kafka topic leader = -1 or leader = none
date: 2016-03-01 22:08:55
tags: Kafka学习
---
# Kafka topic 查询leader为-1 #
## 问题原因 ##
该问题的主要原因是在创建topic时，该partition的所有Replica都宕机了，Leader会自动设置为-1.

## 解决方法 ##
1. 查看Kafka配置文件 `/config/server.properties`
添加如下配置

```yaml
auto.leader.rebalance.enable=true
```

2. 重启kafka的broker：
进入kafka的bin目录

```SHELL
> nohup ./kafka-server-start.sh ../config/server.properties &
``` 