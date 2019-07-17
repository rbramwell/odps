# JSON数据从MongoDB迁移至MaxCompute {#concept_z2b_fl2_xfb .concept}

本文将为您介绍如何通过DataWorks的数据集成功能，将从MongoDB提取的JSON字段迁移至MaxCompute。

## 准备工作 {#section_qv1_nm2_xfb .section}

1.  账号准备

    在数据库内新建用户，用于DataWorks添加数据源。本示例执行如下命令。

    ``` {#codeblock_118_qh9_eas .language-json}
    db.createUser({user:"bookuser",pwd:"123456",roles:["root"]})
    ```

    新建用户名为bookuser，密码为123456，权限为root。

2.  数据准备

    首先您需要将数据上传至您的MongoDB数据库。本示例使用阿里云的[云数据库MongoDB版](../../../../intl.zh-CN/单节点快速入门/MongoDB单节点实例使用流程.md#)，网络类型为VPC（需申请公网地址，否则无法与DataWorks默认资源组互通），测试数据如下。

    ``` {#codeblock_68l_o3p_q7p .language-json}
    {
                            "store": {
                            "book": [
                            {
                            "category": "reference",
                            "author": "Nigel Rees",
                            "title": "Sayings of the Century",
                            "price": 8.95
                            },
                            {
                            "category": "fiction",
                            "author": "Evelyn Waugh",
                            "title": "Sword of Honour",
                            "price": 12.99
                            },
                            {
                            "category": "fiction",
                            "author": "J. R. R. Tolkien",
                            "title": "The Lord of the Rings",
                            "isbn": "0-395-19395-8",
                            "price": 22.99
                            }
                            ],
                            "bicycle": {
                            "color": "red",
                            "price": 19.95
                            }
                            },
                            "expensive": 10
                            }
    ```

    登录MongoDB的DMS控制台，本示例使用的数据库为admin，集合为userlog。您可以在查询窗口执行如下命令，查看已上传的数据。

    ``` {#codeblock_dmy_50a_enx .language-json}
    db.userlog.find().limit(10)
    ```

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/64919/156335838532875_zh-CN.png)


## 通过DataWorks将JSON数据从MongoDB迁移至MaxCompute {#section_sk4_zq2_xfb .section}

1.  新增MongoDB数据源
    1.  以项目管理员身份进入[DataWorks控制台](https://workbench.data.aliyun.com/console)，单击对应工作空间操作栏中的**进入数据集成**。
    2.  选择**同步资源管理** \> **数据源**，单击**新增数据源**。

        ![新增数据源](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/64919/156335838551682_zh-CN.png)

    3.  在新增数据源弹出框中，选择数据源类型为**MongoDB**。
    4.  填写MongoDB数据源的各配置项。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/64919/156335838532877_zh-CN.png)

        |配置|说明|
        |--|--|
        |**数据源类型**|由于本文中MongoDB处于VPC环境下，因此**数据源类型**需选择**连接串模式（数据集成网络可直接连通）**。|
        |**数据源名称**|数据源名称必须以字母、数字、下划线组合，且不能以数字和下划线开头。|
        |**数据源描述**|对数据源进行简单描述，不得超过80个字符。|
        |**适用环境**|可以选择**开发**或**生产**环境。 **说明：** 仅标准模式工作空间会显示此配置。

 |
        |**访问地址**|访问地址及端口号可以通过单击MongoDB控制台中的实例名称获取。|
        |**数据库名**|该数据源对应的数据库名称。|
        |**用户名/密码**|数据库对应的用户名和密码。|

    5.  单击**测试连通性**。
    6.  测试连通性通过后，单击**完成**。
2.  新建数据同步任务

    在DataWorks上新建数据同步节点，详情请参见[数据同步节点](../../../../intl.zh-CN/使用指南/数据开发/节点类型/数据同步节点.md#)。

    ![新建数据同步节点](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/156335838631543_zh-CN.png)

    新建的同时，在DataWorks新建一个[建表任务](../../../../intl.zh-CN/使用指南/数据开发/表管理.md#)，用于存放JSON数据，本示例新建表名为mqdata。

    ![新建表](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/156335838631544_zh-CN.png)

    表参数可以通过图形化界面完成。本例中mqdata表仅有一列，类型为string，列名为MQ data。

    ![图形化界面新建表](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/156335838631545_zh-CN.png)

3.  配置同步任务参数

    完成上述新建后，您可以在图形化界面配置数据同步任务参数，如下图所示。选择数据来源类型为MongoDB，来源表为mongodb\_userlog。目标数据源名称为odps\_first，目标表为刚新建的mqdata。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/64919/156335838632880_zh-CN.png)

    由于MongoDB数据源不支持向导模式开发，您直接**点击转换为脚本**，即可跳转至脚本模式进行配置。

    ``` {#codeblock_rl7_nmi_8zy .language-json}
    {
        "type": "job",
        "steps": [
        {
            "stepType": "mongodb",
            "parameter": {
                "datasource": "mongodb_userlog",//数据源名称。
                "column": [
                    {
                    "name": "store.bicycle.color", //JSON字段路径，本例中提取color值。
                    "type": "document.String" //非一层子属性以最终获取的类型为准。假如您选取的JSON字段为一级字段，如本例中的expensive，则直接填写string即可。
                    }
                  ],
                "collectionName //集合名称": "userlog"
                },
            "name": "Reader",
            "category": "reader"
            },
            {
                "stepType": "odps",
                "parameter": {
                "partition": "",
                "isCompress": false,
                "truncate": true,
                "datasource": "odps_first",
                "column": [
                "mqdata"  //MaxCompute表列名。
                ],
                "emptyAsNull": false,
                "table": "mqdata"
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
                "record": ""
                },
                "speed": {
                "concurrent": 2,
                "throttle": false,
                }
                }
            }
    ```

    完成上述配置后，单击**运行**即可。运行成功日志示例如下所示。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/156335838631550_zh-CN.png)


## JSON数据从MongoDB迁移至MaxCompute结果验证 {#section_jvp_ix3_kfz .section}

1.  在您的[业务流程](../../../../intl.zh-CN/使用指南/数据开发/业务流程/业务流程介绍.md#)中新建一个ODPS SQL节点。

    ![新建ODPS SQL节点](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/156335838631551_zh-CN.png)

2.  输入`SELECT * from mqdata;`语句，查看当前mqdata表中数据。

    ![结果验证](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/64919/156335838732881_zh-CN.png)

    **说明：** 您也可以直接在[MaxCompute客户端](../../../../intl.zh-CN/工具及下载/客户端.md#)中输入命令运行。


