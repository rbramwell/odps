# 导出SQL的运行结果 {#concept_bsq_ctc_5db .concept}

本文将通过示例，为您介绍几种下载MaxCompute SQL计算结果的方法。

**说明：** 本文中所有SDK部分仅举例介绍Java的例子。

您可以通过以下几种方法导出SQL的运行结果：

-   如果数据比较少，可以直接用[SQL Task](../../../../intl.zh-CN/SDK参考/Java SDK/Java SDK介绍.md)得到全部的查询结果。
-   如果只是想导出某个表或者分区，可以用 [Tunnel](../../../../intl.zh-CN/开发/数据上传下载/Tunnel上传下载命令.md)直接导出数据。
-   如果SQL比较复杂，需要Tunnel和SQL相互配合才行。
-   [DataWorks](https://data.aliyun.com/product/ide?) 可以方便地帮您运行SQL，同步数据，并有定时调度，配置任务依赖的功能。
-   开源工具**DataX** 可帮助您方便地把MaxCompute中的数据导出到目标数据源。

## SQLTask方式导出 {#section_pyd_ntc_5db .section}

[SQLTask](../../../../intl.zh-CN/SDK参考/Java SDK/Java SDK介绍.md) 是 SDK直接调用MaxCompute SQL的接口，能很方便地运行SQL并获得其返回结果。

从文档可以看到，SQLTask.getResult\(i\)返回的是一个 List，可以循环迭代这个List，获得完整的SQL计算返回结果。不过该方法有一个缺陷，详情请参见[SET操作](../../../../intl.zh-CN/开发/常用命令/SET操作.md#)中的`SetProject READ_TABLE_MAX_ROW`功能。

目前Select语句返回给客户端的数据条数最大可以调整到 **1万**。也即如果在客户端上（包括 SQLTask）直接进行Select操作，相当于查询结果上最后加上了Limit N参数（如果使用CREATE TABLE XX AS SELECT或者用INSERT INTO/OVERWRITE TABLE把结果固化到具体的表里则没有影响）。

**说明：** SQLTask.getResult\(i\)用于导出Select查询结果，不适用于导出show tables;等其他MaxCompute命令操作结果。

## Tunnel 方式导出 {#section_ryd_ntc_5db .section}

如果您需要导出的查询结果是某张表的全部内容（或者是具体的某个分区的全部内容），可以通过Tunnel来实现，详情请参见 [命令行工具](../../../../intl.zh-CN/开发/数据上传下载/Tunnel上传下载命令.md) 和基于SDK编写的 [Tunnel SDK](../../../../intl.zh-CN/开发/数据上传下载/批量数据通道SDK介绍/批量数据通道概要.md)。

此处提供一个Tunnel命令行导出数据的简单示例，Tunnel SDK的编写适用于Tunnel命令行无法支持的场景，详情请参见 [批量数据通道概述](../../../../intl.zh-CN/开发/数据上传下载/批量数据通道SDK介绍/批量数据通道概要.md)。

``` {#codeblock_7vy_nnv_30w}
tunnel d wc_out c:\wc_out.dat;
2016-12-16 19:32:08 - new session: 201612161932082d3c9b0a012f68e7 total lines: 3
2016-12-16 19:32:08 - file [0]: [0, 3), c:\wc_out.dat
downloading 3 records into 1 file
2016-12-16 19:32:08 - file [0] start
2016-12-16 19:32:08 - file [0] OK. total: 21 bytes
download OK
```

## SQLTask + Tunnel方式导出 {#section_vyd_ntc_5db .section}

从前面SQL Task方式导出的介绍可以看到，SQL Task不能处理超过1万条记录，而 Tunnel可以，两者可以互补。所以可以基于两者实现数据的导出。

代码实现的示例如下：

``` {#codeblock_67j_khp_d74 .language-java}
private static final String accessId = "userAccessId";
    private static final String accessKey = "userAccessKey";
    private static final String endPoint = "http://service.odps.aliyun.com/api";
    private static final String project = "userProject";
    private static final String sql = "userSQL";
    private static final String table = "Tmp_" + UUID.randomUUID().toString().replace("-", "_");//其实也就是随便找了个随机字符串作为临时表的名字
    private static final Odps odps = getOdps();

    public static void main(String[] args) {
        System.out.println(table);
        runSql();
        tunnel();
    }

    /*
     * 把SQLTask的结果下载过来
     * */
    private static void tunnel() {
        TableTunnel tunnel = new TableTunnel(odps);
        try {
            DownloadSession downloadSession = tunnel.createDownloadSession(
                    project, table);
            System.out.println("Session Status is : "
                    + downloadSession.getStatus().toString());
            long count = downloadSession.getRecordCount();
            System.out.println("RecordCount is: " + count);
            RecordReader recordReader = downloadSession.openRecordReader(0,
                    count);
            Record record;
            while ((record = recordReader.read()) != null) {
                consumeRecord(record, downloadSession.getSchema());
            }
            recordReader.close();
        } catch (TunnelException e) {
            e.printStackTrace();
        } catch (IOException e1) {
            e1.printStackTrace();
        }
    }

    /*
     * 保存这条数据
     * 数据量少的话直接打印后拷贝走也是一种取巧的方法。实际场景可以用Java.io写到本地文件，或者写到远端数据等各种目标保存起来。
     * */
    private static void consumeRecord(Record record, TableSchema schema) {
        System.out.println(record.getString("username")+","+record.getBigint("cnt"));
    }

    /*
     * 运行SQL，把查询结果保存成临时表，方便后面用Tunnel下载
     * 这里保存数据的lifecycle为1天，所以哪怕删除步骤出了问题，也不会太浪费存储空间
     * */
    private static void runSql() {
        Instance i;
        StringBuilder sb = new StringBuilder("Create Table ").append(table)
                .append(" lifecycle 1 as ").append(sql);
        try {
            System.out.println(sb.toString());
            i = SQLTask.run(getOdps(), sb.toString());
            i.waitForSuccess();

        } catch (OdpsException e) {
            e.printStackTrace();
        }
    }

    /*
     * 初始化MaxCompute(原ODPS)的连接信息
     * */
    private static Odps getOdps() {
        Account account = new AliyunAccount(accessId, accessKey);
        Odps odps = new Odps(account);
        odps.setEndpoint(endPoint);
        odps.setDefaultProject(project);
        return odps;
    }
```

## 大数据开发套件的数据同步方式导出 {#section_d12_ntc_5db .section}

前面介绍的方式解决了数据下载后保存的问题，但是没解决数据的生成以及两个步骤之间的调度依赖的问题。

[数加·DataWorks](https://data.aliyun.com/product/ide?) 可以运行SQL、配置数据同步任务，还可以设置自动 [周期性运行](https://www.alibabacloud.com/help/doc-detail/50130.htm) 和 [多任务之间依赖](https://www.alibabacloud.com/help/doc-detail/50130.htm)，彻底解决了前面的烦恼。

接下来将用一个简单示例，为您介绍如何通过大数据开发套件运行SQL并配置数据同步任务，以完成数据生成和导出需求。

 **操作步骤** 

1.  创建一个工作流，工作流里创建一个SQL节点和一个数据同步节点，并将两个节点连线配置成依赖关系，SQL节点作为数据产出的节点，数据同步节点作为数据导出节点。
2.  配置SQL节点。

    **说明：** SQL这里的创建表要先执行一次再去配置同步（否则表都没有，同步任务没办法配置）。

3.  配置数据同步任务。
    1.  选择来源。
    2.  选择目标。
    3.  字段映射。
    4.  通道控制。
    5.  预览保存。
4.  工作流调度配置完成后（可以直接使用默认配置），保存并提交工作流，然后单击 **测试运行**。查看数据同步的运行日志，如下所示：

    ``` {#codeblock_9aj_mzh_gn7}
    2016-12-17 23:43:46.394 [job-15598025] INFO JobContainer - 
    任务启动时刻 : 2016-12-17 23:43:34
    任务结束时刻 : 2016-12-17 23:43:46
    任务总计耗时 : 11s
    任务平均流量 : 31.36KB/s
    记录写入速度 : 1668rec/s
    读出记录总数 : 16689
    读写失败总数 : 0
    ```

5.  输入SQL语句查看数据同步的结果，如下图所示：


