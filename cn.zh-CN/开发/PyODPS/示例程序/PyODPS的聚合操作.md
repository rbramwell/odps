# PyODPS的聚合操作 {#task_2005069 .task}

本文为您介绍如何进行PyODPS的聚合操作。

请提前开通MaxCompute和DataWorks服务，并在DataWorks完成业务流程的创建。本例使用DataWorks简单模式。

1.  准备测试数据。 
    1.  单击[此处](http://t.cn/Rf8GeUq)下载鸢尾花数据集iris.data，重命名为iris.csv。
    2.  登录Dataworks控制台，以**DDL模式**创建表pyodps\_iris。![DDL模式](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1227015/156740315954311_zh-CN.jpg)

 建表语句如下。

        ``` {#codeblock_fg5_xi3_khn .lanuage-sql}
        create table if not exists pyodps_iris
        (
        sepallength  DOUBLE comment '片长度(cm)',
        sepalwidth   DOUBLE comment '片宽度(cm)',
        petallength  DOUBLE comment '瓣长度(cm)',
        petalwidth   DOUBLE comment '瓣宽度(cm)',
        name         STRING comment '种类'
        );
        ```

    3.  单击**提交到生产环境**。![提交到生产环境](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1227015/156740316054313_zh-CN.jpg)


    4.  将iris.csv中的数据导入到表pyodps.iris。![数据导入](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1227015/156740316054317_zh-CN.jpg)


2.  新建PyODPS节点，并将节点命名为PyAgg。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1332268/156740316055229_zh-CN.png)

 编辑节点的PyAgg代码如下。

    ``` {#codeblock_h0e_a21_lfl .language-python}
    from odps.df import DataFrame
    from odps.df import Delay
    from odps.df import Scalar
    from odps.df import agg
    import numpy as np
    import pandas as pd
    iris = DataFrame(o.get_table('pyodps_iris'))
    
    #describe函数,查看DataFrame里数字列的数量、最大值、最小值、平均值以及标准差。
    print iris.describe().execute()
    
    #单列聚合操作。
    print iris.sepallength.max().execute()
    
    #消除重复列的聚合。
    print iris.name.unique().cat(sep=',').execute()
    
    #统计总行数。
    print iris.count().execute()
    
    #分组聚合。
    print iris.groupby('name').agg(iris.sepallength.max(),smin=iris.sepallength.min()).head()
    print iris.groupby('name').agg(count=iris.name.count()).sort('count', ascending=False).head(5)
    print iris.groupby('name').petallength.sum().head()
    print iris.groupby('name').agg(iris.petallength.notnull().sum()).head()
    print iris.groupby(Scalar(1)).petallength.sum().head()
    
    
    #自定义聚合,对应单列。
    class Agg(object):
    
        def buffer(self):
            return [0.0, 0]
    
        def __call__(self, buffer, val):
            buffer[0] += val
            buffer[1] += 1
    
        def merge(self, buffer, pbuffer):
            buffer[0] += pbuffer[0]
            buffer[1] += pbuffer[1]
    
        def getvalue(self, buffer):
            if buffer[1] == 0:
                return 0.0
            return buffer[0] / buffer[1]
    
    print iris.sepalwidth.agg(Agg).execute()
    
    #自定义聚合,对应双列。
    class Agg2(object):
    
        def buffer(self):
            return [0.0, 0.0]
    
        def __call__(self, buffer, val1, val2):
            buffer[0] += val1
            buffer[1] += val2
    
        def merge(self, buffer, pbuffer):
            buffer[0] += pbuffer[0]
            buffer[1] += pbuffer[1]
    
        def getvalue(self, buffer):
            if buffer[1] == 0:
                return 0.0
            return buffer[0] - buffer[1]
    
    to_agg = agg([iris.sepalwidth, iris.sepallength], Agg2, rtype='float')  # 对两列调用自定义聚合。
    print iris.groupby('name').agg(val=to_agg).execute()
    
    #HyperLogLog计数。
    
    df5 = DataFrame(pd.DataFrame({'a': np.random.randint(100000, size=100000)}))
    print df5.a.hll_count().execute()
    ```

3.  单击PyODPS节点的运行按钮。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1591404/156740316058671_zh-CN.png)


4.  在**运行日志**中查看结果。 

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1591404/156740316058672_zh-CN.png)

    PyAgg节点运行结果如下。

    ``` {#codeblock_a5h_61n_f12}
    Executing user script with PyODPS 0.8.0
    
    Sql compiled:
    CREATE TABLE tmp_pyodps_28b83337_87f9_4b11_a084_f1896758d53b LIFECYCLE 1 AS
    SELECT pyodps_udf_1567394092_9467b7ba_3121_4105_81dc_431e8e68be39(t3.`sepallength_count`, t3.`sepallength_mean`, t3.`sepallength_std`, t3.`sepallength_min`, t3.`sepallength_quantile_25`, t3.`sepallength_quantile_50`, t3.`sepallength_quantile_75`, t3.`sepallength_max`, t3.`sepalwidth_count`, t3.`sepalwidth_mean`, t3.`sepalwidth_std`, t3.`sepalwidth_min`, t3.`sepalwidth_quantile_25`, t3.`sepalwidth_quantile_50`, t3.`sepalwidth_quantile_75`, t3.`sepalwidth_max`, t3.`petallength_count`, t3.`petallength_mean`, t3.`petallength_std`, t3.`petallength_min`, t3.`petallength_quantile_25`, t3.`petallength_quantile_50`, t3.`petallength_quantile_75`, t3.`petallength_max`, t3.`petalwidth_count`, t3.`petalwidth_mean`, t3.`petalwidth_std`, t3.`petalwidth_min`, t3.`petalwidth_quantile_25`, t3.`petalwidth_quantile_50`, t3.`petalwidth_quantile_75`, t3.`petalwidth_max`) AS (`type`, `sepallength`, `sepalwidth`, `petallength`, `petalwidth`)
    FROM (
      SELECT COUNT(t2.`sepallength`) AS `sepallength_count`, AVG(t2.`sepallength`) AS `sepallength_mean`, STDDEV_SAMP(t2.`sepallength`) AS `sepallength_std`, MIN(t2.`sepallength`) AS `sepallength_min`, PERCENTILE(t2.`sepallength`, 0.25) AS `sepallength_quantile_25`, PERCENTILE(t2.`sepallength`, 0.5) AS `sepallength_quantile_50`, PERCENTILE(t2.`sepallength`, 0.75) AS `sepallength_quantile_75`, MAX(t2.`sepallength`) AS `sepallength_max`, COUNT(t2.`sepalwidth`) AS `sepalwidth_count`, AVG(t2.`sepalwidth`) AS `sepalwidth_mean`, STDDEV_SAMP(t2.`sepalwidth`) AS `sepalwidth_std`, MIN(t2.`sepalwidth`) AS `sepalwidth_min`, PERCENTILE(t2.`sepalwidth`, 0.25) AS `sepalwidth_quantile_25`, PERCENTILE(t2.`sepalwidth`, 0.5) AS `sepalwidth_quantile_50`, PERCENTILE(t2.`sepalwidth`, 0.75) AS `sepalwidth_quantile_75`, MAX(t2.`sepalwidth`) AS `sepalwidth_max`, COUNT(t2.`petallength`) AS `petallength_count`, AVG(t2.`petallength`) AS`petallength_mean`, STDDEV_SAMP(t2.`petallength`) AS `petallength_std`, MIN(t2.`petallength`) AS `petallength_min`, PERCENTILE(t2.`petallength`, 0.25) AS `petallength_quantile_25`, PERCENTILE(t2.`petallength`, 0.5) AS `petallength_quantile_50`, PERCENTILE(t2.`petallength`, 0.75) AS `petallength_quantile_75`, MAX(t2.`petallength`) AS `petallength_max`, COUNT(t2.`petalwidth`) AS `petalwidth_count`, AVG(t2.`petalwidth`) AS `petalwidth_mean`, STDDEV_SAMP(t2.`petalwidth`) AS `petalwidth_std`, MIN(t2.`petalwidth`) AS `petalwidth_min`, PERCENTILE(t2.`petalwidth`, 0.25) AS `petalwidth_quantile_25`, PERCENTILE(t2.`petalwidth`, 0.5) AS `petalwidth_quantile_50`, PERCENTILE(t2.`petalwidth`, 0.75) AS `petalwidth_quantile_75`, MAX(t2.`petalwidth`) AS `petalwidth_max`
      FROM (
        SELECT t1.`sepallength`, t1.`sepalwidth`, t1.`petallength`, t1.`petalwidth`
        FROM DQC_0221_dev.`pyodps_iris` t1
      ) t2
    ) t3
    
    
              type  sepallength  sepalwidth  petallength  petalwidth
    0        count   149.000000  149.000000   149.000000  149.000000
    1         mean     5.848322    3.051007     3.774497    1.205369
    2          std     0.828594    0.433499     1.759651    0.761292
    3          min     4.300000    2.000000     1.000000    0.100000
    4  quantile_25     5.000000    2.000000     1.000000    0.000000
    5  quantile_50     5.000000    3.000000     4.000000    1.000000
    6  quantile_75     6.000000    3.000000     5.000000    1.000000
    7          max     7.900000    4.400000     6.900000    2.500000
    Sql compiled:
    SELECT MAX(t2.`sepallength`) AS `sepallength_max`
    FROM (
      SELECT t1.`sepallength`
      FROM DQC_0221_dev.`pyodps_iris` t1
    ) t2
    
    
    7.9
    Sql compiled:
    SELECT WM_CONCAT(DISTINCT ',', t2.`name`) AS `name_cat`
    FROM (
      SELECT t1.`name`
      FROM DQC_0221_dev.`pyodps_iris` t1
    ) t2
    
    
    Iris-setosa,Iris-versicolor,Iris-virginica
    149
    Sql compiled:
    CREATE TABLE tmp_pyodps_4b00d671_82b1_4cd8_8af4_f11dbdac4570 LIFECYCLE 1 AS
    SELECT t1.`name`, MAX(t1.`sepallength`) AS `sepallength_max`, MIN(t1.`sepallength`) AS `smin`
    FROM DQC_0221_dev.`pyodps_iris` t1
    GROUP BY t1.`name`
    
    
                  name  sepallength_max  smin
    0      Iris-setosa              5.8   4.3
    1  Iris-versicolor              7.0   4.9
    2   Iris-virginica              7.9   4.9
    Sql compiled:
    CREATE TABLE tmp_pyodps_276c901f_7cf0_48ee_aaae_0d3092822b34 LIFECYCLE 1 AS
    SELECT t1.`name`, COUNT(t1.`name`) AS `count`
    FROM DQC_0221_dev.`pyodps_iris` t1
    GROUP BY t1.`name`
    ORDER BY count DESC
    LIMIT 10000
    
                  name  count
    0  Iris-versicolor     50
    1   Iris-virginica     50
    2      Iris-setosa     49
    Sql compiled:
    CREATE TABLE tmp_pyodps_440d12ec_8496_4b87_b33f_b9624941e5bd LIFECYCLE 1 AS
    SELECT SUM(t1.`petallength`) AS `petallength_sum`
    FROM DQC_0221_dev.`pyodps_iris` t1
    GROUP BY t1.`name`
    Instance ID: 20190902031604255go6c472m
    
    
       petallength_sum
    0             71.8
    1            213.0
    2            277.6
    Sql compiled:
    CREATE TABLE tmp_pyodps_61e62f5f_7b4f_4971_8c39_dcaccba37764 LIFECYCLE 1 AS
    SELECT t1.`name`, SUM(IF(t1.`petallength` IS NOT NULL, 1, 0)) AS `petallength_sum`
    FROM DQC_0221_dev.`pyodps_iris` t1
    GROUP BY t1.`name`
    Instance ID: 20190902031607441ghzyr392
    
    
                  name  petallength_sum
    0      Iris-setosa               49
    1  Iris-versicolor               50
    2   Iris-virginica               50
    Sql compiled:
    CREATE TABLE tmp_pyodps_daeffd50_88fe_4b4d_9db9_57810cfe71c3 LIFECYCLE 1 AS
    SELECT SUM(t1.`petallength`) AS `petallength_sum`
    FROM DQC_0221_dev.`pyodps_iris` t1
    GROUP BY 1
    
    
       petallength_sum
    0            562.4
    Sql compiled:
    SELECT pyodps_udf_1567394173_ef78183a_a4f5_4372_91dd_18d6ecc6f651(t2.`sepalwidth`) AS `sepalwidth_aggregation`
    FROM (
      SELECT t1.`sepalwidth`
      FROM DQC_0221_dev.`pyodps_iris` t1
    ) t2
    
    
    Sql compiled:
    CREATE TABLE tmp_pyodps_e312f110_64e7_465e_adae_fd053f125a90 LIFECYCLE 1 AS
    SELECT t1.`name`, pyodps_udf_1567394220_49231d7a_faeb_4f2f_9de5_36c4aefaa29b(t1.`sepalwidth`, t1.`sepallength`) AS `val`
    FROM DQC_0221_dev.`pyodps_iris` t1
    GROUP BY t1.`name`
    
    
                  name    val
    0      Iris-setosa  -77.8
    1  Iris-versicolor -158.3
    2   Iris-virginica -180.7
    
    63021
    ```


