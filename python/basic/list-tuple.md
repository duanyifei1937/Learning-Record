# list & tuple

## 相同点和区别
* 相同
数据结构的有序集合

* 区别

|  | list | tuple |
| --- | --- | --- |
| 变化 | 可变(动态) | 不可变(静态) |
| 存储空间 | 额外存放指针，over-allocating机制 | 轻量级 |
| 性能 | ~ | 优(不变内容可使用资源缓存) |
| 应用 | 元素可变 | 返回数据、数量不变 |


## 支持操作
* 负数索引
* 切片
* 多层嵌套
* count
* index
* list.reverse() & list.soft()
* reversed() & sorted()

