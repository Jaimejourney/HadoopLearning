## 用Spark分析危险指数

### 概述

MapReduce很有用，但是运行作业所花费的时间有时可能是很长的。 此外，MapReduce作业仅适用于一组特定的用例。 需要一种适用于更多用例的计算框架。

Apache Spark被设计为快速，通用，易于使用的计算平台。 它扩展了MapReduce模型并将其带到另一个层次。 速度来自内存中的计算。 内存中运行的应用程序可以更快地处理和响应。



使用Ambari启用Spark

1. 以 `maria_dev`登录Ambari.在最下方的左侧栏, 确认Spark2和Zeppelin Notebook正在运行：

![run](/Users/yikaizhu/Downloads/Hadoop sandbox中文教程/run.png)

2.通过URL: http://sandbox-hdp.hortonworks.com:9995/打开Zeppelin

![z](/Users/yikaizhu/Downloads/Hadoop sandbox中文教程/z.png)

3.新建Zeppelin Notebook，点击顶部的 Notebook并选择Create new note. 命名你的notebook:

```
Compute Riskfactor with Spark
```

![z2](/Users/yikaizhu/Downloads/Hadoop sandbox中文教程/z2.png)



#### 新建一个HIVE上下文

为了改进Hive集成，已为Spark添加了ORC文件支持。 这使Spark可以读取存储在ORC文件中的数据。 Spark可以利用ORC文件更高效的列存储和谓词下推功能来实现更快的内存处理。 HiveContext是Spark SQL执行引擎的实例，与存储在Hive中的数据集成。 更基本的SQLContext提供了不依赖Hive的Spark SQL支持的子集。 它从类路径上的hive-site.xml读取Hive的配置。



#### 实例化SparkSession

```scala
%spark2
val hiveContext = new org.apache.spark.sql.SparkSession.Builder().getOrCreate()
```



#### 读取CSV到Apache Spark中

在本教程中，我们将使用上一部分中存储在HDFS中的CSV文件。 此外，我们将利用SparkSession上的全局临时视图来使用SQL以编程方式查询DataFrame。



#### 在没有用户定义模式的情况下将CSV数据导入数据框

```scala
%spark2
/**
 * Let us first see what temporary views are already existent on our Sandbox
 */
hiveContext.sql("SHOW TABLES").show()
```



首先，我们必须从HDFS读取数据，在这种情况下，我们是在不首先定义架构的情况下从csv文件读取数据：

```scala
%spark2
val geoLocationDataFrame = spark.read.format("csv").option("header", "true").load("hdfs:///tmp/data/geolocation.csv")

/**
 * Now that we have the data loaded into a DataFrame, we can register a temporary view.
 */
geoLocationDataFrame.createOrReplaceTempView("geolocation")
```

让我们确认CSV文件中的数据已正确加载到我们的数据框中：

```scala
%spark2
hiveContext.sql("SELECT * FROM geolocation LIMIT 15").show()
```

![z3](/Users/yikaizhu/Downloads/Hadoop sandbox中文教程/z3.png)

请注意，当我们将其注册为临时视图时，我们的数据将转换为临时类型：

```scala
%spark2
hiveContext.sql("DESCRIBE geolocation").show()
```

![z4](/Users/yikaizhu/Downloads/Hadoop sandbox中文教程/z4.png)



#### 输入CSV数据

```scala
%spark2
/**
 * The SQL Types library allows us to define the data types of our schema
 */
import org.apache.spark.sql.types._

/**
 * Recall from the previous tutorial section that the driverid schema only has two relations:
 * driverid (a String), and totmiles (a Double).
 */
val drivermileageSchema = new StructType().add("driverid",StringType,true).add("totmiles",DoubleType,true)
```

现在我们能引入`drivermileageSchema` :

```scala
%spark2
val drivermileageDataFrame = spark.read.format("csv").option("header", "true").schema(drivermileageSchema)load("hdfs:///tmp/data/drivermileage.csv")
```

最后，我们创建一个临时的视图：

```scala
%spark2
drivermileageDataFrame.createOrReplaceTempView("drivermileage")
```

我们能用SparkSession和SQL来查询 `drivermileage`：

```scala
%spark2
hiveContext.sql("SELECT * FROM drivermileage LIMIT 15").show()
```

![z5](/Users/yikaizhu/Downloads/Hadoop sandbox中文教程/z5.png)



#### 查询表来构建Spark RDD

我们会从 `geolocation` 和 `drivermileage` 做简单的查询来得到spark变量. 

```scala
%spark2
val geolocation_temp0 = hiveContext.sql("SELECT * FROM geolocation")
val drivermileage_temp0 = hiveContext.sql("SELECT * FROM drivermileage")
```

```scala
%spark2
geolocation_temp0.createOrReplaceTempView("geolocation_temp0")
drivermileage_temp0.createOrReplaceTempView("drivermileage_temp0")

hiveContext.sql("SHOW TABLES").show()
```

![z6](/Users/yikaizhu/Downloads/Hadoop sandbox中文教程/z6.png)



#### 查询已注册的临时表

接下来，我们将执行迭代和过滤操作。 首先，我们需要过滤与非正常事件关联的驱动程序，然后为每个驱动程序计算非正常事件的数量。

```scala
%spark2
val geolocation_temp1 = hiveContext.sql("SELECT driverid, COUNT(driverid) occurance from geolocation_temp0 WHERE event!='normal' GROUP BY driverid")
/**
 * Show RDD
 */
geolocation_temp1.show(10)
```

```scala
%spark2
geolocation_temp1.createOrReplaceTempView("geolocation_temp1")
hiveContext.sql("SHOW TABLES").show()
```

![z7](/Users/yikaizhu/Downloads/Hadoop sandbox中文教程/z7.png)

```scala
%spark2
hiveContext.sql("SELECT * FROM geolocation_temp1 LIMIT 15").show()
```

![z8](/Users/yikaizhu/Downloads/Hadoop sandbox中文教程/z8.png)



#### 进行Join操作

在本节中，我们将执行联接操作geolocation_temp1表，该表包含驾驶员的详细信息以及他们各自的非正常事件的计数。 drivermileage_temp0表包含每个驾驶员行驶的总里程的详细信息。

```scala
%spark2
val joined = hiveContext.sql("select a.driverid,a.occurance,b.totmiles from geolocation_temp1 a,drivermileage_temp0 b where a.driverid=b.driverid")
```

```scala
%spark2
joined.createOrReplaceTempView("joined")
hiveContext.sql("SHOW TABLES").show()
```

![z9](/Users/yikaizhu/Downloads/Hadoop sandbox中文教程/z9.png)

```scala
%spark2
/**
 * We can view the result from our query with a select statement
 */
hiveContext.sql("SELECT * FROM joined LIMIT 10").show()
```

![z10](/Users/yikaizhu/Downloads/Hadoop sandbox中文教程/z10.png)



#### 计算驾驶员风险概率

在本节中，我们将把驾驶员风险因素与每个驾驶员相关联。 每个驾驶员的风险因素是异常发生次数占里程驾驶员总数的总和。 简而言之，在短短的行驶里程内，大量的异常事件是高风险的指标。 让我们将此直觉转换为SQL查询：

```scala
%spark2
val risk_factor_spark = hiveContext.sql("SELECT driverid, occurance, totmiles, totmiles/occurance riskfactor FROM joined")
```

结果数据集将为我们提供总里程和非正常事件总数，以及特定驾驶员面临的风险。 将此筛选后的表注册为临时表，以便可以对其应用后续的SQL查询。

```scala
%spark2
risk_factor_spark.createOrReplaceTempView("risk_factor_spark")
hiveContext.sql("SHOW TABLES").show()
```

```scala
%spark2
risk_factor_spark.show(10)
```

![z11](/Users/yikaizhu/Downloads/Hadoop sandbox中文教程/z11.png)



#### 把表保存成CSV

在找到每个驾驶员的风险因素之后，我们可能希望将结果存储为CSV在HDFS上：

```scala
%spark2
risk_factor_spark.coalesce(1).write.option("header", "true").csv("hdfs:///tmp/data/riskfactor")
```

在 `user/maria_dev/data/` 目录下将会有一个叫 `riskfactor` 在那里我们能找到一个名字随机命名的CSV文件。



#### 总结

恭喜你！ 让我们总结一下我们用来计算与每个驾驶员相关的风险因素的Spark编码技能和知识。 Apache Spark具有内存中的数据处理引擎，因此计算效率很高。 我们学习了如何通过创建Hive上下文将Hive与Spark集成。 我们使用来自Hive的现有数据来创建RDD。 我们学会了执行RDD转换和操作以从现有RDD创建新数据集。 这些新的数据集包括经过过滤，处理和处理的数据。 在计算了风险因素之后，我们学会了将数据作为ORC加载并保存到Hive中。