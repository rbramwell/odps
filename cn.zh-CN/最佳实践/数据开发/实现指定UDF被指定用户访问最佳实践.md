# 实现指定UDF被指定用户访问最佳实践 {#concept_td1_cw2_4gb .concept}

## 指定资源被指定用户访问产生背景 {#section_g5w_hw2_4gb .section}

本文为您介绍如何实现将具体的某个资源（表、UDF等）设置为仅能被指定的用户访问。此UDF涉及到数据的加密解密算法，属于数据安全管控范畴。

## 前提条件 {#section_ywq_zx2_4gb .section}

您需要提前安装MaxCompute客户端，以实现指定UDF被指定用户访问的操作。详情请参见[安装并配置客户端](../../../../cn.zh-CN/准备工作/安装并配置客户端.md#)。

## 常见方案 {#section_ekd_rx2_4gb .section}

您可通过以下三种方案，实现指定UDF被指定用户访问。

-   package方案，通过打包授权进行权限精细化管控。

    [Package](../../../../cn.zh-CN/管理/安全功能详解/跨项目空间的资源分享/基于Package的跨项目空间的资源分享.md#)通常是为了解决跨项目空间的共享数据及资源的用户授权问题。当通过package授予用户开发者角色后，用户则拥有所有权限，风险不可控。

    -   首先，用户熟知的DataWorks开发者角色的权限如下所示。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/118822/156316489038016_zh-CN.png)

        由上图可见，开发者角色对工作空间中的package、functions、resources和table默认有全部权限，明显不符合权限配置的要求。

    -   其次，通过DataWorks添加子账号并赋予开发者角色，如下所示。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/118822/156316489038018_zh-CN.png)

    由此可见，通过打包授权和DataWorks默认的角色都不能满足我们的需求。比如将子账号RAM$xxxxx.pt@aliyun-test.com:ramtest授予开发者角色，则默认拥有当前工作空间中所有Object的所有操作权限，详情请参见[用户授权](../../../../cn.zh-CN/管理/安全功能详解/用户及授权管理/授权.md#)。

-   在DataWorks中新建角色来进行高级管控。

    您可进入DataWorks控制台中的**工作空间配置** \> **MaxCompute高级配置** \> **自定义用户角色**页面，进行高级管控。但是在MaxCompute高级配置中只能针对某个表/某个项目进行授权，不能对resource和UDF进行授权。

-   Role policy方案，通过role policy自定义role的权限集合。

    通过policy可以精细化地管理到具体用户针对具体资源的具体权限粒度，下文将为您详细描述如何通过role policy方案实现指定UDF被指定用户访问。


**说明：** 为了安全起见，建议初学者使用测试项目来验证policy。

## 通过role policy自定义role的权限集合 {#section_tn4_b1f_4gb .section}

1.  创建默认拒绝访问UDF的角色。
    1.  在客户端输入create role denyudfrole;，创建一个role denyudfrole。
    2.  创建policy授权文件，如下所示。

        ```
        {
        "Version": "1", "Statement"
        
        [{
        "Effect":"Deny",
        "Action":["odps:Read","odps:List"],
        "Resource":"acs:odps:*:projects/sz_mc/resources/getaddr.jar"
        },
        {
        "Effect":"Deny",
        "Action":["odps:Read","odps:List"],
        "Resource":"acs:odps:*:projects/sz_mc/registration/functions/getregion"
        }
         ] }
        ```

    3.  设置和查看role policy。

        在客户端输入put policy /Users/yangyi/Desktop/role\_policy.json on role denyudfrole;命令，设置role policy文件的存放路径等配置。

        通过get policy on role denyudfrole;命令，查看role policy。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/118822/156316489038029_zh-CN.png)

    4.  在客户端输入grant denyudfrole to RAM$xxxx.pt@aliyun-test.com:ramtest;，添加子账号至role denyudfrole。
2.  验证拒绝访问UDF的角色是否创建成功。

    以子账号RAM$xxxx.pt@aliyun-test.com:ramtest登录MaxCompute客户端。

    1.  登录客户端输入whoami;确认角色。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/118822/156316489038052_zh-CN.png)

    2.  通过show grants;查看当前登录用户权限。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/118822/156316489038031_zh-CN.png)

        通过查询发现该RAM子账号有两个角色，一个是role\_project\_dev（即DataWorks默认的开发者角色），另一个是刚自定义创建的denyudfrole。

    3.  验证自建UDF以及依赖的包的权限。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/118822/156316489138032_zh-CN.png)

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/118822/156316489138033_zh-CN.png)

    通过上述验证发现，该子账号在拥有DataWorks开发者角色的前提下并没有自建UDF：getregion的读权限。但还需要结合project policy来实现该UDF只能被指定的用户访问。

3.  配置project policy。
    1.  编写policy。

        ```
        {
        "Version": "1", "Statement":
        [{
        "Effect":"Allow",
        "Principal":"RAM$yangyi.pt@aliyun-test.com:yangyitest",
        "Action":["odps:Read","odps:List","odps:Select"],
        "Resource":"acs:odps:*:projects/sz_mc/resources/getaddr.jar"
        },
        {
        "Effect":"Allow",
         "Principal":"RAM$xxxx.pt@aliyun-test.com:yangyitest",
        "Action":["odps:Read","odps:List","odps:Select"],
        "Resource":"acs:odps:*:projects/sz_mc/registration/functions/getregion"
        }] }
        ```

    2.  设置和查询policy。

        通过put policy /Users/yangyi/Desktop/project\_policy.json;命令设置policy文件的存放路径。

        通过get policy;命令查看policy。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/118822/156316489138036_zh-CN.png)

    3.  通过whoami;和show grants;进行验证。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/118822/156316489138039_zh-CN.png)

    4.  运行SQL任务，查看是否只有指定的RAM子账号能够查看指定的UDF和依赖的包。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/118822/156316489138041_zh-CN.png)

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/118822/156316489238046_zh-CN.png)


## 总结 {#section_ymz_21f_4gb .section}

关于DataWorks和MaxCompute的安全体系，总结如下。

-   如果您不想让其他用户访问工作空间内具体的资源，在DataWorks中添加数据开发者权限后，再在MaxCompute客户端按照role policy的操作，将其配置为拒绝访问权限。
-   如果您要指定用户访问资源，在DataWorks中配置数据开发者权限后，再在MaxCompute客户端按照project policy的操作，将其配置为允许访问权限。

