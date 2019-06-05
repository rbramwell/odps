# TableTunnel {#concept_exm_f3g_vdb .concept}

TableTunnel是访问MaxCompute Tunnel服务的入口类，仅支持表数据（非视图）的上传和下载。

## TableTunnel接口定义及说明 {#section_6ku_far_yjw .section}

TableTunnel接口定义如下，详情请参见[Java-sdk-doc](http://repo.aliyun.com/java-sdk-doc/)。

``` {#codeblock_vlb_twr_z54 .language-java}
public class TableTunnel {
 public DownloadSession createDownloadSession(String projectName, String tableName);
 public DownloadSession createDownloadSession(String projectName, String tableName, PartitionSpec partitionSpec);
 public UploadSession createUploadSession(String projectName, String tableName);
 public UploadSession createUploadSession(String projectName, String tableName, PartitionSpec partitionSpec);
 public DownloadSession getDownloadSession(String projectName, String tableName, PartitionSpec partitionSpec, String id);
 public DownloadSession getDownloadSession(String projectName, String tableName, String id);
 public UploadSession getUploadSession(String projectName, String tableName, PartitionSpec partitionSpec, String id);
 public UploadSession getUploadSession(String projectName, String tableName, String id);
}
```

接口说明如下：

-   生命周期：从TableTunnel实例被创建开始，一直到程序结束。
-   TableTunnel提供创建UploadSession对象和DownloadSession对象的方法。数据的上传和下载分别由TableTunnel.UploadSession和TableTunnel.DownloadSession这两个会话实现。
-   对一张表或Partition上传下载的过程，称为一个Session。Session由一或多个到Tunnel RESTful API的HTTP Request组成。
-   TableTunnel的上传Session是insert into语义，即对同一张表或Partition的多个/多次上传Session互不影响，每个Session上传的数据会位于不同的目录中。
-   在上传Session中，每个RecordWriter对应一个HTTP Request，由一个Block ID标识，对应Service端一个文件（实际上Block id就是对应的文件名）。
-   同一Session中，使用同一Block ID多次打开RecordWriter会导致覆盖行为，最后一个调用close\(\)的RecordWriter所上传的数据会被保留。该特性可用于Block的上传失败重传。

## TableTunnel接口实现流程 {#section_4d2_fvq_o6v .section}

1.  RecordWriter.write\(\)将数据上传到临时目录的文件。
2.  RecordWriter.close\(\)将相应的文件从临时目录挪移到data目录。
3.  session.commit\(\)将相应Data目录下的所有文件挪移到相应表所在目录，并更新表Meta，即数据进表，对其他MaxCompute任务（例如SQL、MR）可见。

## TableTunnel接口限制 {#section_e76_tln_rmo .section}

-   Block ID的取值范围是\[0, 20000\)，单个Block上传的数据限制为100G。
-   Session用Session ID来标识。Session的超时时间为24小时。如果大批量数据传送导致超过24小时，需要自行拆分成多个Session。
-   RecordWriter对应的HTTP Request超时为120s。如果120s内HTTP连接上没有数据流过，Service端会主动关闭连接。

    **说明：** HTTP本身还有8k Buffer，因此并不是每次调用RecordWriter.write\(\)都能保证HTTP连接上有数据流过。TunnelRecordWriter.flush\(\)可以将Buffer内数据强制刷出。

-   对于日志类写入MaxCompute的场景，无法预测数据的到达会造成RecordWriter超时。此时：
    -   不建议对每条数据打开一个RecordWriter（因为每个RecordWriter对应一个文件，会造成小文件过多，严重影响MaxCompute后续使用性能）。
    -   建议用户代码Cache至少64M的数据后，再使用一个RecordWriter进行一次性批量写入。
-   RecordReader的超时为300s。

