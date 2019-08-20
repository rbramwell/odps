# Grant packages {#concept_cth_bb3_4gb .concept}

This topic describes how to grant packages to service analysis personnel, so that these personnel can be granted the corresponding permissions to operate on tables of multiple production projects all at once.

## Scenario {#section_tpj_gb3_4gb .section}

Service analysis personnel require to view production tables, but often may not have the corresponding permissions. In such scenarios, you can create packages for multiple projects separately and add the tables that can allow service analysis personnel to view the packages. Specifically, you can create an independent analysis project. Then, install the packages in the analysis project, and grant the packages to service analysis personnel. This method can reduce the cost of management because service analysis personnel do not need to be added to all production projects. Service analysis personnel can view only the tables specified in the packages that are installed in the analysis project.

## Procedure {#section_mw4_gb3_4gb .section}

1.  Create packages in production projects.

    ``` {#codeblock_5b3_w2g_izm}
    CREATEPACKAGE PACKAGE_NAME;
    For example:
    CREATEPACKAGE prj_prod2bi; 
    ```

2.  Add resources to be shared to the packages in the production projects.

    ``` {#codeblock_ekd_923_gwi}
    ADD table TO PACKAGE [Package name]; 
    For example:
    ADD table adl_test_table TO PACKAGE prj_prod2bi;
    ```

3.  Create an independent analysis project.

    ``` {#codeblock_dv6_2i4_wjg}
    ALLOW PROJECT [Project in which packages can be installed] TO INSTALL PACKAGE [Package name];
    For example:
    ALLOW PRJ_BI TO INSTALL PACKAGE prj_prod2bi;
    ```

4.  Install the packages in the analysis project.

    ``` {#codeblock_gp8_ww9_z7i}
    INSTALLPACKAGE [Application name].[Package name]; 
    For example:
    INSTALLPACKAGE prj_prod.prj_prod2bi;
    ```

5.  Grant the packages to specified users.

    ``` {#codeblock_jcx_df1_k5c}
    Grant the package to a user:
    GRANTreadonpackage prj_prod2bi TOUSER [Cloud account];
    Grant the package to a role:
    GRANTreadonpackage prj_prod2bi TOROLE [Role name];
    ```


