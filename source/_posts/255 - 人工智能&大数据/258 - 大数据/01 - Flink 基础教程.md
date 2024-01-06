---
title: Flink 基础教程
date: 2024-01-04
categories:
  - 人工智能&大数据
tags:
  - Flink
published: false
---
# 0 参考资料
## 0.1 学习资料

1. [2022 Flink 学习路线总结](https://www.zhihu.com/tardis/zm/art/105620944?source_id=1003)
2. [学习经验](https://www.baispace.cn/article/flink-source.html)

## 0.2 推荐书籍
- 《Flink大数据分析实战》
- Flink入门与实战
- Flink基础教程
- Flink原理、实战与性能优化

# 1 初识 Flink

## 1.1 大数据开发总体架构

- 总体架构图

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240104163945.png)

- 数据来源层

在大数据领域，数据的来源往往是关系型数据库、日志文件（用户在Web网站和手机App中浏览相关内容时，服务器端会生成大量的日志文件）、其他非结构化数据等。要想对这些大量的数据进行离线或实时分析，需要使用数据传输工具将其导入Hadoop平台或其他大数据集群中。

- 数据传输层
在大数据领域，数据的来源往往是关系型数据库、日志文件（用户在Web网站和手机App中浏览相关内容时，服务器端会生成大量的日志文件）、其他非结构化数据等。要想对这些大量的数据进行离线或实时分析，需要使用数据传输工具将其导入Hadoop平台或其他大数据集群中。

- 数据存储层
数据可以存储于分布式文件系统HDFS中，也可以存储于分布式数据库HBase中，而HBase的底层实际上还是将数据存储于HDFS中。此外，为了满足对大量数据的快速检索与统计，可以使用Elasticsearch作为全文检索引擎。

- 资源管理层
YARN是大数据开发中常用的资源管理器，它是一个通用资源（内存、CPU）管理系统，不仅可以集成于Hadoop中，也可以集成于Flink、Spark等其他大数据框架中。

- 数据计算层
MapReduce是Hadoop的核心组成部分，可以结合Hive通过SQL的方式进行数据的离线计算，当然也可以单独编写MapReduce应用程序进行计算。Storm用于进行数据的实时计算，可以非常容易地实时处理无限的流数据。Flink提供了离线计算库和实时计算库两种，离线计算库支持FlinkML（机器学习）、Gelly（图计算）、基于Table的关系操作，实时计算库支持CEP（复杂事件处理），同时也支持基于Table的关系操作。

- 任务调度层
Oozie是一个用于Hadoop平台的工作流调度引擎，可以使用工作流的方式对编写好的大数据任务进行调度。若任务不复杂，则可以使用Linux系统自带的Crontab定时任务进行调度。

- 业务模型层
对大量数据的处理结果最终需要通过可视化的方式进行展示。可以使用Java、PHP等处理业务逻辑，查询结果数据库，最终结合ECharts等前端可视化框架展示处理结果。

- Flink 在大数据开发架构中的位置
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240104164306.png)


## 1.2 Flink 主要组件

Flink是由多个组件构成的软件栈，整个软件栈可分为4层：
1. 存储层

Flink本身并没有提供分布式文件系统，因此Flink的分析大多依赖于HDFS，也可以从HBase和Amazon S3（亚马逊云存储服务）等持久层读取数据。

2. 调度层

Flink自带一个简易的资源调度器，称为独立调度器(Standalone)。若集群中没有任何资源管理器，则可以使用自带的独立调度器。当然，Flink也支持在其他的集群管理器上运行，包括Hadoop YARN、Apache Mesos等。

3. 计算层
Flink自带一个简易的资源调度器，称为独立调度器(Standalone)。若集群中没有任何资源管理器，则可以使用自带的独立调度器。当然，Flink也支持在其他的集群管理器上运行，包括Hadoop YARN、Apache Mesos等。

4. 工具层
在Flink Runtime的基础上，Flink提供了面向流处理(DataStream API)和批处理(DataSet API)的不同计算接口，并在此接口上抽象出了不同的应用类型组件库，例如基于流处理的CEP（复杂事件处理库）、Table&SQL（结构化表处理库）和基于批处理的Gelly（图计算库）、FlinkML（机器学习库）、Table&SQL（结构化表处理库）。

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240104164547.png)


## 1.3 Flink 编程接口

Flink提供了丰富的数据处理接口，并将接口抽象成4层，由下向上分别为Stateful Stream Processing API、DataStream/DataSet API、Table API以及SQL API，开发者可以根据具体需求选择任意一层接口进行应用开发。

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240104165159.png)

### 1.3.1 Stateful Stream Processing API

Flink中处理有状态流最底层的接口，它通过Process Function（低阶API，Flink提供的最具表达力的底层接口）嵌入DataStream API中，允许用户自由地处理一个或多个流中的事件，并使用一致的容错状态。此外，用户可以注册事件时间和处理时间回调，从而允许程序实现复杂的计算。用户可以通过这个API接口操作状态、时间等底层数据。

### 1.3.2 DataStream/DataSet API

大多数应用程序不需要上述低级抽象，而是针对核心API进行编程的，例如DataStream API和DataSet API。DataStream API用于处理无界数据集，即流处理；DataSet API用于处理有界数据集，即批处理。这两种API都提供了用于数据处理的通用操作，例如各种形式的转换、连接、聚合等。

低级Process Function与DataStream API集成在一起，从而使得仅对某些操作进行低级抽象成为可能。DataSet API在有限的数据集上提供了其他原语，例如循环／迭代。

### 1.3.3 Table API
Table API作为批处理和流处理统一的关系型API，即查询在无界实时流或有界批数据集上以相同的语义执行，并产生相同的结果。Flink中的Table API通常用于简化数据分析、数据流水线和ETL应用程序的定义。

Table API构建在DataStream/DataSet API之上，提供了大量编程接口，例如GroupByKey、Join等操作，是批处理和流处理统一的关系型API，使用起来更加简洁。使用Table API允许在表与DataStream/DataSet数据集之间无缝切换，并且可以将Table API与DataStream/DataSet API混合使用。

Table API的原理是将内存中的DataStream/DataSet数据集在原有的基础上增加Schema信息，将数据类型统一抽象成表结构，然后通过Table API提供的接口处理对应的数据集，从而简化数据分析。

此外，Table API程序还会通过优化规则在数据处理过程中对处理逻辑进行大量优化。

### 1.3.4 SQL API
Flink提供的最高级别的抽象是SQL API。这种抽象在语义和表达方式上均类似于Table API，但是将程序表示为SQL查询表达式。SQL抽象与Table API紧密交互，并且可以对Table API中定义的表执行SQL查询。


## 1.4 Flink 程序结构

一个Flink应用程序由3部分构成，或者说将Flink的操作算子可以分成3部分，分别为Source、Transformation和Sink：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240104170129.png)


- Source：数据源部分。负责读取指定存储介质中的数据，转为分布式数据流或数据集，例如readTextFile()、socketTextStream()等算子。Flink在流处理和批处理上的Source主要有4种：基于本地集合、基于文件、基于网络套接字Socket和自定义Source。
- Transformation：数据转换部分。负责对一个或多个数据流或数据集进行各种转换操作，并产生一个或多个输出数据流或数据集，例如map()、flatMap()、keyBy()等算子。
- Sink：数据输出部分。负责将转换后的结果数据发送到HDFS、文本文件、MySQL、Elasticsearch等目的地，例如writeAsText()算子。图1-15描述了一个Flink应用程序的3部分。

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240104170301.png)


# 2 Flink 运行架构和原理

## 2.1 Flink 运行时架构

### 2.1.1 Flink Standalone架构

Flink Standalone模式为经典的主从(Master/Slave)架构，资源调度是Flink自己实现的。集群启动后，主节点上会启动一个JobManager进程，主节点称为JobManager节点；各个从节点上会启动一个TaskManager进程，从节点称为TaskManager节点。从Flink 1.6版本开始，将主节点上的进程名称改为了StandaloneSessionClusterEntrypoint，从节点的进程名称改为了，在这里为了方便使用，仍然沿用之前版本的称呼，即JobManager和TaskManager。

Client接收到Flink应用程序后，将作业提交给JobManager。JobManager要做的第一件事就是分配Task（任务）所需的资源。完成资源分配后，Task将被JobManager提交给相应的TaskManager，TaskManager会启动线程开始执行。在执行过程中，TaskManager会持续向JobManager汇报状态信息，例如开始执行、进行中或完成等状态。作业执行完成后，结果将通过JobManager发送给Client。


![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240104170507.png)

#### 2.1.1.1 Task
Flink中的每一个操作算子称为一个Task（任务）。每个Task在一个JVM线程中执行。多个Task可以在同一个JVM进程中共享TCP连接（通过多路复用技术）和心跳信息。它们还可能共享数据集和数据结构，从而降低每个Task的开销。

#### 2.1.1.2 Task Slot

TaskManager为了控制执行的Task数量，将计算资源（内存）划分为多个Task Slot（任务槽），每个Task Slot代表TaskManager的一份固定内存资源，Task则在Task Slot中执行。

Task Slot是一个静态概念，TaskManager在启动的时候就设置好了Task Slot的数量。Task Slot的数量决定了TaskManager具有的并发执行能力。因此，建议将Task Slot的数量设置为节点CPU的核心数，以最大化利用资源，提高计算效率。Task Slot的数量设置可以修改Flink配置文件flink-conf.yaml中的taskmanager.numberOfTaskSlots属性值，默认为1。

### 2.1.2 YARN 集群架构



## 2.2 Flink 任务调度原理

### 2.2.1 任务链
Flink中的每一个操作算子称为一个Task（任务），算子的每个具体实例则称为SubTask（子任务），SubTask是Flink中最小的处理单元，多个SubTask可能在不同的机器上执行。一个TaskManager进程包含一个或多个执行线程，用于执行SubTask。TaskManager中的一个Task Slot对应一个执行线程，一个执行线程可以执行一个或多个SubTask。
由于每个SubTask只能在一个线程中执行，为了能够减少线程间切换和缓冲的开销，在降低延迟的同时提高整体吞吐量，Flink可以将多个连续的SubTask链接成一个Task在一个线程中执行。这种将多个SubTask连在一起的方式称为任务链。如图：一个Source类算子的SubTask和一个map()算子的SubTask连在了一起，组成了任务链。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240104172145.png)

> 任务链的好处是，同一任务链内的SubTask可以彼此直接传递数据，而无须通过序列化或Flink的网络栈。

### 2.2.2 并行度
为了充分利用计算资源，提高计算效率，可以增加算子的实例数（SubTask数量）。一个特定算子的SubTask数量称为该算子的并行度，且任意两个算子的并行度之间是独立的，不同算子可能拥有不同的并行度。例如，将Source算子、map()算子、keyby()/window()/apply()算子的并行度设置为2，Sink算子的并行度设置为1，运行效果如图：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240104172326.png)

由于一个Task Slot对应一个执行线程，因此并行度为2的算子的SubTask将被分配到不同的Task Slot中执行。
假设一个作业图(JobGraph)有A、B、C、D、E五个算子，其中A、B、D的并行度为4，C、E的并行度为2，该作业在TaskManager中的详细数据流程可能如图：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240104172406.png)

Flink中并行度的设置有4种级别：算子级别、执行环境(Execution Environment)级别、客户端（命令行）级别、系统级别。

1. 算子级别
每个算子、Source和Sink都可以通过调用setParallelism()方法指定其并行度。例如以下代码设置flatMap()算子的并行度为2：
```scala
data.flatMap(_.split(" ")).setParallelism(2)
```

2. 执行环境级别
调用执行环境对象的setParallelism()方法可以指定Flink应用程序中所有算子的默认并行度，代码如下：
```scala
val env=ExecutionEnvironment.getExecutionEnvironment 
env.setParallelism(2)
```

3. 客户端（命令行）级别
在向集群提交Flink应用程序时使用-p选项可以指定并行度。例如以下提交命令：
```bash
bin/flink run -p 2 WordCount.jar
```

4. 系统级别
影响所有运行环境的系统级别的默认并行度可以在配置文件flink-conf.yaml中的parallelism.default属性中指定，默认为1。

> 4种并行度级别的作用顺序为：算子级别>执行环境级别>客户端级别>系统级别。

### 2.2.3 共享Task Slot

默认情况下，Flink允许SubTask之间共享Task Slot，即使它们是不同Task（算子）的SubTask，只要它们来自同一个作业(Job)即可。在没有共享Task Slot的情况下，简单的SubTask（source()、map()等）将会占用和复杂的SubTask（keyBy()、window()等）一样多的资源，通过共享Task Slot可以充分利用Task Slot的资源，同时确保繁重的SubTask在TaskManager之间公平地获取资源。例如，将图2-7中的算子并行度从2增加到6，并行效果如图：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240104172833.png)

Flink集群的Task Slot数量最好与作业中使用的最高并行度一致，这样不需要计算作业总共包含多少个具有不同并行度的Task。

### 2.2.4 数据流

一个Flink应用程序会被映射成逻辑数据流(Dataflow)，而Dataflow都是以一个或多个Source开始、以一个或多个Sink结束的，且始终包括Source、Transformation、Sink三部分。Dataflow描述了数据如何在不同算子之间流动，将这些算子用带方向的直线连接起来会形成一个关于计算路径的有向无环图，称为DAG（Directed Acyclic Graph，有向无环图）或Dataflow图。
假设一个Flink应用程序在读取数据后先对数据进行了map()操作，然后进行了keyBy()/window()/apply()操作，最后将计算结果输出到了指定的文件中，则该程序的Dataflow图如图：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240104173332.png)

假设该程序的Source、map()、keyBy()/window()/apply()算子的并行度为2，Sink算子的并行度为1，则该程序的逻辑数据流图、物理（并行）数据流图和Flink优化后的数据流图如图：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240104173358.png)


Flink应用程序在执行时，为了降低线程开销，会将多个SubTask连接在一起组成任务链，在一个线程中运行。对于上图的物理（并行）数据流来说，Flink执行时会对其进行优化，将Source[1]和map()[1]、Source[2]和map()[2]分别连接成一个任务，这是因为Source和map()之间采用了一对一的直连模式，而且没有任何的重分区（重分区往往发生在聚合阶段，类似于Spark的Shuffle），它们之间可以直接通过缓存进行数据传递，而不需要通过网络或序列化（如果不使用任务链，Source和map()可能在不同的机器上，它们之间的数据传递就需要通过网络）。这种优化在很大程度上提升了Flink的执行效率。

### 2.2.5 执行图

Flink应用程序执行时会根据数据流生成多种图，每种图对应了作业的不同阶段，根据不同图的生成顺序，主要分为4层：StreamGraph→JobGraph→ExecutionGraph→物理执行图。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240104173706.png)

- StreamGraph：流图。使用DataStream API编写的应用程序生成的最初的图代表程序的拓扑结构，描述了程序的执行逻辑。StreamGraph在Flink客户端中生成，在客户端应用程序最后调用execute()方法时触发StreamGraph的构建。
- JobGraph：作业图。所有高级别API都需要转换为JobGraph。StreamGraph经过优化（例如任务链）后生成了JobGraph，以提高执行效率。StreamGraph和JobGraph都是在本地客户端生成的数据结构，而JobGraph需要被提交给JobManager进行解析。
- ExecutionGraph：执行图。JobManager对JobGraph进行解析后生成的并行化执行图是调度层最核心的数据结构。它包含对每个中间数据集或数据流、每个并行任务以及它们之间的通信的描述。
- 物理执行图：JobManager根据ExecutionGraph对作业进行调度后，在各个TaskManager上部署Task后形成的“图”。物理执行图并不是一个具体的数据结构，而是各个Task分布在不同的节点上所形成的物理上的关系表示。


# 3 Flink DataStream API
DataStream API是Flink的核心层API。一个Flink程序，其实就是对DataStream的各种转换。具体来说，代码基本上都由以下几部分构成：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240104174200.png)

## 3.1 执行模式

DataStream API支持不同的运行时执行模式，可以根据用例的要求和作业的特征从中进行选择。DataStream API比较“经典”的执行行为称为“流”执行模式，主要用于需要连续增量处理并无限期保持在线的无限作业。

可以通过execution.runtime-mode属性来配置执行模式。其有3种可能的值：
- STREAMING：典型的DataStream执行模式（默认）。
- BATCH：在DataStream API上以批处理方式执行。
- AUTOMATIC：让系统根据数据源的有界性来决定。

**Flink Task任务作业流程：**

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240104174500.png)

## 3.2 作业流程
在一个作业提交前，JobManager和TaskManager等进程需要先被启动。可以在Flink安装目录中执行bin/start-cluster.sh命令来启动这些进程。JobManager和TaskManager被启动后，TaskManager需要将自己注册给JobManager中的ResourceManager（资源注册）。这个初始化和资源注册过程发生在单个作业提交前。

Flink作业的具体执行流程如下：

1. 用户编写应用程序代码，并通过Flink客户端提交作业。程序一般为Java或Scala语言，调用Flink API构建逻辑数据流图，然后转为作业图JobGraph，并附加到StreamExecutionEnvironment中。代码和相关配置文件被编译打包，被提交到JobManager的Dispatcher，形成一个应用作业。
2. Dispatcher（JobManager的一个组件）接收到这个作业，启动JobManager，JobManager负责本次作业的各项协调工作。
3. 接下来JobManager向ResourceManager申请本次作业所需的资源。
4. JobManager将用户作业中的作业图JobGraph转化为并行化的物理执行图，对作业并行处理并将其子任务分发部署到多个TaskManager上执行。每个作业的并行子任务将在Task Slot中执行。至此，一个Flink作业就开始执行了。
5. TaskManager在执行计算任务的过程中可能会与其他TaskManager交换数据，会使用相应的数据交换策略。同时，TaskManager也会将一些任务状态信息反馈给JobManager，这些信息包括任务启动、运行或终止的状态、快照的元数据等。

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240104174753.png)

## 3.3 Source 数据源

DataStream API中直接提供了对一些基本数据源的支持，例如文件系统、Socket连接等，也提供了非常丰富的高级数据源连接器(Connector)，例如Kafka Connector、Elasticsearch Connector等。用户也可以实现自定义Connector数据源，以便使Flink能够与其他外部系统进行数据交互。

**数据源相关类的继承关系：**
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240104175405.png)


从Flink1.12开始，主要使用流批统一的新Source架构：
```java
DataStreamSource<String> stream = env.fromSource(…)
```

**实现一个自定义数据源的示例：**
```java
package com.atguigu.source;  
  
import org.apache.flink.api.common.eventtime.WatermarkStrategy;  
import org.apache.flink.api.common.typeinfo.Types;  
import org.apache.flink.api.connector.source.util.ratelimit.RateLimiterStrategy;  
import org.apache.flink.connector.datagen.source.DataGeneratorSource;  
import org.apache.flink.connector.datagen.source.GeneratorFunction;  
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;  

public class DataGeneratorDemo {  
    public static void main(String[] args) throws Exception {  
    
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
  
        // 如果有n个并行度， 最大值设为a  
        // 将数值 均分成 n份，  a/n ,比如，最大100，并行度2，每个并行度生成50个  
        // 其中一个是 0-49，另一个50-99  
        env.setParallelism(2);  
  
        /**  
         * 数据生成器Source，四个参数：  
         *     第一个： GeneratorFunction接口，需要实现， 重写map方法， 输入类型固定是Long  
         *     第二个： long类型， 自动生成的数字序列（从0自增）的最大值(小于)，达到这个值就停止了  
         *     第三个： 限速策略， 比如 每秒生成几条数据  
         *     第四个： 返回的类型  
         */  
        DataGeneratorSource<String> dataGeneratorSource = new DataGeneratorSource<>(  
                new GeneratorFunction<Long, String>() {  
                    @Override  
                    public String map(Long value) throws Exception {  
                        return "Number:" + value;  
                    }  
                },  
                100,  
                RateLimiterStrategy.perSecond(1),  
                Types.STRING  
        );  
  
        env  
                .fromSource(dataGeneratorSource, WatermarkStrategy.noWatermarks(), "data-generator")  
                .print();  
  
  
        env.execute();  
    }  
}
```
以上程序每秒自动产生随机数，直到达到100为止。

## 3.4 Transformation 数据转换

在Flink中，Transformation（转换）算子就是将一个或多个DataStream转换为新的DataStream，可以将多个转换组合成复杂的数据流(Dataflow)拓扑。
Transformation应用于一个或多个数据流或数据集，并产生一个或多个输出数据流或数据集。Transformation可能会在每个记录的基础上更改数据流或数据集，但也可以只更改其分区或执行聚合。

### 3.4.1 基本转换算子

#### 3.4.1.1 map(func)

map()算子接收一个函数作为参数，并把这个函数应用于DataStream的每个元素，最后将函数的返回结果作为结果DataStream中对应元素的值，即将DataStream的每个元素转换成新的元素。

**map()算子运行过程：**

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240104175633.png)

只需要基于DataStream调用map()方法就可以进行转换处理。方法需要传入的参数是接口MapFunction的实现；返回值类型还是DataStream，不过泛型（流中的元素类型）可能改变。

**示例代码：**
```java
package com.atguigu.transfrom;  
  
import com.atguigu.bean.WaterSensor;  
import com.atguigu.functions.MapFunctionImpl;  
import org.apache.flink.api.common.functions.MapFunction;  
import org.apache.flink.streaming.api.datastream.DataStream;  
import org.apache.flink.streaming.api.datastream.DataStreamSource;  
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;  
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;  
  
public class MapDemo {  
    public static void main(String[] args) throws Exception {  
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
        env.setParallelism(1);  
  
        DataStreamSource<WaterSensor> sensorDS = env.fromElements(  
                new WaterSensor("s1", 1L, 1),  
                new WaterSensor("s2", 2L, 2),  
                new WaterSensor("s3", 3L, 3)  
        );  
  
        // TODO map算子： 一进一出  
  
        // TODO 方式一： 匿名实现类  
//        SingleOutputStreamOperator<String> map = sensorDS.map(new MapFunction<WaterSensor, String>() {  
//            @Override  
//            public String map(WaterSensor value) throws Exception {  
//                return value.getId();  
//            }  
//        });  
  
        // TODO 方式二： lambda表达式  
//        SingleOutputStreamOperator<String> map = sensorDS.map(sensor -> sensor.getId());  
  
        // TODO 方式三： 定义一个类来实现MapFunction  
//        SingleOutputStreamOperator<String> map = sensorDS.map(new MyMapFunction());  
        SingleOutputStreamOperator<String> map = sensorDS.map(new MapFunctionImpl());  
        map.print();  
  
  
        env.execute();  
    }  
  
    public static class MyMapFunction implements MapFunction<WaterSensor,String>{  
  
        @Override  
        public String map(WaterSensor value) throws Exception {  
            return value.getId();  
        }  
    }  
}
```

**输出：**
```bash
s1
s2
s3
```
MapFunction实现类的泛型类型，与输入数据类型和输出数据的类型有关。在实现MapFunction接口的时候，需要指定两个泛型，分别是输入事件和输出事件的类型，还需要重写一个map()方法，定义从一个输入事件转换为另一个输出事件的具体逻辑。
#### 3.4.1.2 flatMap(func)

flatMap操作又称为扁平映射，主要是将数据流中的整体（一般是集合类型）拆分成一个一个的个体使用。消费一个元素，可以产生0到多个元素。flatMap可以认为是“扁平化”（flatten）和“映射”（map）两步操作的结合，也就是先按照某种规则对数据进行打散拆分，再对拆分后的元素做转换处理。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240106205452.png)

同map一样，flatMap也可以使用Lambda表达式或者FlatMapFunction接口实现类的方式来进行传参，返回值类型取决于所传参数的具体逻辑，可以与原数据流相同，也可以不同。
```java
package com.atguigu.transfrom;  
  
import com.atguigu.bean.WaterSensor;  
import org.apache.flink.api.common.functions.FilterFunction;  
import org.apache.flink.api.common.functions.FlatMapFunction;  
import org.apache.flink.streaming.api.datastream.DataStreamSource;  
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;  
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;  
import org.apache.flink.util.Collector;  
  
/**  
 * TODO 如果输入的数据是sensor_1，只打印vc；如果输入的数据是sensor_2，既打印ts又打印vc  
 *  
 * @author cjp  
 * @version 1.0  
 */public class FlatmapDemo {  
    public static void main(String[] args) throws Exception {  
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
        env.setParallelism(1);  
  
        DataStreamSource<WaterSensor> sensorDS = env.fromElements(  
                new WaterSensor("s1", 1L, 1),  
                new WaterSensor("s1", 11L, 11),  
                new WaterSensor("s2", 2L, 2),  
                new WaterSensor("s3", 3L, 3)  
        );  
  
  
        /**  
         * TODO flatmap： 一进多出（包含0出）  
         *      对于s1的数据，一进一出  
         *      对于s2的数据，一进2出  
         *      对于s3的数据，一进0出（类似于过滤的效果）  
         *  
         *    map怎么控制一进一出：  
         *      =》 使用 return  
         *         *    flatmap怎么控制的一进多出  
         *      =》 通过 Collector来输出， 调用几次就输出几条  
         *  
         *         */        SingleOutputStreamOperator<String> flatmap = sensorDS.flatMap(new FlatMapFunction<WaterSensor, String>() {  
            @Override  
            public void flatMap(WaterSensor value, Collector<String> out) throws Exception {  
                if ("s1".equals(value.getId())) {  
                    // 如果是 s1，输出 vc                    out.collect(value.getVc().toString());  
                } else if ("s2".equals(value.getId())) {  
                    // 如果是 s2，分别输出ts和vc  
                    out.collect(value.getTs().toString());  
                    out.collect(value.getVc().toString());  
                }  
            }  
        });  
  
        flatmap.print();  
  
  
        env.execute();  
    }  
  
  
}
```
如果输入的数据是sensor_1，只打印vc；如果输入的数据是sensor_2，既打印ts又打印vc。

#### 3.4.1.3 filter
进行filter转换之后的新数据流的数据类型与原数据流是相同的。filter转换需要传入的参数需要实现FilterFunction接口，而FilterFunction内要实现filter()方法，就相当于一个返回布尔类型的条件表达式。

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240106205231.png)

```java
package com.atguigu.transfrom;  
  
import com.atguigu.bean.WaterSensor;  
import com.atguigu.functions.FilterFunctionImpl;  
import com.atguigu.functions.MapFunctionImpl;  
import org.apache.flink.api.common.functions.FilterFunction;  
import org.apache.flink.api.common.functions.MapFunction;  
import org.apache.flink.streaming.api.datastream.DataStreamSource;  
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;  
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;  
  
/**  
 * TODO  
 *  
 * @author cjp  
 * @version 1.0  
 */public class FilterDemo {  
    public static void main(String[] args) throws Exception {  
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
        env.setParallelism(1);  
  
        DataStreamSource<WaterSensor> sensorDS = env.fromElements(  
                new WaterSensor("s1", 1L, 1),  
                new WaterSensor("s1", 11L, 11),  
                new WaterSensor("s2", 2L, 2),  
                new WaterSensor("s3", 3L, 3)  
        );  
  
        // TODO filter： true保留，false过滤掉  
//        SingleOutputStreamOperator<WaterSensor> filter = sensorDS.filter(new FilterFunction<WaterSensor>() {  
//            @Override  
//            public boolean filter(WaterSensor value) throws Exception {  
//                return "s1".equals(value.getId());  
//            }  
//        });  
  
//        SingleOutputStreamOperator<WaterSensor> filter = sensorDS.filter(new FilterFunctionImpl("s1"));  
        SingleOutputStreamOperator<WaterSensor> filter = sensorDS.filter(waterSensor -> "s1".equals(waterSensor.id));  
  
        filter.print();  
  
  
        env.execute();  
    }  
  
  
}
```

### 3.4.2 聚合算子

#### 3.4.2.1 简单聚合（sum/min/max/minBy/maxBy）
使用聚合算子前都需要使用keyBy算子，把数据从DataStream转换为KeyedStream。KeyedStream可以认为是“分区流”或者“键控流”，它是对DataStream按照key的一个逻辑分区，所以泛型有两个类型：除去当前流中的元素类型外，还需要指定key的类型。
- sum()：在输入流上，对指定的字段做叠加求和的操作。
- min()：在输入流上，对指定的字段求最小值。
- max()：在输入流上，对指定的字段求最大值。
- minBy()：与min()类似，在输入流上针对指定字段求最小值。不同的是，min()只计算指定字段的最小值，其他字段会保留最初第一个数据的值；而minBy()则会返回包含字段最小值的整条数据。
- maxBy()：与max()类似，在输入流上针对指定字段求最大值。两者区别与min()/minBy()完全一致。

简单聚合算子返回的，同样是一个SingleOutputStreamOperator，也就是从KeyedStream又转换成了常规的DataStream。所以可以这样理解：keyBy和聚合是成对出现的，先分区、后聚合，得到的依然是一个DataStream。而且经过简单聚合之后的数据流，元素的数据类型保持不变。

#### 3.4.2.2 reduce

reduce可以对已有的数据进行归约处理，把每一个新输入的数据和当前已经归约出来的值，再做一个聚合计算。

```java
package com.atguigu.aggreagte;  
  
import com.atguigu.bean.WaterSensor;  
import org.apache.flink.api.common.functions.ReduceFunction;  
import org.apache.flink.api.java.functions.KeySelector;  
import org.apache.flink.streaming.api.datastream.DataStreamSource;  
import org.apache.flink.streaming.api.datastream.KeyedStream;  
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;  
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;  
  
 public class ReduceDemo {  
    public static void main(String[] args) throws Exception {  
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
        env.setParallelism(1);  
  
  
        DataStreamSource<WaterSensor> sensorDS = env.fromElements(  
                new WaterSensor("s1", 1L, 1),  
                new WaterSensor("s1", 11L, 11),  
                new WaterSensor("s1", 21L, 21),  
                new WaterSensor("s2", 2L, 2),  
                new WaterSensor("s3", 3L, 3)  
        );  
  
  
        KeyedStream<WaterSensor, String> sensorKS = sensorDS  
                .keyBy(new KeySelector<WaterSensor, String>() {  
                    @Override  
                    public String getKey(WaterSensor value) throws Exception {  
                        return value.getId();  
                    }  
                });  
  
        /**  
         * TODO reduce:  
         * 1、keyby之后调用  
         * 2、输入类型 = 输出类型，类型不能变  
         * 3、每个key的第一条数据来的时候，不会执行reduce方法，存起来，直接输出  
         * 4、reduce方法中的两个参数  
         *     value1： 之前的计算结果，存状态  
         *     value2： 现在来的数据  
         */  
        SingleOutputStreamOperator<WaterSensor> reduce = sensorKS.reduce(new ReduceFunction<WaterSensor>() {  
            @Override  
            public WaterSensor reduce(WaterSensor value1, WaterSensor value2) throws Exception {  
                System.out.println("value1=" + value1);  
                System.out.println("value2=" + value2);  
                return new WaterSensor(value1.id, value2.ts, value1.vc + value2.vc);  
            }  
        });  
  
        reduce.print();  
        env.execute();  
    }  
}
```
**输出：**
```bash
WaterSensor{id='s1', ts=1, vc=1}
value1=WaterSensor{id='s1', ts=1, vc=1}
value2=WaterSensor{id='s1', ts=11, vc=11}
WaterSensor{id='s1', ts=11, vc=12}
value1=WaterSensor{id='s1', ts=11, vc=12}
value2=WaterSensor{id='s1', ts=21, vc=21}
WaterSensor{id='s1', ts=21, vc=33}
WaterSensor{id='s2', ts=2, vc=2}
WaterSensor{id='s3', ts=3, vc=3}
```

在输入第一条数据时不会触发reduce，后边每次数据到来会触发数据聚合，value1是当前聚合结果，value2是这次新的数据。

### 3.4.3 用户自定义函数
#### 3.4.3.1 函数类

Flink暴露了所有UDF函数的接口，具体实现方式为接口或者抽象类，例如MapFunction、FilterFunction、ReduceFunction等。所以用户可以自定义一个函数类，实现对应的接口。
上边的示例，都是实现了对应UDF函数的类，可以是匿名类，Lambda，继承了接口或抽象类的自定义类。
#### 3.4.3.2 富函数类
“富函数类”也是DataStream API提供的一个函数类的接口，所有的Flink函数类都有其Rich版本。富函数类一般是以抽象类的形式出现的。例如：RichMapFunction、RichFilterFunction、RichReduceFunction等。

与常规函数类的不同主要在于，富函数类可以获取运行环境的上下文，并拥有一些生命周期方法，所以可以实现更复杂的功能。

Rich Function有生命周期的概念。典型的生命周期方法有：
- open()方法，是Rich Function的初始化方法，也就是会开启一个算子的生命周期。当一个算子的实际工作方法例如map()或者filter()方法被调用之前，open()会首先被调用。
- close()方法，是生命周期中的最后一个调用的方法，类似于结束方法。一般用来做一些清理工作。

**示例代码：**
```java
package com.atguigu.transfrom;  
  
import com.atguigu.bean.WaterSensor;  
import org.apache.flink.api.common.functions.RichMapFunction;  
import org.apache.flink.api.common.functions.RuntimeContext;  
import org.apache.flink.configuration.Configuration;  
import org.apache.flink.streaming.api.datastream.DataStreamSource;  
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;  
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;  
  
 public class RichFunctionDemo {  
    public static void main(String[] args) throws Exception {  
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
        env.setParallelism(2);  
  
        DataStreamSource<WaterSensor> sensorDS = env.fromElements(  
                new WaterSensor("s1", 1L, 1),  
                new WaterSensor("s2", 2L, 2),  
                new WaterSensor("s3", 3L, 3)  
        );  
          
        SingleOutputStreamOperator<String> map = sensorDS.map(new RichMapFunction<WaterSensor, String>() {  
  
            @Override  
            public void open(Configuration parameters) throws Exception {  
                super.open(parameters);  
                System.out.println(  
                        "子任务编号=" + getRuntimeContext().getIndexOfThisSubtask()  
                                + "，子任务名称=" + getRuntimeContext().getTaskNameWithSubtasks()  
                                + ",调用open()");  
            }  
  
            @Override  
            public void close() throws Exception {  
                super.close();  
                System.out.println(  
                        "子任务编号=" + getRuntimeContext().getIndexOfThisSubtask()  
                                + "，子任务名称=" + getRuntimeContext().getTaskNameWithSubtasks()  
                                + ",调用close()");  
            }  
  
            @Override  
            public String map(WaterSensor value) throws Exception {  
                return value.getId();  
            }  
        });  
  
  
        /**  
         * TODO RichXXXFunction: 富函数  
         * 1、多了生命周期管理方法：  
         *    open(): 每个子任务，在启动时，调用一次  
         *    close():每个子任务，在结束时，调用一次  
         *      => 如果是flink程序异常挂掉，不会调用close  
         *      => 如果是正常调用 cancel命令，可以close  
         * 2、多了一个 运行时上下文  
         *    可以获取一些运行时的环境信息，比如 子任务编号、名称、其他的.....  
         *///        DataStreamSource<Integer> source = env.fromElements(1, 2, 3, 4); 
    
        map.print();  
        env.execute();  
    }  
}
```
**输出：**
```bash
子任务编号=0，子任务名称=Map -> Sink: Print to Std. Out (1/2)#0,调用open()
子任务编号=1，子任务名称=Map -> Sink: Print to Std. Out (2/2)#0,调用open()
1> s1
2> s2
1> s3
子任务编号=0，子任务名称=Map -> Sink: Print to Std. Out (1/2)#0,调用close()
子任务编号=1，子任务名称=Map -> Sink: Print to Std. Out (2/2)#0,调用close()
```
以上设置并行度为2，每个子任务启动时会调用open()，退出时调用close()。
### 3.4.4 

## 3.5 Sink 数据输出




