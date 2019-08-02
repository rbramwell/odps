# Export SQL operation result {#concept_bsq_ctc_5db .concept}

This article provides examples to illustrate how to download the MaxCompute SQL computing results by using several methods.

**Note:** The Java SDK is used as an example throughout this article.

You can use one of the following methods to export the SQL statement execution results:

-   If the data volume is small, use [SQL Task](../../../../reseller.en-US/SDK Reference /Java SDK/Java SDK.md) to list all query results.

-   If you want to export the results of a specific table or partition, use [Tunnel](../../../../reseller.en-US/User Guide/Data upload and download/Tunnel commands.md).

-   If the SQL statements are complex, use Tunnel and SQL Task in combination.

-   [DataWorks](https://data.aliyun.com/product/ide?) allows you to conveniently run SQL statements and synchronize data. It supports regular scheduling and task dependency configuration.DatadataDatadependency

-   The Open Source Tool **DataX**helps you easily export data from maxcompute to the target data source.


## Use SQL Task to export data {#section_pyd_ntc_5db .section}

[SQL Task](../../../../reseller.en-US/SDK Reference /Java SDK/Java SDK.md) is the interface where the SDK calls maxcompute SQL directly, you can easily run SQL and get its return results.

`SQLTask.getResult(i);` returns a list which can be iterated cyclically to obtain the complete SQL computing results. However, there is a flaw in this method. For more information, see the `SetProject READ_TABLE_MAX_ROW` maid feature mentioned in [other actions](../../../../reseller.en-US/User Guide/Common commands/Other operations.md).

Currently, you can adjust the maximum number of data records that the SELECT statement returns to the client up to **10,000**. If you run the SELECT statement on a client or using SQL Task, the query results are appended with Limit N. Limit N does not apply to the CREATE TABLE XX AS SELECT statement or in the case that the results are solidified in a specific table through INSERT INTO/OVERWRITE TABLE.

## Use Tunnel to export data {#section_ryd_ntc_5db .section}

If you need to export a query that results in the entire contents of a table \(or a specific partition\) all of the content \), you can do this through tunnel, see [command-line](../../../../reseller.en-US/User Guide/Data upload and download/Tunnel commands.md) tools for details, and the [tunnel SDK](../../../../reseller.en-US/User Guide/Data upload and download/Tunnel SDK/Tunnel overview.md) written based on the SDK.

An example is provided to illustrate how to export data by using the Tunnel command line. You can compile the Tunnel SDK only when data cannot be exported using some command lines. For more information, see [Batch data tunnel overview](../../../../reseller.en-US/User Guide/Data upload and download/Tunnel SDK/Tunnel overview.md).

``` {#codeblock_66u_ogp_vii}
tunnel d wc_out c:\wc_out.dat;
2016-12-16 19:32:08 - new session: 201612161932082d3c9b0a012f68e7 total lines: 3
2016-12-16 19:32:08 - file [0]: [0, 3), c:\wc_out.dat
downloading 3 records into 1 file
2016-12-16 19:32:08 - file [0] start
2016-12-16 19:32:08 - file [0] OK. total: 21 bytes
download OK
```

## Use SQL Task and Tunnel to export data {#section_vyd_ntc_5db .section}

SQL Task cannot export more than 10,000 records, whereas Tunnel can. You can use them in combination You can use them in combination to export data.

The sample code is as follows:

``` {#codeblock_6sn_igm_7p9}
private static final String accessId = "userAccessId";
private static final String accessKey = "userAccessKey";
private static final String endPoint = "http://service.odps.aliyun.com/api";
private static final String project = "userProject";
private static final String sql = "userSQL";
private static final String table = "Tmp_" + UUID.randomUUID().toString().replace("-", "_");//The name of the temporary table is a random string.
private static final Odps odps = getOdps();
public static void main(String[] args) {
System.out.println(table);
runSql();
tunnel();
}
/*
* Download the results returned by SQL Task.
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
while ((record = recordReader.read()) ! = null) {
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
* Save the data record.
* If the data volume is small, you can print and copy the data directly.  In reality, you can use Java.io to write the data to a local file or a remote data storage.
* */
private static void consumeRecord(Record record, TableSchema schema) {
System.out.println(record.getString("username")+","+record.getBigint("cnt"));
}
/*
* Run an SQL statement to save the query results to a temporary table. The saved results can be downloaded using Tunnel.
* The lifecycle of the saved data is 1 day. The data does not occupy much storage space even when an error occurs while you delete the data.
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
* Initialize the connection information of the MaxCompute (formerly ODPS) instance.
* */
private static Odps getOdps() {
Account account = new AliyunAccount(accessId, accessKey);
Odps odps = new Odps(account);
odps.setEndpoint(endPoint);
odps.setDefaultProject(project);
return odps;
}
```

## Use DataWorks to export data using synchronization {#section_d12_ntc_5db .section}

Using the preceding method, you can save the downloaded data. Other methods are required to create the data and implement the scheduling dependency between data creation and storage.

[DataWorks](https://data.aliyun.com/product/ide?) allows you to configure a data synchronization task and configure [Periodic running](https://www.alibabacloud.com/help/doc-detail/50130.htm) and [Dependency among multiple tasks](https://www.alibabacloud.com/help/doc-detail/50130.htm) to complete the process from data creation to storage.

An example is provided to illustrate how to use Data IDE to run SQL statements and configure a data synchronization task to create and export data.

 **Procedure** 

1.  Create a workflow with an SQL node and a data synchronization node. Connect the two nodes and configure an inter-node dependency, with the SQL node as the data production node and the data synchronization node as the data export node.
2.  Configure the SQL node.

    **Note:** Run an SQL statement to create a table before you configure synchronization. If no table exists, the synchronization task cannot be configured.

3.  Perform the following to configure the data synchronization task.
    1.  Select a Source.
    2.  Select a Target.
    3.  Map fields.
    4.  Control the tunnel.
    5.  Preview and Save.
4.  After workflow scheduling is configured, save and submit the workflow. Click **Test Run**. If you do not configure workflow scheduling, you can use the default scheduling configuration directly. View the running log on data synchronization as as in the following figure.

    ``` {#codeblock_w63_nln_ydh}
    2016-12-17 23:43:46.394 [job-15598025] INFO JobContainer - 
    Task start time : 2016-12-17 23:43:34
    Task end time : 2016-12-17 23:43:46
    Total task time : 11s
    Average data per task : 31.36 KB/s-
    Write speed : 1,668 rec/s
    Read records : 16,689
    Failed read-write attempts : 0
    ```

5.  Run an SQL statement to view the data synchronization results.


