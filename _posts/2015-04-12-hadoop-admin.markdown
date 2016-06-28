---
layout: post
title:  "Hadoop管理"
date:   2015-04-12 13:30:00
author: white clover
categories: ops
tags: ops
---


## dfsadmin命令详解

dfsadmin是一个多任务的工具，我们可以使用它来获取HDFS的状态信息，以及在HDFS上执行的一系列管理操作。


使用方式：

     bin/hadoop dfsadmin -report


### 参数帮助


-report：查看文件系统的基本信息和统计信息。

#### -safeadmin enter | leave | get | wait

安全模式命令。安全模式是NameNode的一种状态，在这种状态下，NameNode不接受对名字空间的更改（只读）；不复制或删除块。NameNode在启动时自动进入安全模式，当配置块的最小百分数满足最小副本数的条件时，会自动离开安全模式。enter是进入，leave是离开。

#### -refreshNode

重新读取hosts和exclude文件，使新的节点或需要退出集群的节点能够被NameNode重新识别。这个命令在新增节点或注销节点时用到。

#### -finalizeUpgrade

终结HDFS的升级操作。DataNode删除前一个版本的工作目录，之后NameNode也这样做。

#### -upgradeProgress status | details | force

请求当前系统的升级状态 ， 升级状态的细节 ，强制升级操作

#### -metasave filename

保存NameNode的主要数据结构到hadoop.log.dir属性指定的目录下的<filename> 文件中。

#### -setQuota <quota><dirname>……<dirname>
 为每个目录<dirname>设定配额<quota>。目录配额是一个长整形整数，强制设定目录树下的名字个数。

#### -clrQuota <dirname>……<dirname>

为每个目录<dirname>清除配额设定。

#### -help 

这个命令就不多说了。

## fsck

fsck工具来检验HDFS中的文件是否正常可用。这个工具可以检测文件块是否在DataNode中丢失，是否低于或高于文件副本。


使用方法：
    
     hadoop fsck /user/admin/In/hello.txt -files -blocks -racks  (如果不加选项，则执行所有的指令)

### 命令详细


-files: 显示文件的文件名称、大小、块数量及是否可用；

-blocks: 显示每个块在文件中的信息，一个块用一行显示；

-racks: 展示了每个块所处的机架位置及DataNode的位置；


可能出现的输出：

Over-replicated blocks：一些文件块副本数超出了它所属文件的限定

Under-replicated blocks : 文件块数未达到所属文件要求的副本数量

Misreplicated blocks ：指出不满足块副本存储位置策略的块

Corrupt blocks : 所有的块副本全部出现问题

Missing replicas : 集群不存在副本的文件块