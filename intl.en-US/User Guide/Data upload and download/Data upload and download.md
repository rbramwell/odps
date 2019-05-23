# Data upload and download {#concept_am2_p3f_vdb .concept}

This topic provides a brief introduction about the upload and download process of the MaxCompute system data, including service connection, SDKs, tools, and cloud data migration.

The DataHub and Tunnel offers the real-time data tunnel and the batch data tunnel respectively to access the MaxCompute system.

Both DataHub and Tunnel provide their own SDKs. The SDKs and derivative data upload and download tools can suffice your data upload and download requirements in various scenarios.

Data upload and download tools include: DataWorks, DTS, OGG plugin, Sqoop, Flume plugin, Logstash plugin, Fluentd plugin, Kettle plugin, MaxCompute console.

Underlying data tunnels used by these tools include:

-   DataHub tools
    -   OGG
    -   Flume
    -   LogStash
    -   Fluentd
-   Tunnel tools
    -   DataWorks
    -   DTS
    -   Sqoop
    -   Kettle
    -   MaxCompute client

A wide range of data upload and download tools are applicable to most of the cloud data migration scenarios. The subsequent topics introduce the tools, [Hadoop data migration](../../../../reseller.en-US/Best Practices/Data migration/Migrate data from Hadoop to MaxCompute.md#), database data synchronization, log collection, and other cloud migration scenarios. We recommend that you refer to these topics when you select the technical solutions.

**Note:** For information about how to synchronize data in offline mode, we recommend that you see [Data integration overview](../../../../reseller.en-US/User Guide/Data integration/Data integration introduction/Data integration overview.md#).

## Limits {#section_53g_php_d2w .section}

**Limits on uploading data by using Tunnel** 

-   Your upload speed is dependent on your network bandwidth and server performance.
-   The number of retransmission attempts has an upper limit. When this upper limit is exceeded, the system still continues to upload the next data block. However, your data may be lost as a result. Therefore, we recommend that you run the `select count(*)` command after the data upload is completed to check whether any data is lost.
-   Tunnel commands cannot be used to upload or download the data types ARRAY, MAP, and STRUCT.
-   On the server, the lifecycle for the session of each tunnel spans 24 hours. Each session can be shared among processes and threads on the server. During these sessions, you must ensure that each BlockId is unique.

 BlockId

**Limits on uploading data by using DataHub** 

-   The size of each field cannot exceed its upper limit. For information about the limits of each field, see [Data types](reseller.en-US/User Guide/Definitions/Data types.md#).

    **Note:** The length of a STRING-type field cannot exceed 8 MB.

-   Multiple data entries are packetized into one package before they are uploaded.

**Limits on TableTunnel SDK interfaces** 

-   The value of BlockId must be greater than or equal to 0 and less than 20000. The size of data to be uploaded in a block cannot exceed 100 GB.
-   The lifecycle of a session is 24 hours. If your session times out due to large volume data uploads, you must split your data into multiple sessions.
-   The lifecycle of an HTTP request of the RecordWriter class is 120s. If no data flows over an HTTP connection within 120 seconds, the server closes the connection.

