---
title: ElasticSearch学习笔记
date: 2019-11-16 16:01:14
tags:
categories: elasticsearch
---

官网：https://www.elastic.co/cn/

<!--more-->

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

### 节点类型

- Master-eligible nodes：可以参加选主流程，第一个节点启动时，它会将自己选举成Master节点
- Master Node：主节点，可以修改集群的状态信息
- Data Node：可以保存数据的节点
- Coordinating Node：负责接收Client的请求，将请求分发到合适的节点，最后汇集结果
- Hot & Warm Node
- Machine Learning Node
- Tribe Node

### 分片

主分片：解决水平扩展

- 一个主分片是一个运行的Lucene的实例
- 主分片数在索引创建时指定，后续不允许修改，除非重建索引

副分片：解决数据高可用，是主分片的拷贝

- 数量可以动态调整  

查看集群的健康状况：`/_cluster/health`