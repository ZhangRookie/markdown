#### explain sql

---

| 列名          | 描述                                           |
| ------------- | :--------------------------------------------- |
| id            | 一个大查询每个select都对应一个id               |
| select_type   | select关键字对应的查询类型                     |
| table         | 表名                                           |
| partitions    | 匹配的分区信息                                 |
| type          | 针对单表的访问方式                             |
| possible_keys | 可能用到的索引                                 |
| key           | 实际用到的索引                                 |
| key_len       | 实际用到的索引长度                             |
| ref           | 使用索引列等值查询，与索引列等值匹配的对象信息 |
| rows          | 预估需要读取的记录条数                         |
| filtered      | 某个表经过条件过滤后剩余记录的条数百分比       |
| extra         | 额外信息                                       |
|               |                                                |

__id：__ 对于连接查询，from子句可以跟多个表，对应执行计划中多条记录，id相同。<u> 出现在前面的是驱动表，后面的是被驱动表</u>  对于子查询，每个select对应一个id <u> 查询优化器可能会对嵌套查询优化为关联查询，id相同</u> 

__select_type:__  

- simple: 查询语句中不包含union、子查询的 包含连接
- primary: union、union all 最左边的查询
- union:  union\union all 除最左边的查询都是union
- union result: union产生的临时表用来去重的select_type
- subquery: 

__type:__ 对表访问的方法

- const:根据主键或唯一二级索引进行访问
- eq_ref:连接查询被驱动表通过主键或唯一二级索引等值访问，被驱动表访问方式为eq_ref
- ref:通过普通二级索引与常量进行等值匹配
- index:使用索引覆盖，但需要扫描全部索引记录
- range:使用索引进行某些范围查询，如in或> <

__possible_keys和key__ 

​	possible_key 可能用到哪些索引，key表示经过执行器通过成本计算，实际用到的索引

​	<u>通过index方式访问时possible_keys为空，key为实际索引</u>

__ref:__ 当使用索引列进行等值访问时(const、eq_ref、ref、ref_or_null...)，ref的值为索引列等值匹配的值是什么

- const: 匹配的值为常量  如：‘a’
- 某个表的列名 如：DB1.T1.id
- func 函数 