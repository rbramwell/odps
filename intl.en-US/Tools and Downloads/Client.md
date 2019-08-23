# Client {#concept_dvj_dzw_5db .concept}

This article describes how to use the basic functions of the MaxCompute using client command line tool. Before using the MaxCompute client, you must [install and configure the client](../../../../reseller.en-US/Prepare/Install and configure a client.md) .

**Note:** 

-   Do not perform the analysis operation based on the output format of the client. The output format of the client is not ensured for forward compatibility. Clients of different versions are different in their command formats and behaviors.
-   For more information about basic commands of the client, see [Basic commands](../../../../reseller.en-US/User Guide/Common commands/List of common commands.md).
-   [Click here](https://odps-repo.oss-cn-hangzhou.aliyuncs.com/odpscmd/latest/odpscmd_public.zip) to download the new version of MaxCompute client.
-   The client supports JDK 1.9 from the 0.28.0 version, and the previous version can only use JDK 1.8.
-   The client supports MaxCompute 2.0 from the 0.27.0 version [New data type](../../../../reseller.en-US/User Guide/Data types.md#).

After the client is installed and configured, you can use a command line to perform the following operations.

## Get Help {#section_p5h_41x_5db .section}

To view the help information of the console, the command format is as follows:

``` {#codeblock_3qn_eg9_2f0}
odps @>./bin/odpscmd-h;
```

You can also input `h;` or `help;`\(case-insensitive\) in an interactive mode.

The console also provides the `help [keyword];` command to get the command prompts related to the keyword. For example, input `help table;` to get command prompts related to the table operation as follows:

``` {#codeblock_45b_ip2_4g0}



odps @ odps> help table;
Usage: alter table merge smallfiles
Usage: show tables [in]
      a list of tables | ls [-p,-project]
Usage: describe deserves mention | [.] [partition ()]
Usage: read [.]  [partition ()] [line_num]
```

## Start parameters {#section_u5h_41x_5db .section}

When start the console, you can specify a series of parameters as follows:

``` {#codeblock_hc3_yan_fiw}



Usage: odpscmd [option]...
where options include:
    --help (-h) for help
    --project= use project
    --endpoint= set endpoint
    -u-p user name and password
    -k will skip begining queries and start from specified position
    -r set retry times
    -f <"file_path;"> execute command in file
    -e <"command; [command;]..."> execute command, include sql command
    -C will display job counters
```

Take the -f parameter as an example, the operation is as follows:

1.  Prepare the local script file script.txt. Suppose that the file is located in the disk D, and the content is shown as follows:

    ``` {#codeblock_632_w46_ikc}
    
    
    
    DROP TABLE IF EXISTS test_table_mj;
    CREATE TABLE test_table_mj (id string, name string);
    DROP TABLE test_table_mj;
    ```

2.  Run the following command:

    ``` {#codeblock_m9e_j8l_n5s}
    odpscmd\bin>odpscmd -f D:/script.txt;
    ```


## Interactive mode {#section_evh_41x_5db .section}

Run the console to directly enter the interactive mode:

``` {#codeblock_xex_8f8_t5r}



[admin: ~]$odpscmd
Aliyun ODPS Command Line Tool
Version 1.0
@Copyright 2012 Alibaba Cloud Computing Co., Ltd. All rights reserved.
odps@ odps> INSERT OVERWRITE TABLE DUAL SELECT * FROM DUAL;
```

Enter the command at the cursor position \(use a semicolon as a statement terminator\), and press Enter to run.

## Continuous running {#section_hvh_41x_5db .section}

-   When using -e or -f option to run a command, if there are multiple statements, and you want to start running from a middle statement, you can specify the parameter -k, indicating to ignore the previous statements and to start running from the specified position. When the parameter <= 0 is specified, the execution starts from the first statement.
-   Each statement separated by a semicolon is considered as a valid statement. The statements which run successfully or fail to run are printed out at runtime.

For example,

suppose there are three SQL statements in the file /tmp/dual.sql:

``` {#codeblock_8o5_rtt_oon}



drop table dual;
create table dual (dummy string);
insert overwrite table dual select count(*) from dual;
```

To ignore the first two statements, and start running from the third statement, the command format is as follows:

``` {#codeblock_xca_vl1_zk5}
odpscmd -k 3 -f dual.sql
```

## Get current logon user {#section_lvh_41x_5db .section}

To get current logon user, the command format is as follows:

``` {#codeblock_yxo_pv6_xjv}
whoami;
```

Use example:

``` {#codeblock_q2d_o9g_yhc}



odps@ hiveut>whoami;
Name: odpstest@aliyun.com
End_Point: http://service.odps.aliyun.com/api
Project: lijunsecuritytest
```

Use the preceding command to get the current logon user Alibaba Cloud account, endpoint configuration, and project name.

## Exit {#section_ovh_41x_5db .section}

To exit the console, the command format is as follows:

``` {#codeblock_4fp_lbh_8ea}
odps@ > quit;
```

You can also use the following command to exit the console:

``` {#codeblock_u0q_03h_bc2}
odps@ > q;
```

