# MaxCompute permissions and DataWorks permissions {#concept_f4n_mc1_4gb .concept}

This topic describes how permissions can be authorized when MaxCompute interoperates with DataWorks and the limitations of using the permissions of only one service. When you use the security model of MaxCompute to control permissions, project members can perform authorized operations on any interfaces in DataWorks. However, when you use DataWorks to assign roles to users, the permissions of project members on MaxCompute resources are more limited.

## Project relationship {#section_hcc_qc1_4gb .section}

If you log on to the [console](../../../../reseller.en-US/User Guide/Workbench/Overview.md#) from the MaxCompute or DataWorks official website, you can create one of the following two types of projects:

-   **Projects in simple mode**: In simple mode, a DataWorks workspace is associated with a MaxCompute project. A number of roles are created in the MaxCompute project. For details about the role permissions, see **Member roles and permissions** in this topic.
-   **Projects in standard mode**: In standard mode, a DataWorks workspace is associated with a MaxCompute development project and a MaxCompute production project. A number of roles are created in each MaxCompute project. For details about the role permissions, see **Member roles and permissions** in this topic.

## Account authentication {#section_ur4_sc1_4gb .section}

In a DataWorks project, an Alibaba Cloud account must be the owner of the project. That is, a RAM user account cannot be the owner. In a MaxCompute project, an Alibaba Cloud account can be the owner or a user. When you add members by using the project member management system of DataWorks, you can add only the RAM users under your Alibaba Cloud account. In MaxCompute, however, you can add other Alibaba Cloud accounts by running the `add user xxx;` command on the command line interface \(CLI\).

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/118689/156134341038006_en-US.png)

## Member roles and permissions {#section_bvy_gf1_4gb .section}

Project members require permissions on MaxCompute resources during extract, transform, and load \(ETL\) operations. Therefore, the roles for DataWorks projects are also created for MaxCompute projects. For more information, see [User management](../../../../reseller.en-US/User Guide/Project management/User management.md#). In addition to the **project owner** role, the **admin** role is also provided by MaxCompute. The following table describes the interoperation between MaxCompute role permissions and DataWorks role permissions.

|MaxCompute role|MaxCompute permission|DataWorks role|DataWorks permission|
|---------------|---------------------|--------------|--------------------|
|project owner|This role has permissions to operate on all MaxCompute projects.|None|None|
|admin| When you create a project, the system creates an **admin** role for it and grant the following permissions to the role: accessing all objects in the project, managing users or roles, and granting permissions to users or roles.

 Unlike a **project owner**, an **admin** role cannot grant **admin** role permissions to users, set security policies for workspaces, or change the authentication models of workspaces. The permissions of an **admin** role cannot be changed.

 The **project owner** role can assign an **admin** role to a user so that the user is authorized with security management.

 |None|None|
|role\_project\_admin|This role has all permissions on projects, tables, functions, resources, instances, jobs, and packages.|Administrator|This role is the administrator of a workspace. It can manage the basic properties, data sources, compute engine configurations, and project members in the workspace. It also can assign administrator, development, O&M, deploy, and visitor roles to project members.|
|role\_project\_dev|This role has permissions to operate on projects, functions, resources, instances, jobs, packages, and tables.|Development|A user with this role can create workflows, script files, resources, and user-defined functions \(UDFs\), create or delete tables, and create packages. However, this role does not have the permission to publish.|
|role\_project\_pe|This role has permissions to operate on projects, functions, resources, instances, and jobs. It also has read permissions for packages and also read and describe permissions for tables.|O&M|A user with this role has publish and online O&M permissions, which are granted by the project administrator. This role does not have the permission to develop data.|
|role\_project\_deploy|This role does not have any permissions by default.|Deploy|This role is similar to the O&M role, except that a user with the deploy role does not have the online O&M permission.|
|role\_project\_guest|This role does not have any permissions by default.|Visitor|A user with this role can only view data, but cannot edit workflows or code.|
|role\_Project\_security|This role does not have any permissions by default.|Security Administrator|This role is only used to configure sensitivity rules and audit data risks in Data Security Guard.|

**Note:** According to the preceding table, the mapping between DataWorks roles and MaxCompute permissions is fixed. After a user is assigned a DataWorks role and obtains the permissions of the MaxCompute role that is associated with this DataWorks role, if you assign other MaxCompute permissions to the user on the CLI, the user's permissions in MaxCompute become inconsistent with those in DataWorks.

## Users and permissions {#section_pbf_wpg_4gb .section}

After a DataWorks workspace is associated with one MaxCompute project, you can specify whether members of another DataWorks workspace have permissions to operate on this MaxCompute project. Specifically, you can choose **Workspace Manage** from the main menu. On the page that is displayed, you can then choose Workspace Management from the left pane. Last, on the Settings page that is displayed, set the AccessKey ID parameter.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/118689/156134341038072_en-US.png)

AccessKey ID has two values: Personal Account and Compute Engine Designated Account. The following figure shows the interoperation between users and permissions.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/118689/156134341138076_en-US.png)

In standard mode, a DataWorks workspace is associated with a MaxCompute development project and a MaxCompute production project. **Members of other DataWorks workspaces can be granted the permissions of the roles assigned to this MaxCompute development project. However, they cannot be granted the permissions of the roles assigned to this MaxCompute production project.** To execute a MaxCompute task, you need to publish it to the production project, and then submit it to MaxCompute as the owner.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/118689/156134341138077_en-US.png)

