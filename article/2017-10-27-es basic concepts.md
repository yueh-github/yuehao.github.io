````---
title: ES Basic Concepts
layout: post
author: 王召辉
catalog: true
tags:
  - elasticsearch
---

## 近实时

ES是一个近实时搜索平台。这意味着，从索引文档到可搜索的时间，有一个微小的延迟(通常是1秒)。

## 集群

集群是一个或多个节点node(服务器)的集合，它们一起保存您的整个数据，并在所有节点上提供联合索引和搜索功能。集群由一个惟一的名称标识，默认是“elasticsearch”。这个名称很重要，因为如果节点设置为以其名称加入集群，那么节点是集群的一部分。

确保在不同的环境中不重用相同的集群名称，否则可能会导致连接错误集群的节点。例如，您可以使用``logging-dev``、``logging-stage``和``logging-prod``，用于开发、分段和生产集群。

注意在集群中只有一个节点是有效和可以的。此外，您还可能拥有多个独立的集群，每个集群都有自己独特的集群名称。

## 节点
节点是一个单独的服务器，它是集群的一部分，存储数据，并参与到集群的索引和搜索功能中。就像集群一样，节点由名称标识，默认情况下是一个随机的全局惟一标识符(UUID)，该标识符是在启动时分配给节点的。如果您不想要缺省值，可以定义任何您想要的节点名。这个名称对于管理目的非常重要，因为您需要确定您的网络中的哪些服务器对应于您的ES搜索集群中的哪些节点。

一个节点可以通过Cluster名子配置为加入一个指定的集群。默认情况下，每个节点都被设置加入一个名为``elasticsearch``的集群，这意味着如果您在网络上启动多个节点并假设它们能够发现彼此，那么它们将自动形成并连接一个名为``elasticsearch``的集群。

在单个集群中，您可以拥有任意数量的节点。此外，如果在您的网络上没有其他的ES节点，那么启动一个单节点将默认形成一个名为``elasticsearch``的新单节点集群。

## 索引

索引是一组具有类似特征的文档集合。例如，您可以有一个客户数据的索引、产品目录的另一个索引，以及订单数据的另一个索引。索引是由名称标识的(必须是小写的)，并且在对其中的文档执行索引、搜索、更新和删除操作时，使用这个名称来引用索引。

在单个集群中，您可以定义任意数量的索引。

## 类型

在一个索引中，您可以定义一个或多个类型。类型是您的索引的逻辑分类/分区，其语义完全取决于您。通常，类型被定义为具有一组公共字段的文档。例如，假设您运行一个博客平台，并将您的所有数据存储在一个索引中。在这个索引中，您可以为用户数据定义一种类型，另一种用于博客数据，而另一种类型用于评论数据。

## 文档

文档是可以被索引的基本信息单元。例如，您可以为单个客户提供文档，为单个产品提供另一个文档，而为单个订单提供另一个文档。这个文档是用JSON表示的，它是一种无处不在的互联网数据交换格式。

在索引/类型中，您可以存储任意数量的文档。请注意，尽管文档物理驻留在索引中，但实际上必须将文档索引/分配给索引中的类型。

## 分区/复制

索引可以存储大量数据，这些数据可以超过单个节点的硬件限制。例如，一个索引10亿文档占用1TB的磁盘空间的索引可能不适合单个节点的磁盘，或者可能太慢，不能单独地为单个节点提供搜索请求。为了解决这个问题，Elasticsearch提供了将你的索引细分为多个分片的能力。当您创建一个索引时，您可以简单地定义您想要的分片数量。每个分片本身都是一个功能齐全且独立的“索引”，可以在集群中的任何节点上托管。

分片的重要性有两个主要原因：

* 它允许你水平地分裂/缩放你的内容体积
* 它允许您跨分片(可能在多个节点上)分发和并行化操作，从而提高性能/吞吐量。

一个shard是如何分布的，以及它的文档如何被聚合到搜索请求中，这是由Elasticsearch完全管理的，对用户来说是透明的。在一个网络/云环境中，在任何时候都可以预期故障，这是非常有用的，并且强烈建议有一个故障转移机制，以防止shard/节点以某种方式脱机或消失。为此，ES允许你为你的index'shard制作一个或多个副本称为replica shards，或 replica。

复制很重要有两个主要原因:

* 它提供了高可用性，以防shard/node失败。因此，需要注意的是复制shard从未在与它复制的原始/主shard相同的节点上分配是非常重要的。
* 它允许您扩展您的搜索量/吞吐量，因为搜索可以在所有副本上并行执行。

总之，每个索引都可以分成多个shard。一个索引也可以被复制为零(意思是没有副本)或更多次。一旦复制，每个索引将拥有primary shard(复制的原始sahrd)和replica shard(primary shard的副本)。在创建索引时，可以在每个索引中定义碎片和副本的数量。创建索引之后，您可以随时更改副本的数量，但您无法更改shard数量。默认情况下，ES中的每个索引都被分配了5个primary shard和1个replica，这意味着如果您的集群中至少有两个节点，那么索引将有5个主碎片和另外5个复制碎片(1个完整副本)，每个索引总共有10个shard。

```
每一个ES shard 都是一个lucene index。在一个Lucene索引中，您可以拥有最多的文档数量。lucene-5843限制是2,147,483,519 (= Integer.MAX_VALUE - 128)个文档。你可以通过_cat/shards api来监控。
```````