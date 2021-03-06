---
layout: post
title:  "Redis在数据分析平台的用例"
date:   2015-12-23 15:21:00
author: White clover
categories: database
tags:   redis
---

## Java中redis&kyro的缓存应用

在做服务化拆分过程中了，为了更好方便在各个应用需求中减少逻辑代码，而进一步方便拓展高可用。我们引入了thrift rpc+ zookeeper的分布式rpc服务。
过程中为了减少数据库的压力，加入redis做缓存层。用来缓存数据库查询结果和一些业务逻辑处理结果。开始使用的java原生的序列化发现字节空间比较浪费。
于是引入的[kyro](https://github.com/EsotericSoftware/kryo)层,kyrox相对`protobuf`更加方便的在java使用，但缺点是对其他语言的基本是不支持。

## 在实时曲线最近60分，60秒曲线的应用

问题前景可以描述如下： 

有 100 个产品，需要知道每个产品最近 60 秒的每秒请求负载数，并且可以看全国 400 个主要城市下以及华北，华东等聚合大区的负载指标。 

分产品和城市储存的话 400*100 = 4w 个 k/v 数据 每秒写入 4w 条数据，对于任何数据库都是非常有挑战的 （放弃） 

分产品存储，把同一时间戳各个大区和城市的指标聚成一条记录： 400 条每秒的记录量，数据库写入毫无压力。 

为什么放弃 list 或者 有序集合， 因为需要多余的机制保证删除过期的数据，而使用 redis 过期 k/v 存储策略 （ 120 秒过期）能更好完成这项需求实现。（这些数据也会实时落地插入到 mysql 等备份）。 

当展示最近 60 秒的数据时，我们发现可以在服务层缓存使用前 59 秒从 redis 已经拉取的数据。所以只要在服务后端加上一个内部缓存就可以加速数据的前端展示，因为后面的前端发请求带上时间戳，只要给时间戳后到现在时间几秒的数据（主要考虑到用户的网络延迟，大多数时间都是下一秒的数据）。最后当用户首次打开前端时，在网络正常的情况下，展示 60 秒数据也不到 0.03s 。 已经很好的满足用户的刷屏感。 


核心部分还是服务内部缓存，其实如果使用数据直接存 mysql 在比较少用户时同样可以达到每秒刷新展示性能，但因为考虑到高峰期负载问题，所以使用 redis.

应用层使用了java的spring，应用层缓存使用`guava`内存缓存，加速曲线展现。

### 一些优化细节

在redos储存开始我们使用了json方便解析，但是后面发现数据空间比较大，比较耗费内存，字节数过于大也会造成redis的访问和写入性能，`msgpack`的的序列比也不错相对json也能省40%左右的空间。但因为底层kafka那块的数据传输开始就是用了protobuf是引入`protobuf`协议减少存储内存空间使用，并加速解析，也保证了平台的协议一致性。

## 当内存数据库

在服务中大概接入了1.2kw终端设备，日活在50%左右。对于如此活跃的用户平台，对于用户的资料更新和访问时非常频繁了，为了减少对mysql的访问，于是我们直接把数据在redis内写了一份内存快照。加速用户资料的获取。

## 作为消息队列

在老架构的数据平台中， 服务单一--单例服务。那时前人技术挖的坑还是很为坑爹，过于臃肿的服务，处理对使用了http 数据访问api外，还集成一系列调度定时任务和常驻任务比如处理。因为是在线的实时的数据分析平台，对于日志的处理还是比较简单，logstash监听不同机器上的服务日志文件，发送但redis的list队列。简单完成的一个系统的整合。redis整理来说非常稳定，没有出现什么大故障，不过最基本的还是每秒日志量不到2w条而已，redis还能应付。瓶颈还是在redis队列上，redis单线程处理请求在每秒日志超过单进程redis的处理能力，这时拓展起来相当麻烦。教训就是请评估好数据的大小，而定redis做消息队列是否满足你的需求


## 做分布式锁和分布式定时任务调度

很想吐槽前人的留下的老系统，不引入zookeeper也摆。使用redis的做分布式锁机制，来保证服务master和slave机制，这样单一的master大多数情况还是分布式定时任务调度任务。master负责加载mysql储存的定时任务信息，并设置刷新轮询任务发布到redis的queue队列。worker从队列实时取出任务去处理。缺点是redis是单点队列，这类单一的队列很难保证每次任务执行都是成功的，同样也不能指定任务到那台机器上执行。每分钟的执行调度一天下理论上应该执行1440次，结果可能执行了1423次。对于每十分钟，每天每小时执行的任务，相对还是能比较好的保证执行率。