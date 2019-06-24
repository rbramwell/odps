# Manage users, roles, and permissions {#concept_mw3_2qg_4gb .concept}

This topic describes how to manage users, roles, and permissions for MaxCompute and DataWorks. We recommend that you read this topic before you configure [User management](intl.en-US/Management/Configure security features/Manage users and permissions/Manage users.md#).

## Manage users {#section_ltc_brg_4gb .section}

You can manage users by either adding users or by deleting or locking accounts that do not have owners, remain inactive, or that are owned by employees that have resigned. Furthermore, you can control the permissions of the Project Manager and O&M roles in DataWorks.

**Note:** If you create an account in DataWorks, a default role in MaxCompute is assigned to this account.

|Item|MaxCompute|DataWorks|
|----|----------|---------|
|Role|project owner or admin|Project Manager|
|View details| 1.  Run the list users; command to view the users in the current project.
2.  Run the show grants for <username\>; command to view the permissions of a specified user.

 |Log on to the DataWorks console, and navigate to the management page for a workspace. In the left-side navigation pane, choose **User Management**. Then, view the members and roles in the workspace, and check the validity of the permissions of each member.|
|Grant permissions| Run the add user <username\>; command to add a specified user to the current project.

 MaxCompute project members can be operated only in combination with object, role, and label permissions. You need to check whether MaxCompute project members have object, role, or label permissions and need to delete these permissions when required.

**Note:** MaxCompute project members are not associated with DataWorks. DataWorks allows you only to add Alibaba Cloud accounts and RAM user accounts.

 |Log on to the DataWorks console, and navigate to the management page for a workspace. In the left-side navigation pane, choose **User Management**. Then, add members, and assign roles to the members. -   You can add only a RAM user account under the owner of a workspace to be a member for this workspace.
-   After you add a member and assign a role to this member, this member is granted the default permissions of this role. For more information, see [MaxCompute permissions and DataWorks permissions](intl.en-US/Management/Security management/MaxCompute permissions and DataWorks permissions.md#).

 |
|Roll back settings| Run the remove user <username\>; command to delete a specified user from the current project.

 |Delete the permissions of members or roles from DataWorks. After doing so, the system deletes the corresponding users and roles from MaxCompute.|

## Manage roles {#section_cw5_3rg_4gb .section}

Roles are managed differently in MaxCompute and DataWorks. In MaxCompute, you can manage roles by creating roles, by granting permissions to these roles, and by deleting necessary accounts, resources, or permissions from roles. In DataWorks, you can assign roles by either changing the roles of members or controlling the assignment of the Project Manager and O&M roles.

**Note:** After a MaxCompute project is created, it is assigned with both the default admin role in MaxCompute and another role in DataWorks. For more information, see [MaxCompute permissions and DataWorks permissions](intl.en-US/Management/Security management/MaxCompute permissions and DataWorks permissions.md#).

| |MaxCompute|DataWorks role|
|--|----------|--------------|
|Role|project owner or admin|Project Manager|
|View details| Run the list roles; command to view all the roles in the current project.

 Run the describe role <role\_name\>; command to view the permissions of a specified role.

 Run the show grants for <username\>; command to view the role assigned to a specified user.

 **Note:** The users to whom a specified role is assigned are invisible.

 |Log on to the DataWorks console, and navigate to the management page for a workspace. In the left-side navigation pane, choose **User Management**. Then, view the members to which each role is assigned.|
|Grant permissions| In addition to the default roles provided by MaxCompute, you can define custom role permissions and assign the custom roles to users. The overall process is as follows:

 1.  Run the create role <role\_name\>; command to create a role.
2.  Run the grant actions on object to <role\_name\>; command to grant permissions to a role.
3.  Run the GRANT <roleName\> TO <full\_username\>; command to assign a role to a user.

 Log on to the DataWorks console, and navigate to the management page for a workspace. In the left-side navigation pane, choose **MaxCompute Management**. In the middle pane, choose **Custom User Roles**, Then, define roles, and assign the roles to members.

**Note:** The roles that are created by using commands are not displayed.

 |For DataWorks, the roles that are created by running commands are not displayed. Instead, you need to log on to the DataWorks console, and navigate to the management page for a workspace. Then, in the left-side navigation pane, choose **MaxCompute Management**. In the middle pane, choose **Custom User Roles**, define roles, and assign the roles to members.|
|Roll back settings| 1.  Run the REVOKE <roleName\> FROM <full\_username\>; command to delete a user from a role.
2.  Run the revoke <privList\> on <objType\> <objName\> from role <rolename\>; command to revoke the permissions that are granted to a role.
3.  Run the DROP ROLE <roleName\>; command to delete a role.

 |In DataWorks, roles cannot be deleted. You can only delete a member from a role.|

## Grant permissions by using ACLs {#section_ytj_lrg_4gb .section}

You can revoke unnecessary object permissions, which involve various objects and operation types that must be verified before the permissions on them are revoked.

|Item|Description|
|----|-----------|
|Role|project owner or admin|
|View details| -   Run the show grants for <username\>; command to view the permissions of a specified user.
-   Run the show grants; command to view the permissions of the current user.
-   Run the show acl for <objectName\> \[on type <objectType\>\]; command to view the permissions to operate a specified object.
-   Run the show acl for alipaydw.alipaydw\_for\_alisec\_app on type package; command to view the permissions that are included in a specified package.

 |
|Grant permissions| Run the grant actions on object to subject; command to grant the permissions to operate a specified object.

 Subject, object, and action types are expressed as follows:

 -   Subject types: user full\_username,role role\_name
-   Object types: project project\_name,table schema\_name,instance inst\_name,function func\_name,resource res\_name
-   Action types: action\_item1, action\_item2, ...

 For more information about subjects, objects, and actions, see [Authorization](intl.en-US/Management/Configure security features/Manage users and permissions/Authorize users.md#).

 |
|Roll back settings| Run the revoke actions on object from subject; command to revoke the permissions to operate a specified object.

 |

## Grant permissions by using packages {#section_xq4_nrg_4gb .section}

You can grant the permissions to operate the projects that have [ProjectProtection](intl.en-US/Management/Configure security features/Data protection of projects.md#) enabled but are not in the same trusted project group. The packages for these projects and the permissions included in these packages must be properly planned. No packages can remain inactive.

|Item|Description|
|----|-----------|
|Role|project owner|
|View details| -   View the packages for the current project and the permissions included in these packages.
    -   To view the packages that are created and installed in the project, run the show packages; command.
    -   To view details about the packages, run the describe package <pkgname\>; command.
-   Run the show acl for <project\_name.package\_name\> on type package; command to view which packages are granted to users in the current project.

 |
|Grant permissions|**As a user who creates packages, you can:** -   Run the create package <pkgname\>; command to create a package.
-   Run the add project\_object to package package\_name \[with privileges privileges\]; command to add shared resources to a package.

project\_object is expressed as follows: table table\_name, instance inst\_name, function func\_name, resource res\_name.

-   Run the allow project <prjname\> to install package <pkgname\> \[using label<number\>\]; command to grant the package for one project to another project.

 **As a user who installs packages, you can:** -   Run the install package <pkgname\>; command to install a package.
-   Grant a package to a user or role. When you do so, you are not allowed to specify a label, whereas a user with the project owner or admin role can.
    -   To grant a package to a user, run the grant actions on package <pkgName\> to user <username\>; command.
    -   To grant a package to a role, run the grant actions on package <pkgName\> to role <role\_name\>; command.

For more information about action types, see [Authorization](intl.en-US/Management/Configure security features/Manage users and permissions/Authorize users.md#).


 In most cases, to enable a user to access the resources in a package, you only need to grant the read permission for the package to the user.

 |
|Roll back settings| -   Run the disallow project <prjname\> to install package <pkgname\>; command to revoke the permissions that enable a specified project to operate a package.
-   Run the delete package <pkgname\>; command to delete a package.
-   Run the remove project\_object from package package\_name; command to remove shared resources from a package.

project\_object is expressed as follows: table table\_name, instance inst\_name, function func\_name, resource res\_name.

-   Revoke a package for a user or role:
    -   To revoke a package for a user, run the revoke actions on package <pkgName\> from user <username\>; command.
    -   To revoke a package for a role, run the revoke actions on package <pkgName\> from role <role\_name\>; command.

 |

## Grant permissions by using labels {#section_f5z_qrg_4gb .section}

You can grant labels for fields, tables, and packages, which fall into one to four levels of labels, in MaxCompute.

|Item|Description|
|----|-----------|
|Role|project owner|
|View details| -   Run the SHOW LABEL \[<level\>\] GRANTS \[FOR USER <username\>\]; command to check which sensitive data sets are accessible to a user.
    -   If \[FOR USER <username\>\] is not specified, the sensitive data sets accessible to the current user are displayed.
    -   If <level\> is not specified, labels of all levels that are granted to the specified user are displayed.
    -   If <level\> is specified, only the labels of the specified level that are granted to the specified user are displayed.
-   Run the SHOW LABEL \[<level\>\] GRANTS ON TABLE <tablename\>; command to check which users can access a specified table that contains sensitive data.
-   Run the SHOW LABEL \[<level\>\] GRANTS ON TABLE <tablename\> FOR USER <username\>; command to view all the column-level labels that a user has on a data table.

For more information, see [Column-level access control](intl.en-US/Management/Configure security features/Column-level access control.md#).


 |
|Grant permissions| -   Run the following command to grant a label for a table or field to a user:

GRANT LABEL <number\> ON TABLE <tablename\>\[\(column\_list\)\] TO USER <username\> \[WITH EXP <days\>\];

The default value of days in \[WITH EXP <days\>\] is 180.

For example, if you run the GRANT LABEL 2 ON TABLE t1 TO USER alice WITH EXP 1; command, the user named alice is explicitly granted the permissions to access data, whose sensitivity levels are 2 or lower, in the t1 table. Furthermore, the permissions remain valid for one day.

-   Run the following command to grant a label for a project to a user:

SET LABEL <number\> TO USER <username\>;

-   Run the following command as the creator of a package to grant the permissions for sensitive resources in the package to a user who installs the package:

ALLOW PROJECT <prjName\> TO INSTALL PACKAGE <pkgName\> \[USING LABEL <number\>\];

-   Grant a package to a user or role. When you do so, you are not allowed to specify a label.
    -   To grant a package to a user, run the grant actions on package <pkgName\> to user <username\>; command.
    -   To grant a package to a role, run the grant actions on package <pkgName\> to role <role\_name\>; command.

 |
|Roll back settings| -   Revoke the label for a table or field from a user.

    -   To revoke permissions, run the REVOKE LABEL ON TABLE <tablename\>\[\(column\_list\)\] FROM USER <username\>; command.
    -   To delete expired permissions, run the CLEAR EXPIRED GRANTS; command.
For example, to revoke the permissions that enable the user named alice to access sensitive data in the t1 table, run the REVOKE LABEL ON TABLE t1 FROM USER alice; command.

-   Run the SET LABEL <number\> TO USER <username\>; command to change the level of label that a user has for a project.

The default level of label is 0.

-   Run the following command to change the level of label that enables a user, who installs a package, to access the sensitive resources in the package:

ALLOW PROJECT <prjName\> TO INSTALL PACKAGE <pkgName\> \[USING LABEL <number\>\];

The default level of label is 0.

-   Revoke the permissions of a user or role.
    -   To revoke the permissions of a user, run the revoke actions on package <pkgName\> from user <username\>; command.
    -   To revoke the permissions of a role, run the revoke actions on package <pkgName\> from role <role\_name\>; command.

 |

