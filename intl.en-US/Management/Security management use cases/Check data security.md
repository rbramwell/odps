# Check data security {#concept_hfc_cb3_4gb .concept}

This topic describes how to check the security of your data and the adjustments that you can make for better data security.

## Background information {#section_tpj_gb3_4gb .section}

Often when a project is initially created, its users and permissions may be loosely managed so to expedite the project progress. However, as the project matures, data security becomes an increasingly important aspect of the management of the project. To ensure better data security, we recommend that you check the security of your data and thereafter formulate a data security plan accordingly.

## Methods {#section_mw4_gb3_4gb .section}

1.  Calculate the number of accounts in your DataWorks projects and in MaxCompute projects. Also, make sure that each member or user has only one RAM user account so that the operations performed by each member or user can be tracked and managed more easily.
2.  Calculate the number of accounts that have been discarded and the permissions of these accounts.

    If a RAM user account has a role in a MaxCompute or DataWorks project, the account must be unbound from the role and then deleted from its workspace. If you do not do so, the account is displayed as p4\_xxxxxxxxxxxxxxxxxxxx, which means that the account cannot be removed from the workspace \(even though the workspace stills runs properly\).

    If the RAM user account of a member or user changes due to role changes, the account and its permissions must be recycled. We recommend that, after you survey account usage and notify the involved users, you delete or recycle short-term accounts and the accounts that remain inactive for an extended period of time.

3.  Survey and analyze the data retrieval and computing tasks \(most of which are SQL tasks\) that are submitted by RAM user accounts within the last three months. Specifically, identify which accounts submit the most tasks and analyze the tasks submitted by specific accounts.

    For example, the account owned by a member occupies a position in an algorithm development project, and this member executes more SQL tasks for querying and writing tables than it executes algorithm tasks and MapReduce tasks. Based on this fact, the system preferentially calls SQL to process data for this member.

    In another example, an account submits a large number of tasks. After a thorough survey and analysis, the user who owns this account is found to be designing an application with the Software Development Kit \(SDK\). Multiple users can use this user's Access Key \(AK\) to query data by using this application. However, such behavior is forbidden.

4.  Calculate the number of tasks for downloading data from each project, and plan the projects from which data can be downloaded.

## Adjustments {#section_ywq_cc3_4gb .section}

-   **Allocate accounts properly**

    Each member or user can have only one RAM user account, which is properly allocated. For example, the account is allocated based on service groups such as management, data integration, data model, algorithm, analysis, O&M, and security groups.

    Members and users are granted data access permissions according to their groups and roles, and their accounts cannot be shared. This is to avoid data security risks that may be incurred by improperly managed permissions.


-   **Manage the flow of data** 

    The permissions of members or users to export data from projects must be overseen and managed. For example, you can restrict the flow of data to only specified projects or locations. We recommend that you restrict the unlimited flow of data among projects because it may interrupt the Alibaba Cloud data architecture and cause data leakage.

-   **Limit data exporting**

    Roles must be divided and bound to service groups properly, so that only users in specified groups can export data as files. Data is no longer in your control once it is exported as files from MaxCompute.


