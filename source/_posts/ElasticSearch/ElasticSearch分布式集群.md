---
title: ElasticSearch分布式集群
time: 2019-10-30
categories: ElasticSearch
tag: ElasticSearch
---
# ElasticSearch分布式集群简介
--- 
## 集群内部工作方式
Elasticsearch用于构建高可用和可扩展的系统。扩展的方式可以是购买更好的服务器(纵向扩展(vertical scale or scaling up))或者购买更多的服务器（横向扩展(horizontal scale or scaling out)）。

Elasticsearch虽然能从更强大的硬件中获得更好的性能，但是纵向扩展有它的局限性。真正的扩展应该是横向的，它通过增加节点来均摊负载和增加可靠性。
对于大多数数据库而言，横向扩展意味着你的程序将做非常大的改动才能利用这些新添加的设备。对比来说，Elasticsearch天生就是分布式的：它知道如何管理节点来提供高扩展和高可用。这意味着你的程序不需要关心这些。
在这章我们将探索如何创建你的集群(cluster)、节点(node)和分片(shards)，使其按照你的需求进行扩展，并保证在硬件故障时数据依旧安全。