# DownloadSession {#concept_yvg_zkg_vdb .concept}

**This DownloadSession interface is defined as follows:** 

``` {#codeblock_bch_ova_ce8}
public class DownloadSession {
        DownloadSession(Configuration conf, String projectName, String tableName,
            String partitionSpec) throws TunnelException
        DownloadSession(Configuration conf, String projectName, String tableName,
            String partitionSpec, String downloadId) throws TunnelException
        public String getId()
        public long getRecordCount()
        public TableSchema getSchema()
        public TableTunnel.DownloadStatus getStatus()
        public RecordReader openRecordReader(long start, long count)
        public RecordReader openRecordReader(long start, long count, boolean compress)
    }
```

**Parameters:** 

-   **Life cycle**: Begins with the creation of the Download instance and ends with the completion of a download process.
-   **Create Download instance**: An instance can be created either by Calling the Constructor or by using the TableTunnel.
    -   Request mode: Synchronous.
    -   The server creates a session for this download instance and a unique DownloadId is generated. Obtain this ID using the getId on the client.
    -   This operation incurs high costs. The server creates an index for the data files. Large files generally take longer time to download.
    -   Simultaneously, the server returns the total number of Records and starts multiple concurrent downloads based on this value.
-   **Download data**:
    -   Request mode: Asynchronous.
    -   Call the openRecordReader method to generate a RecordReader instance. “start” identifies the start position of downloading this record, which cannot be less than zero. “count” specifies the number of records for this download which must be greater than zero.
-   **View download**:
    -   Request mode: Synchronous.
    -   Call getStatus to obtain the current download status.
-   Following is the list of 4 states:
    -   UNKNOWN: The initial value when the server creates a session.
    -   NORMAL: The download object is successfully created.
    -   CLOSED: The download is now complete.
    -   EXPIRED: The download session is timed out.

