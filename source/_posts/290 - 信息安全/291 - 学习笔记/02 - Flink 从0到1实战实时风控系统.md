
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

![[Pasted image 20231220160848.png]]


**风控系统模块：**
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202312201627516.png)


业务架构图
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202312201638937.png)

应用架构图：需要划分出系统的层级，各个层级的应用服务
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202312201636722.png)

数据架构图
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2023/202312201639917.png)
- Redis 指标的缓存存储，HBase 指标的持久化存储

技术架构图

## 1.1 风控规则引擎 

主流开源规则引擎：Groovy 和 Drools
Groovy：简单，与Java兼容
Drools：复杂




