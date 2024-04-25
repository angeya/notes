## 简介

**InfluxData** 平台是领先的现代[时序](https://docs.influxdata.com/platform/faq/#what-is-time-series-data)平台，专为指标和事件而构建。有 **InfluxData2.x**和**InfluxData1.x**版本，**InfluxData2.x**在性能和功能上有很大提升，并且设计理念和API也和**InfluxData1.x**提供了不太一样，这里主要介绍**InfluxData2.x**。InfluxData2.x和InfluxData1.x的区别如下：

|          | InfluxData1.x                                                | InfluxData2.x                                                |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 登录凭证 | 用户名密码                                                   | token                                                        |
| 查询语言 | [InfluxQL](https://docs.influxdata.com/influxdb/v2/get-started/query/?t=#influxql-query-basics)：<br/>一种类似 SQL 的查询语言，旨在查询来自 InfluxDB。 | [Flux ](https://docs.influxdata.com/influxdb/v2/get-started/query/?t=#execute-a-flux-query)：<br/>一种用于查询和处理数据的函数式脚本语言，来自 InfluxDB 和其他数据源。 |
| UI界面   | 不提供                                                       | http://127.0.0.1:8086/                                       |
| 数据组织 | Database                                                     | Organization和Bucket                                         |

**InfluxData2.x**的官方文档地址为：https://docs.influxdata.com/influxdb/v2/。

**InfluxData2.x**提供了图形用户界面，在安装完成之后可以使用浏览器访问 http://127.0.0.1:8086/ 进行初始化设置。

## 一些概念

在开始使用 InfluxDB 之前，请务必了解时间序列如何 数据被组织并存储在 InfluxDB 和一些使用的关键定义中 贯穿本文档。

### 数据组织

InfluxDB 数据模型将时间序列数据组织到存储桶和度量中。 一个存储桶可以包含多个测量值。测量值包含多个 标记和字段。

- **存储桶(Bucket)**：存储时序数据的命名位置。 一个存储桶可以包含多个*测量值*。
  - **度量(Measurement)**：时间序列数据的逻辑分组。 给定测量中的所有*点*都应具有相同的*标签*。 测量包含多个*标签*和*字段*。
    - **标签(tag)**：值不同但不经常更改的键值对。 标签用于存储每个点的元数据，例如， 用于识别数据源的东西，例如主机、位置、站点等。有索引。
    - **字段(field)**：键值对，其值随时间变化，例如：温度、压力、股票价格等。
    - **时间戳(timestamp)**：与数据关联的时间戳。 当存储在磁盘上并进行查询时，所有数据都按时间排序。

*有关 InfluxDB 数据模型的详细信息和示例，请参阅[数据元素](https://docs.influxdata.com/influxdb/v2/reference/key-concepts/data-elements/)。*

### 重要定义

以下是使用 InfluxDB 时需要了解的重要定义：

- **点**：由测量值、标签键、*标签值、字段键和时间戳*标识的单个数据记录。
- **系列**：具有相同*测量值、标签键和标签值*的一组点。

**InfluxDB 查询结果示例，下面测量值为天气，标签键为城市和国家**

![image-20240103162258133](InFluxDB学习笔记.assets\image-20240103162258133.png)

![image-20240103162349281](InFluxDB学习笔记.assets\image-20240103162349281.png)

## 图形用户界面的使用

第一次进入用户界面，会提示需要设置用户名，密码，组织和桶等信息。

### 写入数据

Load Data -> Buckets -> 选择需要的桶 ADD DATA -> Line Protocol -> ENTER MANUALLY

然后通过输入行协议的数据，点击 **WRITE DATA** 按钮即可。

**行协议元素：**

每行线路协议都包含以下元素：

**必填*

- **measurement**：标识要在其中存储数据的度量值的字符串。
- **tag set**：以逗号分隔的键值对列表，每个键值对代表一个标签。 标签键和值是不带引号的字符串。*空格、逗号和相等字符必须转义。*
- **字段集**：逗号分隔的列表键值对，每个键值对代表一个字段。 字段键是不带引号的字符串。*空格和逗号必须转义。*字段值可以是字符串（带引号）、浮点数、整数、无符号整数、 或布尔值。
- **timestamp**：与数据关联的 [Unix 时间戳](https://docs.influxdata.com/influxdb/v2/reference/syntax/line-protocol/#unix-timestamp)。InfluxDB 支持高达纳秒级的精度。如果时间戳的精度不是以纳秒为单位，则必须指定 将数据写入 InfluxDB 时的精度。

#### [行协议元素解析](https://docs.influxdata.com/influxdb/v2/get-started/write/#line-protocol-element-parsing)

- **measurement**：第一个空格之前第一个未转义逗号之前的所有内容。
- **tag set**：第一个未转义逗号和第一个未转义空格之间的键值对。
- **字段集**：第一个和第二个未转义空格之间的键值对。
- **timestamp**：第二个未转义空格之后的整数值。
- 行之间用换行符 `\n` 分隔。 线路协议区分空格。

Syntax:

```
measurementName,tagKey=tagValue fieldKey="fieldValue" 1465839830100400200
--------------- --------------- --------------------- -------------------
       |               |                  |                    |
  Measurement       Tag set           Field set            Timestamp
```

Example（采集客厅和厨房的温度和声音）:

```
home,room=Living\ Room temp=21.1,hum=35.9,co=0i 1641024000
home,room=Kitchen temp=21.0,hum=35.9,co=0i 1641024000
home,room=Living\ Room temp=21.4,hum=35.9,co=0i 1641027600
home,room=Kitchen temp=23.0,hum=36.2,co=0i 1641027600
home,room=Living\ Room temp=21.8,hum=36.0,co=0i 1641031200
home,room=Kitchen temp=22.7,hum=36.1,co=0i 1641031200
```

### 查询数据

Data Explorer中可以使用Query Builder或者Script Editor进行查询。

使用 Flux 脚本查询 InfluxDB 时，您主要使用三个函数：

- [from()](https://docs.influxdata.com/flux/v0/stdlib/influxdata/influxdb/from/)： 查询InfluxDB数据。如`from(bucket: "demo-bucket")`

- [range()](https://docs.influxdata.com/flux/v0/stdlib/universe/range/)： 根据时间范围筛选数据。Flux 需要“有界”查询 — 查询 限制在特定的时间范围内。可以使用相对时间或者绝对时间。

  如：相对时间到此刻`|> range(start: -12h)`

  绝对时间`|> range(start: 2021-05-22T23:30:00Z, stop: 2021-05-23T00:00:00Z)`或者时间戳`|> range(start: 1621726200, stop: 1621728000)`

- [filter()](https://docs.influxdata.com/flux/v0/stdlib/universe/filter/)： 根据列值筛选数据。每行都由`r` 表示，每列由 `r`的属性表示。 您可以应用多个后续过滤器。如`|> filter(fn: (r) => r._measurement == "home")`

> 注意：当一个测量中包含多个字段(field)，使用图形用户界面查询以表格展示时，每一行只会显示一个字段键和值，分别在`_field`列和`_value`列。并且会以字段名称和时间归类成列表展示在左侧。因此有时候在查询的时候，返回的结果是`List<FluxTable>`。

## Influx CLI的使用

Influx CLI是InfluxDB的命令行客户端，支持使用命令来管理InfluxDB。

Influx CLI安装及使用链接：https://docs.influxdata.com/influxdb/cloud/reference/cli/influx/。InfluxDB的Docker容器中默认已经包含了Influx CLI。

### 写入数据

网络CSV文件

```
influx write --bucket sample-bucket --url https://influx-testdata.s3.amazonaws.com/air-sensor-data-annotated.csv
```

本地CSV文件

```
influx write --bucket sample-bucket --file <YOUR_PATH>/air-sensor-data-annotated.csv
```

csv的内容格式如下：

```
#group,false,false,false,false,true,true,true
#datatype,string,long,dateTime:RFC3339,double,string,string,string
#default,_result,,,,,,
,result,table,_time,_value,_field,_measurement,sensor_id
,,0,2024-01-03T23:19:57Z,0.4938242960528907,co,airSensors,TLM0100
,,0,2024-01-03T23:20:07Z,0.503711643685156,co,airSensors,TLM0100
,,0,2024-01-03T23:20:17Z,0.4901465934432477,co,airSensors,TLM0100
,,0,2024-01-03T23:20:27Z,0.498262663160897,co,airSensors,TLM0100
```

### 查询和删除数据

根据目前的经验，官方给出的命令总是报错，不知道为什么。

#### Java中使用InfluxDB2.x

1. 引入maven依赖

   ```xml
   <dependency>
   	<groupId>com.influxdb</groupId>
   	<artifactId>influxdb-client-java</artifactId>
   	<version>3.1.0</version>
   </dependency>
   ```

2. 定义实体类

   ```java
   @Data
   @Measurement(name="SystemInfo")
   public class SystemInfo {
       // name用来配置字段名，名称和实体类一样可以不写，timestamp用来指定该列是够为时间列，tag用来标注该字段是否是索引（索引列将会参与查询分组）
       // 查询的每一条记录只包含一个字段的键和值，分别对应_field和_value列
       @Column(name="instant", timestamp = true, tag = true)
       private Instant instant;
   
       @Column(name = "cpu")
       private Double cpu;
   
       @Column(name = "memory")
       private Double memory;
   
       @Column(name = "userCount", tag = true)
       private Integer userCount;
   
   }
   ```

   

3. 使用客户端连接

   ```java
   // 创建连接
   InfluxDBClient influxDbClient = InfluxDBClientFactory.create(url, token.toCharArray(), org, bucket);
   ```

4. 写入数据

   ```java
   // 创建实体类对象
   SystemInfo measurement = new SystemInfo();
   measurement.setInstant(Instant.now());
   measurement.setCpu(3.6);
   measurement.setMemory(66.8);
   // 获取写API
   WriteApi writeApi = influxDbClient.getWriteApi();
   // 指定时间精度，写入测量
   writeApi.writeMeasurement(bucket, org, WritePrecision.MS, measurement);
   ```

   

5. 查询数据

   ```java
   // 获取查询API
   QueryApi queryApi = influxDbClient.getQueryApi();
   // 查询语句
   String fluxQuery = "from(bucket:\"sciyon-bucket\") " +
   		"|> range(start: 0) " +
   		"|> filter(fn: (r) => r[\"_measurement\"] == \"si\")";
   // 调用查询接口
   List<FluxTable> tables = queryApi.query(fluxQuery);
   // 根据时间和字段名归类，一类下面可能有多个表，所以这里是两层循环
   for (FluxTable fluxTable : tables) {
   	List<FluxRecord> records = fluxTable.getRecords();
   	for (FluxRecord fluxRecord : records) {
           // 获取字段值
   		Object filed = fluxRecord.getValueByKey("_field");
   		Object value = fluxRecord.getValueByKey("_value");
   		System.out.println(fluxRecord.getTime() + "--- " + filed + ":" + value);
   	}
   }
   ```

6. 删除数据

   ```java
   // 获取删除API
   DeleteApi deleteApi = influxDbClient.getDeleteApi();
   // 删除16年到现在、测量值为demo的数据
   deleteApi.delete(OffsetDateTime.of(LocalDateTime.of(2016, 1, 1, 0, 0), ZoneOffset.UTC),
   	OffsetDateTime.of(LocalDateTime.now(), ZoneOffset.UTC),
   	"_measurement=\"demo\"", bucket, org);
   ```

   

7. 关闭连接

   ```java
   influxDbClient.close();
   ```


#### 合并多个属性为一行显示

InfluxDB默认将多个属性分多行显示，但是很多时候我们需要在一行中显示多个属性值。

`pivot` 函数是一种数据重组函数，它可以将数据从纵向（列存储）转为横向（行存储）的形式。这意味着 `pivot` 函数可以将数据集中相同标签的不同字段值转换成单独的列，从而将多条数据转化为一条数据，并保留原始标签。这有助于将多条记录组合成一条记录，并在查询结果中将不同字段的值并列显示。

`pivot` 函数的主要参数有：

- `rowKey`: 一个列表参数，用于指定作为行键的列（一般为标签）。这些列将决定输出数据的行分组方式。
- `columnKey`: 一个列表参数，用于指定作为列键的列（通常为字段）。这些列将决定数据的列分组方式。
- `valueColumn`: 用于指定包含数据值的列（通常为 `_value`）。

下面是一个 `pivot` 函数的示例，展示如何将多个字段的值转化为同一行：

```flux
from(bucket: "my_bucket")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "my_measurement")
  |> pivot(rowKey: ["_time"], columnKey: ["_field"], valueColumn: "_value")
```

通过使用 `pivot` 函数，你可以将数据从传统的纵向形式转换为横向形式，从而使不同字段的值在查询结果中以列的形式并列显示。



InfluxDB2.x 官方文档网址：[influxdata/influxdb-client-java: InfluxDB 2 JVM Based Clients (github.com)](https://github.com/influxdata/influxdb-client-java)
