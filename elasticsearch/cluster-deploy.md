# 集群部署方式

## 单一角色节点
* 节点类型

| Role | 备注 | node.master & node.ingest & node.data | 配置 |
| --- | --- | --- | --- | 
| Master Eligible | cluster state manager | true & false & false | 3 | 
| Data | data storage and deal client request | false & false & true | |
| Ingest | 数据处理 | false & true & false | |
| Coordinating | client node | false & false & false | oom |

## 基本部署
![](media/15734009416038.jpg)

## 扩展

### 水平扩展
![](media/15734009122730.jpg)

### 读写分离
![](media/15734008983573.jpg)

### kibana高可用
![](media/15734008636431.jpg)

### 异地多活
![](media/15734006758849.jpg)

