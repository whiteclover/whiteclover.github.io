---
layout: post
title:  "Spark编译安装"
date:   2015-04-11 13:30:00
author: white clover
categories: ops
tags: ops
---

## 编译安装


    build/mvn -Pyarn -Phadoop-2.4 -Dhadoop.version=2.4.0 -DskipTests clean package

Other build examples can be found below.

Setting up Maven’s Memory Usage
You’ll need to configure Maven to use more memory than usual by setting MAVEN_OPTS. We recommend the following settings:

    export MAVEN_OPTS="-Xmx2g -XX:MaxPermSize=512M -XX:ReservedCodeCacheSize=512m"


For Apache Hadoop versions 1.x, Cloudera CDH “mr1” distributions, and other Hadoop versions without YARN, use:

```bash

     # Apache Hadoop 1.2.1
     mvn -Dhadoop.version=1.2.1 -DskipTests clean package

     # Cloudera CDH 4.2.0 with MapReduce v1
     mvn -Dhadoop.version=2.0.0-mr1-cdh4.2.0 -DskipTests clean package

     # Apache Hadoop 0.23.x
     mvn -Phadoop-0.23 -Dhadoop.version=0.23.7 -DskipTests clean package
```


## Building with SBT



Maven is the official recommendation for packaging Spark, and is the “build of reference”. But SBT is supported for day-to-day development since it can provide much faster iterative compilation. More advanced developers may wish to use SBT.

The SBT build is derived from the Maven POM files, and so the same Maven profiles and variables can be set to control the SBT build. For example:

    
    build/sbt -Pyarn -Phadoop-2.3 assembly

### Testing with SBT
Some of the tests require Spark to be packaged first, so always run build/sbt assembly the first time. The following is an example of a correct (build, test) sequence:

    build/sbt -Pyarn -Phadoop-2.3 -Phive -Phive-thriftserver assembly
    build/sbt -Pyarn -Phadoop-2.3 -Phive -Phive-thriftserver test