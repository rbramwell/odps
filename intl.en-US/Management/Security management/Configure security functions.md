# Configure security functions {#concept_szy_rch_4gb .concept}

This topic describes how to enable, set, and disable specific security functions of MaxCompute and DataWorks for improved security purposes. For more information, see the documents [Security configurations](intl.en-US/Management/Configure security features/Security configurations.md#), [Data protection of projects](intl.en-US/Management/Configure security features/Data protection of projects.md#), and [Column-level access control](intl.en-US/Management/Configure security features/Column-level access control.md#).

## **Enable ProjectProtection** {#section_jcj_hdh_4gb .section}

ProjectProtection helps to limit the transmission of data from workspaces. It does this by prohibiting data from being downloaded in batches to personal computers. We recommend that you enable this function. It is disabled by default. For more information, see [Security configurations](intl.en-US/Management/Configure security features/Security configurations.md#).

|Item|Description|
|----|-----------|
|Role|project owner|
|View the function status| Run the show SecurityConfiguration; command to check whether ProjectProtection is set to true.

 |
|Set the function| **Use one of the following two methods to enable ProjectProtection:**

 -   Log on to the DataWorks console, navigate to **MaxCompute Management**, and enable Protect workspace data in **Basic Settings**.
-   On the command line interface \(CLI\) of MaxCompute, run the SET ProjectProtection=true \[WITH EXCEPTION <policyFile\>\]; command.

 If some Alibaba Cloud accounts or private accounts require the permissions to transmit data from workspaces after ProjectProtection is enabled, you can set exception policies \(namely, enable the whitelist function\).

 **We recommend that you configure exception policies if:** 

-   You want to specify the number of Alibaba Cloud accounts that can transmit data from workspaces or a number of IP addresses from which data can be transmitted.
-   You want to specify the number of tables that private accounts can download.

 **Add trusted projects:**

 If you add project A as a trusted project of project B, data can be exchanged between them.

 -   To view all the trusted projects of the current project, run the list trustedprojects; command.
-   To add a trusted project to the current project, run the add trustedproject <projectname\>; command.
-   To remove a trusted project from the current project, run the remove trustedproject <projectname\>; command.

 **Note:** If project C requires data from project D but it is not a trusted project of project D, you need to grant permissions to project C by using a package. For more information, see [Package](intl.en-US/Management/Configure security features/Resource share across project space/Package usage method.md#)

 |
|Roll back settings| To disable ProjectProtection for the current project, run the SET ProjectProtection=false; command.

 To remove a trusted project from the current project, run the remove trustedproject <projectname\>; command.

 |

## Enable LabelSecurity {#section_mqc_rfh_4gb .section}

LabelSecurity is a type of mandatory access control \(MAC\) for workspaces. It helps workspace owners to manage user access to column-level security-sensitive data more flexibly. This can allow you to keep fields in your tables more secure. We recommend that you enable LabelSecurity, which is disabled by default. For more information, see [Column-level access control](intl.en-US/Management/Configure security features/Column-level access control.md#).

|Item|Description|
|----|-----------|
|Role|project owner|
|View the function status|Run the show SecurityConfiguration; command to check whether LabelSecurity is set to true.|
|Set the function|Run the Set LabelSecurity=true; command to enable LabelSecurity.|
|Roll back settings| Run the Set LabelSecurity=false; command to disable LabelSecurity.

 Before you disable LabelSecurity for a project, check whether any other projects depend on it and whether the labels for tables in this project are also granted to the other projects.

 |

## Set labels for fields {#section_prj_3gh_4gb .section}

We recommend that you set labels for tables, which may be divided into different levels of labels according to data sensitivity in MaxCompute.

|Item|Description|
|----|-----------|
|View the function status| Use one of the following two methods to view the labels for fields in a MaxCompute table:

 -   Run the DESCRIBE <tablename\>; command.
-   Log on to the DataWorks console, navigate to **Data Management**, and view details about fields in the table.

 |
|Set the function| Use one of the following two methods to set labels for fields:

 -   Method 1 \(recommended\)

Log on to the DataWorks console, and navigate to **Data Management**. You can set labels for fields in new and existing tables.

**Note:** The label attributes are visible in Data Management only when LabelSecurity is set to true.

-   Method 2

Run the SET LABEL <number\> TO TABLE tablename\[\(column\_list\)\]; command.

Examples:

    -   To set the label for the t1 table to 1, run the SET LABEL 1 TO TABLE t1; command.
    -   To set the labels for the mobile and addr columns in the t1 table to 2, run the SET LABEL 2 TO TABLE t1\(mobile, addr\); command.
    -   To set the label for the t1 table to 3, run the SET LABEL 3 TO TABLE t1; command. After you do so, the labels for the mobile and addr columns remain 2.
**Note:** After you enable automatic labels by using the command line interface \(CLI\), the labels of fields in Data Management cannot be updated according to the automatically allocated labels. Therefore, we recommend that you set labels for fields by using DataWorks.


 |
|Roll back settings|Return fields to their original labels. **Note:** When you reconfigure labels for fields to make the fields more secure, the original permissions owned by packages, production accounts, and private accounts are no longer valid. To mitigate the impact, you must notify the involved users before the reconfiguration.

 |

## Enable IP address whitelist {#section_j4b_xhh_4gb .section}

We recommend that you enable an IP address whitelist to specify the IP addresses from which users can access a specified project. Each of these IP addresses correspond to pages in the console or SDKs. For more information, see [Console](../../../../intl.en-US/Tools and Downloads/Client.md#) and [SDK](../../../../intl.en-US/SDK Reference /Java SDK/Java SDK.md#).

**Note:** 

-   The whitelist takes effect on all the accounts \(including project owner\) of the project.
-   The whitelist does not take effect on the servers that run DataWorks. If your server runs DataWorks, you can submit MaxCompute tasks by using DataWorks even though the IP address of your server is not included in the whitelist.

|Item|Description|
|----|-----------|
|Role|project owner|
|View the function status| Run the setproject; command on the DataWorks console.

 In the command output, check whether information follows odps.security.ip.whitelist=. If no information follows odps.security.ip.whitelist=, the whitelist is disabled.

 |
|Set the function| **Note:** Before you enable the whitelist, you must add the IP address of your computer to the whitelist. Otherwise, you will not be able to operate the project after the whitelist takes effect.

 Run the following command on the client:

 setproject odps.security.ip.whitelist=xxx.xxx.xxx.xxx,xxx.xxx.x.x/xx,xxx.xxx.xxx.xxx-xxx.xxx.xxx.xxx;

 The whitelist supports IPv6 addresses, and the IP addresses in the whitelist can be expressed in one of the following three ways:

-   IP addresses, for example, 101.132.236.134 and FE80:0202:B3FF:FE1E:8329
-   Subnet masks, for example, 100.116.0.0/16 and FE80:0101:4567:F456:0202:B3FF:1111:1111/126
-   Network segments, for example, 101.132.236.134-101.132.236.144 and FE80:0101:4567:F456:0202:B3FF:FE1E:8330-FE80:0101:4567:F456:0202:B3FF:FE1E:8331

 The whitelist takes effect 5 minutes after you set it.

 If you want to manage permissions at finer levels, you can grant permissions by using policies.

 |
|Rollback settings| Delete the information following setproject odps.security.ip.whitelist=:

 setproject odps.security.ip.whitelist=;

 After you do so, the whitelist is disabled for the project.

 |

## **Disable SELECT result download in DataWorks** {#section_s52_zhh_4gb .section}

Disable SELECT result download in DataWorks. We recommend that you disable SELECT result download. When you analyze data by using DataWorks, the data analysis results are displayed in IDE and can be downloaded. After ProjectProtection is set to true, you only need to have the permissions to read tables in the project for you to be able to select and download data analysis results in Data Analytics in the DataWorks console.

|Item|Description|
|----|-----------|
|Role|Project Manager in DataWorks|
|View the function status|Log on to the DataWorks console, navigate to **Workspace Settings**, and check whether SELECT result download is enabled.|
|Set the function|Log on to the DataWorks console, navigate to **Workspace Settings**, and disable SELECT result download.|
|Roll back settings|Log on to the DataWorks console, navigate to **Workspace Settings**, and enable SELECT result download.|

## Promote security management by using other cloud services {#section_hgl_plh_4gb .section}

You may use other cloud services while using MaxCompute. Therefore, you can promote the security management of MaxCompute by using the other associated cloud services. For example, when you use MaxCompute on the DataWorks console, you need to use RAM user accounts to add members to projects. The following describes how to promote security management on RAM user accounts.

MaxCompute supports two account systems: the Alibaba Cloud account system and the RAM user account system. MaxCompute can identify RAM users but cannot identify their permissions, which enables you to add any RAM users under the project owner account of a project to this project. When MaxCompute authenticates these RAM users, it does not verify their permissions. Therefore, you only need to promote security management on logons of RAM users.

**Set password policies for RAM users**

If you allow RAM users to change their passwords, you need to specify strong password policies such as the password length, whether characters other than letters are required, and the intervals at which RAM users change their passwords.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/119945/156134395438120_en-US.png)

**Set logon address masks for RAM users**

You can set log address masks to specify from which IP addresses RAM users can log on the DataWorks console.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/119945/156134395538121_en-US.png)

**Revoke the permissions that RAM users no longer require**

When some permissions of a RAM user are not longer used because the user's position changes, you need to revoke these permissions promptly.

