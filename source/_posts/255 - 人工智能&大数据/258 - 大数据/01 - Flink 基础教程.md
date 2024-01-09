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

### 3.4.4 物理分区算子
#### 3.4.4.1 随机分区
随机分区服从均匀分布（uniform distribution），所以可以把流中的数据随机打乱，均匀地传递到下游任务分区。因为是完全随机的，所以对于同样的输入数据, 每次执行得到的结果也不会相同。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240108135209.png)

经过随机分区之后，得到的依然是一个DataStream。
```java
sensorDS.shuffle().print();
```

#### 3.4.4.2 轮询分区
轮询，按照先后顺序将数据做依次分发。

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240108135714.png)

```java
stream.rebalance()
```

#### 3.4.4.3 重缩放分区

重缩放分区和轮询分区非常相似。当调用rescale()方法时，其实底层也是使用Round-Robin算法进行轮询，但是只会将数据轮询发送到下游并行任务的一部分中。rescale的做法是分成小团体，发牌人只给自己团体内的所有人轮流发牌。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240108140109.png)

重缩放分区与轮询分区相比，因为轮询的范围更小，性能更好。
```java
stream.rescale()
```
#### 3.4.4.4 广播/全局分区
- 广播分区

数据会在不同的分区都保留一份，可能进行重复处理。可以通过调用DataStream的broadcast()方法，将输入数据复制并发送到下游算子的所有并行任务中去。
```java
stream.broadcast()
```

- 全局分区

会将所有的输入流数据都发送到下游算子的第一个并行子任务中去。这就相当于强行让下游任务并行度变成了1，所以使用这个操作需要非常谨慎，可能对程序造成很大的压力。
```java
stream.global()
```

#### 3.4.4.5 自定义分区
如果所有分区策略无法满足需求，可以通过使用partitionCustom()方法来自定义分区策略。

- 自定义分区 MyPartitioner

```java
package com.atguigu.partition;  
  
import org.apache.flink.api.common.functions.Partitioner;  
  
public class MyPartitioner implements Partitioner<String> {  
    @Override  
    public int partition(String key, int numPartitions) {  
        return Integer.parseInt(key) % numPartitions;  
    }  
}
```

继承Partitioner 后需要实现 partition方法，第一个参数为 从流数据中提取的计算分区的key，第二个参数为 并行度。


- 使用自定义分区

```java
package com.atguigu.partition;  
  
import org.apache.flink.api.java.functions.KeySelector;  
import org.apache.flink.configuration.Configuration;  
import org.apache.flink.streaming.api.datastream.DataStreamSource;  
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;  
  
public class PartitionCustomDemo {  
    public static void main(String[] args) throws Exception {  
  
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
        env.setParallelism(2);  
  
        DataStreamSource<String> elementsDs = env.fromElements("1", "2", "3", "4", "5");  
  
        elementsDs  
                .partitionCustom(new MyPartitioner(), r->r)  
                .print();  
  
        env.execute();  
    }  
}
```
以上设置并行度为2，会把数据按奇偶分到两个分区中。

#### 3.4.4.6 总结
Flink提供了 7种分区器+ 1种自定义：
1. shuffle：随机分区
2. rebalance：轮询
3. recale：重缩放分区
4. broadcast：广播
5. global：全局
6. keyBy：one-by-one
7. partitionCustom：自定义分区

### 3.4.5 侧输出流
在处理流数据时，可以使用侧输出流把部分数据输出到其他流中，达到分流的效果。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240108144051.png)

示例：实现将WaterSensor按照Id类型进行分流。

```java
package com.atguigu.split;  
  
import com.atguigu.bean.WaterSensor;  
import com.atguigu.functions.WaterSensorMapFunction;  
import org.apache.flink.api.common.functions.FilterFunction;  
import org.apache.flink.api.common.typeinfo.Types;  
import org.apache.flink.configuration.Configuration;  
import org.apache.flink.streaming.api.datastream.DataStreamSource;  
import org.apache.flink.streaming.api.datastream.SideOutputDataStream;  
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;  
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;  
import org.apache.flink.streaming.api.functions.ProcessFunction;  
import org.apache.flink.util.Collector;  
import org.apache.flink.util.OutputTag;  
  
public class SideOutputDemo {  
    public static void main(String[] args) throws Exception {  
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
//        StreamExecutionEnvironment env = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());  
  
        env.setParallelism(1);  
  
//        SingleOutputStreamOperator<WaterSensor> sensorDS = env  
//                .socketTextStream("10.211.55.4", 7777)  
//                .map(new WaterSensorMapFunction());  
  
        DataStreamSource<WaterSensor> sensorDS = env.fromElements(  
                new WaterSensor("s1", 1L, 1),  
                new WaterSensor("s1", 11L, 11),  
                new WaterSensor("s2", 2L, 2),  
                new WaterSensor("s3", 3L, 3)  
        );  
  
        /**  
         * TODO 使用侧输出流 实现分流  
         * 需求： watersensor的数据，s1、s2的数据分别分开  
         *  
         * TODO 总结步骤：  
         *    1、使用 process算子  
         *    2、定义 OutputTag对象  
         *    3、调用 ctx.output  
         *    4、通过主流 获取 测流  
         */  
  
        /**         * 创建OutputTag对象  
         * 第一个参数： 标签名  
         * 第二个参数： 放入侧输出流中的 数据的 类型，Typeinformation  
         */        OutputTag<WaterSensor> s1Tag = new OutputTag<>("s1", Types.POJO(WaterSensor.class));  
        OutputTag<WaterSensor> s2Tag = new OutputTag<>("s2", Types.POJO(WaterSensor.class));  
  
        SingleOutputStreamOperator<WaterSensor> process = sensorDS  
                .process(  
                        new ProcessFunction<WaterSensor, WaterSensor>() {  
                            @Override  
                            public void processElement(WaterSensor value, Context ctx, Collector<WaterSensor> out) throws Exception {  
                                String id = value.getId();  
                                if ("s1".equals(id)) {  
                                    // 如果是 s1，放到侧输出流s1中  
                                    /**  
                                     * 上下文ctx 调用ouput，将数据放入侧输出流  
                                     * 第一个参数： Tag对象  
                                     * 第二个参数： 放入侧输出流中的 数据  
                                     */  
                                    ctx.output(s1Tag, value);  
                                } else if ("s2".equals(id)) {  
                                    // 如果是 s2，放到侧输出流s2中  
  
                                    ctx.output(s2Tag, value);  
                                } else {  
                                    // 非s1、s2的数据，放到主流中  
                                    out.collect(value);  
                                }  
  
                            }  
                        }  
                );  
        // 从主流中，根据标签 获取 侧输出流  
        SideOutputDataStream<WaterSensor> s1 = process.getSideOutput(s1Tag);  
        SideOutputDataStream<WaterSensor> s2 = process.getSideOutput(s2Tag);  
  
  
        // 打印主流  
        process.print("主流-非s1、s2");  
  
        //打印 侧输出流  
//        s1.printToErr("s1");  
        s1.filter(new FilterFunction<WaterSensor>() {  
            @Override  
            public boolean filter(WaterSensor waterSensor) throws Exception {  
                return waterSensor.getTs() == 11;  
            }  
        }).print("s1");  
        s2.printToErr("s2");  
  
        env.execute();  
    }  
}  
```
**输出：**
```bash
s1> WaterSensor{id='s1', ts=11, vc=11}
主流-非s1、s2> WaterSensor{id='s3', ts=3, vc=3}
s2> WaterSensor{id='s2', ts=2, vc=2}
```

按照s1，s2，s3 类型进行分流，分成三条流，对s1流再进行 filter 操作。

### 3.4.6 合流
#### 3.4.6.1 联合

最简单的合流操作，就是直接将多条流合在一起，叫作流的“联合”（union）。联合操作要求必须流中的数据类型必须相同，合并之后的新流会包括所有流中的元素，数据类型不变。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240108145943.png)

```java
stream1.union(stream2, stream3, ...)
```

**示例代码：**
```java
package com.atguigu.combine;  
  
import org.apache.flink.streaming.api.datastream.DataStream;  
import org.apache.flink.streaming.api.datastream.DataStreamSource;  
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;  
  
  
public class UnionDemo {  
    public static void main(String[] args) throws Exception {  
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
        env.setParallelism(1);  
  
        DataStreamSource<Integer> source1 = env.fromElements(1, 2, 3);  
        DataStreamSource<Integer> source2 = env.fromElements(11, 22, 33);  
        DataStreamSource<String> source3 = env.fromElements("111", "222", "333");  
  
        /**  
         * TODO union：合并数据流  
         * 1、 流的数据类型必须一致  
         * 2、 一次可以合并多条流  
         */  
//        DataStream<Integer> union = source1.union(source2).union(source3.map(r -> Integer.valueOf(r)));  
        DataStream<Integer> union = source1.union(source2, source3.map(r -> Integer.valueOf(r)));  
        union.print();  
  
  
        env.execute();  
    }  
}
```
所有数据合并到一起输出。

#### 3.4.6.2 连接
流的联合虽然简单，不过受限于数据类型不能改变，灵活性大打折扣，所以实际应用较少出现。

连接操作允许流的数据类型不同，但我们知道一个DataStream中的数据只能有唯一的类型，所以连接得到的并不是DataStream，而是“连接流”（ConnectedStreams），在连接以后，实际上内部仍保持各自的数据形式不变，彼此之间相互独立。要得到新的DataStream，需要进一步定义一个“同处理”（co-process）转换操作，用来说明对于不同来源、不同类型的数据，怎样分别进行处理转换、得到统一的输出类型。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240108150840.png)

- CoMapFunction

**代码实现：** 需要分为两步：
1. 基于一条DataStream调用 `.connect()` 方法，传入另一条DataStream作为参数，将两条流连接起来，得到一个ConnectedStreams
2. 再调用同处理方法得到DataStream。这里可以的调用的同处理方法有.map()/.flatMap()，以及.process()方法。

```java
package com.atguigu.combine;  
  
import org.apache.flink.streaming.api.datastream.ConnectedStreams;  
import org.apache.flink.streaming.api.datastream.DataStream;  
import org.apache.flink.streaming.api.datastream.DataStreamSource;  
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;  
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;  
import org.apache.flink.streaming.api.functions.co.CoMapFunction;  
  
public class ConnectDemo {  
    public static void main(String[] args) throws Exception {  
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
        env.setParallelism(1);  
  
        DataStreamSource<Integer> source1 = env.fromElements(1, 2, 3);  
        DataStreamSource<String> source2 = env.fromElements("a", "b", "c");  
  
        /*** TODO 使用 connect 合流  
         * 1、一次只能连接 2条流  
         * 2、流的数据类型可以不一样  
         * 3、 连接后可以调用 map、flatmap、process来处理，但是各处理各的  
         */  
        ConnectedStreams<Integer, String> connect = source1.connect(source2);  
  
        SingleOutputStreamOperator<String> result = connect.map(new CoMapFunction<Integer, String, String>() {  
            @Override  
            public String map1(Integer value) throws Exception {  
                return "来源于数字流:" + value.toString();  
            }  
  
            @Override  
            public String map2(String value) throws Exception {  
                return "来源于字母流:" + value;  
            }  
        });  
  
        result.print();  
        env.execute();  
    }  
}
```
**输出：**
```bash
来源于数字流:1
来源于字母流:a
来源于数字流:2
来源于字母流:b
来源于数字流:3
来源于字母流:c
```

ConnectedStreams有两个类型参数，分别表示内部包含的两条流各自的数据类型；
由于是两条不同类型的流，调用 `.map()`方法时传入的不再是一个简单的MapFunction，而是一个CoMapFunction，表示分别对两条流中的数据执行map操作。这个接口有三个类型参数，依次表示第一条流、第二条流，以及合并后的流中的数据类型。需要实现的方法：`.map1()`就是第一条流中数据的 `map` 操作，`.map2()` 则是针对第二条流的操作。

- CoProcessFunction

调用.process()时，传入的则是一个CoProcessFunction。它也是“处理函数”家族中的一员，用法非常相似。它需要实现的就是processElement1()、processElement2()两个方法，在每个数据到来时，会根据来源的流调用其中的一个方法进行处理。
ConnectedStreams也可以直接调用.keyBy()进行按键分区的操作，得到的还是一个ConnectedStreams：
```java
connectedStreams.keyBy(keySelector1, keySelector2);
```
两个参数keySelector1和keySelector2，是两条流中各自的键选择器；当然也可以直接传入键的位置值（keyPosition），或者键的字段名（field），这与普通的keyBy用法完全一致。ConnectedStreams进行keyBy操作，其实就是把两条流中key相同的数据放到了一起，然后针对来源的流再做各自处理，这在一些场景下非常有用。

**代码示例：**
```java
package com.atguigu.combine;  
  
import org.apache.flink.api.java.tuple.Tuple2;  
import org.apache.flink.api.java.tuple.Tuple3;  
import org.apache.flink.streaming.api.datastream.ConnectedStreams;  
import org.apache.flink.streaming.api.datastream.DataStreamSource;  
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;  
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;  
import org.apache.flink.streaming.api.functions.co.CoProcessFunction;  
import org.apache.flink.util.Collector;  
  
import java.util.ArrayList;  
import java.util.HashMap;  
import java.util.List;  
import java.util.Map;  
  
public class ConnectKeybyDemo {  
    public static void main(String[] args) throws Exception {  
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
        env.setParallelism(2);  
  
        DataStreamSource<Tuple2<Integer, String>> source1 = env.fromElements(  
                Tuple2.of(1, "a1"),  
                Tuple2.of(1, "a2"),  
                Tuple2.of(2, "b"),  
                Tuple2.of(3, "c")  
        );  
        DataStreamSource<Tuple3<Integer, String, Integer>> source2 = env.fromElements(  
                Tuple3.of(1, "aa1", 1),  
                Tuple3.of(1, "aa2", 2),  
                Tuple3.of(2, "bb", 1),  
                Tuple3.of(3, "cc", 1)  
        );  
  
        ConnectedStreams<Tuple2<Integer, String>, Tuple3<Integer, String, Integer>> connect = source1.connect(source2);  
  
        // 多并行度下，需要根据 关联条件进行 keyby，才能保证 key相同的数据到一起去，才能匹配上  
        ConnectedStreams<Tuple2<Integer, String>, Tuple3<Integer, String, Integer>> connectKeyby = connect.keyBy(s1 -> s1.f0, s2 -> s2.f0);  
  
        /**  
         * 实现互相匹配的效果：  两条流，，不一定谁的数据先来  
         *  1、每条流，有数据来，存到一个变量中  
         *      hashmap  
         *      =》key=id，第一个字段值  
         *      =》value=List<数据>  
         *  2、每条流有数据来的时候，除了存变量中， 不知道对方是否有匹配的数据，要去另一条流存的变量中 查找是否有匹配上的  
         */  
        SingleOutputStreamOperator<String> process = connectKeyby.process(  
                new CoProcessFunction<Tuple2<Integer, String>, Tuple3<Integer, String, Integer>, String>() {  
                    // 每条流定义一个hashmap，用来存数据  
                    Map<Integer, List<Tuple2<Integer, String>>> s1Cache = new HashMap<>();  
                    Map<Integer, List<Tuple3<Integer, String, Integer>>> s2Cache = new HashMap<>();  
  
                    /**  
                     * 第一条流的处理逻辑  
                     * @param value 第一条流的数据  
                     * @param ctx   上下文  
                     * @param out   采集器  
                     * @throws Exception  
                     */  
                    @Override  
                    public void processElement1(Tuple2<Integer, String> value, Context ctx, Collector<String> out) throws Exception {  
                        Integer id = value.f0;  
                        // TODO 1. s1的数据来了，就存到变量中  
                        if (!s1Cache.containsKey(id)) {  
                            // 1.1 如果key不存在，说明是该key的第一条数据，初始化，put进map中  
                            List<Tuple2<Integer, String>> s1Values = new ArrayList<>();  
                            s1Values.add(value);  
                            s1Cache.put(id, s1Values);  
                        } else {  
                            // 1.2 key存在，不是该key的第一条数据，直接添加到 value的list中  
                            s1Cache.get(id).add(value);  
                        }  
  
                        // TODO 2.去 s2Cache中查找是否有id能匹配上的,匹配上就输出，没有就不输出  
                        if (s2Cache.containsKey(id)) {  
                            for (Tuple3<Integer, String, Integer> s2Element : s2Cache.get(id)) {  
                                out.collect("s1:" + value + "<========>" + "s2:" + s2Element);  
                            }  
                        }  
  
                    }  
  
                    /**  
                     * 第二条流的处理逻辑  
                     * @param value 第二条流的数据  
                     * @param ctx   上下文  
                     * @param out   采集器  
                     * @throws Exception  
                     */  
                    @Override  
                    public void processElement2(Tuple3<Integer, String, Integer> value, Context ctx, Collector<String> out) throws Exception {  
                        Integer id = value.f0;  
                        // TODO 1. s2的数据来了，就存到变量中  
                        if (!s2Cache.containsKey(id)) {  
                            // 1.1 如果key不存在，说明是该key的第一条数据，初始化，put进map中  
                            List<Tuple3<Integer, String, Integer>> s2Values = new ArrayList<>();  
                            s2Values.add(value);  
                            s2Cache.put(id, s2Values);  
                        } else {  
                            // 1.2 key存在，不是该key的第一条数据，直接添加到 value的list中  
                            s2Cache.get(id).add(value);  
                        }  
  
                        // TODO 2.去 s1Cache中查找是否有id能匹配上的,匹配上就输出，没有就不输出  
                        if (s1Cache.containsKey(id)) {  
                            for (Tuple2<Integer, String> s1Element : s1Cache.get(id)) {  
                                out.collect("s1:" + s1Element + "<========>" + "s2:" + value);  
                            }  
                        }  
                    }  
                }  
        );  
  
        process.print();  
        env.execute();  
    }  
}
```

## 3.5 Sink 数据输出
Flink作为数据处理框架，最终还是要把计算处理的结果写入外部存储，为外部应用提供支持。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240108152743.png)


### 3.5.1 连接到外部系统
Flink1.12开始，重构了Sink架构，使用如下方式：
```java
stream.sinkTo(…)
```

官方提供了众多 Sink 连接器，如下：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240108153223.png)


Flink官方之外，Apache Bahir框架，也实现了一些其他第三方系统与Flink的连接器。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240108153257.png)

### 3.5.2 输出到文件
FileSink支持行编码（Row-encoded）和批量编码（Bulk-encoded）格式。这两种不同的方式都有各自的构建器（builder），可以直接调用FileSink的静态方法：
- 行编码： FileSink.forRowFormat（basePath，rowEncoder）。
- 批量编码： FileSink.forBulkFormat（basePath，bulkWriterFactory）。

**导入依赖：**
```xml
<dependency>  
    <groupId>org.apache.flink</groupId>  
    <artifactId>flink-connector-files</artifactId>  
    <version>${flink.version}</version>  
    <scope>compile</scope>  
</dependency>
```

**示例代码：**
```java
package com.atguigu.sink;  
  
import org.apache.flink.api.common.eventtime.WatermarkStrategy;  
import org.apache.flink.api.common.serialization.SimpleStringEncoder;  
import org.apache.flink.api.common.typeinfo.Types;  
import org.apache.flink.api.connector.source.util.ratelimit.RateLimiterStrategy;  
import org.apache.flink.configuration.MemorySize;  
import org.apache.flink.connector.datagen.source.DataGeneratorSource;  
import org.apache.flink.connector.datagen.source.GeneratorFunction;  
import org.apache.flink.connector.file.sink.FileSink;  
import org.apache.flink.core.fs.Path;  
import org.apache.flink.streaming.api.CheckpointingMode;  
import org.apache.flink.streaming.api.datastream.DataStreamSource;  
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;  
import org.apache.flink.streaming.api.functions.sink.filesystem.OutputFileConfig;  
import org.apache.flink.streaming.api.functions.sink.filesystem.bucketassigners.DateTimeBucketAssigner;  
import org.apache.flink.streaming.api.functions.sink.filesystem.rollingpolicies.DefaultRollingPolicy;  
  
import java.time.Duration;  
import java.time.ZoneId;  
import java.util.TimeZone;  
  
public class SinkFile {  
    public static void main(String[] args) throws Exception {  
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
  
        // TODO 每个目录中，都有 并行度个数的 文件在写入  
        env.setParallelism(2);  
  
        // 必须开启checkpoint，否则一直都是 .inprogress        env.enableCheckpointing(2000, CheckpointingMode.EXACTLY_ONCE);  
  
  
        DataGeneratorSource<String> dataGeneratorSource = new DataGeneratorSource<>(  
                new GeneratorFunction<Long, String>() {  
                    @Override  
                    public String map(Long value) throws Exception {  
                        return "Number:" + value;  
                    }  
                },  
                Long.MAX_VALUE,  
                RateLimiterStrategy.perSecond(1000),  
                Types.STRING  
        );  
  
        DataStreamSource<String> dataGen = env.fromSource(dataGeneratorSource, WatermarkStrategy.noWatermarks(), "data-generator");  
  
        // TODO 输出到文件系统  
        FileSink<String> fieSink = FileSink  
                // 输出行式存储的文件，指定路径、指定编码  
                .<String>forRowFormat(new Path("./output/"), new SimpleStringEncoder<>("UTF-8"))  
                // 输出文件的一些配置： 文件名的前缀、后缀  
                .withOutputFileConfig(  
                        OutputFileConfig.builder()  
                                .withPartPrefix("atguigu-")  
                                .withPartSuffix(".log")  
                                .build()  
                )  
                // 按照目录分桶：如下，就是每个小时一个目录  
                .withBucketAssigner(new DateTimeBucketAssigner<>("yyyy-MM-dd HH", ZoneId.systemDefault()))  
                // 文件滚动策略:  1分钟 或 1m                .withRollingPolicy(  
                        DefaultRollingPolicy.builder()  
                                .withRolloverInterval(Duration.ofMinutes(1))  
                                .withMaxPartSize(new MemorySize(1024*1024))  
                                .build()  
                )  
                .build();  
  
  
        dataGen.sinkTo(fieSink);  
        env.execute();  
    }  
}
```

### 3.5.3 输出到Kafka
**导入依赖：**
```xml
<dependency>  
    <groupId>org.apache.flink</groupId>  
    <artifactId>flink-connector-kafka</artifactId>  
    <version>${flink.version}</version>  
</dependency>
```

**示例代码**：
```java
package com.atguigu.sink;  
  
import org.apache.flink.api.common.serialization.SimpleStringSchema;  
import org.apache.flink.connector.base.DeliveryGuarantee;  
import org.apache.flink.connector.kafka.sink.KafkaRecordSerializationSchema;  
import org.apache.flink.connector.kafka.sink.KafkaSink;  
import org.apache.flink.streaming.api.CheckpointingMode;  
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;  
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;  
import org.apache.kafka.clients.producer.ProducerConfig;  
  
public class SinkKafka {  
    public static void main(String[] args) throws Exception {  
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
        env.setParallelism(1);  
  
        // 如果是精准一次，必须开启checkpoint（后续章节介绍）  
        env.enableCheckpointing(2000, CheckpointingMode.EXACTLY_ONCE);  
  
  
        SingleOutputStreamOperator<String> sensorDS = env  
                .socketTextStream("hadoop102", 7777);  
  
        /**  
         * Kafka Sink:         * TODO 注意：如果要使用 精准一次 写入Kafka，需要满足以下条件，缺一不可  
         * 1、开启checkpoint（后续介绍）  
         * 2、设置事务前缀  
         * 3、设置事务超时时间：   checkpoint间隔 <  事务超时时间  < max的15分钟  
         */  
        KafkaSink<String> kafkaSink = KafkaSink.<String>builder()  
                // 指定 kafka 的地址和端口  
                .setBootstrapServers("hadoop102:9092,hadoop103:9092,hadoop104:9092")  
                // 指定序列化器：指定Topic名称、具体的序列化  
                .setRecordSerializer(  
                        KafkaRecordSerializationSchema.<String>builder()  
                                .setTopic("ws")  
                                .setValueSerializationSchema(new SimpleStringSchema())  
                                .build()  
                )  
                // 写到kafka的一致性级别： 精准一次、至少一次  
                .setDeliveryGuarantee(DeliveryGuarantee.EXACTLY_ONCE)  
                // 如果是精准一次，必须设置 事务的前缀  
                .setTransactionalIdPrefix("atguigu-")  
                // 如果是精准一次，必须设置 事务超时时间: 大于checkpoint间隔，小于 max 15分钟  
                .setProperty(ProducerConfig.TRANSACTION_TIMEOUT_CONFIG, 10*60*1000+"")  
                .build();  
  
  
        sensorDS.sinkTo(kafkaSink);  
	    env.execute();  
    }  
}
```

### 3.5.4 输出到 MySQL（JDBC）

**导入依赖：**
```xml
<!--目前中央仓库还没有 jdbc的连接器，暂时用一个快照版本-->  
<dependency>  
    <groupId>org.apache.flink</groupId>  
    <artifactId>flink-connector-jdbc</artifactId>  
    <version>1.17-SNAPSHOT</version>  
</dependency>
```

**示例代码：**
```java
package com.atguigu.sink;  
  
import com.atguigu.bean.WaterSensor;  
import com.atguigu.functions.WaterSensorMapFunction;  
import org.apache.flink.api.common.restartstrategy.RestartStrategies;  
import org.apache.flink.connector.jdbc.JdbcConnectionOptions;  
import org.apache.flink.connector.jdbc.JdbcExecutionOptions;  
import org.apache.flink.connector.jdbc.JdbcSink;  
import org.apache.flink.connector.jdbc.JdbcStatementBuilder;  
import org.apache.flink.streaming.api.CheckpointingMode;  
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;  
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;  
import org.apache.flink.streaming.api.functions.sink.SinkFunction;  
  
import java.sql.PreparedStatement;  
import java.sql.SQLException;  
  
public class SinkMySQL {  
    public static void main(String[] args) throws Exception {  
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
        env.setParallelism(1);  
  
        SingleOutputStreamOperator<WaterSensor> sensorDS = env  
                .socketTextStream("hadoop102", 7777)  
                .map(new WaterSensorMapFunction());  

        /**  
         * TODO 写入mysql  
         * 1、只能用老的sink写法： addsink  
         * 2、JDBCSink的4个参数:  
         *    第一个参数： 执行的sql，一般就是 insert into  
         *    第二个参数： 预编译sql， 对占位符填充值  
         *    第三个参数： 执行选项 ---》 攒批、重试  
         *    第四个参数： 连接选项 ---》 url、用户名、密码  
         */  
        SinkFunction<WaterSensor> jdbcSink = JdbcSink.sink(  
                "insert into ws values(?,?,?)",  
                new JdbcStatementBuilder<WaterSensor>() {  
                    @Override  
                    public void accept(PreparedStatement preparedStatement, WaterSensor waterSensor) throws SQLException {  
                        //每收到一条WaterSensor，如何去填充占位符  
                        preparedStatement.setString(1, waterSensor.getId());  
                        preparedStatement.setLong(2, waterSensor.getTs());  
                        preparedStatement.setInt(3, waterSensor.getVc());  
                    }  
                },  
                JdbcExecutionOptions.builder()  
                        .withMaxRetries(3) // 重试次数  
                        .withBatchSize(100) // 批次的大小：条数  
                        .withBatchIntervalMs(3000) // 批次的时间  
                        .build(),  
                new JdbcConnectionOptions.JdbcConnectionOptionsBuilder()  
                        .withUrl("jdbc:mysql://hadoop102:3306/test?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=UTF-8")  
                        .withUsername("root")  
                        .withPassword("000000")  
                        .withConnectionCheckTimeoutSeconds(60) // 重试的超时时间  
                        .build()  
        );  
  
        sensorDS.addSink(jdbcSink);   
        env.execute();  
    }  
}
```

### 3.5.5 自定义 Sink 输出
 如果我们想将数据存储到我们自己的存储设备中，而Flink并没有提供可以直接使用的连接器，就只能自定义Sink进行输出了。与Source类似，Flink为我们提供了通用的SinkFunction接口和对应的RichSinkDunction抽象类，只要实现它，通过简单地调用DataStream的.addSink()方法就可以自定义写入任何外部存储。
```java
stream.addSink(new MySinkFunction<String>());
```
在实现SinkFunction的时候，需要重写的一个关键方法invoke()，在这个方法中我们就可以实现将流里的数据发送出去的逻辑。

这种方式比较通用，对于任何外部存储系统都有效；不过自定义Sink想要实现状态一致性并不容易，所以一般只在没有其它选择时使用。实际项目中用到的外部连接器Flink官方基本都已实现，而且在不断地扩充，因此自定义的场景并不常见。

## 3.6 Flink 中的窗口
### 3.6.1 窗口
Flink是一种流式计算引擎，主要是来处理无界数据流的，数据源源不断、无穷无尽。想要更加方便高效地处理无界流，一种方式就是将无限数据切割成有限的“数据块”进行处理，这就是所谓的“窗口”（Window）。
在Flink中窗口可以把流切割成有限大小的多个“存储桶”（bucket），每个数据都会分发到对应的桶中，当到达窗口结束时间时，就对每个桶中收集的数据进行计算处理。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240109091736.png)

> Flink中窗口并不是静态准备好的，而是动态创建——当有落在这个窗口区间范围的数据达到时，才创建对应的窗口。


### 3.6.2 窗口分类
#### 3.6.2.1 按照驱动类型划分
- 时间窗口（Time Window）
时间窗口以时间点来定义窗口的开始和结束，所以截取出的就是某一段时间段的数据，到达结束时间时，窗口不再收集数据，触发计算输出结果，并将窗口关闭销毁。
```java
sensorKS.window(TumblingProcessingTimeWindows.of(Time.seconds(10))) // 滚动窗口，窗口长度10s  
sensorKS.window(SlidingProcessingTimeWindows.of(Time.seconds(10), Time.seconds(2))) // 滑动窗口，窗口长度10s，滑动步长2s  
sensorKS.window(ProcessingTimeSessionWindows.withGap(Time.seconds(5))) // 会话窗口，超时间隔5s
```

- 计数窗口（Count Window）
计数窗口基于元素的个数来截取数据，到达固定的个数时就触发计算并关闭窗口，每个窗口截取数据的个数，就是窗口的大小。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240109093047.png)

```java
sensorKS.countWindow(5)  // 滚动窗口，窗口长度=5个元素  
sensorKS.countWindow(5,2) // 滑动窗口，窗口长度=5个元素，滑动步长=2个元素  
sensorKS.window(GlobalWindows.create())  // 全局窗口，计数窗口的底层就是用的这个，需要自定义的时候才会用
```
#### 3.6.2.2 按照窗口分配数据的规则分类
根据分配数据的规则，窗口具体实现可以分为4类：滚动窗口（Tumbling Window）、滑动窗口（Sliding Window）、会话窗口（Session Window）、全局窗口（Global Window）。

- 滚动窗口
窗口有固定的大小，是一种对数据进行“均匀切片”的划分方式，窗口之间没有重叠，是“首尾相接”的状态，每个数据都会被分配到一个窗口，而且只会属于一个窗口。

滚动窗口可以基于时间定义，也可以基于数据个数定义；需要的参数只有一个，就是**窗口的大小（window size）**。

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240109093518.png)

滚动处理时间窗口：
```java
stream.keyBy(...)
		//长度为5秒的滚动窗口
       .window(TumblingProcessingTimeWindows.of(Time.seconds(5)))
       .aggregate(...)
```

滚动事件时间窗口：
```java
stream.keyBy(...)
       .window(TumblingEventTimeWindows.of(Time.seconds(5)))
       .aggregate(...)
```


- 滑动窗口
滑动窗口的大小也是固定的，但是窗口之间并不是首尾相接，而是可以“错开”一定的位置。
定义滑动窗口的参数有两个：除了窗口大小（window size），还有一个“滑动步长”（window slide），它其实就代表了窗口计算的频率。窗口在结束时间触发计算输出结果，那么滑动步长就代表计算频率。

当滑动步长小于窗口大小时，滑动窗口就会出现重叠，这时数据也可能会被同时分配到多个窗口中。而具体的个数，就由窗口大小和滑动步长的比值（size/slide）来决定。
滚动窗口也可以看作是一种特殊的滑动窗口（窗口大小等于滑动步长）。滑动窗口适合计算结果更新频率非常高的场景。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240109094006.png)

滑动事件时间窗口：
```java
stream.keyBy(...)
       .window(SlidingEventTimeWindows.of(Time.seconds(10)，Time.seconds(5)))
       .aggregate(...)
```

滑动计数窗口：
```java
stream.keyBy(...)
       .countWindow(10，3)
```
定义了一个长度为10、滑动步长为3的滑动计数窗口。每个窗口统计10个数据，每隔3个数据就统计输出一次结果。

- 会话窗口
会话窗口，基于“会话（session）”来对数据进行分组，会话窗口只能基于时间来定义。
会话窗口中，最终的参数就是会话的超时时间，也就是两个会话窗口之间的最小距离。如果相邻两个数据到来的时间间隔（Gap）小于指定的大小（size），那说明还在保持会话，他们就属于同一个窗口；如果

gap大于size，那么新来的数据就应该属于新的会话窗口，而前一个窗口就应该关闭了。
会话窗口的长度不固定，起始和结束时间也是不确定的，各个分区之间窗口没有任何关联，会话窗口之间一定是不会重叠的，而是会留有至少为size的间隔。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240109094413.png)

事件时间会话窗口：
```java
stream.keyBy(...)
       .window(EventTimeSessionWindows.withGap(Time.seconds(10)))
       .aggregate(...)
```

- 全局窗口
这种窗口全局有效，会把相同key的所有数据都分配到同一个窗口中，这种窗口没有结束的时候，默认是不会做触发计算的。如果希望他能对数据进行计算处理，还需要自定义“触发器”（Trigger）。

全局窗口没有结束的时间点，所以一般在希望做更加灵活的窗口处理时自定义使用，Flink中的计数窗口（Count Window），底层就是用全局窗口实现。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240109094644.png)
```java
stream.keyBy(...)
       .window(GlobalWindows.create());
```
需要注意使用全局窗口，必须自行定义触发器才能实现窗口计算，否则起不到任何作用。

### 3.6.3 窗口API
1. 按键分区窗口（Keyed Windows）

经过按键分区keyBy操作后，数据流会按照key被分为多条逻辑流（logical streams），这就是KeyedStream。基于KeyedStream进行窗口操作时，窗口计算会在多个并行子任务上同时执行。相同key的数据会被发送到同一个并行子任务，而窗口操作会基于每个key进行单独的处理。所以可以认为，每个key上都定义了一组窗口，各自独立地进行统计计算。

在代码实现上，我们需要先对DataStream调用.keyBy()进行按键分区，然后再调用.window()定义窗口。
```java
stream.keyBy(...)
       .window(...)
```

2. 非按键分区（Non-Keyed Windows）

如果没有进行keyBy，那么原始的DataStream就不会分成多条逻辑流。这时窗口逻辑只能在一个任务（task）上执行，就相当于并行度变成了1。
在代码中，直接基于DataStream调用.windowAll()定义窗口。
```java
stream.windowAll(...)
```
> 对于非按键分区的窗口操作，手动调大窗口算子的并行度也是无效的，windowAll本身就是一个非并行的操作。

以 时间类型的 滚动窗口 为例，分析原理：
1. 窗口什么时候触发输出：时间进展 >= 窗口的最大时间戳（end - 1ms），窗口区间为 左闭右开。
2. 窗口的划分：
start = 向下取整，取窗口长度的整数倍（不是取当前数据到达的时间）
end= start + 窗口长度

3. 窗口的生命周期？
创建： 属于本窗口的第一条数据来的时候，现new的，放入一个singleton单例的集合中  
销毁（关窗）： 时间进展 >=  窗口的最大时间戳（end - 1ms） + 允许迟到的时间（默认0）


**示例代码：**
```java
package com.atguigu.window;  
  
import com.atguigu.bean.WaterSensor;  
import com.atguigu.functions.WaterSensorMapFunction;  
import org.apache.commons.lang3.time.DateFormatUtils;  
import org.apache.flink.streaming.api.datastream.KeyedStream;  
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;  
import org.apache.flink.streaming.api.datastream.WindowedStream;  
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;  
import org.apache.flink.streaming.api.functions.windowing.ProcessWindowFunction;  
import org.apache.flink.streaming.api.windowing.assigners.ProcessingTimeSessionWindows;  
import org.apache.flink.streaming.api.windowing.assigners.SessionWindowTimeGapExtractor;  
import org.apache.flink.streaming.api.windowing.assigners.SlidingProcessingTimeWindows;  
import org.apache.flink.streaming.api.windowing.assigners.TumblingProcessingTimeWindows;  
import org.apache.flink.streaming.api.windowing.time.Time;  
import org.apache.flink.streaming.api.windowing.windows.TimeWindow;  
import org.apache.flink.util.Collector;  
  
public class TimeWindowDemo {  
    public static void main(String[] args) throws Exception {  
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
        env.setParallelism(1);  
  
        SingleOutputStreamOperator<WaterSensor> sensorDS = env  
                .socketTextStream("hadoop102", 7777)  
                .map(new WaterSensorMapFunction());  
  
        KeyedStream<WaterSensor, String> sensorKS = sensorDS.keyBy(sensor -> sensor.getId());  
  
        // 1. 窗口分配器  
        WindowedStream<WaterSensor, String, TimeWindow> sensorWS = sensorKS  
                .window(TumblingProcessingTimeWindows.of(Time.seconds(10))); // 滚动窗口，窗口长度10秒  
//                .window(SlidingProcessingTimeWindows.of(Time.seconds(10), Time.seconds(5)));//滑动窗口，长度10s，步长5s  
//                .window(ProcessingTimeSessionWindows.withGap(Time.seconds(5)));//会话窗口，间隔5s  
//                .window(ProcessingTimeSessionWindows.withDynamicGap(  
//                        new SessionWindowTimeGapExtractor<WaterSensor>() {  
//                            @Override  
//                            public long extract(WaterSensor element) {  
//                                // 从数据中提取ts，作为间隔,单位ms  
//                                return element.getTs() * 1000L;  
//                            }  
//                        }  
//                ));// 会话窗口，动态间隔，每条来的数据都会更新 间隔时间  
  
  
        SingleOutputStreamOperator<String> process = sensorWS  
                .process(  
                        new ProcessWindowFunction<WaterSensor, String, String, TimeWindow>() {  
                            /**  
                             * 全窗口函数计算逻辑：  窗口触发时才会调用一次，统一计算窗口的所有数据  
                             * @param s   分组的key  
                             * @param context  上下文  
                             * @param elements 存的数据  
                             * @param out      采集器  
                             * @throws Exception  
                             */  
                            @Override  
                            public void process(String s, Context context, Iterable<WaterSensor> elements, Collector<String> out) throws Exception {  
                                // 上下文可以拿到window对象，还有其他东西：侧输出流 等等  
                                long startTs = context.window().getStart();  
                                long endTs = context.window().getEnd();  
                                String windowStart = DateFormatUtils.format(startTs, "yyyy-MM-dd HH:mm:ss.SSS");  
                                String windowEnd = DateFormatUtils.format(endTs, "yyyy-MM-dd HH:mm:ss.SSS");  
  
                                long count = elements.spliterator().estimateSize();  
  
                                out.collect("key=" + s + "的窗口[" + windowStart + "," + windowEnd + ")包含" + count + "条数据===>" + elements.toString());  
  
  
                            }  
                        }  
                );  
  
        process.print();  
        env.execute();  
    }  
}
```

### 3.6.4 窗口函数
以上关于窗口的API是为了定义窗口分配器，知道了数据属于哪个窗口，可以将数据收集起来，至于收集起来到底要做什么，必须再接上一个定义窗口如何进行计算的操作，这就是“窗口函数”（window functions）。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20240109112005.png)
窗口函数定义了要对窗口中收集的数据做的计算操作，根据处理的方式可以分为两类：增量聚合函数和全窗口函数。
#### 3.6.4.1 增量聚合函数（ReduceFunction/AggregateFunction）

1. ReduceFunction

Reduce算子接收相同类型的数据，进行增量聚合，输出类型与输入类型一样。
```java
package com.atguigu.window;  
  
import com.atguigu.bean.WaterSensor;  
import com.atguigu.functions.WaterSensorMapFunction;  
import org.apache.flink.api.common.functions.ReduceFunction;  
import org.apache.flink.streaming.api.datastream.KeyedStream;  
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;  
import org.apache.flink.streaming.api.datastream.WindowedStream;  
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;  
import org.apache.flink.streaming.api.windowing.assigners.TumblingProcessingTimeWindows;  
import org.apache.flink.streaming.api.windowing.time.Time;  
import org.apache.flink.streaming.api.windowing.windows.TimeWindow;  
  
public class WindowReduceDemo {  
    public static void main(String[] args) throws Exception {  
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
        env.setParallelism(1);  
  
        SingleOutputStreamOperator<WaterSensor> sensorDS = env  
                .socketTextStream("10.211.55.4", 7777)  
                .map(new WaterSensorMapFunction());  
  
  
        KeyedStream<WaterSensor, String> sensorKS = sensorDS.keyBy(sensor -> sensor.getId());  
  
        // 1. 窗口分配器  
        WindowedStream<WaterSensor, String, TimeWindow> sensorWS = sensorKS.window(TumblingProcessingTimeWindows.of(Time.seconds(10)));  
  
        // 2. 窗口函数： 增量聚合 Reduce        
        /*** 窗口的reduce：  
         * 1、相同key的第一条数据来的时候，不会调用reduce方法  
         * 2、增量聚合： 来一条数据，就会计算一次，但是不会输出  
         * 3、在窗口触发的时候，才会输出窗口的最终计算结果  
         */  
        SingleOutputStreamOperator<WaterSensor> reduce = sensorWS.reduce(  
                new ReduceFunction<WaterSensor>() {  
                    @Override  
                    public WaterSensor reduce(WaterSensor value1, WaterSensor value2) throws Exception {  
                        System.out.println("调用reduce方法，value1=" + value1 + ",value2=" + value2);  
                        return new WaterSensor(value1.getId(), value2.getTs(), value1.getVc() + value2.getVc());  
                    }  
                }  
        );  
  
        reduce.print();  
        env.execute();  
    }  
}
```
以上实现一个10秒滚动窗口，对窗口中的Vc值进行累加操作。10秒后输出窗口结果。

2. AggregateFunction

AggregateFunction可以看作是ReduceFunction的通用版本，这里有三种类型：输入类型（IN）、累加器类型（ACC）和输出类型（OUT）。输入类型IN就是输入流中元素的数据类型；累加器类型ACC则是我们进行聚合的中间状态类型；而输出类型当然就是最终计算结果的类型了。
接口中有四个方法：
- createAccumulator()：创建一个累加器，这就是为聚合创建了一个初始状态，每个聚合任务只会调用一次。
- add()：将输入的元素添加到累加器中。
- getResult()：从累加器中提取聚合的输出结果。
- merge()：合并两个累加器，并将合并后的状态作为一个累加器返回。

**AggregateFunction的工作原理是**：首先调用createAccumulator()为任务初始化一个状态（累加器）；而后每来一个数据就调用一次add()方法，对数据进行聚合，得到的结果保存在状态中；等到了窗口需要输出时，再调用getResult()方法得到计算结果。很明显，与ReduceFunction相同，AggregateFunction也是增量式的聚合；而由于输入、中间状态、输出的类型可以不同，使得应用更加灵活方便。

```java
package com.atguigu.window;  
  
import com.atguigu.bean.WaterSensor;  
import com.atguigu.functions.WaterSensorMapFunction;  
import org.apache.flink.api.common.functions.AggregateFunction;  
import org.apache.flink.api.common.functions.ReduceFunction;  
import org.apache.flink.streaming.api.datastream.KeyedStream;  
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;  
import org.apache.flink.streaming.api.datastream.WindowedStream;  
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;  
import org.apache.flink.streaming.api.windowing.assigners.TumblingProcessingTimeWindows;  
import org.apache.flink.streaming.api.windowing.time.Time;  
import org.apache.flink.streaming.api.windowing.windows.TimeWindow;  
 
public class WindowAggregateDemo {  
    public static void main(String[] args) throws Exception {  
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
        env.setParallelism(1);  
  
        SingleOutputStreamOperator<WaterSensor> sensorDS = env  
                .socketTextStream("hadoop102", 7777)  
                .map(new WaterSensorMapFunction());  
  
        KeyedStream<WaterSensor, String> sensorKS = sensorDS.keyBy(sensor -> sensor.getId());  
  
        // 1. 窗口分配器  
        WindowedStream<WaterSensor, String, TimeWindow> sensorWS = sensorKS.window(TumblingProcessingTimeWindows.of(Time.seconds(10)));  
  
        // 2. 窗口函数： 增量聚合 Aggregate        
        /**
         * 1、属于本窗口的第一条数据来，创建窗口，创建累加器  
         * 2、增量聚合： 来一条计算一条， 调用一次add方法  
         * 3、窗口输出时调用一次getresult方法  
         * 4、输入、中间累加器、输出 类型可以不一样，非常灵活  
         */  
        SingleOutputStreamOperator<String> aggregate = sensorWS.aggregate(  
                /**  
                 * 第一个类型： 输入数据的类型  
                 * 第二个类型： 累加器的类型，存储的中间计算结果的类型  
                 * 第三个类型： 输出的类型  
                 */  
                new AggregateFunction<WaterSensor, Integer, String>() {  
                    /**  
                     * 创建累加器，初始化累加器  
                     * @return  
                     */  
                    @Override  
                    public Integer createAccumulator() {  
                        System.out.println("创建累加器");  
                        return 0;  
                    }  
  
                    /**  
                     * 聚合逻辑  
                     * @param value  
                     * @param accumulator  
                     * @return  
                     */  
                    @Override  
                    public Integer add(WaterSensor value, Integer accumulator) {  
                        System.out.println("调用add方法,value="+value);  
                        return accumulator + value.getVc();  
                    }  
  
                    /**  
                     * 获取最终结果，窗口触发时输出  
                     * @param accumulator  
                     * @return  
                     */  
                    @Override  
                    public String getResult(Integer accumulator) {  
                        System.out.println("调用getResult方法");  
                        return accumulator.toString();  
                    }  
  
                    @Override  
                    public Integer merge(Integer a, Integer b) {  
                        // 只有会话窗口才会用到  
                        System.out.println("调用merge方法");  
                        return null;  
                    }  
                }  
        );  
  
        aggregate.print();  
        env.execute();  
    }  
}
```
以上聚合逻辑计算后，返回类型与输入类型不同。

#### 3.6.4.2 全窗口函数（WindowFunction/ProcessWindowFunction）
有些场景下，我们要做的计算必须基于全部的数据才有效，这时做增量聚合就没什么意义了；另外，输出的结果有可能要包含上下文中的一些信息（比如窗口的起始时间），这是增量聚合函数做不到的。

所以，我们还需要有更丰富的窗口计算方式。窗口操作中的另一大类就是全窗口函数。与增量聚合函数不同，全窗口函数需要先收集窗口中的数据，并在内部缓存起来，等到窗口要输出结果的时候再取出数据进行计算。

在Flink中，全窗口函数也有两种：WindowFunction和ProcessWindowFunction。

1. 窗口函数（WindowFunction）

逐渐被 ProcessWindowFunction 替代，弃用。

2. 处理窗口函数（ProcessWindowFunction）

ProcessWindowFunction是Window API中最底层的通用窗口函数接口。之所以说它“最底层”，是因为除了可以拿到窗口中的所有数据之外，ProcessWindowFunction还可以获取到一个“上下文对象”（Context）。这个上下文对象非常强大，不仅能够获取窗口信息，还可以访问当前的时间和状态信息。这里的时间就包括了处理时间（processing time）和事件时间水位线（event time watermark）。这就使得ProcessWindowFunction更加灵活、功能更加丰富，其实就是一个增强版的WindowFunction。

```java
package com.atguigu.window;  
  
import com.atguigu.bean.WaterSensor;  
import com.atguigu.functions.WaterSensorMapFunction;  
import org.apache.commons.lang3.time.DateFormatUtils;  
import org.apache.flink.api.common.functions.AggregateFunction;  
import org.apache.flink.streaming.api.datastream.KeyedStream;  
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;  
import org.apache.flink.streaming.api.datastream.WindowedStream;  
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;  
import org.apache.flink.streaming.api.functions.windowing.ProcessWindowFunction;  
import org.apache.flink.streaming.api.functions.windowing.WindowFunction;  
import org.apache.flink.streaming.api.windowing.assigners.TumblingProcessingTimeWindows;  
import org.apache.flink.streaming.api.windowing.time.Time;  
import org.apache.flink.streaming.api.windowing.windows.TimeWindow;  
import org.apache.flink.util.Collector;  
  
public class WindowProcessDemo {  
    public static void main(String[] args) throws Exception {  
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
        env.setParallelism(1);  
  
  
        SingleOutputStreamOperator<WaterSensor> sensorDS = env  
                .socketTextStream("10.211.55.4", 7777)  
                .map(new WaterSensorMapFunction());  
  
  
        KeyedStream<WaterSensor, String> sensorKS = sensorDS.keyBy(sensor -> sensor.getId());  
  
        // 1. 窗口分配器  
        WindowedStream<WaterSensor, String, TimeWindow> sensorWS = sensorKS.window(TumblingProcessingTimeWindows.of(Time.seconds(10)));  
  
        // 老写法  
//        sensorWS  
//                .apply(  
//                        new WindowFunction<WaterSensor, String, String, TimeWindow>() {  
//                            /**  
//                             *  
//                             * @param s  分组的key  
//                             * @param window 窗口对象  
//                             * @param input 存的数据  
//                             * @param out   采集器  
//                             * @throws Exception  
//                             */  
//                            @Override  
//                            public void apply(String s, TimeWindow window, Iterable<WaterSensor> input, Collector<String> out) throws Exception {  
//  
//                            }  
//                        }  
//                )  
  
  
        SingleOutputStreamOperator<String> process = sensorWS  
                .process(  
                        new ProcessWindowFunction<WaterSensor, String, String, TimeWindow>() {  
                            /**  
                             * 全窗口函数计算逻辑：  窗口触发时才会调用一次，统一计算窗口的所有数据  
                             * @param s   分组的key  
                             * @param context  上下文  
                             * @param elements 存的数据  
                             * @param out      采集器  
                             * @throws Exception  
                             */  
                            @Override  
                            public void process(String s, Context context, Iterable<WaterSensor> elements, Collector<String> out) throws Exception {  
                                // 上下文可以拿到window对象，还有其他东西：侧输出流 等等  
                                long startTs = context.window().getStart();  
                                long endTs = context.window().getEnd();  
                                String windowStart = DateFormatUtils.format(startTs, "yyyy-MM-dd HH:mm:ss.SSS");  
                                String windowEnd = DateFormatUtils.format(endTs, "yyyy-MM-dd HH:mm:ss.SSS");  
  
                                long count = elements.spliterator().estimateSize();  
  
                                out.collect("key=" + s + "的窗口[" + windowStart + "," + windowEnd + ")包含" + count + "条数据===>" + elements.toString());  
  
  
                            }  
                        }  
                );  
  
        process.print();  
		env.execute();  
    }  
}
```
**输出：**
```bash
key=s1的窗口[2024-01-09 11:37:40.000,2024-01-09 11:37:50.000)包含2条数据===>[WaterSensor{id='s1', ts=1, vc=1}, WaterSensor{id='s1', ts=2, vc=3}]
key=s1的窗口[2024-01-09 11:37:50.000,2024-01-09 11:38:00.000)包含1条数据===>[WaterSensor{id='s1', ts=3, vc=3}]
```
全窗口函数不是来一条数据就计算一条，是窗口时间到达后，再计算输出。

#### 3.6.4.3 增量聚合和全窗口函数结合
通过结合可以兼具两者的优点，调用WindowedStream的.reduce()和.aggregate()方法时，只是简单地直接传入了一个ReduceFunction或AggregateFunction进行增量聚合。除此之外，其实还可以传入第二个参数：一个全窗口函数，可以是WindowFunction或者ProcessWindowFunction。
```java
// ReduceFunction与WindowFunction结合
public <R> SingleOutputStreamOperator<R> reduce(
        ReduceFunction<T> reduceFunction，WindowFunction<T，R，K，W> function) 

// ReduceFunction与ProcessWindowFunction结合
public <R> SingleOutputStreamOperator<R> reduce(
        ReduceFunction<T> reduceFunction，ProcessWindowFunction<T，R，K，W> function)

// AggregateFunction与WindowFunction结合
public <ACC，V，R> SingleOutputStreamOperator<R> aggregate(
        AggregateFunction<T，ACC，V> aggFunction，WindowFunction<V，R，K，W> windowFunction)

// AggregateFunction与ProcessWindowFunction结合
public <ACC，V，R> SingleOutputStreamOperator<R> aggregate(
        AggregateFunction<T，ACC，V> aggFunction,
        ProcessWindowFunction<V，R，K，W> windowFunction)
```

基于第一个参数（增量聚合函数）来处理窗口数据，每来一个数据就做一次聚合；等到窗口需要触发计算
时，则调用第二个参数（全窗口函数）的处理逻辑输出结果。需要注意的是，这里的全窗口函数就不再缓存所有数据了，而是直接将增量聚合函数的结果拿来当作了Iterable类型的输入。

**代码示例：**
```java
package com.atguigu.window;  
  
import com.atguigu.bean.WaterSensor;  
import com.atguigu.functions.WaterSensorMapFunction;  
import org.apache.commons.lang3.time.DateFormatUtils;  
import org.apache.flink.api.common.functions.AggregateFunction;  
import org.apache.flink.streaming.api.datastream.KeyedStream;  
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;  
import org.apache.flink.streaming.api.datastream.WindowedStream;  
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;  
import org.apache.flink.streaming.api.functions.windowing.ProcessWindowFunction;  
import org.apache.flink.streaming.api.windowing.assigners.TumblingProcessingTimeWindows;  
import org.apache.flink.streaming.api.windowing.time.Time;  
import org.apache.flink.streaming.api.windowing.windows.TimeWindow;  
import org.apache.flink.util.Collector;  
  
public class WindowAggregateAndProcessDemo {  
    public static void main(String[] args) throws Exception {  
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
        env.setParallelism(1);  
  
        SingleOutputStreamOperator<WaterSensor> sensorDS = env  
                .socketTextStream("hadoop102", 7777)  
                .map(new WaterSensorMapFunction());  
  
        KeyedStream<WaterSensor, String> sensorKS = sensorDS.keyBy(sensor -> sensor.getId());  
  
        // 1. 窗口分配器  
        WindowedStream<WaterSensor, String, TimeWindow> sensorWS = sensorKS.window(TumblingProcessingTimeWindows.of(Time.seconds(10)));  
  
        // 2. 窗口函数：  
        /**  
         * 增量聚合 Aggregate + 全窗口 process  
         * 1、增量聚合函数处理数据： 来一条计算一条  
         * 2、窗口触发时， 增量聚合的结果（只有一条） 传递给 全窗口函数  
         * 3、经过全窗口函数的处理包装后，输出  
         *  
         * 结合两者的优点：  
         * 1、增量聚合： 来一条计算一条，存储中间的计算结果，占用的空间少  
         * 2、全窗口函数： 可以通过 上下文 实现灵活的功能  
         */  
  
//        sensorWS.reduce()   //也可以传两个  
  
        SingleOutputStreamOperator<String> result = sensorWS.aggregate(  
                new MyAgg(),  
                new MyProcess()  
        );  
  
        result.print();  
        env.execute();  
    }  
  
    public static class MyAgg implements AggregateFunction<WaterSensor, Integer, String>{  
  
        @Override  
        public Integer createAccumulator() {  
            System.out.println("创建累加器");  
            return 0;  
        }  
  
  
        @Override  
        public Integer add(WaterSensor value, Integer accumulator) {  
            System.out.println("调用add方法,value="+value);  
            return accumulator + value.getVc();  
        }  
  
        @Override  
        public String getResult(Integer accumulator) {  
            System.out.println("调用getResult方法");  
            return accumulator.toString();  
        }  
  
        @Override  
        public Integer merge(Integer a, Integer b) {  
            System.out.println("调用merge方法");  
            return null;  
        }  
    }  
  
    // 全窗口函数的输入类型 = 增量聚合函数的输出类型  
    public static class MyProcess extends ProcessWindowFunction<String,String,String,TimeWindow>{  
  
        @Override  
        public void process(String s, Context context, Iterable<String> elements, Collector<String> out) throws Exception {  
            long startTs = context.window().getStart();  
            long endTs = context.window().getEnd();  
            String windowStart = DateFormatUtils.format(startTs, "yyyy-MM-dd HH:mm:ss.SSS");  
            String windowEnd = DateFormatUtils.format(endTs, "yyyy-MM-dd HH:mm:ss.SSS");  
  
            long count = elements.spliterator().estimateSize();  
  
            out.collect("key=" + s + "的窗口[" + windowStart + "," + windowEnd + ")包含" + count + "条数据===>" + elements.toString());  
  
        }  
    }  
}
```

### 3.6.5 其他 API

对于一个窗口算子而言，窗口分配器和窗口函数是必不可少的。除此之外，Flink还提供了其他一些可选的API，让我们可以更加灵活地控制窗口行为。

触发器、移除器： 现成的几个窗口，都有默认的实现，一般不需要自定义
#### 3.6.5.1 触发器（Trigger）

触发器主要是用来控制窗口什么时候触发计算。所谓的“触发计算”，本质上就是执行窗口函数，所以可以认为是计算得到结果并输出的过程。
基于WindowedStream调用.trigger()方法，就可以传入一个自定义的窗口触发器（Trigger）。
```java
stream.keyBy(...)
       .window(...)
       .trigger(new MyTrigger())
```
#### 3.6.5.2 移除器（Evictor）
移除器主要用来定义移除某些数据的逻辑。基于WindowedStream调用.evictor()方法，就可以传入一个自定义的移除器（Evictor）。Evictor是一个接口，不同的窗口类型都有各自预实现的移除器。
```java
stream.keyBy(...)
       .window(...)
       .evictor(new MyEvictor())
```

## 3.7 Flink 中的时间
### 3.7.1 时间语义
