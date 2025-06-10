---
title: 💎 如何用MongoDB聚合管道(pipeline)处理数据
date: 2025-06-10 22:00:00
tags:
  - mongodb
  - pipeline
categories:
  - 指南
cover: https://cdn.jsdelivr.net/gh/chendx97/CPics/img/202506102209816.png
---

MongoDB的聚合管道（Aggregation Pipeline）是一种强大的数据处理工具，允许通过多个阶段对文档进行逐步处理、转换和分析。其核心思想是将数据视为流经一系列操作的“管道”，每个阶段处理后的结果传递给下一阶段，最终输出目标数据。


![](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/202506102204461.png)


数据按照管道中的每个阶段依次流动；
每个阶段对数据进行特定的转换或筛选；
前一阶段的输出作为下一阶段的输入；


# 阶段操作符
每个阶段代表一个数据处理操作（如过滤、分组、排序等），按照顺序执行。
前一阶段的输出作为下一阶段的输入，形成处理链。

## $match
功能：**过滤**文档，类似SQL的WHERE。
```py
{
    "$match": {
        "date": {
            "$gte": beginday,
            "$lte": endday
        }
    }
},
```

## $unwind
功能：展开数组字段，将数组元素拆分为独立文档。
```py
{
    "$unwind": "$operations"
},
```
如果一条原数据中有一个长度为5的operations数组，使用$unwind可以得到5条数据。

## $group
功能：按字段分组并计算聚合值（总和、平均等），类似SQL的GROUP BY。
```py
{
    "$group": {
        "_id": "$userid",
        "email": {"$last": "$operations.email"},
        "name": {"$last": "$operations.name"},
    }
},
```
第一个键值对是分组依据，根据此字段划分出多个逻辑组。
后面几个键值对是新文档的结果，使用操作符对文档进行计算，后面的阶段只能使用此阶段声明的属性。

分组依据可以是字段、表达式，或者嵌套对象。
```py
# 字段
{
    "$group": {
        "_id": "$userid",
        "email": {"$last": "$operations.email"},
        "name": {"$last": "$operations.name"},
    }
},

# 表达式
{
    $group: {
      _id: {
        // 提取 "@" 后的域名部分
        domain: { $substr: ["$email", { $indexOfBytes: ["$email", "@"] }, -1] }
      },
      userCount: { $sum: 1 }
    }
}

# 嵌套对象
{
    "$group": {
        "_id": {
            year: { $year: "$date" },
            month: { $month: "$date" }
        },
        "email": {"$last": "$operations.email"},
        "name": {"$last": "$operations.name"},
    }
},
```

在 $group 前使用 $match 或 $project 减少处理的数据量。

## $sort
功能：排序结果，类似SQL的ORDER BY。
```py
{
    "$sort": {
        "date": -1
    }
},
```

## $project
功能：选择或重命名字段，类似SQL的SELECT。
```py
{
    "$project": {
        "_id": 0,
        "email": 1,
        "name": 1,
    }
}
```

## $limit/$skip
功能：分页查询，类似SQL的LIMIT和OFFSET。
```py
{ $sort: { amount: -1 } },   // 1. 按金额降序排序
{ $skip: 3 },                // 2. 跳过前3条（即跳过第1页）
{ $limit: 3 }                // 3. 取接下来的3条（第2页）
```

## $addFields / $set
功能：新增字段或修改现有字段（不删除其他字段）。
```py
{ 
    $addFields: { 
        tax: { 
            $multiply: ["$amount", 0.05] 
        } 
    } 
}
```

## $replaceRoot
功能：将嵌套文档提升为根文档。
```py
# 将 user.details 提升为根
{ 
    $replaceRoot: { 
        newRoot: "$details" 
    } 
}
```

## $lookup
功能：关联其他集合，类似SQL的JOIN。
```py
{ 
    $lookup: { 
        from: "products", 
        localField: "productId", 
        foreignField: "_id", 
        as: "productDetails" 
    } 
}
```

## $natural
按文档在磁盘上的物理存储顺序（自然顺序）排序。
```py
{ 
    $natural: 1 # 1是正序，-1是倒序
}
```

## 性能优化建议
- 尽早过滤：在管道前段使用$match、$project减少后续处理的数据量。
- 合理使用索引：为$match或$sort涉及的字段创建索引。
- 避免过度拆分阶段：合并可简化的操作，减少管道阶段数。

# 表达式操作符

每个阶段使用特定的操作符（如 $match、$group）定义数据处理逻辑。
操作符可操作字段、计算聚合值或转换数据结构。

## 数学操作符
常用操作符：$add, $subtract, $multiply, $divide, $mod, $floor, $ceil

$add 加法
$substract 减法
$multiply 乘法
$divide 除法
$mod 取余
$floor 向下取整
$ceil 向上取整
$toDouble 转换为浮点数

```py
# 计算订单总价（单价 × 数量）
{ 
    $project: { 
        total: { 
            $multiply: ["$price", "$quantity"] 
        } 
    } 
}
```

## 日期操作符
常用操作符：$year, $month, $dayOfMonth, $hour, $dateToString

$year 获取年份
$month 获取月份
$dayOfMonth 获取日
$hour 获取当前小时
$dateToString 转换成字符串

```py
{
  $dateToString: {
    format: "<格式字符串>",  // 必填，定义输出格式
    date: "<日期字段或表达式>", // 必填，日期字段或生成日期的表达式
    timezone: "<时区>",        // 可选，指定时区（如 "Asia/Shanghai"）
    onNull: "<替代值>"         // 可选，日期为空时的默认值（如 "N/A"）
  }
}
```

```py
# 提取订单年份和月份
{
    $project: { 
        year: { $year: "$orderDate" }, 
        month: { $month: "$orderDate" } 
    } 
}
```

## 字符串操作符
常用操作符：$substr, $concat, $toLower, $toUpper, $trim
$substr 截取字符串
$concat 拼接
$toLower 转换为小写
$toUpper 转换为大写
$trim 去掉首尾空格
$split 将字符串拆分为数组

```py
{
    $project: {
      fruits: { $split: ["$textField", ","] } // 按逗号分割
    }
}
```

```py
# 拼接用户名并转为大写
{ 
    $project: { 
        fullName: { 
            $toUpper: { $concat: ["$firstName", " ", "$lastName"] 
            } 
        } 
    }
}
```

## 逻辑操作符
常用操作符：$and, $or, $not, $cond（条件判断）

$and 逻辑与
$or 逻辑或
$not 逻辑否
$cond 条件判断
$ne 不等于指定值
$ifNull 字段值为 null 或字段不存在的情况，返回指定的默认值

```py
# 若 nickname 字段为 null 或不存在，显示为 "Anonymous"
{
    $project: {
      displayName: { $ifNull: ["$nickname", "Anonymous"] }
    }
}
```

```py
# 标记高价值订单（金额 ≥ 1000）
{ 
    $project: { 
        isHighValue: { 
            $cond: { 
                if: { $gte: ["$amount", 1000] }, 
                then: "Yes", 
                else: "No" 
            } 
        } 
    } 
}
```

```py
# 数组第一个元素是判断条件，第二个是条件为真时的取值，第三个是为假时的取值
"$cond": [
    {"$eq": ["$operations.opcode", 1001]},
    "$operations.email",
    "$$REMOVE"
]
```

## 聚合操作符
常用操作符：$sum, $avg, $max, $min, $push, $addToSet

$sum 取和
$avg 取平均数
$max 取最大值
$min 取最小值
$push 往数组中添加元素
$addToSet 往Set中添加元素，自动去重

```py
# 收集每个客户的订单 ID
{ 
    $group: { 
        _id: "$customer", 
        orderIds: { $push: "$_id" } 
    } 
}
```

## 数组操作符
常用操作符：$size, $slice, $map, $filter

$size 获取数组长度
$slice 截取数组元素
$map 遍历数组
$filter 过滤数组
$nin 不属于指定数组中的任意值

```py
# 筛选评分 ≥ 4 的评论
{ 
    $project: { 
        topReviews: { 
            $filter: { 
                input: "$reviews", 
                as: "review", 
                cond: { $gte: ["$$review.rating", 4] 
                }
            } 
        } 
    } 
}
```

# 自定义脚本操作符

## $function
$function 允许在聚合管道中执行自定义的 JavaScript 函数，用于处理复杂逻辑或实现内置操作符无法直接完成的操作。

基本语法：
```py
{
  $function: {
    body: <function>,  // JavaScript 函数
    args: [<表达式1>, <表达式2>, ...],  // 参数列表（可引用字段或计算结果）
    lang: "js"         // 目前仅支持 JavaScript
  }
}
```

举例：
```py
{
    $project: {
      formattedName: {
        $function: {
          body: function(name) {
            return name.charAt(0).toUpperCase() + name.slice(1).toLowerCase();
          },
          args: ["$name"],  // 参数为字段 name 的值
          lang: "js"
        }
      }
    }
}
```

JavaScript 执行效率低于内置操作符，避免在大数据集或高频操作中使用。
函数体必须是 单行字符串（需转义换行符）或通过 toString() 序列化。
函数参数通过 args 传递，支持聚合表达式（如 "$field"、{ $add: [...] }）。

## $accumulator
$accumulator 操作符允许在聚合管道的 $group 阶段执行自定义累加逻辑，适用于复杂的分组计算场景（如加权平均、动态数据结构维护）。

基本语法：
```py
{
  $accumulator: {
    init: <初始化函数>,          // 初始化累加状态的函数（返回初始值）
    accumulate: <累加函数>,      // 处理单个文档，更新累加状态
    accumulateArgs: [<参数列表>], // 传递给 accumulate 函数的参数（可引用字段）
    merge: <合并函数>,           // 合并不同分片/线程的累加状态
    finalize: <终止函数>,        // （可选）对最终结果进行后处理
    lang: "js"                   // 目前仅支持 JavaScript
  }
}
```

init 初始化累加器的状态（如 () => ({ sum: 0, count: 0 })）。

accumulate	对每个文档执行，更新累加状态（如 (state, value) => { state.sum += value }）。

merge	合并多个分片/并行计算的中间结果（如 (state1, state2) => { ... }）。

finalize	（可选）对最终状态进行加工（如 (state) => state.sum / state.count）。

```py
# 根据 score（分数）和 weight（权重）字段，计算分组的加权平均分
# 加权平均 = Σ(score * weight) / Σ(weight)
{
    $group: {
      _id: "$class",
      weightedAvg: {
        $accumulator: {
          init: function() { 
            return { totalScore: 0, totalWeight: 0 }; 
          },
          accumulate: function(state, score, weight) {
            state.totalScore += score * weight;
            state.totalWeight += weight;
            return state;
          },
          accumulateArgs: ["$score", "$weight"], // 传递字段值作为参数
          merge: function(state1, state2) {
            return {
              totalScore: state1.totalScore + state2.totalScore,
              totalWeight: state1.totalWeight + state2.totalWeight
            };
          },
          finalize: function(state) {
            return state.totalScore / state.totalWeight;
          },
          lang: "js"
        }
      }
    }
}
```

自定义 JavaScript 代码执行效率低于内置操作符（如 $sum、$avg）。
函数体需为字符串或通过 toString() 序列化。
在 accumulate 和 merge 中需返回新状态，避免直接修改输入状态。