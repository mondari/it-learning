# InfluxDB 数据元素

InfluxDB 2.0 包含以下数据元素:

- [timestamp](https://docs.influxdata.com/influxdb/v2.0/reference/key-concepts/data-elements/#timestamp)
- [field key](https://docs.influxdata.com/influxdb/v2.0/reference/key-concepts/data-elements/#field-key)
- [field value](https://docs.influxdata.com/influxdb/v2.0/reference/key-concepts/data-elements/#field-value)
- [field set](https://docs.influxdata.com/influxdb/v2.0/reference/key-concepts/data-elements/#field-set)
- [tag key](https://docs.influxdata.com/influxdb/v2.0/reference/key-concepts/data-elements/#tag-key)
- [tag value](https://docs.influxdata.com/influxdb/v2.0/reference/key-concepts/data-elements/#tag-value)
- [tag set](https://docs.influxdata.com/influxdb/v2.0/reference/key-concepts/data-elements/#tag-set)
- [measurement](https://docs.influxdata.com/influxdb/v2.0/reference/key-concepts/data-elements/#measurement)
- [series](https://docs.influxdata.com/influxdb/v2.0/reference/key-concepts/data-elements/#series)
- [point](https://docs.influxdata.com/influxdb/v2.0/reference/key-concepts/data-elements/#point)
- [bucket](https://docs.influxdata.com/influxdb/v2.0/reference/key-concepts/data-elements/#bucket)
- [organization](https://docs.influxdata.com/influxdb/v2.0/reference/key-concepts/data-elements/#organization)

## 示例数据

为了方便讲解数据元素概念，以下给出示例数据。

**bucket:** `my_bucket`

| _time                | _measurement | location | scientist | _field | _value |
| :------------------- | :----------- | :------- | :-------- | :----- | :----- |
| 2019-08-18T00:00:00Z | census       | klamath  | anderson  | bees   | 23     |
| 2019-08-18T00:00:00Z | census       | portland | mullen    | ants   | 30     |
| 2019-08-18T00:06:00Z | census       | klamath  | anderson  | bees   | 28     |
| 2019-08-18T00:06:00Z | census       | portland | mullen    | ants   | 32     |

更多示例数据参见：https://docs.influxdata.com/influxdb/v2.0/reference/sample-data

## 时间戳 Timestamp

表示数据的日期和时间，默认精度是纳秒，按照 [RFC3339](https://docs.influxdata.com/influxdb/v2.0/reference/glossary/#rfc3339-timestamp) UTC 标准时间存储。

示例数据中的 `_time` 列就是时间戳。

## 测量 Measurement

**测量**相当于关系数据库中表的概念。

**测量**是时间戳(Timestamp)、标签(Tags)、字段(Fields)的容器。

示例数据中的 `_measurement` 列就是**测量**的名称。

## 字段 Fields

字段包括存储在 `_field` 列中的字段键(Field key)和存储在 `_value` 列中的字段值(Field value)。

**字段是不会被索引的，且字段在 InfluxDB 中是必需的。**过滤字段值的查询必须扫描所有字段值以匹配查询条件。因此，对标签(Tags)的查询比对字段的查询性能更好。通常将常用查询的元数据存储在标签(Tags)中。

### 字段键 Field key

字段键是表示字段名称的字符串。在上面的示例数据中， `bees` 和 `ants` 是字段键。

### 字段值 Field value

字段值表示关联字段键的值。在上面的示例数据中，`bees` 字段键的字段值是23和28，`ants` 字段键的字段 值是30和32。

### 字段集 Field set

字段集是与时间戳相关联的字段键值对的集合。示例数据中包括以下字段集：

```bash
census bees=23i,ants=30i 1566086400000000000
census bees=28i,ants=32i 1566086760000000000
       -----------------
           Field set
```

## 标签 Tags

示例数据中的 `location` 和 `scientist` 列就是标签。标签包含标签键(Tag key) 和标签值(Tag value)。

**标签是有索引的，且标签在 InfluxDB 中是非必需的。** 对标签的查询要比对字段的查询快，所以标签非常适合存储常见查询的元数据。

用标签来存储高度可变的信息 (比如 UUID、散列和随机字符串) 将导致数据库中出现大量的唯一序列，称为高序列基数 (high series cardinality) 。对于大多数数据库来说，高系列基数是导致内存占用变高的主要因素。

### 标签键 Tag key

示例数据中的 `location` 和 `scientist` 就是标签键。

### 标签值 Tag value

示例数据中标签键 `location` 有两个标签值 `klamath` 和 `portland` ，标签键 `scientist` 同样也有两个标签值 `anderson` 和 `mullen` 。

### 标签集 Tag set

标签键值对的集合组成一个标签集。示例数据包括以下四个标签集：

```bash
location = klamath, scientist = anderson
location = portland, scientist = anderson
location = klamath, scientist = mullen
location = portland, scientist = mullen
```

## 时间序列 Series

### 时间序列键 Series Key

时间序列键包含测量名称、标签集合、字段键。

例如，示例数据中包括两个唯一的时间序列键：

| _measurement | tag set                             | _field |
| :----------- | :---------------------------------- | :----- |
| census       | location=klamath,scientist=anderson | bees   |
| census       | location=portland,scientist=mullen  | ants   |

### 时间序列 Series

时间序列是时间序列键的时间戳和字段值。示例数据中有一个时间系列键和对应的时间系列

```bash
# series key
census,location=klamath,scientist=anderson bees

# series
2019-08-18T00:00:00Z 23
2019-08-18T00:06:00Z 28        
```

## 时序记录 Point

时序记录相当于关系数据库中的行记录的概念。

示例数据中其中一条时序记录如下：

```
2019-08-18T00:06:00Z  census  portland  mullen  ants  32
```

## 保留策略 Retention Policy (RP)

定义了时序记录在数据库的保留时长，超过时长将被删除。

这是 InfluxDB 1.x 的概念，在 InfluxDB 2.0 中合并进 Bucket 中。

参考：

https://docs.influxdata.com/influxdb/v1.8/concepts/glossary/#retention-policy-rp

## 桶 Bucket

InfluxDB 1.x 中数据存储在数据库和保留策略中（两者合起来叫 DBRP）。而在 InfluxDB 2.0 中，数据存储在桶中。桶结合了数据库和保留策略的概念。

## 组织 Organization

组织是一组用户的工作空间。所有的仪表板(Dashboard)、任务(Task)、桶和用户都属于一个组织。

## 参考

https://docs.influxdata.com/influxdb/v2.0/reference/key-concepts/data-elements

# InfluxDB 其它概念

## InfluxDB 1.x TICK 监控系统架构图

![Influx-1.0-Diagram](InfluxDB.assets\Influx-1.0-Diagram.png)

参考：https://www.influxdata.com/time-series-platform/

## InfluxDB 2.0 监控系统架构图

![APM diagram](InfluxDB.assets\APM-Diagram-2.png)

参考：https://www.influxdata.com/products/influxdb/

## Telegraf

Telegraf 是 Influxdata 公司的数据收集代理(Agent)，用于收集和报告指标。它丰富的数据收集插件和即插即用架构让您可以快速、轻松地从许多不同的来源收集指标。

## 数据抓取器 Scraper

Scraper 负责定期从指定的目标 URL 收集数据，然后将收集到的数据写入桶中。目标 URL 以Prometheus 数据格式提供数据。

参考：https://docs.influxdata.com/influxdb/v2.0/write-data/no-code/scrape-data/

## InfluxQL

InfluxQL 是 InfluxDB 的类 SQL 查询语言，在 InfluxDB 2.0 后被 Flux 替代。

参考：

https://docs.influxdata.com/influxdb/v2.0/query-data/influxql/

https://docs.influxdata.com/influxdb/v2.0/reference/flux/flux-vs-influxql/

## Flux

Flux 是 Influxdata 公司的函数式脚本语言，用于查询、分析和处理数据。

Flux 是 InfluxQL 的替代品。

参考：https://docs.influxdata.com/influxdb/v2.0/query-data/get-started/

## 连续查询 Continuous Query (CQ)

连续查询是在数据库中自动定期运行的查询。连续查询需要在 `SELECT` 子句中的使用函数，并且必须包含 `GROUP BY time()` 子句。

这是 InfluxDB 1.x 的概念，在 InfluxDB 2.0 中被 Task 替代。

参考：

https://docs.influxdata.com/influxdb/v1.8/concepts/glossary/#continuous-query-cq

https://docs.influxdata.com/influxdb/v1.8/query_language/continuous_queries/

## Task

Task 是一个被定时调度的 Flux 脚本，用于分析和处理输入的数据流，然后将结果存储在一个新的桶中或执行其他操作。

参考：https://docs.influxdata.com/influxdb/v2.0/process-data/get-started/

