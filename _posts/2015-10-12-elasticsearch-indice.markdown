---
layout: post
title:  "Elasticsearch index"
date:   2015-10-12 13:21:50
author: white clover
categories: database
tags: database
---


## 创建index

创建一个3个分片， 3个副本的索引库

    PUT /blogs

```json
{
    "settings" : {
        "number_of_replicas": 3,
        "number_of_shards": 3
    }
}
```


`number_of_replicas` 副本数量, 副本数是可以动态修改的。 `number_of_shards` 分片数，分片数量是固定的，指定后就不能修改了。


设置副本位2：

    PUT /blogs

```json
{
    "settings" : {
        "number_of_replicas": 2
    }
}
```




## 文档原元素(Document Metadata)

文档不尽含有数据，而且含有原元素。

### index

文档储存所在的索引库

Where the document lives

### type

文档的类型

### id

文档的唯一标识符

### _index

存储相关文档的索引信息