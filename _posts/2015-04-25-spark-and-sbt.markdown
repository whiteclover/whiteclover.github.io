---
layout: post
title:  "SBT&Spark使用"
date:   2015-04-25 09:43:50
author: white clover
categories: "big-data"
tags: spark
---


## Spark项目打包和发布


###SBT 常用命令

    actions – 显示对当前工程可用的命令
    update – 下载依赖
    compile – 编译代码
    test – 运行测试代码
    package – 创建一个可发布的jar包
    publish-local – 把构建出来的jar包安装到本地的ivy缓存
    publish – 把jar包发布到远程仓库（如果配置了的话)

## sbt-assembly

使用sbt-assembly打jar主要有两个版本0.11.2和0.13.0


### 对于0.11.2版

在project目录加 assembly.sbt 写入:

    $ cat project/assembly.sbt
    addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "0.11.2")

然后在项目根目录加入 build.sbt ，因为spark包含了scala库所以不需要再次包含。

```scala
    import AssemblyKeys._

    name := "Simple Project"
    version := "1.0"
    organization := "com.databricks"
    scalaVersion := "2.10.4"

    // 加入第三方依赖
    libraryDependencies ++= Seq(
            "org.apache.hadoop" % "hadoop-client" % "2.3.0" % "provided",
            "org.apache.spark" %% "spark-core" % "1.3.0" % "provided"
            // Third-party libraries
            "net.sf.jopt-simple" % "jopt-simple" % "4.3",
            "joda-time" % "joda-time" % "2.0"
    )

    // This statement includes the assembly plug-in capabilities
    assemblySettings
    // Configure JAR used with the assembly plug-in
    jarName in assembly := "my-project-assembly.jar"
    // A special option to exclude Scala itself form our assembly JAR, since Spark
    // already bundles Scala.
    assemblyOption in assembly :=
    (assemblyOption in assembly).value.copy(includeScala = false)
```

### 对于0.13.0版
在project目录加 assembly.sbt 写入:

    $ cat project/assembly.sbt
    addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "0.13.0")

然后在项目根目录加入 build.sbt ，因为spark包含了scala库所以不需要再次包含。

```scala
name := "hello"
version := "1.0"
scalaVersion := "2.10.4"

libraryDependencies ++= Seq(
    "org.apache.hadoop" % "hadoop-client" % "2.3.0" % "provided",
    "org.apache.spark" %% "spark-core" % "1.3.0" % "provided"
)

// Configure JAR used with the assembly plug-in
assemblyJarName  in assembly := "wordcount-assembly.jar"
// A special option to exclude Scala itself form our assembly JAR, since Spark
// already bundles Scala.
assemblyOption in assembly :=
(assemblyOption in assembly).value.copy(includeScala = false)
```

然后使用 sbt assembly就可以打jar包了。

    > assembly
    [info] Checking every *.class/*.jar file's SHA-1.
    [info] Merging files...
    [info] Assembly up to date: /usr/home/service/projects/hello/target/scala-2.11/wordcount-assembly.jar
    [success] Total time: 0 s, completed 2015-4-2 11:27:43

## 使用eclipse sbt 插件

在相关的sbt用户目录加入全局插件配置:

    // ~\.sbt\0.13\plugins global plugin dir
    addSbtPlugin("com.typesafe.sbteclipse" % "sbteclipse-plugin" % "3.0.0")

然后就可以使用 sbt eclipse构建初始化eclipse项目

    >> sbt eclipse
    [info] About to create Eclipse project files for your project(s).
    [info] Successfully created Eclipse project files for project(s):
    [info] app

## 依赖导出

在build.sbt中设置:

    retrieveManaged := true

## SBT打包Java应用

```scala
使用sbt管理java项目，build.sbt配置如下

// Project name (artifact name in Maven)
name := "$Project_name"

// orgnization name (e.g., the package name of the project)
organization := "$Organization"

version := "1.0-SNAPSHOT"

// project description
description := "Treasure Data Project"

// Enables publishing to maven repo
publishMavenStyle := true

// Do not append Scala versions to the generated artifacts
crossPaths := false

// This forbids including Scala related libraries into the dependency
autoScalaLibrary := false

// library dependencies. (orginization name) % (project name) % (version)
libraryDependencies ++= Seq(
   "org.apache.commons" % "commons-math3" % "3.1.1",
   "org.fluentd" % "fluent-logger" % "0.2.10",
   "org.mockito" % "mockito-core" % "1.9.5" % "test"  // Test-only dependency
)
```

## Maven打包Java应用

POM配置

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <!-- Information about your project -->
    <groupId>com.databricks</groupId>
    <artifactId>example-build</artifactId>
    <name>Simple Project</name>
    <packaging>jar</packaging>
    <version>1.0</version>
    <dependencies>
        <!-- Spark dependency -->
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.10</artifactId>
            <version>1.2.0</version>
            <scope>provided</scope>
        </dependency>
        <!-- Third-party library -->
        <dependency>
            <groupId>net.sf.jopt-simple</groupId>
            <artifactId>jopt-simple</artifactId>
            <version>4.3</version>
        </dependency>
        <!-- Third-party library -->
        <dependency>
            <groupId>joda-time</groupId>
            <artifactId>joda-time</artifactId>
            <version>2.0</version>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <!-- Maven shade plug-in that creates uber JARs -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.3</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                    </execution>
                    Packaging Your Code and Dependencies | 125
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

## 打包命令

    $ mvn package
    # In the target directory, we'll see an uber JAR and the original package JAR
    $ ls target/
    example-build-1.0.jar
    original-example-build-1.0.jar
    # Listing the uber JAR will reveal classes from dependency libraries
    $ jar tf target/example-build-1.0.jar
    ...
    joptsimple/HelpFormatter.class
    ...
    org/joda/time/tz/UTCProvider.class
    ...
    # An uber JAR can be passed directly to spark-submit
    $ /path/to/spark/bin/spark-submit --master local ... target/example-build-1.0.jar

## 部署提交

spark的app通过spark的spark-sumbit提交任务可以提交python，java，scala写的程序。


提交 Python 应用

    bin/spark-submit my_script.py

    Using spark-submit with various options
    # Submitting a Java application to Standalone cluster mode
    $ ./bin/spark-submit \
    --master spark://hostname:7077 \
    --deploy-mode cluster \
    --class com.databricks.examples.SparkExample \
    --name "Example Program" \
    --jars dep1.jar,dep2.jar,dep3.jar \
    --total-executor-cores 300 \
    --executor-memory 10g \
    myApp.jar "options" "to your application" "go here"
    # Submitting a Python application in YARN client mode
    $ export HADOP_CONF_DIR=/opt/hadoop/conf
    $ ./bin/spark-submit \
    --master yarn \
    --py-files somelib-1.2.egg,otherlib-4.4.zip,other-file.py \
    --deploy-mode client \
    --name "Example Program" \
    --queue exampleQueue \
    --num-executors 40 \
    --executor-memory 10g \
    my_script.py "options" "to your application" "go here"