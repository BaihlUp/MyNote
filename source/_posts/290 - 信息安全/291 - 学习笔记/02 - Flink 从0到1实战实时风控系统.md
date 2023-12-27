
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

