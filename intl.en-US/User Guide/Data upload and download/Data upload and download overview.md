# Data upload and download overview {#concept_am2_p3f_vdb .concept}

This topic provides an overview of the upload and download processes of MaxCompute system data, including service connection information, SDKs, tools, and how to migrate data to the cloud.

DataHub provides a real-time data tunnel, and Tunnel provides a batch data tunnel. Both of these tunnels can access MaxCompute and provide their own SDKs and derivative data upload and download tools. Specifically, the tools include DataWorks, DTS, OGG plugin, Sqoop, Flume plugin, LogStash plugin, Fluentd plugin, Kettle plugin, and MaxCompute client.

The underlying data tunnels used by these tools include:

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

A wide range of data upload and download tools are applicable to most cloud data migration scenarios. The subsequent topics introduce the tools,[Hadoop data migration](../../../../reseller.en-US/Best Practices/Data migration/Migrate data from Hadoop to MaxCompute.md#), database data synchronization, log collection, and other cloud migration scenarios. We recommend that you refer to the relevant topics describing these scenarios.

**Note:** For information about how to synchronize data in offline mode, we recommend that you read[Data integration overview](../../../../reseller.en-US/User Guide/Data integration/Data integration introduction/Data integration overview.md#).

## Limits {#section_ih3_e3v_sp8 .section}

**Limits to uploading data while using Tunnel** 

-   Your upload speed is dependent on your network bandwidth and server performance.
-   The number of retransmission attempts has an upper limit. When this upper limit is exceeded, the system still continues to upload the next data block. However, your data may be lost as a result. Therefore, we recommend that you run the `select count(*)` command after the data upload is completed to check whether any data is lost.
-   Tunnel commands cannot be used to upload or download the data types ARRAY, MAP, and STRUCT.
-   By default, each project supports up to 2,000 concurrent Tunnel connections.
-   On the server, the lifecycle for the session of each tunnel spans 24 hours. Each session can be shared among processes and threads on the server. During these sessions, you must ensure that each BlockId is unique.
-   When data is written concurrently, MaxCompute guarantees the concurrent sessions according to the Atomicity, Consistency, Isolation, Durability \(ACID\) principle. For more information, see [ACID semantics](../../../../reseller.en-US/.md#).

**Limits to uploading data while using DataHub** 

-   The size of each field cannot exceed its upper limit. For information about the limits of each field, see [Data types](reseller.en-US/User Guide/Data types.md#).

    **Note:** The length of a STRING-type field cannot exceed 8 MB.

-   Multiple data entries are packetized into one package before they are uploaded.

**Limits of TableTunnel SDK interfaces** 

-   The value of BlockId must be greater than or equal to 0 and less than 20000. The size of data to be uploaded in a block cannot exceed 100 GB.
-   The lifecycle of a session is 24 hours. If your session times out due to large volume data uploads, you must split your data into multiple sessions.
-   The lifecycle of an HTTP request of the RecordWriter class is 120s. If no data flows over an HTTP connection within 120 seconds, the server closes the connection.

