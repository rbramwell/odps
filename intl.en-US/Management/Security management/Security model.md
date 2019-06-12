# Security model {#concept_bcx_q25_ngb .concept}

This topic describes the security model of MaxCompute and that of DataWorks. The security model of MaxCompute can be used by MaxCompute project owners and security administrators for better overall O&M and regular security operations. To ensure better data security, we recommend that you read about the security model before you configure any security functions on Alibaba Cloud.

A security model can be configured for MaxC ompute and [DataWorks](../../../../reseller.en-US/User Guide/Project management/Permission list.md#). When you interwork MaxCompute with ​​​​DataWorks but the security model of DataWorks does not meet your service security requirements, you need to use the security models of both MaxCompute and DataWorks combined together.

## MaxCompute security model {#section_scz_jf5_ngb .section}

**Benefits**

MaxCompute supports multi-tenant data security, which has the following benefits:

-   **[User authentication](reseller.en-US/Management/Configure security features/Manage users and permissions/Authenticate users.md#)**

    MaxCompute supports two account systems: the Alibaba Cloud account system and RAM user system. Note that MaxCompute recognizes RAM users but cannot recognize RAM permissions. That is, you can add RAM users under your Alibaba Cloud account to a MaxCompute project. However, MaxCompute does not consider the RAM permission definitions when it verifies the permissions of RAM users.

-   **[User management](reseller.en-US/Management/Configure security features/Manage users and permissions/Manage users.md#)**

    User management operations such as adding and removing users and granting permissions to users are supported for MaxCompute projects. You can manage permissions by using roles. For each project, an **admin** role is provided automatically. Next, you can grant permissions by using access control lists \(ACLs\) or by setting policies.

    ACLs are similar to the GRANT and REVOKE statements defined in SQL-92. You can use simple statements to grant or revoke permissions for objects in your workspace. An example is as follows:

    ```
    grant actions on object to subject;
    ```

-   **[LabelSecurity](reseller.en-US/Management/Configure security features/Column-level access control.md#)**

    LabelSecurity is a workspace-level mandatory access control \(MAC\) policy that enables workspace administrators to control user access to column-level sensitive data more flexibly.

-   **[Resource sharing across projects based on package](reseller.en-US/Management/Configure security features/Resource share across project space/Resource sharing across projects based on package.md#)**

    You can share data and resources, such as tables and functions, among workspaces by using packages. For these operations, you only need to manage the users in your project.

-   **[Data protection of projects](reseller.en-US/Management/Configure security features/Data protection of projects.md#)**

    Multi-tenant data security meets customer requirements on **not allowing user data to be transmitted outside workspaces**.


**Permissions, roles, and labels**

The security system provided by MaxCompute includes a variety of policies. Permissions are granted by the application of different policies, and help maintain fine-grained authorization. The following describes an example of how to grant the permission on an L4 table to a user to illustrate how permissions are granted by the use of policies:

1.  If no permissions have been granted to the user and the user does not belong to the project, add the user to the project. The user does not have any permissions before they are added to the project.
2.  Grant operation permissions to the user. For details, see [Authorization](reseller.en-US/Management/Configure security features/Manage users and permissions/Authorize users.md#).
    1.  Grant a specific operation permission to the user.
    2.  Grant an ACL to a role and then to the user. If a resource does not have a label, the user has obtained the permission on the resource.
3.  If the user manages resources that have [labels](reseller.en-US/Management/Configure security features/Column-level access control.md#), such as datasheets and packages with datasheets, grant label permissions to the user. Four types of label permissions are provided:
    1.  Permissions on fields in a datasheet
    2.  Permissions on a datasheet \(This type of permission is not supported currently.\)
    3.  Permissions on a package
    4.  Permissions on a user \(Label permissions cannot be granted to a role.\)

The following figure shows how permissions are granted by means of fine-grained authorization and access control.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/116981/156032709937987_en-US.png)

**DataProtection and packages**

DataProtection prevents data from leaking from a project. After DataProtection is enabled, data can be exchanged only between projects that are in the same trusted project group. If two projects are not in the same trusted project group, you need to grant permissions on resources in one project to users in the other project by using a package. For more information, see [Data protection of projects](reseller.en-US/Management/Configure security features/Data protection of projects.md#).

You can group some resources, such as commonly used tables and user-defined functions \(UDFs\), into a package, and then grant the permissions on this package to another project.

In some scenarios, ProjectProtection allows you to configure exception policies specific to application IP addresses and Alibaba Cloud accounts, so that data can be exchanged if needed.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/116981/156032709937993_en-US.png)

## DataWorks security model {#section_jpx_gxz_ngb .section}

DataWorks supports the access of multiple users to shared data sources to help develop data analytics applications. Its security model ensures the following requirements:

-   Isolation of data among organizations.
-   Security of [data development](../../../../reseller.en-US/User Guide/Data development/Business flow/Business flow.md#) during extract, transform, and load \(ETL\) processes. Specifically, it helps limit changes to production tasks, manage which members can edit and debug code, and manage which members can publish production tasks.
-   Permissions can be granted on MaxCompute resources \(such as tables, functions, and instances\) even though MaxCompute provides its own security model that does not include the permissions for such processes as ETL.

Authentication and interoperation with [RAM](../../../../reseller.en-US/Preparation/RAM User Operations.md#) is supported. Specially, you can use your Alibaba Cloud account to create and activate a DataWorks project, and then authorize RAM users under your Alibaba Cloud account the permissions to operate resources in DataWorks.

Using the same account to create all your projects could comprise an organization. To avoid doing so, you can configure dependencies among tasks from different projects. The data of various tasks from projects created by using different accounts is isolated.

To ensure better security, DataWorks distinguishes between [**development projects** and **production projects**](../../../../reseller.en-US/Product Introduction/Simple mode and standard mode.md#) by services to isolate task development and debug from stable production. You can use roles to specify which members can develop and debug tasks and which members can operate and maintain production tasks.

For permissions on MaxCompute resources, while a MaxCompute project is created, roles are created in the project based on roles in DataWorks and permissions are granted to these roles in the project.

