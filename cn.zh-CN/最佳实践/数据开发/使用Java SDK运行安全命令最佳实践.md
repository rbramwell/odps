# 使用Java SDK运行安全命令最佳实践 {#concept_nhc_1ln_4gb .concept}

本文为您介绍在MaxCompute Console上通过Java SDK接口的SecurityManager.runQuery\(\)方法运行安全相关的命令的最佳实践。

## 背景信息 {#section_apv_rqn_4gb .section}

用户运行安全相关的命令有以下两种方式：

-   通过使用MaxCompute[Console](../../../../cn.zh-CN/工具及下载/客户端.md#) 运行，详细的使用说明请参见[安全指南](../../../../cn.zh-CN/管理/安全功能详解/目标用户.md#)和[安全相关语句汇总](../../../../cn.zh-CN/管理/安全功能详解/项目空间的安全配置.md#)。

    以下关键字开头的命令为MaxCompute安全相关的操作命令。

    ```language-sql
    GRANT/REVOKE ...
    SHOW  GRANTS/ACL/PACKAGE/LABEL/ROLE/PRINCIPALS
    SHOW  PRIV/PRIVILEGES
    LIST/ADD/REOVE  USERS/ROLES/TRUSTEDPROJECTS
    DROP/CREATE   ROLE
    CLEAR EXPIRED  GRANTS
    DESC/DESCRIBE   ROLE/PACKAGE
    CREATE/DELETE/DROP  PACKAGE
    ADD ... TO  PACKAGE
    REMOVE ... FROM  PACKAGE
    ALLOW/DISALLOW  PROJECT
    INSTALL/UNINSTALL  PACKAGE
    LIST/ADD/REMOVE   ACCOUNTPROVIDERS
    SET  LABLE  ...
    ```

-   使用Java SDK接口`SecurityManager.runQuery()`方式运行，详细的SDK使用说明请参见[MaxCompute SDK Java Doc](http://www.javadoc.io/doc/com.aliyun.odps/odps-sdk-core/0.29.11-oversea-public?spm=a2c4e.11153940.blogcont686985.22.57a97573bI8DuQ&file=0.29.11-oversea-public)。

    **说明：** MaxCompute安全相关的命令不是SQL命令，不能创建instance通过SQL Task方式来运行。


## 前提条件 {#section_e42_pqn_4gb .section}

在开始使用Java SDK接口方法运行安全命令之前，您需要做如下准备工作：

1.  准备IntelliJ IDEA开发工具，请参见[安装Studio](../../../../cn.zh-CN/工具及下载/MaxCompute Studio/工具安装与版本信息/IntelliJ IDEA安装步骤.md#)。
2.  准备AccessKey ID和Access Key Secret。

    您可以登录阿里云官网，在右上角的用户名下单击**accesskeys**进入AccessKey管理页面获取，如下图。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/120378/155918656338210_zh-CN.png)

3.  配置Endpoint，详细内容请参见[配置Endpoint](../../../../cn.zh-CN/准备工作/配置Endpoint.md#)。
4.  创建空间项目<your\_project\>，通过IntelliJ IDEA MaxCompute Studio[创建MaxCompute项目连接](../../../../cn.zh-CN/工具及下载/MaxCompute Studio/项目空间连接管理.md#)。
5.  在MaxCompute Studio上添加项目依赖。

    SecurityManager类在odps-sdk-core包中，因此在使用时需要做如下配置。

    ```language-xml
    <dependency>
      <groupId>com.aliyun.odps</groupId>
      <artifactId>odps-sdk-core</artifactId>
      <version>0.29.11-oversea-public</version>
    </dependency>
    ```


准备工作完成后，您可以开始运行安全命令示例操作。例如，设置表test\_label列的访问级别为2，运行如下命令。

```
SET LABEL 2 TO TABLE test_label(key, value);
```

## 实施步骤 {#section_un5_1sn_4gb .section}

1.  创建测试表test\_label，命令如下。

    ```
    CREATE TABLE IF NOT EXISTS test_label(
     key  STRING,
     value BIGINT
    );
    ```

2.  测试运行。
    -   Java代码

        ```language-java
        import com.aliyun.odps.Column;
        import com.aliyun.odps.Odps;
        import com.aliyun.odps.OdpsException;
        import com.aliyun.odps.OdpsType;
        import com.aliyun.odps.TableSchema;
        import com.aliyun.odps.account.Account;
        import com.aliyun.odps.account.AliyunAccount;
        import com.aliyun.odps.security.SecurityManager;
        public class test {
          public static void main(String [] args) throws OdpsException {
            try {
              // init odps
              Account account = new AliyunAccount("<your_accessid>", "<your_accesskey>");
              Odps odps = new Odps(account);
              odps.setEndpoint("http://service-corp.odps.aliyun-inc.com/api");
              odps.setDefaultProject("<your_project>");
              // set label 2 to table columns
              SecurityManager securityManager = odps.projects().get().getSecurityManager();
              String res = securityManager.runQuery("SET LABEL 2 TO TABLE test_label(key, value);", false);
              System.out.println(res);
            } catch (OdpsException e) {
              e.printStackTrace();
            }
          }
        }
        ```

    -   查看运行结果。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/120378/155918656338175_zh-CN.png)

3.  结果验证。

    程序运行完成后，在MaxCompute Console中运行desc test\_label命令，可以看到set label命令已经生效了。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/120378/155918656338189_zh-CN.png)

    其它安全相关的命令，都可以通过Java SDK来运行。


## 更多信息 {#section_x5j_f1p_4gb .section}

了解更多安全命令操作相关的内容，可参考[授权](../../../../cn.zh-CN/管理/安全功能详解/用户及授权管理/授权.md#)、[列级别访问控制](../../../../cn.zh-CN/管理/安全功能详解/列级别访问控制.md#)、[项目空间的安全配置](../../../../cn.zh-CN/管理/安全功能详解/项目空间的安全配置.md#)以及[基于Package的跨项目空间的资源分享](../../../../cn.zh-CN/管理/安全功能详解/跨项目空间的资源分享/基于Package的跨项目空间的资源分享.md#)。

