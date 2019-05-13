# Simple download {#concept_jzt_3ng_vdb .concept}

This topic describes how to use MaxCompute Java SDK to download data.

## Download data by using the DownloadSession interface of TableTunnel {#section_5ri_tji_vad .section}

```language-java
import java.io.IOException;
 import java.util.Date;
 import com.aliyun.odps.Column;
 import com.aliyun.odps.Odps;
 import com.aliyun.odps.PartitionSpec;
 import com.aliyun.odps.TableSchema;
 import com.aliyun.odps.account.Account;
 import com.aliyun.odps.account.AliyunAccount;
 import com.aliyun.odps.data.Record;
 import com.aliyun.odps.data.RecordReader;
 import com.aliyun.odps.tunnel.TableTunnel;
 import com.aliyun.odps.tunnel.TableTunnel.DownloadSession;
 import com.aliyun.odps.tunnel.TunnelException;
 public class DownloadSample {
     private static String accessId = "<your access id>";
     private static String accessKey = "<your access Key>";
     private static String odpsUrl = "http://service.odps.aliyun.com/api";
     private static String tunnelUrl = "http://dt.cn-shanghai.maxcompute.aliyun-inc.com";
     //tunnelUrl specifies the tunnel URL. This parameter is mandatory when you download data over your intranet. If this parameter is set to null, your data is downloaded via the Internet.
     private static String project = "<your project>";
     private static String table = "<your table name>";
     private static String partition = "<your partition spec>";
     public static void main(String args[]) {
         Account account = new AliyunAccount(accessId, accessKey);
         Odps odps = new Odps(account);
         odps.setEndpoint(odpsUrl);
         odps.setDefaultProject(project);
         TableTunnel tunnel = new TableTunnel(odps);
         tunnel.setEndpoint(tunnelUrl);//Set tunnelUrl.
         PartitionSpec partitionSpec = new PartitionSpec(partition);
           try {
                  DownloadSession downloadSession = tunnel.createDownloadSession(project, table,
                                      partitionSpec);
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
         private static void consumeRecord(Record record, TableSchema schema) {
                 for (int i = 0; i < schema.getColumns().size(); i++) {
                         Column column = schema.getColumn(i);
                         String colValue = null;
                         switch (column.getType()) {
                         case BIGINT: {
                                 Long v = record.getBigint(i);
                                 colValue = v == null ? null : v.toString();
                                 break;
                         }
                         case BOOLEAN: {
                                 Boolean v = record.getBoolean(i);
                                 colValue = v == null ? null : v.toString();
                                 break;
                         }
                         case DATETIME: {
                                 Date v = record.getDatetime(i);
                                 colValue = v == null ? null : v.toString();
                                 break;
                         }
                         case DOUBLE: {
                                 Double v = record.getDouble(i);
                                 colValue = v == null ? null : v.toString();
                                 break;
                         }
                         case STRING: {
                                 String v = record.getString(i);
                                 colValue = v == null ? null : v.toString();
                                 break;
                         }
                         default:
                                 throw new RuntimeException("Unknown column type: "
                                                 + column.getType());
                         }
                         System.out.print(colValue == null ? "null" : colValue);
                         if (i != schema.getColumns().size())
                                 System.out.print("\t");
                 }
                 System.out.println();
         }
 }
```

**Note:** 

-   In the preceding command, a tunnel endpoint on the classic network in the China East 2 \(Shanghai\) region is used as an example. For more information about how to configure tunnel endpoints in the other regions, see [Access domains and data centers](../../../../reseller.en-US/Prepare/Configure Endpoint.md#).
-   To make the test easier, we use System.out.printl to output data. You can choose to output data as a file in TXT format.

## Download data by using the DownloadSession interface of InstanceTunnel {#section_al5_lg7_a3a .section}

```language-java
Odps odps = OdpsUtils.newDefaultOdps(); //Initialize Open Data Processing Service (ODPS) objects.
    Instance i = SQLTask.run(odps, "select * from wc_in;");
    i.waitForSuccess();

    //Create an instance tunnel.
    InstanceTunnel tunnel = new InstanceTunnel(odps);
    //Create a download session based on the specified instance ID.
    InstanceTunnel.DownloadSession session = tunnel.createDownloadSession(odps.getDefaultProject(), i.getId());

    long count = session.getRecordCount();
     //Specify the number of records that will be presented.
    System.out.println(count);

    //Obtain data by using the same method as you do with TableTunnel.
    TunnelRecordReader reader = session.openRecordReader(0, count);
    Record record;
    while ((record = reader.read()) != null) {
      for (int col = 0; col < session.getSchema().getColumns().size(); ++col) {
        //Specify that all the fields in the wc_in table are strings and will be directly printed.
        System.out.println(record.get(col));
      }
    }
    reader.close();
```

## Download data by using SQLTask.getResultSet\(\) {#section_ypz_rch_t1s .section}

```language-java
Odps odps = OdpsUtils.newDefaultOdps(); //Initialize ODPS objects.
    Instance i = SQLTask.run(odps, "select * from wc_in;");
    i.waitForSuccess();

    //Obtain the result iterator based on the specified instance object.
    ResultSet rs = SQLTask.getResultSet(i);
    for (Record r : rs) {
      //Specify the number of records that will be presented.
      System.out.println(rs.getRecordCount());

      for (int col = 0; col < rs.getTableSchema().getColumns().size(); ++col) {
        //Specify that all the fields in the wc_in table are strings and will be directly printed.
        System.out.println(r.get(col));
      }
    }
```

