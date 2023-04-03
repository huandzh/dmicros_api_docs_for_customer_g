# FAQ

## `clustedProjects`和`clusted`是什么关系？

**两个资源创建的先后顺序** :

先创建`clustedProjects`：

* 创建`clustedProjects`资源
* 创建`clusted`资源，包括`clusted_project`字段
* 需要基于项目批量使用数据时，调用一次`PATCH clustedProjects/reload-clusteds`API挂载`clusted`资源到项目下

**两个资源的关联关系修改**

mongodb不是关系数据库，这两个API提供一种双向引用关系维护机制。

但由于支持批量插入内嵌资源，这些API实际用起来调用频次会比较低

* `POST https://dmicros.iamhd.top/clustedProjects`API支持在创建project的同时批量创建内嵌的资源
* `PATCH clustedProjects/append-clusteds`API支持同步批量新增内嵌资源（如项目下样本很多则不推荐）
* `PATCH clustedProjects/reload-clusteds`主要负责应对关系挂错时做修改

## 创建 clusted 资源时，哪些数据是必填的?

`raw`字段和`v`字段是必须填写的，此外API还会自己维护必须存储的一些功能性字段，如`hashed`字段和id等，其他数据字段均为可选。
