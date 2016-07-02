---
layout: post
title:  "水塘抽样"
date:   2015-09-10 16:23:56
author: white clover
categories: algorithm
tags: algorithm,spark
---


水塘抽样主要用来对未知大小数据集合或者数据非常大的集合中抽取随机的样本。

## 理论

假设有个N个元素的未知集合，取样品k个。

1. 当N=1时， 选取到N的概率为K/1 = K。
2. 当N=2， 因保证前面第一次抽样概览为K。 那么当此次抽样的概率的K/Q时， 保证了前面抽样不被替换概览为 K*（1-Q） = K/Q 得到新的均衡概览K/2.
3. 因此 第N次时，概率为K/N。

## Spark中的水塘抽样算法

当抽样样品大小大于输入集合大小时取全集。
否则使用概览选择替换保一次性遍历且每个输入元素的概率都为1/N。

```scala
private[spark] object SamplingUtils {

  /**
   * Reservoir sampling implementation that also returns the input size.
   *
   * @param input input size
   * @param k reservoir size
   * @param seed random seed
   * @return (samples, input size)
   */
  def reservoirSampleAndCount[T: ClassTag](
      input: Iterator[T],
      k: Int,
      seed: Long = Random.nextLong())
    : (Array[T], Long) = {
    val reservoir = new Array[T](k)
    // Put the first k elements in the reservoir.
    var i = 0
    while (i < k && input.hasNext) {
      val item = input.next()
      reservoir(i) = item
      i += 1
    }

    // If we have consumed all the elements, return them. Otherwise do the replacement.
    if (i < k) {
      // If input size < k, trim the array to return only an array of input size.
      val trimReservoir = new Array[T](i)
      System.arraycopy(reservoir, 0, trimReservoir, 0, i)
      (trimReservoir, i)
    } else {
      // If input size > k, continue the sampling process.
      var l = i.toLong
      val rand = new XORShiftRandom(seed)
      while (input.hasNext) {
        val item = input.next()
        val replacementIndex = (rand.nextDouble() * l).toLong
        if (replacementIndex < k) {
          reservoir(replacementIndex.toInt) = item
        }
        l += 1
      }
      (reservoir, l)
    }
  }
```

参考 [MurmurHash](https://en.wikipedia.org/wiki/MurmurHash)