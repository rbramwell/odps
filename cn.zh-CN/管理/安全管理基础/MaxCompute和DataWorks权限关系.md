# MaxCompute和DataWorks权限关系 {#concept_f4n_mc1_4gb .concept}

通过MaxCompute的安全模型进行权限控制，并不会影响成员在DataWorks任何界面操作。而通过DataWorks的用户角色分配，则有可能影响成员的MaxCompute资源权限。下面详细介绍这两个产品之间的权限如何交叉关联。

## 项目关系 {#section_hcc_qc1_4gb .section}

通过MaxCompute或DataWorks官网产品页进入[管理控制台](../../../../intl.zh-CN/使用指南/管理控制台/管理控制台概览.md#)创建的项目时，有两种选择：

-   **简单模式**的项目实际上是创建了关联绑定好的一个MaxCompute Project和一个DataWorks项目空间（工作空间）。同时在MaxCompute的Project里创建对应的几个Role，具体Role权限请参见**成员角色权限关系**。
-   **标准模式**的项目实际上是创建了关联绑定好的一个开发（dev）MaxCompute Project、一个生产（prod）MaxCompute Project来同时对应一个DataWorks项目空间（工作空间）。同时在MaxCompute的Project里创建对应的几个Role，具体Role权限请参见**成员角色权限关系**。

## 账号认证 {#section_ur4_sc1_4gb .section}

云账号在DataWorks项目中只能是主账号，即项目Owner。在MaxCompute既可以为Owner也可以为普通User。当通过DataWorks项目成员管理添加成员时只能添加当前项目主账号对应的RAM子账号。而MaxCompute可以通过命令行`add user xxx;`命令添加其他云账号。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/118689/156292734038006_zh-CN.png)

## 成员角色与权限关系 {#section_bvy_gf1_4gb .section}

如前文所述，DataWorks为了解决项目成员在ETL过程中需要的MaxCompute相关资源权限，绑定了一些MaxCompute Role。具体是指DataWorks项目固定有几个[成员角色](../../../../intl.zh-CN/使用指南/项目管理/成员管理.md#)，同时在对应MaxCompute Project上创建了对应的几个Role。此外，MaxCompute Project本身除Project Owner之外，还包含有一个admin Role。具体权限对应如下表所示。

|MaxCompute角色|MaxCompute数据权限|DataWorks成员角色|DataWorks平台权限特征|
|------------|--------------|-------------|---------------|
|project owner|MaxCompute Project的Owner，拥有所有Project的权限。|无|无|
|admin| 每一个Project在创建时，会自动创建一个admin的role，并且为该role授予了确定的权限：可以访问Project的所有对象、对user或role进行管理、对user或role进行授权。

 与Project Owner相比：admin角色不能将 admin 权限指派给用户，不能设定项目空间的安全配置，不能修改项目空间的鉴权模型，admin角色所对应的权限不能被修改。

 Project Owner可以将这admin role赋权给一个user，让该user代理安全管理。

 |无|无|
|role\_project\_admin|project/table/fuction/resource/instance/job/package的所有权限|项目管理员|指项目空间的管理者。可对该项目空间的基本属性、数据源、当前项目空间计算引擎配置和项目成员等进行管理，并为项目成员赋予项目管理员、开发、运维、部署、访客角色。|
|role\_project\_dev|project/fuction/resource/instance/job/package/table 的所有权限|开发|开发角色的用户能够创建工作流、脚本文件、资源和UDF，新建/删除表，同时可以创建发布包，但不能执行发布操作。|
|role\_project\_pe|project/fuction/resource/instance/job的所有权限，拥有package的read权限和table 的read/describe权限。|运维|运维角色的用户由项目管理员分配运维权限，拥有发布及线上运维的操作权限，没有数据开发的操作权限。|
|role\_project\_deploy|默认无权限|部署|部署角色与运维角色相似，但是它没有线上运维的操作权限。|
|role\_project\_guest|默认无权限|访客|访客角色的用户只具备查看权限，没有权限进行编辑工作流和代码等操作。|
|role\_Project\_security|默认无权限|安全管理员|安全管理员仅在数据保护伞中用到，用于敏感规则配置，数据风险审计等。|

**说明：** 由上表可知，DataWorks角色对应的MaxCompute权限是固定的：一旦某个user通过DataWorks角色获取MaxCompute相关role权限后，如果此后又通过命令行方式获得了其他的MaxCompute权限，会使该user在MaxCompute的权限与在DataWorks上看到的不一致。

## 用户和权限关系图 {#section_pbf_wpg_4gb .section}

一个DataWorks项目空间绑定一个MaxCompute project，此时您可以根据DataWorks**项目管理** \> **工作空间配置** \> **访问身份**这个属性设置决定DataWorks其他项目成员是否拥有MaxCompute project的权限。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/118689/156292734038072_zh-CN.png)

访问身份分为**个人账号**和**计算引擎指定账号（系统账号）**，具体用户和权限对应关系如下图所示。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/118689/156292734038076_zh-CN.png)

对于标准模式，一个DataWorks项目空间绑定两个MaxCompute project：一个开发项目和一个生产项目。**DataWorks其他项目成员根据成员角色拥有MaxCompute开发project对应的role权限，但没有MaxCompute生产project的权限。**MaxCompute任务需要通过发布流程发布到生产project后，以owner账号提交到MaxCompute执行。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/118689/156292734038077_zh-CN.png)

