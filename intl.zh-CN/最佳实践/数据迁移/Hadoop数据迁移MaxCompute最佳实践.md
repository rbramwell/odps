# Hadoop数据迁移MaxCompute最佳实践 {#concept_rn1_jcm_bfb .concept}

本文将为您介绍如何通过DataWorks数据同步功能，将HDFS上的数据迁移至MaxCompute，或从MaxCompute将数据迁移至HDFS。无论您使用Hadoop还是Spark，均可以与MaxCompute之间的数据进行双向同步。

## 环境准备 {#section_vv3_pqy_bfb .section}

1.  Hadoop集群搭建

    进行数据迁移前，您需要保证自己的Hadoop集群环境正常。本文使用阿里云EMR服务自动化搭建Hadoop集群，详细过程请参见[创建集群](../../../../intl.zh-CN/快速入门/步骤三：创建集群.md#)。

    本文使用的EMR Hadoop版本信息如下：

    EMR版本: EMR-3.11.0

    集群类型: HADOOP

    软件信息: HDFS2.7.2 / YARN2.7.2 / Hive2.3.3 / Ganglia3.7.2 / Spark2.2.1 / HUE4.1.0 / Zeppelin0.7.3 / Tez0.9.1 / Sqoop1.4.6 / Pig0.14.0 / ApacheDS2.0.0 / Knox0.13.0

    Hadoop集群使用经典网络，区域为华东1（杭州），主实例组ECS计算资源配置公网及内网IP，高可用选择为否（非HA模式）。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21113/156315157411593_zh-CN.png)

2.  MaxCompute

    [开通MaxCompute](../../../../intl.zh-CN/准备工作/开通MaxCompute.md#)并创建好项目。本文以在华东1（杭州）区域创建项目bigdata\_DOC为例，同时启动DataWorks相关服务。


## 数据准备 {#section_p2m_nty_bfb .section}

1.  Hadoop集群创建测试数据
    1.  进入EMR Hadoop集群控制台界面，使用交互式工作台，新建交互式任务doc。本例中HIVE建表语句如下。

        ``` {#codeblock_trw_v2i_y2q}
        CREATE TABLE IF NOT
        EXISTS hive_doc_good_sale(
           create_time timestamp,
           category STRING,
           brand STRING,
           buyer_id STRING,
           trans_num BIGINT,
           trans_amount DOUBLE,
           click_cnt BIGINT
           )
           PARTITIONED BY (pt string) ROW FORMAT
        DELIMITED FIELDS TERMINATED BY ',' lines terminated by '\n'
        ```

    2.  选择运行，出现`Query executed successfully`提示，则说明成功在EMR Hadoop集群上创建了表hive\_doc\_good\_sale。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21113/156315157411595_zh-CN.png)

    3.  插入测试数据。您可以选择从OSS或其他数据源导入测试数据，也可以手动插入少量的测试数据。本文中手动插入数据如下。

        ``` {#codeblock_sf0_hm3_aoy}
        insert into
        hive_doc_good_sale PARTITION(pt =1 ) values('2018-08-21','外套','品牌A','lilei',3,500.6,7),('2018-08-22','生鲜','品牌B','lilei',1,303,8),('2018-08-22','外套','品牌C','hanmeimei',2,510,2),(2018-08-22,'卫浴','品牌A','hanmeimei',1,442.5,1),('2018-08-22','生鲜','品牌D','hanmeimei',2,234,3),('2018-08-23','外套','品牌B','jimmy',9,2000,7),('2018-08-23','生鲜','品牌A','jimmy',5,45.1,5),('2018-08-23','外套','品牌E','jimmy',5,100.2,4),('2018-08-24','生鲜','品牌G','peiqi',10,5560,7),('2018-08-24','卫浴','品牌F','peiqi',1,445.6,2),('2018-08-24','外套','品牌A','ray',3,777,3),('2018-08-24','卫浴','品牌G','ray',3,122,3),('2018-08-24','外套','品牌C','ray',1,62,7) ;
        ```

    4.  完成插入数据后，您可以执行`select * from hive_doc_good_sale where pt =1;`语句，检查Hadoop集群表中是否已存在数据可以用于迁移。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21113/156315157411596_zh-CN.png)

2.  利用DataWorks新建目标表
    1.  登录DataWorks控制台，单击相应工作空间操作栏下的**进入数据开发**。
    2.  进入DataStudio（数据开发）页面，选择**新建** \> **表**。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21113/156315157411597_zh-CN.png)

    3.  在新建表对话框中，填写表名，并单击**提交**。
    4.  进入新建表页面，选择**DDL模式**。
    5.  在**DDL模式**对话框中输入建表语句，单击**生成表结构**，并确认操作。本示例的建表语句如下所示：

        ``` {#codeblock_nqt_lyr_klv}
        CREATE TABLE IF NOT EXISTS hive_doc_good_sale(
           create_time string,
           category STRING,
           brand STRING,
           buyer_id STRING,
           trans_num BIGINT,
           trans_amount DOUBLE,
           click_cnt BIGINT
           )
           PARTITIONED BY (pt string) ;
        ```

        在建表过程中，需要考虑到HIVE数据类型与MaxCompute数据类型的映射，当前数据映射关系请参见[与Hive数据类型映射表](../../../../intl.zh-CN/开发/SQL及函数/附录/与Hive数据类型映射表.md#)。

        由于本文使用DataWorks进行数据迁移，而DataWorks数据同步功能当前暂不支持TIMESTAMP类型数据。因此在DataWorks建表语句中，将create\_time设置为STRING类型。上述步骤同样可通过odpscmd命令行工具完成，命令行工具安装和配置请参见[安装并配置客户端](../../../../intl.zh-CN/准备工作/安装并配置客户端.md#)。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21113/156315157411598_zh-CN.png)

        **说明：** 考虑到部分HIVE与MaxCompute数据类型的兼容问题，建议在odpscmd客户端上执行以下命令。

        ``` {#codeblock_xue_zlj_by9}
        set odps.sql.type.system.odps2=true;
        set odps.sql.hive.compatible=true;
        ```

    6.  完成建表后，单击左侧导航栏中的**表管理**，即可查看当前创建的MaxCompute表。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21113/156315157411599_zh-CN.png)


## 数据同步 {#section_esz_swy_bfb .section}

1.  新建自定义资源组

    由于MaxCompute项目所处的网络环境与Hadoop集群中的数据节点（data node）网络通常不可达，您可以通过自定义资源组的方式，将DataWorks的同步任务运行在Hadoop集群的Master节点上（Hadoop集群内Master节点和数据节点通常可达）。

    1.  查看Hadoop集群data node

        进入EMR控制台，选择**首页** \> **集群管理** \> **集群** \> **主机列表**。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21113/156315157411600_zh-CN.png)

        您也可以通过单击上图中Master节点的ECS ID，进入ECS实例详情页。然后单击远程连接进入ECS，执行`hadoop dfsadmin –report`命令查看data node。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21113/156315157411601_zh-CN.png)

        本示例的data node只具有内网地址，很难与DataWorks默认资源组互通，所以需要设置自定义资源组，将master node设置为执行DataWorks数据同步任务的节点。

    2.  新建自定义资源组
        1.  进入**数据集成** \> **资源组**页面，单击右上角的**新增自定义资源组**。关于自定义资源组的详细信息请参见[新增任务资源](../../../../intl.zh-CN/使用指南/数据集成/常见配置/新增任务资源.md#)。

            **说明：** 目前仅专业版及以上版本方可使用此入口。

        2.  添加服务器时，需要输入ECS UUID和机器IP等信息（对于经典网络类型，需要输入服务器名称。对于专有网络类型，需要输入服务器UUID）。目前仅DataWorks V2.0华东2区支持经典网络类型的调度资源添加，对于其他区域，无论您使用的是经典网络还是专有网络类型，在添加调度资源组时都请选择专有网络类型。

            机器IP需要填写master node公网IP（内网IP有可能不可达）。ECS的UUID需要进入master node管理终端，通过命令dmidecode | grep UUID获取（如果您的hadoop集群并非搭建在EMR环境上，也可以通过该命令获取）。

            ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21113/156315157411603_zh-CN.png)

        3.  添加服务器后，需要保证master node与DataWorks网络可达。如果您使用的是ECS服务器，需要设置服务器安全组。
            -   如果您使用的内网IP互通，请参见[添加安全组](../../../../intl.zh-CN/使用指南/数据集成/常见配置/添加安全组.md#)。
            -   如果您使用的是公网IP，可以直接设置安全组公网出入方向规则。本文中设置公网入方向放通所有端口（实际应用场景中，为了您的数据安全，强烈建议设置详细的放通规则）。

                ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21113/156315157411604_zh-CN.png)

        4.  完成上述步骤后，按照提示安装自定义资源组agent。当前状态显示为**可用**时，则新增自定义资源组成功。

            ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21113/156315157511605_zh-CN.png)

            如果状态为**不可用**，您可以登录master node，执行`tail –f/home/admin/alisatasknode/logs/heartbeat.log`命令查看DataWorks与master node之间心跳报文是否超时。

            ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21113/156315157511606_zh-CN.png)

2.  新建数据源

    DataWorks新建工作空间后，默认设置自己为数据源odps\_first。因此只需要添加Hadoop集群数据源。更多详情请参见[配置HDFS数据源](../../../../intl.zh-CN/使用指南/数据集成/数据源配置/配置HDFS数据源.md#)。

    1.  进入数据集成页面，选择**同步资源管理** \> **数据源**，单击**新增数据源**。
    2.  在新增数据源弹出框中，选择数据源类型为**HDFS**。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21113/156315157511607_zh-CN.png)

    3.  填写HDFS数据源的各配置项。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21113/156315157511608_zh-CN.png)

        |配置|说明|
        |:-|:-|
        |**数据源名称**|数据源名称必须以字母、数字、下划线组合，且不能以数字和下划线开头。|
        |**数据源描述**|对数据源进行简单描述，不得超过80个字符。|
        |**适用环境**|可以选择**开发**或**生产**环境。 **说明：** 仅标准模式工作空间会显示此配置。

 |
        |**DefaultFS**|对于EMR Hadoop集群而言，如果Hadoop集群为HA集群，则此处地址为`hdfs://emr-header-1的IP:8020`。如果Hadoop集群为非HA集群，则此处地址为`hdfs://emr-header-1的IP:9000`。 本实验中的emr-header-1与DataWorks通过公网连接，因此此处填写公网IP并放通安全组。

 |

    4.  完成配置后，单击**测试连通性**。
    5.  测试连通性通过后，单击**完成**。

        **说明：** 如果EMR Hadoop集群设置网络类型为专有网络，则不支持连通性测试。

3.  配置数据同步任务
    1.  进入数据开发页面，选择**新建** \> **数据集成** \> **数据同步**。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21113/156315157511609_zh-CN.png)

    2.  在新建节点对话框中，输入**节点名称**，单击**提交**。
    3.  成功创建数据同步节点后，单击工具栏中的**转换脚本**按钮。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21113/156315157551387_zh-CN.png)

    4.  单击提示对话框中的**确认**，即可进入脚本模式进行开发。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21113/156315157551388_zh-CN.png)

    5.  单击工具栏中的**导入模板**按钮。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21113/156315157551389_zh-CN.png)

    6.  在导入模板对话框中，选择**来源类型**、**数据源**、**目标类型**及**数据源**，单击**确认**。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21113/156315157551390_zh-CN.png)

    7.  新建同步任务完成后，通过导入模板已生成了基本的读取端配置。此时您可以继续手动配置数据同步任务的读取端数据源，以及需要同步的表信息等。本示例的代码如下所示，更多详情请参见[配置HDFS Reader](../../../../intl.zh-CN/使用指南/数据集成/作业配置/配置Reader插件/配置HDFS Reader.md#)。

        ``` {#codeblock_81n_kt8_zm0}
        {
          "configuration": {
            "reader": {
              "plugin": "hdfs",
              "parameter": {
                "path": "/user/hive/warehouse/hive_doc_good_sale/", 
                "datasource": "HDFS1",
                "column": [
                  {
                    "index": 0,
                    "type": "string"
                  },
                  {
                    "index": 1,
                    "type": "string"
                  },
                  {
                    "index": 2,
                    "type": "string"
                  },
                  {
                    "index": 3,
                    "type": "string"
                  },
                  {
                    "index": 4,
                    "type": "long"
                  },
                  {
                    "index": 5,
                    "type": "double"
                  },
                  {
                    "index": 6,
                    "type": "long"
                  }
                ],
                "defaultFS": "hdfs://121.199.11.138:9000",
                "fieldDelimiter": ",",
                "encoding": "UTF-8",
                "fileType": "text"
              }
            },
            "writer": {
              "plugin": "odps",
              "parameter": {
                "partition": "pt=1",
                "truncate": false,
                "datasource": "odps_first",
                "column": [
                  "create_time",
                  "category",
                  "brand",
                  "buyer_id",
                  "trans_num",
                  "trans_amount",
                  "click_cnt"
                ],
                "table": "hive_doc_good_sale"
              }
            },
            "setting": {
              "errorLimit": {
                "record": "1000"
              },
              "speed": {
                "throttle": false,
                "concurrent": 1,
                "mbps": "1",
                "dmu": 1
              }
            }
          },
          "type": "job",
          "version": "1.0"
        }
        ```

        其中，path参数为数据在Hadoop集群中存放的位置.您可以在登录master node后，执行`hdfs dfs –ls /user/hive/warehouse/hive_doc_good_sale`命令确认。对于分区表，您可以不指定分区，DataWorks数据同步会自动递归到分区路径。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21113/156315157511611_zh-CN.png)

    8.  完成配置后，单击**运行**。如果提示任务运行成功，则说明同步任务已完成。如果运行失败，可以通过日志进行排查。

## 验证结果 {#section_txr_ybz_bfb .section}

1.  单击左侧导航栏中的**临时查询**。
2.  选择**新建** \> **ODPS SQL**。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21113/156315157551391_zh-CN.png)

3.  编写并执行SQL语句，查看导入hive\_doc\_good\_sale的数据。SQL语句如下所示：

    ``` {#codeblock_3qm_2vu_rq4}
    --查看是否成功写入MaxCompute
    select * from hive_doc_good_sale where pt=1;
    ```

    您也可以在odpscmd命令行工具中输入`select * FROM hive_doc_good_sale where pt =1;`，查询表结果。


## MaxCompute数据迁移到Hadoop {#section_z44_tdb_3fb .section}

如果您想实现MaxCompute数据迁移至Hadoop，步骤与上述步骤类似，不同的是同步脚本内的reader和writer对象需要对调，具体实现脚本示例如下。

``` {#codeblock_7c2_2af_uv6}

{
  "configuration": {
    "reader": {
      "plugin": "odps",
      "parameter": {
      "partition": "pt=1",
      "isCompress": false,
      "datasource": "odps_first",
      "column": [
        "create_time",
        "category",
        "brand",
      "buyer_id",
      "trans_num",
      "trans_amount",
      "click_cnt"
    ],
    "table": "hive_doc_good_sale"
    }
  },
  "writer": {
    "plugin": "hdfs",
    "parameter": {
    "path": "/user/hive/warehouse/hive_doc_good_sale",
    "fileName": "pt=1",
    "datasource": "HDFS_data_source",
    "column": [
      {
        "name": "create_time",
        "type": "string"
      },
      {
        "name": "category",
        "type": "string"
      },
      {
        "name": "brand",
        "type": "string"
      },
      {
        "name": "buyer_id",
        "type": "string"
      },
      {
        "name": "trans_num",
        "type": "BIGINT"
      },
      {
        "name": "trans_amount",
        "type": "DOUBLE"
      },
      {
        "name": "click_cnt",
        "type": "BIGINT"
      }
    ],
    "defaultFS": "hdfs://47.99.162.100:9000",
    "writeMode": "append",
    "fieldDelimiter": ",",
    "encoding": "UTF-8",
    "fileType": "text"
    }
  },
  "setting": {
    "errorLimit": {
      "record": "1000"
  },
  "speed": {
    "throttle": false,
    "concurrent": 1,
    "mbps": "1",
    "dmu": 1
  }
  }
},
"type": "job",
"version": "1.0"
}
```

您需要参见[配置HDFS Writer](../../../../intl.zh-CN/使用指南/数据集成/作业配置/配置Writer插件/配置HDFS Writer.md#)，在运行上述同步任务前，对Hadoop集群进行设置。在运行同步任务后，手动复制同步过去的文件。

