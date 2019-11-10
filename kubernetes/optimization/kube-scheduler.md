# kube-scheduler
## 参加priority score node number
* schuduler选择bonding node:
    1. 找出该 Pod 的所有 可选节点
    2. 按照某种方式对每一个 可选节点 评分
    3. 选择评分最高的 可选节点
    4. 将最终选择结果通知 API Server，这个过程，我们称其为绑定（binding）
* 参与评分的节点的比例 node > 1000

## 挑选node自定义scheduler的实现
1. Predicates： 过滤策略的优先级问题；
2. priority: 评分权重自定义；

## pod优先调度问题：
`PriorityClass` 不同 `values` 定义将要调度的pod的优先级问题：
自定义几类的优先级：`high/medium/low`, 创建时指定字段；

## multip az 自定义调度器遍历节点的方法
拆分为两个array


## reference:
* https://www.qikqiak.com/post/kube-scheduler-introduction/
