# 离线数据分析使用指南

针对大规模数据的分析任务一般都比较耗时。LeanCloud 为开发者提供了部分兼容 SQL 语法的离线数据分析功能。离线数据分析的数据来源是**前一天的数据备份**，并非最新的在线数据，在这里进行的任何操作都不会对应用的线上数据产生影响。

当一个应用的所有数据表（即存储功能中的 Class）的总记录条数超过 10,000 时，离线数据分析功能将自动开启。云端准备离线数据的过程一般会持续数分钟或更长时间。记录数少于一万而无法使用离线数据分析的应用，可以使用 API 或直接在 [数据存储页面](/data.html?appid={{appid}}#/) 进行处理。

离线数据分析页面的访问路径为 [控制台 > **存储** > **离线数据分析**](/dataquery.html?appid={{appid}}#/)。如果该功能不能正常使用，请通过 [工单系统](https://leanticket.cn/) 或 [用户论坛](https://forum.leancloud.cn) 联系我们。

## 类似 SQL 的查询分析语法

LeanCloud 的离线数据分析服务基于 Spark SQL，目前支持 HiveQL 的功能子集，常用的 HiveQL 功能都能正常使用，例如：

### Hive 查询语法

* SELECT
* GROUP BY
* ORDER BY
* CLUSTER BY
* SORT BY

**针对 `GROUP BY` 的特别说明**：

有不少用户都有过 MySQL 的使用经验，这里主要是列举几种在 MySQL 中可用而在我们的服务（基于 Spark SQL）中却**会报错的 `GROUP BY` 用法**。

```
SELECT * FROM table GROUP BY columnA;
```

* MySQL：如果 columnA 这个字段有 10 种不同的值，那么这条查询语句得到的结果应该包含 10 行记录。
* Spark SQL：如果 table 只有 columnA 这个字段，那么查询结果和 MySQL 相同。相反，如果 table 包含不止 columnA 这个字段，查询会报错。


```
SELECT columnB FROM table GROUP BY columnA;
```

* MySQL：同前一条查询差不多，返回正确结果。结果记录数由 columnA 值的种类决定。
* Spark SQL：一定会报错。

当且仅当 SELECT 后面的表达式（expressions）为聚合函数（aggregation function）或包含 `GROUP BY` 中的字段，Spark SQL 的查询才会合法。列举几个常见的合法查询：

```
SELECT columnA, count(columnB) as `count` FROM table GROUP BY columnA
SELECT columnA, count(*) as `count` FROM table GROUP BY columnA
SELECT columnA, columnB FROM table GROUP BY columnA, columnB
```

### Hive 运算符

* 关系运算符（`= ⇔ == <> < > >= <=`  ...）
* 算术运算符（`+ - * / %`  ...）
* 逻辑运算符（`AND && OR ||`  ...）

### 常用函数

#### 字符串类

函数|描述
:---|:---
`instr`|返回一个字符串在另一个字符串中首次出现的位置
`length`|字符串长度
`printf`|格式化输出
...|...

#### 计算类

函数|描述
:---|:---
`abs`|绝对值
`acos`|反余弦
`asin`|反正弦
`bin`|二进制
`ceil`|向上取整
`ceiling`|向上取整
`conv`|进制转换
`cos`|余弦
`exp`|自然指数
`floor`|向下取整
`hex`|十六进制
`ln`|自然对数
`log`|对数
`log10`|以 10 为底对数
`log2`|以 2 为底对数
`negative`|negative 
`pmod`|正取余
`positive`|positive 
`pow`|幂运算
`power`|幂运算
`rand`|取随机数
`round`|取整
`sin`|正弦
`sqrt`|开平方
`unhex`|反转十六进制
...|...

#### 日期类

函数|描述
:---|:---
`date_add`|日期增加
`date_sub`|日期减少
`datediff`|日期比较
`day`|日期转天
`from_unixtime`|UNIX 时间戳转日期
`hour`|日期转小时
`minute`|日期转分钟
`month`|日期转月
`second`|日期转秒
`to_date`|日期时间转日期
`unix_timestamp`|获取当前 UNIX 时间戳
`unix_timestamp`|日期转 UNIX 时间戳
`unix_timestamp`|指定格式日期转 UNIX 时间戳
`weekofyear`|日期转周
`year`|日期转年
...|...

更详尽的 Hive 运算符和内置函数，可以参考 [Hive Language Manual - Built-in Operators](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-Built-inOperators)。

### 多表 Join

```
JOIN
{LEFT|RIGHT|FULL} OUTER JOIN
LEFT SEMI JOIN
CROSS JOIN
```

### 子查询

```
SELECT col FROM ( SELECT a + b AS col from t1) t2
```

### Hive 数据类型

* TINYINT
* SMALLINT
* INT
* BIGINT
* BOOLEAN
* FLOAT
* DOUBLE
* STRING
* BINARY
* TIMESTAMP
* ARRAY<>
* MAP<>
* STRUCT<>

详细信息请参考 [Spark SQL Supported Hive Features](http://spark.apache.org/docs/latest/sql-programming-guide.html#supported-hive-features)。

不支持的 Hive 功能，请参考 [Spark SQL Unsupported Hive Functionality](http://spark.apache.org/docs/latest/sql-programming-guide.html#unsupported-hive-functionality)。

### 数据分析举例

* 简单的 SELECT 查询

```
select * from Post

select count(*) from _User
```

* 复杂的 SELECT 查询

```
select * from Post where createdAt > '2014-12-10'

select avg(age) from _User

select Post.objectId from Post left join _User where _User.name=Post.pubUser limit 10

select * from _User where name in (select name form OtherUser)

select sum(upvotes) from Post

select count(*) as `count`, pubUser from Post group by pubUser
```

更多例子可以参考我们的博客文章[《LeanCloud 离线数据分析功能介绍》](https://blog.leancloud.cn/2559/)。

## 云引擎和 JavaScript SDK 对离线分析的支持

JavaScript SDK 0.5.5 版本开始支持离线数据分析。**请注意，离线数据分析要求使用 Master Key，否则下面所述内容都没有权限运行，请参考 [《权限说明》](leanengine_guide-cloudcode.html#权限说明)。**

### Job 启动

```js
  AV.Insight.startJob({
        sql: "select * from `_User`",
        saveAs: {
          className: 'InsightResult',
          limit:1
        }
   }).done(function(id) {
      //返回 job id
   }).catch(function(err)){
      //发生错误
   });
```

`AV.Insight.startJob` 启动一个离线分析任务，它可以指定：

* **sql**：本次任务的查询的 SQL。
* **saveAs**：（可选）本次任务查询结果保存的参数，比如要保存的表名和数量，limit 最大为 1000。

任务如果能正常启动，将返回任务的 job id，后续可以拿这个 id 去查询任务状态和结果。


### Job 完成

在云引擎里，可以通过一个 hook 函数来监听 job 完成情况：

```js
AV.Insight.on('end', function(err, result) {
    console.dir(result);
});
 ```

目前仅支持 `end` 事件，当 job 完成（可能成功或者失败）就通知到这个 hook 函数，结果 result 里会包含 job id 以及任务状态信息：

```
{  
  "id":        "29f24a909074453622856528359caddc",
  "startTime": 1435569183000,
  "endTime":   1435569185000,
  "status":    "OK"
}
```

如果 status 是 `OK` ，表示任务成功，其他状态包括 `RUNNING` 表示正在运行，以及 `ERROR` 表示本次任务失败，并将返回失败信息 `message`。

如果任务成功，你可以拿 id 去主动查询任务结果，参见下文。

### Job 状态和结果查询

在知道任务 id 的情况下（startJob 返回或者云引擎监听到任务完成），可以主动查询本次任务的结果：

```js
  var id = '已知任务 id';
  var q = new AV.Insight.JobQuery(id);
  q.find().then(function(result) {
    //返回查询结果 result 对象
  }, function(err) {
    //查询失败
  });
```

result 是一个 JSON 对象，形如：

```js
{  
  "id":         "976c94ef0847f4ff3a65e661bf7b809a", //任务 id 
  "status":     "OK", //任务状态 
  "totalCount": 50, //结果总数 
  "results":[  
     ……结果数组……
  ]
}
```

`AV.Insight.JobQuery` 也可以设置 `skip` 和 `limit` 做分页查询。
