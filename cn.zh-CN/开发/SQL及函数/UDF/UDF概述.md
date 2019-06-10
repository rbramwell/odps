# UDF概述 {#concept_dkq_tn2_vdb .concept}

UDF全称为User Defined Function，即用户自定义函数。MaxCompute提供多种内建函数来满足您的计算需求，同时您还可以通过创建自定义函数来满足不同的计算需求。

UDF的使用方法与普通的[内建函数](cn.zh-CN/开发/SQL及函数/内建函数/数学函数.md#)类似。Java和MaxCompute的数据类型的对应关系，请参见[参数与返回值类型](cn.zh-CN/开发/SQL及函数/UDF/Java UDF.md#)。

## UDF分类 {#section_yc9_8pm_blq .section}

MaxCompute支持的UDF有三种，如下表所示。

|UDF 分类|描述|
|:-----|:-|
|UDF（User Defined Scalar Function）|用户自定义标量值函数（User Defined Scalar Function）。其输入与输出是一对一的关系，即读入一行数据，写出一条输出值 。|
|UDTF（User Defined Table Valued Function）|自定义表值函数。用来解决一次函数调用输出多行数据场景。它是唯一能够返回多个字段的自定义函数，而UDF只能一次计算输出一条返回值。|
|UDAF（User Defined Aggregation Function）|自定义聚合函数。其输入与输出是多对一的关系， 即将多条输入记录聚合成一条输出值。它可以与SQL中的GROUP BY语句联用。具体语法请参见[聚合函数](cn.zh-CN/开发/SQL及函数/内建函数/聚合函数.md#)。|

**说明：** 

-   UDF广义的说法代表了自定义标量函数、自定义聚合函数及自定义表函数三种类型的自定义函数的集合。狭义的说法仅代表用户自定义标量函数。在帮助文档中将经常出现UDF这一名词，请您根据上下文内容理解其具体含义。
-   在SQL语句中使用自定义函数时，可能提示内存不足。这是由于计算的数据量过大并且存在倾斜，导致任务超出默认分配的内存。这种情况下，您可以通过`set odps.sql.udf.joiner.jvm.memory=xxxx;set odps.sql.udf.joiner.jvm.memory=xxxx;`语句手动设置更大的内存。

## 跨项目分享 {#section_uw6_u3e_4yz .section}

MaxCompute的UDF支持跨项目分享，即在project\_a中可以使用project\_b的UDF。具体的授权方式参见[跨项目空间Table、Resource、Function分享示例](../../../../cn.zh-CN/管理/安全功能详解/用户及授权管理/授权.md#section_fba_w66_us6)。使用代码举例：`select other_project:udf_in_other_project(arg0, arg1) as res from table_t;`。

## UDF SDK依赖安装 {#section_2sd_vmn_fpd .section}

如果您使用[Maven](http://search.maven.org/)，可以从 [Maven 库](http://search.maven.org/)中搜索odps-sdk-udf，从而获取不同版本的Java SDK，相关配置信息如下所示：

``` {#codeblock_1y6_9eo_sjp .language-xml}
<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>odps-sdk-udf</artifactId>
   <version>0.29.10-public</version>
</dependency>
```

