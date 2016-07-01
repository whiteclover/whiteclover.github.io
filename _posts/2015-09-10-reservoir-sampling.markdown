---
layout: post
title:  "水塘抽样"
date:   2015-09-10 16:23:56
author: white clover
categories: algorithm
tags: algorithm,spark
---

## 水塘抽样


水塘抽样主要用来对未知大小数据集合或者数据非常大的集合中抽取随机的样本。

```python
from random import randrange

def reservoir_sampling(items, k):
    """ 
    Reservoir sampling algorithm for large sample space or unknow end list
    See <http://en.wikipedia.org/wiki/Reservoir_sampling> for detail>
    Type: ([a] * Int) -> [a]
    Prev constrain: k is positive and items at least of k items
    Post constrain: the length of return array is k
    """
    sample = items[0:k]

    for i in range(k, len(items)):
        j = randrange(1, i + 1)
        if j <= k:
            sample[j] = items[i]

    return sample
```