# Elasticsearch数据迁移至MaxCompute {#task_439831 .task}

本文将为您介绍如何通过DataWorks数据同步功能，将阿里云Elasticsearch集群上的数据迁移至MaxCompute。

-   搭建阿里云Elasticsearch集群

    进行数据迁移前，您需要保证自己的阿里云Elasticsearch集群环境正常。搭建阿里云Elasticsearch集群的详细过程请参见[Elasticsearch快速入门](https://help.aliyun.com/document_detail/57876.html)。

    本示例中阿里云Elasticsearch的具体配置如下：

    -   地域：华东2（上海）
    -   可用区：上海可用区B
    -   版本：5.5.3 with Commercial Feature
    ![ES配置](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/354761/156344708848454_zh-CN.jpg)

-   创建MaxCompute项目

    开通MaxCompute服务并创建好项目，详情请参见[开通MaxCompute](../../../../cn.zh-CN/准备工作/开通MaxCompute.md#)。本示例中在华东1（杭州）区域创建项目bigdata\_DOC，同时启动DataWorks相关服务。


Elasticsearch是一个基于Lucene的搜索服务器，它提供了一个分布式多用户功能的全文搜索引擎。Elasticsearch是遵从Apache开源条款的一款开源产品，是当前主流的企业级搜索引擎。

阿里云Elasticsearch提供Elasticsearch 5.5.3 with Commercial Feature、6.3.2 with Commercial Feature、6.7.0 with Commercial Feature及商业插件X-pack服务，致力于数据分析、数据搜索等场景服务。在开源Elasticsearch基础上提供企业级权限管控、安全监控告警、自动报表生成等功能。

1.  创建测试表 将阿里云上的数据导入至阿里云Elasticsearch（离线），具体操作请参见[云上数据导入](https://help.aliyun.com/document_detail/62584.html)。

    ![测试表](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/354761/156344708848514_zh-CN.jpg)

2.  创建接收表 为保证MaxCompute可以顺利接收Elasticsearch数据，您需要在MaxCompute上创建表。本示例使用非分区表。
    1.  登录DataWorks控制台进行创建表的操作，详情请参见[表管理](../../../../cn.zh-CN/使用指南/数据开发/表管理.md#)。![创建表](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/354761/156344708848494_zh-CN.jpg)


    2.  为表添加字段。请确保添加的字段与需要被同步的Elasticsearch表字段相对应。![添加字段](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/354761/156344708948495_zh-CN.jpg)


3.  同步数据 
    1.  在业务流程中，右键单击**数据集成**，选择**新建数据集成节点** \> **数据同步**。![数据同步](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/354761/156344708948496_zh-CN.jpg)


    2.  单击下图框中的按钮，转换为**脚本模式**。![脚本模式](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/354761/156344708948497_zh-CN.jpg)


    3.  脚本配置如下所示。代码释义请参见[配置Elasticsearch Reader](../../../../cn.zh-CN/使用指南/数据集成/作业配置/配置Reader插件/配置Elasticsearch Reader.md#)。 

        ``` {#codeblock_1t9_kyh_60l .language-json}
        {
            "type": "job",
            "steps": [
                {
                    "stepType": "elasticsearch",
                    "parameter": {
                        "retryCount": 3,
                        "column": [
                            "age",
                            "job",
                            "marital",
                            "education",
                            "default",
                            "housing",
                            "loan",
                            "contact",
                            "month",
                            "day_of_week",
                            "duration",
                            "campaign",
                            "pdays",
                            "previous",
                            "poutcome",
                            "emp_var_rate",
                            "cons_price_idx",
                            "cons_conf_idx",
                            "euribor3m",
                            "nr_employed",
                            "y"
                        ],
                        "scroll": "1m",
                        "index": "es_index",
                        "pageSize": 1,
                        "sort": {
                            "age": "asc"
        },
                        "type": "elasticsearch",
                        "connTimeOut": 1000,
                        "retrySleepTime": 1000,
                        "endpoint": "http://es-cn-xxxx.xxxx.xxxx.xxxx.com:9200",
                        "password": "xxxx",
                        "search": {
                            "match_all": {}
                        },
                        "readTimeOut": 5000,
                        "username": "xxxx"
                    },
                    "name": "Reader",
                    "category": "reader"
                },
                {
                    "stepType": "odps",
                    "parameter": {
                        "partition": "",
                        "truncate": true,
                        "compress": false,
                        "datasource": "odps_first",
                        "column": [
                            "age",
                            "job",
                            "marital",
                            "education",
                            "default",
                            "housing",
                            "loan",
                            "contact",
                            "month",
                            "day_of_week",
                            "duration",
                            "campaign",
                            "pdays",
                            "previous",
                            "poutcome",
                            "emp_var_rate",
                            "cons_price_idx",
                            "cons_conf_idx",
                            "euribor3m",
                            "nr_employed",
                            "y"
                        ],
                        "emptyAsNull": false,
                        "table": "elastic2mc_bankdata"
                    },
                    "name": "Writer",
                    "category": "writer"
                }
            ],
            "version": "2.0",
            "order": {
                "hops": [
                    {
                        "from": "Reader",
                        "to": "Writer"
                    }
                ]
            },
            "setting": {
                "errorLimit": {
                    "record": "0"
                },
                "speed": {
                    "throttle": false,
                    "concurrent": 1,
                    "dmu": 1
                }
            }
        }
        ```

        您可以在创建的阿里云Elasticsearch集群的**基本信息**中，查看公网地址和公网端口，并将这些信息填入上面的代码中。

        ![公网端口](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/354761/156344709048498_zh-CN.jpg)

    4.  完成脚本配置后，单击**运行**。![运行](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/354761/156344709048499_zh-CN.jpg)


    5.  在**运行日志**中查看运行结果。![运行日志](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/354761/156344709048500_zh-CN.jpg)


4.  验证结果 
    1.  新建一个数据开发任务运行SQL语句，查看当前表中是否已存在从Elasticsearch同步过来的数据。![验证结果](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/354761/156344709148502_zh-CN.jpg)


    2.  执行结果如下所示。![执行结果](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/354761/156344709148503_zh-CN.jpg)



