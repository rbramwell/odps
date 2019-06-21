# TableTunnel {#concept_exm_f3g_vdb .concept}

This topic provides a definition and describes the processes and limits of TableTunnel, which itself provides an ingress class for you to use the MaxCompute Tunnel service to upload and download tables.

## Definition {#section_p3f_id8_sge .section}

The TableTunnel interface is defined as follows:

``` {#codeblock_dtv_f71_eum .language-java}
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

For more information, see [Java-sdk-doc](https://www.javadoc.io/doc/com.aliyun.odps/odps-sdk-core/0.31.3-public).

Parameter description:

-   lifecycle: the period of time that starts when a TableTunnel instance is created and ends when the service process is completed.
-   UploadSession and DownloadSession: the objects that you can create by using TableTunnel. After creating an UploadSession or DownloadSession object, you can call a session, TableTunnel.UploadSession or TableTunnel.DownloadSession, to upload or download data.
-   Session: the period of time in which a table or partition is uploaded or downloaded. A session consists of one or more HTTP requests to the Tunnel RESTful API actions that you call.
-   TableTunnel.UploadSession: This session offers the same capabilities as the `INSERT INTO` statement. You can create multiple sessions to upload the same table or partition, and these sessions do not affect each another. The data uploaded by each session is stored to a unique directory.
-   RecordWriter: In TableTunnel.UploadSession, each RecordWriter class corresponds to an HTTP request and is uniquely identified by its block ID. The block ID is the name of the file corresponding to the RecordWriter class.
-   blockId: If you use the same block ID to open a RecordWriter class multiple times in the same session, the data uploaded by the RecordWriter class that is the last to call the `close()` function overwrites all previous data. This is helpful in retransmitting data of a block in case that the block data fails to be transmitted.

## Process {#section_o8x_xyi_n93 .section}

1.  The `RecordWriter.write()` function uploads your data as files to a temporary directory.
2.  The `RecordWriter.close()` function moves the files from the temporary directory to the Data directory.
3.  The `session.commit()` function moves each file in the Data directory to the directory where the corresponding table is located, and updates the table metadata accordingly. In this way, data moved into a table by the current task can be visible to the other MaxCompute tasks such as SQL and MapReduce.

## Limits {#section_59l_nph_a9l .section}

-   The number used for the block ID must be greater than or equal to 0 and less than 20000. The size of data to be uploaded in a block cannot exceed 100 GB.
-   A session is uniquely identified by its ID. The lifecycle of a session is 24 hours. If your session times out due to large data uploads, you will need to split your data into multiple sessions to mitigate the chance of session timeouts.
-   The lifecycle of an HTTP request of a RecordWriter class is 120s. If there is no inbound traffic over an HTTP connection within 120 seconds, the server closes the connection.

    **Note:** HTTP provides an 8-KB buffer. When you call the `RecordWriter.write()` function, your data can be saved to the buffer with no inbound traffic over the corresponding HTTP connection. In such a case, you can call the `TunnelRecordWriter.flush()` function to make this data stored in the buffer into inbound traffic over the HTTP connection.

-   When you create RecordWriter classes to write logs to MaxCompute, the `RecordWriter` classes are likely to time out due to unexpected traffic fluctuations. Therefore, we recommend that you:
    -   Do not create a RecordWriter class for each of your data records because each `RecordWriter` class corresponds to a file. If you create a `RecordWriter` class for every data record, a large number of small files are produced, degrading the overall performance of MaxCompute.
    -   Do not create a RecordWriter class to write data in batches until the size of cached code reaches 65 MB.
-   The lifecycle of a RecordReader class is 300s.

