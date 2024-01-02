
# 0 参考资料

[Flink 从0到1实战实时风控系统](https://coding.imooc.com/class/698.html)

## 0.1 开发工具
---

1. IDEA Community Edition 2022.2.1 ( 社区版 )
2. RDM ( Redis 可视化工具, 课程使用的版本是2021.2 )
```
win下载地址：
https://sourceforge.net/projects/rdm.mirror/

mac安装包：
https://pan.baidu.com/s/10vpdhw7YfDD7G4yZCGtqQg
```

3. dbeaver ( ClickHouse,Hbase,Mysql可视化工具 )
**`注意：尽量使用的版本是 7.2.5`**
```
下载地址：
https://dbeaver.io/files/7.2.5/
```

4. 服务器:
   * a. 云服务器 (1.内存至少要 16G, 2.在安全组打开对应端口)
   * b. vmware + ubuntu
   * c. Mac OS (尽量不要用 M2以上)
   * d. win10 + wsl2 + ubuntu ( 课程使用的是这个, wsl版本是v1.2.5.0)
   * e. docker-destop (win10 or Mac)

## 0.2 相关技术栈
- Flume、Kafka
- Clickhouse
- Flink、Groovy、aviator
- Redis、HBase
- Mysql

# 1 风控引擎架构设计

## 1.1 需求分析

**羊毛党主要判断依据（组合判断）：**

- 行为目的明确：行为序列通常是"登录-领券-下单"。
- 下单商品客单价低：订单商品集中在单价为5~20元以下的商品
- 下单商品种类单一：订单商品种类超过85%为折扣商品。
- 年/月商品订单量少：只对羊毛商品感兴趣
- 下单时间通常在深夜
- 年/月登录次数少：远远小于正常用户的平均值
- 注册时间短：注册和首次下单的时间间隔小于1个小时
- 常用IP和设备更换频繁
- IP和设备出现重叠

## 1.2 架构设计
### 1.2.1 风控系统模块设计
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202312201627516.png)


### 1.2.2 业务架构图
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202312201638937.png)


### 1.2.3 应用架构图

需要划分出系统的层级，各个层级的应用服务
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202312201636722.png)

### 1.2.4 数据架构图
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202312201639917.png)
- Redis 做为指标的缓存，HBase 做为指标的持久化存储

### 1.2.5 项目结构图

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202312271437409.png)

## 1.1 风控规则引擎 

主流开源规则引擎：Groovy 和 Drools
Groovy：简单，与Java兼容
Drools：复杂

# 5 环境搭建和单元测试

## 5.1 


# 8 风控数据流入口--事件接入中心
## 8.1 事件中心架构

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20231227162508.png)


1. Flume 采集数据写入Kafka
2. Flink消费Kafka数据 ，并进行反序列化
3. ClickHouse 存储用户行为

存储用户行为路径序列（定时的脚本计算）
对事件行为指标做轻度的聚合（Flink计算）


# 9 指标计算模块

## 9.1 基于滑动窗口--风控指标采样

基于滑动窗口算法对于动态时间片快速获取数据进行计算（基于ClickHouse 或 Flink）
基于滑动窗口算法对于指定动态时间片快速获取结果（基于 Redis）

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20231228150048.png)

示例：在任何时间点，获取最近3分钟内的登录次数

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/20231228150745.png)


## 9.2 风控指标存储设计

- 任何时间点，通过Redis 快速获取到最近 n 分钟的指标采样：

**解决方案：** 通过 key（指标唯一id+采样编号），以及设置 key 的过期时间
**采样编号：** 第一份采样编号为 n-1，最后的采样编号肯定为 0，做循环，直到找不到为止
**过期时间：** 每一份的指标采样超过了 n 分钟就会自动删除
注意1：过期时间 = 指标采样的真实计算时间+n分钟
注意2：设置好指标采样的单位时间，一般设置为1分钟，否则过期时间要减去采样单位时间

指标Key = 指标 Mysql 自增id + 指标主维度 + 计算方式 + 版本号
采样Key = 指标 Mysql 自增id + 指标主维度 + 计算方式 + 版本号 ＋编号

## 9.3 Flink和POJO对象

**POJO 对象：**
1. 没有继承任何的类，没有实现任何的接口
2. 不包含业务逻辑，单纯用来存储数据
3. 每个字段都有 getter 和 setter 方法 或 成员变量是公有的

Flink 和 POJO 对象之间的关系？
1. Flink 会自动分析 POJO 对象的结构，获取 POJO 对象的字段
2. Flink 处理 POJO 对象比一般的对象类型更高效


## 9.4 基于Flink实现指标通用计算


# 10 规则引擎

## 10.1 风控规则设计
* a. 风控场景和行为事件是多对多, 因为课程主要针对优惠券风控场景, 所以没有创建风控场景和行为事件的多对多关系表
* b. 风控规则和风控场景是单对单的关系
* c. 策略和执行动作是多对多的关系


1. rule ( 原子规则表 )

|字段名 | 字段含义 | 字段类型 | 备注|
| -- | -- | -- | -- |
| auto_id | 自增id | int |
| rule_code | 规则编码 | varchar |规则唯一标识
| version | 规则版本 | varchar |
| rule_name | 规则名称 | varchar |
| rule_type | 规则类型 | varchar | 
| condition_id | 规则条件id | int | 
| scene | 风控场景 | varchar |
| event | 行为事件 (多个) | varchar |以,隔开
| is_enable | 规则是否开启 | varchar |

2. rule_condition ( 规则条件表 )

| 字段名 | 字段含义 | 字段类型 |
| -- | -- | -- |
| auto_id | 自增id | int |
| condition_id | 条件id | int |
| rule_code | 规则编码 | varchar |
| condition_no | 规则编号 | int |对应逻辑运算表表达式内的编号
| metric | 指标 | varchar | 课程就不用风控指标属性表表的id,直接字符串
| threshold | 阈值 | varchar |
| operator | 关系运算符 | varchar | 课程就不用运算符表的id,直接字符串
| expression | 条件表达式 | varchar |


3. rule_set ( 规则组表 )

|字段名 | 字段含义 | 字段类型 | 备注|
| -- | -- | -- | -- |
| auto_id | 自增id | int |
| set_code | 规则组编码 | varchar |规则组唯一标识
| rule_code | 规则编码 | varchar |
| rule_no | 规则编号 | int | 对应逻辑运算表表达式内的编号
| rule_set_name | 规则组名称 | varchar |


## 10.2 字符串表达式解析

1. SpEI
2. Aviator 表达式引擎：性能好
3. Groovy
4. Jpxl：解释型，不支持对内部类的调用

# 11 实时风控--动态规则

## 11.1 Flink-Cep
什么是 Cep？
在流式数据中（事件流），筛选出符合条件的一系列动作（事件）


什么是 Pattern ？
Pattern 就是 Cep 里的规则制定
Pattern 分为 个体模式，组合模式（模式序列）和模式组
模式组是将组合模式作为条件的个体模式


Cep 开发流程？
1. DataStream 或 KeyedStream
2. 定义规则（Pattern）
3. 将规则应用于 KeyedStream，生成 PatternStream
4. 将 PatternStream，通过 Select 方法，将符合规则的数据输出