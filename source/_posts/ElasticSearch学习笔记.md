---
title: ElasticSearch学习笔记
date: 2019-11-16 16:01:14
tags:
categories: elasticsearch
---

官网：https://www.elastic.co/cn/

<!--more-->

[TOC]

## 关键概念

索引(Index)

节点(Node)

分片(Sharding)

类型(Type)

映射(Mapping)

集群(Cluster)

## 基础

配置建议

- Xmx和Xms设置一样
- Xmx不要超过机器内存的50%

启动

```bash
❯ elasticsearch
❯ elasticsearch-plugin list
```

集群启动

```bash
❯ elasticsearch -E node.name=node0 -E cluster.name=test-cluster -E path.data=node0_data -d
❯ elasticsearch -E node.name=node1 -E cluster.name=test-cluster -E path.data=node1_data -d
❯ elasticsearch -E node.name=node2 -E cluster.name=test-cluster -E path.data=node2_data -d
❯ elasticsearch -E node.name=node3 -E cluster.name=test-cluster -E path.data=node3_data -d
```

查看集群：`http://localhost:9200/_cat/nodes`



元数据

- _index：文档所属的索引名
- _type：文档所属的类型名
- _id：文档唯一id
- _source：文档的原始JSON数据
- _all：整合所有字段内容（废除）
- _version：文档的版本信息
- _score：相关性打分 

```json
{
    "_index": ".kibana_1",
    "_type": "doc",
    "_id": "maps-telemetry:maps-telemetry",
    "_score": 1.0,
    "_source": {
        "maps-telemetry": {
            "mapsTotalCount": 0,
            "timeCaptured": "2019-11-16T08:11:32.275Z",
            "attributesPerMap": {
                "dataSourcesCount": {
                    "min": 0,
                    "max": 0,
                    "avg": 0
                },
                "layersCount": {
                    "min": 0,
                    "max": 0,
                    "avg": 0
                },
                "layerTypesCount": {},
                "emsVectorLayersCount": {}
            }
        },
        "type": "maps-telemetry",
        "updated_at": "2019-11-16T08:11:32.275Z"
    }
}
```

index

```json
"index": {
    "number_of_shards": "1",
    "auto_expand_replicas": "0-1",
    "provided_name": "kibana_sample_data_ecommerce",
    "creation_date": "1573891940016",
    "number_of_replicas": "1",
    "uuid": "y5YmkJBbSuOVF91ylFcV5A",
    "version": {
        "created": "6080399"
    }
}
```

关系型数据库与ES对比

| RDBMS  | ElasticSearch |
| ------ | ------------- |
| Table  | Index(Type)   |
| Row    | Document      |
| Column | Field         |
| Schema | Mapping       |
| SQL    | DSL           |

### 节点类型（详细）

- Master-eligible nodes：可以参加选主流程，第一个节点启动时，它会将自己选举成Master节点
- Master Node：主节点，可以修改集群的状态信息
- Data Node：可以保存数据的节点
- Coordinating Node：负责接收Client的请求，将请求分发到合适的节点，最后汇集结果
- Hot & Warm Node
- Machine Learning Node
- Tribe Node

## 节点类型（适合描述）

1. 主节点（Master node）：主要负责集群相关的操作内容，比如创建和删除索引，管理整个集群的变更，同时为了保证集群的稳定性，主节点不要作为数据节点。配置如下：

   ```yaml
   node.master: true
   node.data: false
   ```

2. 数据节点（Data node）：负责保存数据、执行数据相关操作，：CRUD、搜索、聚合等。其对CPU、内存、I/O要求较高。配置如下：

   ```yaml
   node.master: false
   node.data: true
   node.ingest: false
   ```

3. 预处理节点（Ingest node）：5.0版本引入的概念。预处理操作允许在索引文档之前，通过事先定义好的一系列的processors和pipeline，对数据进行某种转换、富化。processors和pipeline拦截bulk和index请求，操作后将文档传回bulk或index API。默认情况所有节点都启用ingest，若只想创建一个预处理节点，可以按照如下配置：

   ```yaml
   node.master: false
   node.data: false
   node.ingest: true
   ```

4. 协调节点（Coordinating node）：客户端的请求可以发到集群的任何节点，每个节点都清楚任意文档所处的位置，然后转发请求，收集数据并返回客户端，处理客户端请求的节点叫协调节点。协调节点收集完数据后会进行合并为单个结果。对结果集进行合并和排序可能会消耗很多CPU和内存资源。若只想创建一个协调节点，可按如下配置：

   ```yaml
   node.master: false
   node.data: false
   node.ingest: false
   ```

5. 部落节点（Tribe node）：在多个集群之间充当联合客户端。5.0倍协调节点取代。

### 分片

主分片：解决水平扩展

- 一个主分片是一个运行的Lucene的实例
- 主分片数在索引创建时指定，后续不允许修改，除非重建索引

副分片：解决数据高可用，是主分片的拷贝

- 数量可以动态调整  

查看集群的健康状况：`/_cluster/health`

## Analyzer(分析器)

### 步骤

Character Filters -> Tokenizers -> Token Filters

### Character Filters(字符过滤器)

- HTML Strip Character Filter：删除HTML元素
- Mapping Character Filter：替换指定的字符
- Pattern Replace Character Filter：基于正则表达式替换指定的字符

### Tokenizers(分词器)

- 转小写，删除（停用词），增加（同义词）

### Token Filters(分词后过滤器)

### 内置Analyzer

- Standard Analyzer：把句子分成单词，英文比较适合
- Simple Analyzer：单词转小写，基于非字母字符分词
- Whitespace Analyzer：基于空格分词
- Stop Analyzer：与Simple Analyzer类似，增加了停用词过滤
- Keyword Analyzer：输入和输出文本全部相同
- Pattern Analyzer：利用正则表达式对文本进行划分，单词转小写，支持停用词
- Language Analyzer：特定语言分词器，如英语、法语
- Fingerprint Analyzer：通过创建标记进行重复检测

### 第三方Analyzer

- elasticsearch-analysis-ik：使用最多的中文分词器

## 常用命令

```
es/elasticsearch-6.4.3-master/bin/elasticsearch-plugin install file:///data/cola/elasticsearch-analysis-pinyin-6.4.3.zip
```

