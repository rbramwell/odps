# PyODPS的列运算 {#task_2005070 .task}

本文为您介绍如何进行PyODPS的列运算。

请提前开通MaxCompute和DataWorks服务，并在DataWorks完成业务流程的创建。本例使用DataWorks简单模式。

1.  准备测试数据。 
    1.  单击[此处](http://t.cn/Rf8GeUq)下载鸢尾花数据集iris.data，重命名为iris.csv。
    2.  登录Dataworks控制台，以**DDL模式**创建表pyodps\_iris。![DDL模式](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1227015/156740322654311_zh-CN.jpg)

 建表语句如下。

        ``` {#codeblock_uo5_ytb_jbf .lanuage-sql}
        create table if not exists pyodps_iris
        (
        sepallength  DOUBLE comment '片长度(cm)',
        sepalwidth   DOUBLE comment '片宽度(cm)',
        petallength  DOUBLE comment '瓣长度(cm)',
        petalwidth   DOUBLE comment '瓣宽度(cm)',
        name         STRING comment '种类'
        );
        ```

    3.  单击**提交到生产环境**。![提交到生产环境](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1227015/156740322654313_zh-CN.jpg)


    4.  将iris.csv中的数据导入到表pyodps.iris。![数据导入](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1227015/156740322654317_zh-CN.jpg)


2.  新建PyODPS节点，并将节点命名为PyCo。 

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1332268/156740322655229_zh-CN.png)

    编辑节点的PyCo代码如下。

    ``` {#codeblock_gmj_s00_i2g .language-python}
    from odps import DataFrame
    import numpy as np
    import pandas as pd
    
    iris = DataFrame(o.get_table('pyodps_iris'))
    
    #判断是否为null。
    print iris.sepallength.isnull().head(5)
    
    #逻辑判断。
    print (iris.sepallength > 5).ifelse('gt5','lte5').rename('cmp5').head(5)
    
    #多条件判断。
    print iris.sepallength.switch(4.9,'eq4.9',5.0,'eq5.0',default='noeq').rename('equalness').head(5)
    from odps.df import switch
    print switch(iris.sepallength == 4.9,'eq4.9',iris.sepallength == 5.0,'eq5.0',default='noeq').rename('equalness').head(5)
    
    #修该数据集某一列的一部分值。
    iris[iris.sepallength > 5,'cmp5'] = 'gt5'
    iris[iris.sepallength <=5,'cmp5'] = 'lte5'
    print iris.head(5)
    
    
    #数学计算。
    print (iris.sepallength * 10).log().head(5)
    fields = [iris.sepallength,(iris.sepallength /2).rename('sepallength/2'),(iris.sepallength ** 2).rename('sepallength的平方')]
    print iris[fields].head(5)
    print  (iris.sepallength < 5).head(5)
    
    
    
    #集合类型相关操作。
    data = {'id': [1,2], 'a': [['a1','b1'],['c1']], 'b': [{'a2': 0, 'b2': 1, 'c2': 2},{'d2': 3, 'e2': 4}]}
    df = pd.DataFrame(data)
    print df
    
    df1 = DataFrame(df, unknown_as_string=True, as_type={'a': 'list<string>','b' : 'dict<string,int64>'})
    print df1.dtypes
    print df1.head()
    
    print df1[df1.id,df1.a[0],df1.b.len()].head()
    print df1.a.explode().head()
    print df1.a.explode(pos=True).head()
    print df1.b.explode().head()
    print df1.b.explode(['key','value']).head()
    
    #explode与其他并列多行输出。
    print df1[df1.id,df1.a.explode()].head()
    
    #isin,notin,cut。
    
    #isin用于判断Sequence里的元素是否在某个集合元素里；notin反之。
    print iris.sepallength.isin([4.9,5.1]).rename('sepallength').head()
    
    #cut提供离散化的操作，可以将Sequence的数据拆成几个区段。
    print iris.sepallength.cut(range(6),labels=['0-1','1-2','2-3','3-4','4-5']).rename('sepallength_cut').head(5)
    
    #include_under和include_over可以分别包括向下和向上的区间。
    labels = ['0-1', '1-2', '2-3', '3-4', '4-5', '5-']
    iris.sepallength.cut(range(6), labels=labels, include_over=True).rename('sepallength_cut').head(5)
    ```

3.  单击PyODPS节点的运行按钮。 

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1591405/156740322658684_zh-CN.png)

4.  在**运行日志**中查看结果。 

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1591405/156740322658685_zh-CN.png)

    运行结果如下。

    ``` {#codeblock_fjt_oqx_7n3}
    Executing user script with PyODPS 0.8.0
    
    Sql compiled:
    CREATE TABLE tmp_pyodps_32e39a9a_8ae6_4815_b465_4b9dfa3bf1b9 LIFECYCLE 1 AS
    SELECT t1.`sepallength` IS NULL AS `sepallength`
    FROM DQC_0221_dev.`pyodps_iris` t1
    
      sepallength
    0       False
    1       False
    2       False
    3       False
    4       False
    Sql compiled:
    CREATE TABLE tmp_pyodps_d684aded_8ad8_4119_8558_1fddd774dd3f LIFECYCLE 1 AS
    SELECT IF(t1.`sepallength` > 5, 'gt5', 'lte5') AS `cmp5`
    FROM DQC_0221_dev.`pyodps_iris` t1
    
       cmp5
    0  lte5
    1  lte5
    2  lte5
    3  lte5
    4   gt5
    Sql compiled:
    CREATE TABLE tmp_pyodps_cfc5bdb9_6d3e_4711_8acc_913b29b96c9a LIFECYCLE 1 AS
    SELECT CASE t1.`sepallength` WHEN 4.9 THEN 'eq4.9' WHEN 5.0 THEN 'eq5.0' ELSE 'noeq' END AS `equalness`
    FROM DQC_0221_dev.`pyodps_iris` t1
    
      equalness
    0     eq4.9
    1      noeq
    2      noeq
    3     eq5.0
    4      noeq
    Sql compiled:
    CREATE TABLE tmp_pyodps_d1fd70b4_2ad8_4dc9_838e_f566f7f5841e LIFECYCLE 1 AS
    SELECT CASE WHEN t1.`sepallength` == 4.9 THEN 'eq4.9' WHEN t1.`sepallength` == 5.0 THEN 'eq5.0' ELSE 'noeq' END AS `equalness`
    FROM DQC_0221_dev.`pyodps_iris` t1
    
      equalness
    0     eq4.9
    1      noeq
    2      noeq
    3     eq5.0
    4      noeq
    Sql compiled:
    CREATE TABLE tmp_pyodps_d9e5d64e_1f21_4ffb_993f_70d3f36b6553 LIFECYCLE 1 AS
    SELECT t1.`sepallength`, t1.`sepalwidth`, t1.`petallength`, t1.`petalwidth`, t1.`name`, IF(t1.`sepallength` <= 5, 'lte5', IF(t1.`sepallength` > 5, 'gt5', CAST(NULL AS STRING))) AS `cmp5`
    FROM DQC_0221_dev.`pyodps_iris` t1
    
       sepallength  sepalwidth  petallength  petalwidth         name  cmp5
    0          4.9         3.0          1.4         0.2  Iris-setosa  lte5
    1          4.7         3.2          1.3         0.2  Iris-setosa  lte5
    2          4.6         3.1          1.5         0.2  Iris-setosa  lte5
    3          5.0         3.6          1.4         0.2  Iris-setosa  lte5
    4          5.4         3.9          1.7         0.4  Iris-setosa   gt5
    Sql compiled:
    CREATE TABLE tmp_pyodps_4bfb0aa0_7ec5_4f35_9480_d36baf2b8079 LIFECYCLE 1 AS
    SELECT LN(t1.`sepallength` * 10) AS `sepallength`
    FROM DQC_0221_dev.`tmp_pyodps_d9e5d64e_1f21_4ffb_993f_70d3f36b6553` t1
    
       sepallength
    0     3.891820
    1     3.850148
    2     3.828641
    3     3.912023
    4     3.988984
    Sql compiled:
    CREATE TABLE tmp_pyodps_deb6fa45_fbb1_41d2_8174_76d35eebea9c LIFECYCLE 1 AS
    SELECT t1.`sepallength`, t1.`sepallength` / 2 AS `sepallength/2`, POW(t1.`sepallength`, 2) AS `sepallength的平方`
    FROM DQC_0221_dev.`tmp_pyodps_d9e5d64e_1f21_4ffb_993f_70d3f36b6553` t1
    
       sepallength  sepallength/2  sepallength的平方
    0          4.9           2.45           24.01
    1          4.7           2.35           22.09
    2          4.6           2.30           21.16
    3          5.0           2.50           25.00
    4          5.4           2.70           29.16
    Sql compiled:
    CREATE TABLE tmp_pyodps_fa326546_c5e8_466f_915d_e783b7dcacdf LIFECYCLE 1 AS
    SELECT t1.`sepallength` < 5 AS `sepallength`
    FROM DQC_0221_dev.`tmp_pyodps_d9e5d64e_1f21_4ffb_993f_70d3f36b6553` t1
    
    
      sepallength
    0        True
    1        True
    2        True
    3       False
    4       False
              a                               b  id
    0  [a1, b1]  {u'c2': 2, u'a2': 0, u'b2': 1}   1
    1      [c1]            {u'd2': 3, u'e2': 4}   2
    odps.Schema {
      a   list<string>
      b   dict<string,int64>
      id  int64
    }
              a                               b  id
    0  [a1, b1]  {u'c2': 2, u'a2': 0, u'b2': 1}   1
    1      [c1]            {u'd2': 3, u'e2': 4}   2
       id   a  b
    0   1  a1  3
    1   2  c1  2
        a
    0  a1
    1  b1
    2  c1
       a_pos   a
    0      0  a1
    1      1  b1
    2      0  c1
      b_key  b_value
    0    c2        2
    1    a2        0
    2    b2        1
    3    d2        3
    4    e2        4
      key  value
    0  c2      2
    1  a2      0
    2  b2      1
    3  d2      3
    4  e2      4
    
       id   a
    0   1  a1
    1   1  b1
    2   2  c1
    Sql compiled:
    CREATE TABLE tmp_pyodps_fb28d9b9_fa45_4712_a564_dd751641286e LIFECYCLE 1 AS
    SELECT t1.`sepallength` IN (4.9, 5.1) AS `sepallength`
    FROM DQC_0221_dev.`tmp_pyodps_d9e5d64e_1f21_4ffb_993f_70d3f36b6553` t1
    
       sepallength
    0         True
    1        False
    2        False
    3        False
    4        False
    5        False
    6        False
    7        False
    8         True
    9        False
    10       False
    11       False
    12       False
    13       False
    14       False
    15       False
    16        True
    17       False
    18        True
    19       False
    20        True
    21       False
    22        True
    23       False
    24       False
    25       False
    26       False
    27       False
    28       False
    29       False
    30       False
    31       False
    32       False
    33        True
    34       False
    35       False
    36        True
    37       False
    38        True
    39       False
    40       False
    41       False
    42       False
    43        True
    44       False
    45        True
    46       False
    47       False
    48       False
    49       False
    50       False
    51       False
    52       False
    53       False
    54       False
    55       False
    56        True
    57       False
    58       False
    59       False
    Sql compiled:
    CREATE TABLE tmp_pyodps_1f85e8e1_a9f0_473f_ac0e_d526fdd80dc4 LIFECYCLE 1 AS
    SELECT CASE WHEN (0 < t1.`sepallength`) AND (t1.`sepallength` <= 1) THEN '0-1' WHEN (1 < t1.`sepallength`) AND (t1.`sepallength` <= 2) THEN '1-2' WHEN (2 < t1.`sepallength`) AND (t1.`sepallength` <= 3) THEN '2-3' WHEN (3 < t1.`sepallength`) AND (t1.`sepallength` <= 4) THEN '3-4' WHEN (4 < t1.`sepallength`) AND (t1.`sepallength` <= 5) THEN '4-5' END AS `sepallength_cut`
    FROM DQC_0221_dev.`tmp_pyodps_d9e5d64e_1f21_4ffb_993f_70d3f36b6553` t1
      sepallength_cut
    0             4-5
    1             4-5
    2             4-5
    3             4-5
    4            None
    Sql compiled:
    CREATE TABLE tmp_pyodps_080a677e_904b_4edc_a961_160f8c664d7f LIFECYCLE 1 AS
    SELECT CASE WHEN (0 < t1.`sepallength`) AND (t1.`sepallength` <= 1) THEN '0-1' WHEN (1 < t1.`sepallength`) AND (t1.`sepallength` <= 2) THEN '1-2' WHEN (2 < t1.`sepallength`) AND (t1.`sepallength` <= 3) THEN '2-3' WHEN (3 < t1.`sepallength`) AND (t1.`sepallength` <= 4) THEN '3-4' WHEN (4 < t1.`sepallength`) AND (t1.`sepallength` <= 5) THEN '4-5' WHEN 5 < t1.`sepallength` THEN '5-' END AS `sepallength_cut`
    FROM DQC_0221_dev.`tmp_pyodps_d9e5d64e_1f21_4ffb_993f_70d3f36b6553` t1
    ```


