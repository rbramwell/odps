# JSON数据从OSS迁移至MaxCompute最佳实践 {#concept_gyw_dhd_5fb .concept}

本文将为您介绍如何通过DataWorks的数据集成，将JSON数据从OSS迁移至MaxCompute，并使用MaxCompute内置字符串函数GET\_JSON\_OBJECT提取JSON信息的最佳实践。

## 准备工作 {#section_fmj_3hd_5fb .section}

-   数据上传OSS

    将您的JSON文件重命名后缀为TXT文件，并上传到OSS。本文中使用的JSON文件示例如下。

    ``` {#codeblock_vpj_54j_c6p}
    
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

    将applog.txt文件上传到OSS，本文中OSS Bucket位于华东2区。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/156318711831516_zh-CN.png)


## 使用DataWorks将JSON数据从OSS迁移到MaxCompute {#section_zcj_s3d_5fb .section}

1.  新增OSS数据源
    1.  以项目管理员身份进入[DataWorks控制台](https://workbench.data.aliyun.com/console)，单击对应工作空间操作栏中的**进入数据集成**。
    2.  选择**同步资源管理** \> **数据源**，单击**新增数据源**。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/156318711831532_zh-CN.png)

    3.  在新增数据源弹出框中，选择数据源类型为**OSS**。
    4.  填写OSS数据源的各配置项。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/156318711851464_zh-CN.png)

        |配置|说明|
        |:-|:-|
        |**数据源名称**|数据源名称必须以字母、数字、下划线组合，且不能以数字和下划线开头。|
        |**数据源描述**|对数据源进行简单描述，不得超过80个字符。|
        |**适用环境**|可以选择**开发**或**生产**环境。 **说明：** 仅标准模式工作空间会显示此配置。

 |
        |**Endpoint**|OSS Endpoint信息，本示例为http://oss-cn-shanghai.aliyuncs.com或http://oss-cn-shanghai-internal.aliyuncs.com。OSS各地域的外网、内网地址请参见[OSS开通Region和Endpoint对照表](../../../../intl.zh-CN/开发指南/访问域名（Endpoint）/访问域名和数据中心.md#section_plb_2vy_5db)。 **说明：** 由于本文中OSS和DataWorks项目处于同一个区域中，所以本文选用后者，通过内网连接。

 |
        |**Bucket**|相应的OSS Bucket信息，指存储空间，是用于存储对象的容器。 您可以创建一个或多个存储空间，每个存储空间可添加一个或多个文件。

 您可以在数据同步任务中查找此处填写的存储空间中相应的文件，没有添加的存储空间，则不能查找其中的文件。

 |
        |**AccessKey ID/AceessKey Secret**|访问秘匙（AccessKeyID和AccessKeySecret），相当于登录密码。|

    5.  单击**测试连通性**。
    6.  测试连通性通过后，单击**完成**。
2.  新建数据同步任务

    在DataWorks上新建数据同步节点，详情请参见[数据同步节点](../../../../intl.zh-CN/使用指南/数据开发/节点类型/数据同步节点.md#)。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/156318711831543_zh-CN.png)

    新建的同时，在DataWorks新建一个[建表任务](../../../../intl.zh-CN/使用指南/数据开发/表管理.md#)，用于存放JSON数据，本示例新建表名为mqdata。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/156318711831544_zh-CN.png)

    表参数可以通过图形化界面完成。本例中mqdata表仅有一列，类型为string，列名为MQ data。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/156318711931545_zh-CN.png)

3.  配置同步任务参数

    完成上述新建后，您可以在图形化界面配置数据同步任务参数，如下图所示。选择目标数据源名称为odps\_first，选择目标表为刚建立的mqdata。数据来源类型为OSS，Object前缀可填写文件路径及名称。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/156318711931546_zh-CN.png)

    **说明：** 列分隔符使用TXT文件中不存在的字符即可，本文使用（^）。对于OSS中的TXT格式数据源，Dataworks支持多字符分隔符，您可以使用（%&%\#^$$^%）这种很难出现的字符串作为列分隔符。

    映射方式选择默认的同行映射即可。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/156318711931548_zh-CN.png)

    单击左上方的切换脚本按钮，切换为脚本模式。修改fileFormat参数为`"fileFormat":"binary"`。脚本模式代码示例如下。

    ``` {#codeblock_rff_3x1_pkw}
    {
        "type": "job",
        "steps": [
            {
                "stepType": "oss",
                "parameter": {
                    "fieldDelimiterOrigin": "^",
                    "nullFormat": "",
                    "compress": "",
                    "datasource": "OSS_userlog",
                    "column": [
                        {
                            "name": 0,
                            "type": "string",
                            "index": 0
                        }
                    ],
                    "skipHeader": "false",
                    "encoding": "UTF-8",
                    "fieldDelimiter": "^",
                    "fileFormat": "binary",
                    "object": [
                        "applog.txt"
                    ]
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
                        "mqdata"
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

    **说明：** 该步骤可以保证OSS中的JSON文件同步到MaxCompute之后存在同一行数据中，即为一个字段，其他参数保持不变。

    完成上述配置后，单击**运行**即可。运行成功日志示例如下所示。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/156318711931550_zh-CN.png)


## JSON数据从OSS迁移到MaxCompute结果验证 {#section_opc_bp3_pgb .section}

1.  在您的[业务流程](../../../../intl.zh-CN/使用指南/数据开发/业务流程/业务流程介绍.md#)中新建一个ODPS SQL节点。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/156318711931551_zh-CN.png)

2.  查看当前mqdata表中数据，输入`SELECT * from mqdata;`语句。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/156318711931552_zh-CN.png)

    **说明：** 这一步及后续步骤，也可以直接在[MaxCompute客户端](../../../../intl.zh-CN/工具及下载/客户端.md#)中输入命令运行。

3.  确认导入表中的数据结果无误后，使用`SELECT GET_JSON_OBJECT(mqdata.MQdata,'$.expensive') FROM mqdata;`获取JSON文件中的expensive值。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/156318711931553_zh-CN.png)


## 更多信息 {#section_y4g_zlb_pgb .section}

在进行迁移后结果验证时，您可以使用MaxCompute内建字符串函数[GET\_JSON\_OBJECT](../../../../intl.zh-CN/开发/SQL及函数/内建函数/字符串函数.md#section_cdt_gxz_vdb)获取您想要的JSON数据。

