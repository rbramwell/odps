# Install and configure a client {#concept_qbk_1kv_tdb .concept}

MaxCompute console is also called odpscmd MaxCompute CLI tool.

Once you install and configure the MaxCompute console, you can access all functions of MaxCompute through the console. For more information, see [Client](../../../../reseller.en-US/Tools and Downloads/Client.md).

**Note:** 

-   We recommend that you use the MaxCompute Studio tool. This tool integrates with the Java environment and allows MaxCompute SQL script development and execution, data management, visual log analysis, and Java \(UDF, MR\) development. For more information. see [What is Studio](../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/What is Studio.md).
-   You can also use MaxCompute through DataWorks. After creating a project in the console, you can click on **Enter Workspace** of the specified project in the project list to operate projects with MaxCompute. For more information, see [DataWorks](../../../../reseller.en-US/Product Introduction/What is DataWorks?.md#).

## Install the console {#section_cnm_c5v_tdb .section}

**Note:** Make sure you have JRE 1.7 or a later version installed to properly use the MaxCompute console.

1.  [Click here](http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/attach/119118/cn_zh/1559120057202/odpscmd_public_May.zip) to download the MaxCompute client.
2.  Decompress it into a folder. After decompression, you can see four following folders:

    ``` {#codeblock_ykk_etz_971}
    bin/ conf/ lib/ plugins/
    ```


## Configure the Console {#section_akn_wvv_tdb .section}

A file called odps\_config.ini is in the conf folder. Edit it by filling in corresponding information to configure the console:

``` {#codeblock_t8n_j8z_48k}
access_id=*******************
access_key=********************* 
 #Your AccessKeyId and AccessKeySecret. You can apply for an AccessKey through the official Alibaba Cloud console.
project_name=my_project # Specify the project you want to use.
end_point=https://service.odps.aliyun.com/api # Access URL of the MaxCompute service.
tunnel_endpoint=https://dt.odps.aliyun.com # Access URL of the MaxCompute Tunnel service.
log_view_host=http://logview.odps.aliyun.com 
# After a user executes a task, the console returns the LogView address of this task. You can view the detailed execution information of this task at this address. 
https_check=true  # Determine whether to enable HTTPS access.
```

**Note:** 

-   We recommend that you configure the console according to the region you designated the MaxCompute service.
-   **\#** is used as a comment symbol in the dps\_config.ini file. However, the MaxCompute console uses two minus signs **--** as a comment symbol.
-   Creating a project in advance is needed so that you can specify in the configuration file. For more information, see [Project](reseller.en-US/Prepare/Create a project.md).
-   MaxCompute provides two service addresses separately on the Internet and the intranet. Different addresses correspond to different download prices.For more information. see [Access Domains and Data Centers](reseller.en-US/Prepare/Configure Endpoint.md).

## Run the console {#section_epm_szx_5db .section}

After the configuration file has been modified, run MaxCompute in bin directory\(For Linux, run ./bin/odpscmd, for Windows, run ./bin/odpscmd.bat\). For example:

``` {#codeblock_8mq_2wv_jnz}
create table tbl1(id bigint);
insert overwrite table tbl1 select count(*) from tbl1;
select 'welcome to MaxCompute!' from tbl1;
```

For more information about SQL statements, see [SQL Summary](../../../../reseller.en-US/User Guide/SQL/SQL summary.md).

