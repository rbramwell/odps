# MaxCompute数据迁移至OTS {#task_1375125 .task}

本文为您介绍如何将MaxCompute数据迁移至Table Store（OTS）。

请提前开通MaxCompute和DataWorks服务。本例使用DataWorks简单模式。

1.  在DataWorks控制台中创建表。 
    1.  新建业务流程，并将业务流程命名为mc2ots。![新建业务流程](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095436/156440141053178_zh-CN.jpg)


    2.  新建表transs。![新建表](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095436/156440141153193_zh-CN.jpg)


    3.  输入表的基本属性。![表基本属性](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095436/156440141153197_zh-CN.jpg)


    4.  单击**添加字段**，创建表字段name、id、gender。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095436/156440141153695_zh-CN.jpg)


    5.  单击**提交​到生产环境**。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095436/156440141153765_zh-CN.jpg)


    6.  导入表数据。![导入表数据](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095436/156440141253347_zh-CN.jpg)

 表数据如下。

        ``` {#codeblock_h42_ql7_80a}
        qwe,145,F
        asd,256,F
        xzc,345,M
        rgth,234,F
        ert,456,F
        dfg,12,M
        tyj,4,M
        bfg,245,M
        nrtjeryj,15,F
        rwh,2344,M
        trh,387,F
        srjeyj,67,M
        saerh,567,M
        ```

2.  在表格存储Table Store控制台中创建表。 
    1.  登录[表格存储Table Store控制台](https://ots.console.aliyun.com/index)，单击**创建实例**。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095436/156440141253656_zh-CN.jpg)


    2.  创建实例mctoots。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095436/156440141253657_zh-CN.jpg)


    3.  单击**创建数据表**。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095436/156440141353661_zh-CN.jpg)


    4.  创建数据表Trans。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095436/156440141353698_zh-CN.jpg)


3.  新增数据源。 
    1.  单击**新增数据源**。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095436/156440141353716_zh-CN.jpg)


    2.  新增MaxCompute（ODPS）数据源，并将数据源命名为odps\_first。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095436/156440141453761_zh-CN.jpg)


    3.  新增Table Store（OTS）数据源，并将数据源命名为Transs。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095436/156440141453719_zh-CN.jpg)


4.  配置MaxCompute（ODPS）Reader和Table Store（OTS）Writer。 
    1.  新建数据同步节点mc2ots。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095436/156440141453721_zh-CN.jpg)


    2.  在脚本模式中，导入模板。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095436/156440141553722_zh-CN.jpg)


    3.  修改JSON代码后，单击运行按钮。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095436/156440141553741_zh-CN.jpg)

 代码如下。

        ``` {#codeblock_kz9_l03_bnp}
        {
            "type": "job",
            "steps": [
                {
                    "stepType": "odps",
                    "parameter": {
                        "partition": [],
                        "datasource": "odps_first",
                        "column": [
                            "name",
                            "id",
                            "gender"
                        ],
                        "table": "Transs"
                    },
                    "name": "Reader",
                    "category": "reader"
                },
                {
                    "stepType": "ots",
                    "parameter": {
                        "datasource": "Transs",
                        "column": [
                            {
                                "name": "Gender",
                                "type": "STRING"
                            }
                        ],
                        "writeMode": "UpdateRow",
                        "table": "Trans",
                        "primaryKey": [
                            {
                                "name": "Name",
                                "type": "STRING"
                            },
                            {
                                "name": "ID",
                                "type": "INT"
                            }
                        ]
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

5.  在表格存储Table Store（OTS）控制台中查看新增的表数据。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1095436/156440141553788_zh-CN.jpg)



