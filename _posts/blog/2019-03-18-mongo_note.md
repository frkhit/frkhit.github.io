---
layout: post
title: mongo学习笔记
category: 技术
tags: mongo
keywords: 
description: 
---

# mongo学习笔记

## 1. mongo document学习笔记
### 1.1 BSON 类型
- ObjectId: 快速, 有序, 时间相关
- String
- Timestamps
- Date


### 1.2 Document类型
- 字段(field)有长度限制: 如field name不超过128, 等
- Dot Notation: `<array>.<index>`, `<embedded document>.<field>`
- 单文档有大小限制:16MB

### 1.3 聚合
```MongoDB provides three ways to perform aggregation: the aggregation pipeline, the map-reduce function, and single purpose aggregation methods.```

聚合字段:
- `$match`: 匹配, `{ $match: { name: "Joe Schmoe" } }`
- `unwind`: 打散,针对array, `{ $unwind: "$resultingArray"}`
- `$project`: 投射, `{"$project":{"author":1,"_id":0} #只提前author`
- `$redact`: 校验, `{ $redact: { $cond: { if: { $eq: [ "$level", 5 ] }, then: "$$PRUNE", else: "$$DESCEND" } } }`
- `$skip`: 跳过, `{ $skip: 5 }`
- `$lookup`: 跨表检索, ```{
  $lookup: {
    from: "otherCollection",
    as: "resultingArray",
    localField: "x",
    foreignField: "y"
  }
}```

**Note:**
- 各字段配合的优化, 参考[文档](https://docs.mongodb.com/manual/core/aggregation-pipeline-optimization/)

聚合限制:
- `Result Size Restrictions`: 单doc <= 16MB
- `Memory Restrictions`: `Pipeline stages have a limit of 100 megabytes of RAM`; `The $graphLookup stage must stay within the 100 megabyte memory limit.`

todo: [聚合操作zip code data set(经纬度)](https://docs.mongodb.com/manual/tutorial/aggregation-zip-code-data-set/)

[Aggregation with User Preference Data](https://docs.mongodb.com/manual/tutorial/aggregation-with-user-preference-data/), 利用用户信息表举例:
- 获取所有员工名称
- 根据加入时间返回员工名称
- 获取每个月新加入的人数
- 获取前五个最受欢迎的爱好

### 1.4 检索

条件检索
- `$or`: `cursor = db.inventory.find({"$or": [{"status": "A"}, {"qty": {"$lt": 30}}]})`
- `$and`

检索列表:
- `Match an Array`: `db.inventory.find({"tags": ["red", "blank"]}) # tags == ["red", "blank"]`; `db.inventory.find({"tags": {"$all": ["red", "blank"]}}) # tags 同时存在"red","blank"两个元素`
- `Query an Array with Compound Filter Conditions on the Array Elements`: `db.inventory.find({"dim_cm": {"$gt": 15, "$lt": 20}}) # 15<x<20; x>15, y<20`
- `Query for an Array Element that Meets Multiple Criteria`: `db.inventory.find({"dim_cm": {"$elemMatch": {"$gt": 22, "$lt": 30}}}) # 其中有一个元素22<x<30`
- `Query for an Element by the Array Index Position`: `db.inventory.find({"dim_cm.1": {"$gt": 25}})`
- `Query an Array by Array Length`: `db.inventory.find({"tags": {"$size": 3}})`

Project：
- `db.inventory.find({"status": "A"}, {"item": 1, "status": 1})` means `SELECT _id, item, status from inventory WHERE status = "A"`
- `db.inventory.find({"status": "A"}, {"item": 1, "status": 1, "_id": 0})` means `SELECT item, status from inventory WHERE status = "A"`
- `db.inventory.find({"status": "A"}, {"status": 0, "instock": 0})` means `return All except for status and instock`


其他
- `db.inventory.find({"item": None})`: `None 或 不存在`
- `db.inventory.find({"item": {"$type": 10}})`: 类型检查, 查询值为None的记录
- `db.inventory.find({"item": {"$exists": False}})`: 不存在


## 2.聚合操作

### 2.1 获取列表元素集合

keywords: group, unwind, aggregate

举例,有如下数据:
```
{"_id": "1", "tags": ["a", "b"]}
{"_id": "2", "tags": ["a", "b", "c"]}
{"_id": "3", "tags": []}
{"_id": "4", "tags": ["c", "d"]}
```
求tags元素集合?

方法:

```
db.test.aggregate([
    {"$unwind": "$tags"},
    {"$group": {"_id": "$tags"}},
])
```


## 3.update操作

### 3.1 修改列表元素的值
keywords: update_many

举例,有如下数据:
```
{"_id": "1", "tags": ["a", "b"]}
{"_id": "2", "tags": ["a", "b", "c"]}
{"_id": "3", "tags": []}
{"_id": "4", "tags": ["c", "d"]}
```
将`tags`中"a"修改为"A"? 

方法:

```
self.db["test"].update_many(
    filter={"tags": "a"},
    update={"$set": {'tags.$': "A"}},
    upsert=False,
    )
    
# 一次操作只能修改一个值
# 如果tags中存在多个"a", 需要多次执行以上代码
```

### 3.2 删除列表元素的值
keywords: update_many

举例,有如下数据:
```
{"_id": "1", "tags": ["a", "b"]}
{"_id": "2", "tags": ["a", "b", "c"]}
{"_id": "3", "tags": []}
{"_id": "4", "tags": ["c", "d"]}
{"_id": "5", "tags": ["c", "d", "a", "a"]}
```
将`tags`中所有"a"删除?

方法:

```
tag_list = ["a"]
self.db["test"].update_many(
    filter={"tags": {"$in": tag_list}},
    update={"$pull": {'tags': {"$in": tag_list}}},
    upsert=False,
)
# 执行后, _id = "5"的记录, tags中两个"a"均被删除
```