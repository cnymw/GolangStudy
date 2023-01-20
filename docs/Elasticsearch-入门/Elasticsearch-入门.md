# Elasticsearch 入门

## Elasticsearch 是什么

Elasticsearch 是一个基于 Lucene 构建的开源，分布式，RESTful 接口全文搜索引擎。

## 如何扩展 Elasticsearch

- 垂直扩展/向上扩展：通过购置性能更强的服务器来完成。缺点是存在性能极限。
- 水平扩展/向外扩展：增加更多的服务器来完成。

## Elasticsearch 分布式

Elasticsearch 被设计为分布式的，它知道如何管理多个节点来完成扩展和实现高可用性。

## Elasticsearch 优点

1. 横向可扩展性：Elasticsearch 可以通过配置新增一台服务器，横向扩展集群。
2. 分片机制提供更好的分布性：同一个索引可以分成多个分片（sharding），提升处理效率。
3. 高可用：提供复制（replica）机制，一个分片可以设置多个复制，使得某台服务器在宕机情况下，集群仍旧可以照常运行，并把丢失的数据复制到其他可用节点上。
4. 使用简单：提供简单的命令和接口，来实现业务逻辑。