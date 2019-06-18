# InstanceTunnel {#concept_kn3_cxy_3hb .concept}

This topic describes the interface, parameters, and limits of InstanceTunnel. You can use InstanceTunnel to call an SQL instance that starts with the `SELECT` keyword and is used for querying data.

The InstanceTunnel interface is defined as follows:

``` {#codeblock_c6u_k9p_zwm .language-java}
public class InstanceTunnel{
 public DownloadSession createDownloadSession(String projectName, String instanceID);
 public DownloadSession createDownloadSession(String projectName, String instanceID, boolean limitEnabled);
 public DownloadSession getDownloadSession(String projectName, String id);
 }
```

For more information, see [Java-sdk-doc](https://www.javadoc.io/doc/com.aliyun.odps/odps-sdk-core/0.31.3-public).

Parameter description:

-   projectName: the name of the project where the specified instance resides.
-   instanceID: the ID of the specified instance.

Limits:

-   If the number of records does not exceed 10,000, all users who have the permissions to read the specified instance can call this instance. This is the same for when calling a RESTful API to query data.
-   If the number of records exceeds 10,000, only users who have the permissions to read all the source tables from which the specified instance queries data can call this instance.

