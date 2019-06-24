# Data protection of projects {#concept_hdk_5g1_wdb .concept}

## Background and motivation {#section_a4y_jh1_wdb .section}

Some companies \(including financial institutions, military enterprises and so on\) are extremely sensitive to data security. Hence, to secure the data, additional security measures are taken, that include not allowing employees to carry USB storage devices or personal hard disks to work; or most of the times the USB ports are disabled. Employees are not allowed to work from home. All these measures are taken to secure the sensitive data.

As a MaxCompute Project Space Administrator, do you have similar security requirements, where users are not allowed to move data out of the project space?

For example, the owner of Project Space prj1 may encounter a situation that prj1's user Alice will transfer the data to prj2, only because she has access to prj2.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12100/156136878250078_en-US.png)

More specifically, assume that Alice has been granted access to myprj, which is the Select permission for table1, and then she is also granted create table permission by the administrator of prj2.

By these permissions, Alice is able to transfer the data to prj2 in any of the following ways:

-   Submit SQL:

    ``` {#codeblock_i62_sa9_35p}
    create table prj2.table2 as select * from myprj.table1;
    ```

-   Write MapReduce to read myprj.table1 and write to the prj2.table2.

If the data in your project space is sensitive, you will be restricted to share data out of your project. MaxCompute can resolve issues pertaining to data protection and the aforementioned operations as well.

## Data protection feature {#section_csy_d31_wdb .section}

MaxCompute provides a project space protection feature that helps to resolve issues mentioned earlier. As a user, set the project as follows:

``` {#codeblock_a45_4fm_msu}
    set projectProtection=true
      -- Set project protection rule: data can only flow and cannot flow out
```

When project protection is set up, the data flow in your project space is controlled , "Data can only flow and cannot flow out ". That is, both of these actions will fail because they are against the project protection rule.

By default, ProjectProtection cannot be set and its value is false.

Also, users authorized to access multiple projects can freely use cross-project data access operations to share or transfer project data. If users are highly sensitive to project data security, the administrator must define a ProjectProtection feature likewise.

## Data outflow method after enabling data protection {#section_v1d_h31_wdb .section}

After setting ProjectProtection in the user's project, the user may soon make requests such as Alice applies to the user for exporting the data of a table out of the user’s project.

Moreover, user review confirms that this table does not contain sensitive data. In order not to affect Alice's normal business requirements, MaxCompute provides two data export methods to the user after setting ProjectProtection.

-   **Set TrustedProject** 

    In case, the current project space is protected, and if you set the target space for the data inflows to the trustedproject for the current space. Then, the data flow to the target project space will not be considered a violation of the project protection rule. If multiple project spaces are set to trustedproject between two and one another, so these project spaces form a trustedproject.

    Group; the data can flow within the project group, but restricted to be shared out of the project group.

    Use the following command to manage the TrustedProject:

    ``` {#codeblock_brx_wq9_wpr}
        list trustedprojects;
          -- View All trustedprojects in the current project
        add trustedproject <projectname>;
          -- Add a trustdproject to the current project
        remove trustedproject <projectname>;
          -- Remove a trustdproject from the current project
    						
    ```

-   **Resource sharing and data protection** 

    In MaxCompute, the [package-based resource sharing](reseller.en-US/Management/Configure security features/Resource share across project space/Resource sharing across projects based on package.md#) feature and the project protection data protection feature are orthogonal, but they are similar to each other in terms of functions.

    MaxCompute rules **give priority to resource sharing over data protection**. Therefore, if a data object allows access by users from other projects through resource sharing, the ProjectProtection rules will not apply to this data object.


## Best practices {#section_bhv_n31_wdb .section}

To prevent data outflow from the project, after setting `ProjectProtection=true`, check the following settings:

-   Make sure the trustedproject is not added. If set, you must assess possible risks;
-   Make sure that package data is not used for sharing. If set, make sure that no sensitive data exists in the package.

