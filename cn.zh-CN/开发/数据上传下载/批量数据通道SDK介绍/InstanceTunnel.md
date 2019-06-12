# InstanceTunnel {#concept_kn3_cxy_3hb .concept}

您可以通过InstanceTunnel下载以SELECT关键字开头的用于获取数据的SQL Instance。

InstanceTunnel接口定义如下，详情请参见[Java-sdk-doc](https://www.javadoc.io/doc/com.aliyun.odps/odps-sdk-core/0.31.3-public)。

``` {#codeblock_8ut_7lj_muu .language-java}
public class InstanceTunnel{
 public DownloadSession createDownloadSession(String projectName, String instanceID);
 public DownloadSession createDownloadSession(String projectName, String instanceID, boolean limitEnabled);
 public DownloadSession getDownloadSession(String projectName, String id);
 }
```

参数说明：

-   projectName：Project名
-   instanceID：Instance ID

InstanceTunnel约束：虽然InstanceTunnel提供了非常方便的获取Instance执行结果的方式，但是为了保护用户数据安全，此功能也受到了权限约束。

-   使用InstanceTunnel下载数据时，若数据条数不超过10000，则只要对该Instance有Read权限的用户即可使用。此行为与使用Restful API获取查询数据的行为一致。
-   使用InstanceTunnel下载数据时，若数据条数超过10000，则需要对Instance SQL查询语句中涉及到的所有源表进行权限检查，用户需要具有这些表的Read权限才可使用。

