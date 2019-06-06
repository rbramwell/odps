# Package的使用方法 {#concept_vj4_rk1_wdb .concept}

本文为您介绍项目空间Package创建者和Package使用者所涉及的操作。

## Package的使用方法 {#section_p5q_211_hfb .section}

Package的使用涉及到两个主体：Package创建者和Package使用者。

-   Package创建者项目空间是资源提供方。它将需要分享的资源及其访问权限进行打包，然后许可Package使用方来安装使用。
-   Package使用者项目空间是资源使用方。它在安装了资源提供方发布的Package之后，便可以直接跨项目空间访问资源。

下面分别介绍Package创建者和Package使用者所涉及的操作。

## Package创建者 {#section_sbs_1l1_wdb .section}

-   创建Package

    ``` {#codeblock_okt_0ur_fze}
    create package <pkgname>;
    ```

    **说明：** 

    -   只有Project的Owner才有权限进行该操作。
    -   目前创建的Package名称不能超过128个字符。
-   将分享的资源添加到Package

    ``` {#codeblock_h7j_l2c_mkm}
    add project_object to package package_name [with privileges privileges]; --将对象添加到Package。
    remove project_object from package package_name; --将对象从Package移除。
    project_object ::= table table_name |
                   instance inst_name |
                   function func_name |
                   resource res_name
    privileges ::= action_item1, action_item2, ...
    ```

    **说明：** 

    -   目前支持的对象类型不包括Project类型，也就是不允许通过Package在其它Project中创对象。
    -   添加资源时，对象名称不能加项目名前缀。例如当前Project为prj1，需要添加表table\_test到某个Package中，则进行Add操作时，表名不能写成prj1.table\_test，应该直接写表名table\_test。
    -   添加到Package中的不仅仅是对象本身，还包括相应的操作权限。当没有通过`[with privileges privileges]`来指定操作权限时，默认为只读权限，即Read/Describe/Select。 对象及其权限被看作一个整体，添加后不可被更新。如果有需要，则只能删除和重新添加。
    -   对象添加到package时，并非快照打包。因此后续对象数据有变更时，通过package授权访问对象也是访问该对象的当前数据。
-   许可其他项目空间使用Package

    ``` {#codeblock_ik5_dfk_ktx}
    allow project <prjname> to install package <pkgname> [using label <number>];
    ```

-   撤销其他项目空间使用Package的许可

    ``` {#codeblock_cb1_ny8_bqq}
    disallow project <prjname> to install package <pkgname>;
    ```

-   删除Package

    ``` {#codeblock_qxf_k7g_zac}
    delete package <pkgname>;
    ```

-   查看已创建和已安装的Package列表

    ``` {#codeblock_65b_zt3_2t0}
    show packages;
    ```

-   查看Package详细信息

    ``` {#codeblock_sfu_8v6_p55}
    describe package <pkgname>;
    ```


## Package使用者 {#section_kwk_fl1_wdb .section}

-   安装Package

    ``` {#codeblock_umq_081_lky}
    install package <pkgname>;
    ```

    安装Package时，要求pkgName的格式为：`<projectName>.<packageName>`。

    **说明：** 只有Project的Owner才有权限进行该操作。

-   卸载Package

    ``` {#codeblock_6tr_l6w_0z9}
    uninstall package <pkgname>;
    ```

    卸载Package时，要求pkgName的格式为：`<projectName>.<packageName>`

-   查看Package

    ``` {#codeblock_qzy_qxa_dnq}
    show packages; --查看已创建和已安装的Package列表。
    describe package <pkgname>; --查看Package详细信息。
    ```

-   使用方项目给本项目其他成员或Role授权访问Package

    被安装的package是一种独立的MaxCompute对象类型。若要访问Package里的资源（即其它项目空间分享给您的资源），您必须拥有该Package的Read权限。如果请求者没有Read权限，则需要向Project Owner或Admin申请。Project Owner或Admin可以通过ACL授权机制完成此授权。

    将Package授权给User或Role：

    ``` {#codeblock_exo_imm_ru3}
    grant actions on package <pkgName> to user <username>;
    grant actions on package <pkgName> to role <role_name>;
    ```

    **说明：** 授权后，User仅在此Project中有权限访问该Package中的对象。

    示例

    -   ACL授权允许云账号用户aliyun$odps\_test@aliyun.com访问Package里的资源

        ``` {#codeblock_9hj_o54_tvt}
        use prj2;
        install package prj1.testpkg;
        grant read on package prj1.testpackage to user aliyun$odps_test@aliyun.com;
        ```

    -   ACL授权允许角色role\_dev的所有成员访问Package里的资源

        ``` {#codeblock_ssm_6jc_9np}
        use prj2;
        install package prj1.testpkg;
        grant read on package prj1.testpackage to role role_dev;
        ```


## 场景示例 {#section_hx1_3l1_wdb .section}

场景描述：Jack是项目空间prj1的管理员。John是项目空间prj2的管理员。由于业务需要，Jack希望将其项目空间prj1中的某些资源（例如datamining.jar及sampletable表）分享给John的项目空间prj2。如果项目空间prj2的用户Bob需要访问这些资源，则管理员John可以通过ACL给Bob自主授权，无需Jack参与。

操作步骤：

1.  项目空间prj1的管理员Jack在项目空间prj1中创建资源包（Package）。

    ``` {#codeblock_40f_uuq_n59}
    use prj1;
    create package datamining; --创建一个Package。
    add resource datamining.jar to package datamining; --添加资源到Package。
    add table sampletable to package datamining; --添加表到Package。
    allow project prj2 to install package datamining; --将Package分享给项目空间prj2。
    ```

2.  项目空间prj2管理员John在项目空间prj2中安装Package。

    ``` {#codeblock_scv_msc_7ub}
    use prj2;
    install package prj1.datamining; --安装一个Package。
    describe package prj1.datamining; --查看Package中的资源列表。
    ```

3.  John对Package给Bob进行自主授权。

    ``` {#codeblock_9py_t80_q42}
    use prj2;
    grant Read on package prj1.datamining to user aliyun$bob@aliyun.com; --通过ACL授权Bob使用Package。
    ```


