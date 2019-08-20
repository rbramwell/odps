# Package赋权 {#concept_cth_bb3_4gb .concept}

业务分析人员需要查看生产表，但是不允许查看生产任务代码。这种情况下，我们可以通Package赋权，将多个生产项目的部分表开放给业务分析人员。

## 场景分析 {#section_tpj_gb3_4gb .section}

业务分析人员需要查看生产表但是又不能查看生产任务，我们可以通过单独创建一个分析项目来实现。首先，在多个生产项目创建package，把需要开放的表加到package中。然后，在分析项目中安装package并授权给分析人员。这样可以减少成员管理成本，无需在所有生产项目中增加分析人员，同时保证这些分析人员只能在分析项目中查看package中的表。

## 操作步骤 {#section_mw4_gb3_4gb .section}

1.  生产项目中创建Package

    ``` {#codeblock_s0b_68d_24j}
    CREATE PACKAGE [pkgname]
    如：
    CREATE PACKAGE prj_prod2bi; 
    ```

2.  生产项目中向Package中添加需要分享的资源

    ``` {#codeblock_etm_kdp_3de}
    ADD table [table_name] TO PACKAGE [包名称]; 
    如：
    ADD table adl_test_table TO PACKAGE prj_prod2bi;
    ```

3.  生产项目许可分析项目空间使用Package

    ``` {#codeblock_fqd_pp3_1l9}
    ALLOW PROJECT [允许安装的 project] TO INSTALL PACKAGE [包名称];
    如:
    ALLOW PROJECT PRJ_BI TO INSTALL PACKAGE prj_prod2bi;
    ```

4.  分析项目安装Package

    ``` {#codeblock_vjg_h40_808}
    INSTALL PACKAGE [应用名].[包名称]; 
    如:
    INSTALL PACKAGE prj_prod.prj_prod2bi;
    ```

5.  Package赋权给使用者

    ``` {#codeblock_jie_ogc_bvz}
    赋权给user：
    GRANT read on package prj_prod2bi TOUSER [云账号];
    赋权给role：
    GRANT read on package prj_prod2bi TOROLE [rolename];
    ```


