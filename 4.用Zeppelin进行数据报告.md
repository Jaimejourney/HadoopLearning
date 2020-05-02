## 用Zeppelin进行数据报告

### 概述

Apache Zeppelin提供了一个强大的基于Web的notebook平台，用于数据分析和发现。 它在后台支持Spark分布式上下文以及Spark之上的其他语言绑定。

在本教程中，我们将使用Apache Zeppelin在我们先前收集的地理位置，卡车和风险因子数据上运行SQL查询，并通过图形和图表将结果可视化。



#### 创建Zeppelin Notebook

进入Zeppelin界面，参考上一章进行操作：

```bash
http://sandbox-hdp.hortonworks.com:9995/
```





#### 下载数据

如果你在上一章中遇到了问题你可以点击这个链接来下载数据， [here to download it](https://raw.githubusercontent.com/hortonworks/data-tutorials/master/tutorials/hdp/hadoop-tutorial-getting-started-with-hdp/assets/datasets/riskfactor.csv) ，并将它上传到HDFS的 `/tmp/data/`这个目录下

![41](/Users/yikaizhu/Downloads/Hadoop sandbox中文教程/41.png)



并运行如下语句：

```scala
%spark2
val hiveContext = new org.apache.spark.sql.SparkSession.Builder().getOrCreate()
val riskFactorDataFrame = spark.read.format("csv").option("header", "true").load("hdfs:///tmp/data/riskfactor.csv")
riskFactorDataFrame.createOrReplaceTempView("riskfactor")
hiveContext.sql("SELECT * FROM riskfactor LIMIT 15").show()
```



#### 进行HIVE查询

在上一个Spark教程中，您已经创建了一个表finalresults或riskfactor，该表给出了与每个驾驶员相关的风险因子。 我们将使用在此表中生成的数据来可视化哪些驾驶员具有最高风险因素。 我们将使用jdbc Hive解释器在Zeppelin中编写查询。



1.运行以下语句：

```sql
%sql
SELECT * FROM riskfactor
```

2.点击运行，或者使用快捷卷shirt+enter

![z12](/Users/yikaizhu/Downloads/Hadoop sandbox中文教程/z12.png)



#### 使用Zeppelin进行数据可视化

1.遍历查询下方显示的每个选项。 每个查询都将显示不同类型的图表，具体取决于查询中返回的数据。

![z13](/Users/yikaizhu/Downloads/Hadoop sandbox中文教程/z13.png)

2.单击图表后，我们可以查看其他高级设置，以定制所需数据的视图。

![z14](/Users/yikaizhu/Downloads/Hadoop sandbox中文教程/z14.png)

3.点击Setting来进行更多图表设置

4.要制作带有riskfactor.driverid和riskfactor.riskfactor SUM的图表，请将表关系拖动到如下图所示的框中。![z15](/Users/yikaizhu/Downloads/Hadoop sandbox中文教程/z15.png)

5.你应该能得到这样一个结果

![z16](/Users/yikaizhu/Downloads/Hadoop sandbox中文教程/z16.png)

6.如果您将鼠标悬停在峰值上，则每个峰值都会给出驾驶员和危险因素。

7.尝试试验不同类型的图表，以及拖放不同的表格字段，以查看可以获得什么样的结果。

8.让我们尝试一个不同的查询，以查找哪些城市和州包含风险因素最高的驾驶员。

```sql
%sql
SELECT a.driverid, a.riskfactor, b.city, b.state
FROM riskfactor a, geolocation b where a.driverid=b.driverid
```

![z18](/Users/yikaizhu/Downloads/Hadoop sandbox中文教程/z18.png)

9.更改一些设置后，我们可以找出哪些城市具有高风险因素。 尝试通过单击散点图图标来更改图表设置。 然后，确保键a.driverid在xAxis字段内，a.riskfactor在yAxis字段内，而b.city在group字段中。 该图表应类似于以下内容。

![z19](/Users/yikaizhu/Downloads/Hadoop sandbox中文教程/z19.png)

您可以将鼠标悬停在最高点上，以确定哪个驾驶员具有最高的风险因素以及在哪个城市。



#### 总结

太好了，现在我们知道如何使用Apache Zeppelin查询和可视化数据。 我们可以利用Zeppelin以及最新获得的Hive和Spark知识，以新的创造性方式解决现实世界中的问题。