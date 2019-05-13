# Simple upload {#concept_ytr_dng_vdb .concept}

```
 import java.io.IOException;
 import java.util.Date;
 import com.aliyun.odps.Column;
 import com.aliyun.odps.Odps;
 import com.aliyun.odps.PartitionSpec;
 import com.aliyun.odps.TableSchema;
 import com.aliyun.odps.account.Account;
 import com.aliyun.odps.account.AliyunAccount;
 import com.aliyun.odps.data.Record;
 import com.aliyun.odps.data.RecordWriter;
 import com.aliyun.odps.tunnel.TableTunnel;
 import com.aliyun.odps.tunnel.TunnelException;
 import com.aliyun.odps.tunnel.TableTunnel.UploadSession;
 public class UploadSample {
         private static String accessId = "<your access id>";
         private static String accessKey = "<your access Key>";
         private static String odpsUrl = "http://service.odps.aliyun.com/api";
         private static String tunnelUrl = "http://dt.cn-shanghai.maxcompute.aliyun-inc.com";
                         //The tunnelURL must be set if you need to connect internal network, otherwise, the system uses public network as default. The example shows the Tunnel Endpoint of classical network in HuaDong 2, for other regions, see Access domain and data centers.
         private static String project = "<your project>";
         private static String table = "<your table name>";
         private static String partition = "<your partition spec>";
         public static void main(String args[]) {
                 Account account = new AliyunAccount(accessId, accessKey);
                 Odps odps = new Odps(account);
                 odps.setEndpoint(odpsUrl);
                 odps.setDefaultProject(project);
                 try {
                         TableTunnel tunnel = new TableTunnel(odps);
                         tunnel.setEndpoint(tunnelUrl);  //set tunnelUrl
                         PartitionSpec partitionSpec = new PartitionSpec(partition);
                         UploadSession uploadSession = tunnel.createUploadSession(project,
                                         table, partitionSpec);
                         System.out.println("Session Status is : "
                                         + uploadSession.getStatus().toString());
                         TableSchema schema = uploadSession.getSchema();
                          // After preparing data, open a Writer to start writing data. The prepared data is written to one block.
                          // When the data written to individual blocks is too small, the system will produce a large number of small files, seriously degrading computing performance. We strongly recommend over 64 MB of data be written each time (up to 100 GB of data can be written to the same block).
                          // You can use the average data volume and record count to estimate the total value. For example: 64MB < Average data size x Record count < 100GB.
                         RecordWriter recordWriter = uploadSession.openRecordWriter(0);
                         Record record = uploadSession.newRecord();
                         for (int i = 0; i < schema.getColumns().size(); i++) {
                                 Column column = schema.getColumn(i);
                                 switch (column.getType()) {
                                 case BIGINT:
                                         record.setBigint(i, 1L);
                                         break;
                                 Case Boolean:
                                         record.setBoolean(i, true);
                                         break;
                                 case DATETIME:
                                         record.setDatetime(i, new Date());
                                         break;
                                 case DOUBLE:
                                         record.setDouble(i, 0.0);
                                         break;
                                 case STRING:
                                         record.setString(i, "sample");
                                         break;
                                 default:
                                         throw new RuntimeException("Unknown column type: "
                                                         + column.getType());
                                 }
                         }
                         for (int i = 0; i < 10; i++) {
                                  // Writes data to the server. Each 8 KB of data written triggers a network transmission.
                                  // If no network transmission occurs for 120 seconds, the server closes the connection. At this time, the Writer becomes unavailable and you must write data again.
                                 recordWriter.write(record);
                         }
                         recordWriter.close();
                         uploadSession.commit(new Long[]{0L});
                         System.out.println("upload success!") ;
                 } catch (TunnelException e) {
                         e.printStackTrace();
                 } catch (IOException e) {
                         e.printStackTrace();
                 }
         }
 }
```

**Constructor:**

PartitionSpec\(String spec\): Uses a string to construct this class of object.

**Parameters**

spec: The partition definition string, such as pt=’1’,ds=’2’.

In this program, the configuration must be as follows:

```
private static String partition = “pt=’XXX’,ds=’XXX’”;
```

**Note:** This paper gives the Tunnel Endpoint of East China 2 classical network. The Tunnel Endpoint settings of other regions can be referred to [Configure Endpoint](../../../../reseller.en-US/Prepare/Configure Endpoint.md#).

