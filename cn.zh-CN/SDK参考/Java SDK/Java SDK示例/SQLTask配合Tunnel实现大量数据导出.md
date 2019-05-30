# SQLTask配合Tunnel实现大量数据导出 {#concept_388499 .concept}

## SQLTask {#section_pd4_rqe_ew3 .section}

SQLTask是MaxCompute SQL的SDK接口。借助SQLTask，您可以方便地运行SQL并获得其返回结果。关于SQLTask的详细描述，请参见[Java SDK概述](cn.zh-CN/SDK参考/Java SDK/Java SDK概述.md#)。

SQLTask.getResult\(i\)返回的是一个list，您可以循环迭代这个list以获得完整的SQL计算返回结果。但是，此方法有限制条件：目前select语句返回至客户端的数据条数最大值为1万（参见[其他操作](../../../../cn.zh-CN/开发/常用命令/其他操作.md#)中READ\_TABLE\_MAX\_ROW的取值范围）。即如果在客户端上（包括使用SQLTask SDK）直接select，相当于查询结果上最后加上了limit 10000（如果使用CREATE TABLE XX AS SELECT或用INSERT INTO/OVERWRITE TABLE把结果固化到具体的表里，则无这项限制）。

## Tunnel {#section_9x7_6y4_o0t .section}

如果需要导出的查询结果是某张表或具体某个分区的全部内容，可以使用Tunnel命令行共计完成。MaxCompute提供了Tunnel命令行工具和Tunnel SDK，详情请参见[上传下载命令](../../../../cn.zh-CN/开发/数据上传下载/上传下载命令.md#)和[批量数据通道概要](../../../../cn.zh-CN/开发/数据上传下载/批量数据通道SDK介绍/批量数据通道概要.md#)。

使用Tunnel命令行工具导出数据示例

``` {#codeblock_jrn_pj4_d2q}
>tunnel d wc_out c:\wc_out.dat;
2017-12-16 19:32:08  -  new session: 201712161932082dxxxxx     total lines: 3
2017-12-16 19:32:08  -  file [0]: [0, 3), c:\wc_out.dat
downloading 3 records into 1 file
2017-12-16 19:32:08  -  file [0] start
2017-12-16 19:32:08  -  file [0] OK. total: 21 bytes
download OK
```

## SQLTask配合Tunnel实现超出1万行的运行结果导出 {#section_fw2_pos_z2b .section}

SQLTask不能处理超过1万条的记录，但是Tunnel可以，两者互补。因此可以基于两者实现数据的导出，代码示例如下。

``` {#codeblock_8m5_8mj_2h7}
private static final String accessId = "userAccessId";
private static final String accessKey = "userAccessKey";
private static final String endPoint = "http://service.odps.aliyun.com/api";
private static final String project = "userProject";
private static final String sql = "userSQL";
private static final String table = "Tmp_" + UUID.randomUUID().toString().replace("-", "_");//此处使用随机字符串作为临时导出存放数据的表的名字
private static final Odps odps = getOdps();
public static void main(String[] args) {
    System.out.println(table);
    runSql();
    tunnel();
}
/*
     * 下载SQLTask的结果。
     * */
private static void tunnel() {
    TableTunnel tunnel = new TableTunnel(odps);
    try {
        DownloadSession downloadSession = tunnel.createDownloadSession(project, table);
        System.out.println("Session Status is : "+ downloadSession.getStatus().toString());
        long count = downloadSession.getRecordCount();
        System.out.println("RecordCount is: " + count);
        RecordReader recordReader = downloadSession.openRecordReader(0, count);
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
     * 保存这条数据。
     * 如果数据量少，可以直接打印结果后拷贝。实际使用场景下也可以使用JAVA.IO写到本地文件，或写到远端保存数据结果。
     * */
private static void consumeRecord(Record record, TableSchema schema) {
    System.out.println(record.getString("username")+","+record.getBigint("cnt"));
}
/*
     * 运行SQL ,把查询结果保存成临时表，方便后续用Tunnel下载。
     * 保存数据的lifecycle此处设置为1天，如果删除步骤失败，也不会浪费过多存储空间。
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
     * 初始化MaxCompute的连接信息。
     * */
private static Odps getOdps() {
    Account account = new AliyunAccount(accessId, accessKey);
    Odps odps = new Odps(account);
    odps.setEndpoint(endPoint);
    odps.setDefaultProject(project);
    return odps;
}
```

