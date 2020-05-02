# HadoopLearning



## 简介

本教程描述如何使用Hortonworks数据平台为一个卡车物联网数据探索用例提炼数据。 这个物联网探索案例包含车辆、设备和人等在地图或者类似表面移动。 我们的分析将兴趣点放在将位置信息和我们的分析数据绑定。 开发人员通常通过建立简单程序实现Hello World来理解新的概念。 本教程通过让参与者开始使用Hadoop和HDP来获得类似效果。 我们使用一个物联网（loT）用例来构建我们的第一个HDP应用。 为了本教程，我们找到了一个卡车足迹的用例。每辆车装备了随时记录位置和时间数据的装置。 这些事件数据以流的方式传回我们将要处理数据的数据中心。



## 准备

- 下载并安装最新版[Hortonworks Sanbox](http://zh.hortonworks.com/products/hortonworks-sandbox/#install)
- 在进入hello HDP实验室请，我们**强烈建议**你过一下[学习Hortonworks Sandbox的线索](https://forevernull.gitbooks.io/hortonworks-getstarted/content/xue_xi_hortonworks_sandbox_de_xian_suo.html)，在虚拟机和Ambari界面中熟悉Sandbox。
- 用到的数据集合: [Geolocation.zip](https://app.box.com/HadoopCrashCourseData)
- 在本课程中，Hortonworks Sandbox是安装在Oracle VirtualBox虚拟机（VM）上的--你的屏幕可能看起来有所不同。
- 注意，其他版本的Excel也可以，但是可视化会被限制在表格或图表。你也可以使用其他可视化工具，比如Zeppelin和Zoomdata。