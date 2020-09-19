# Tunnel commands {#concept_rkf_2wc_5db .concept}

This topic introduces you to the instructions for the use of Upload, Show, Resume and other Tunnel upload and download commands.

## Features {#section_hcs_btf_vdb .section}

The [Client](../../../../reseller.en-US/Tools and Downloads/Client.md) provides [Tunnel](reseller.en-US/User Guide/Data upload and download/Tunnel commands.md) commands for you to use the functions of the original Dship tool.

Tunnel commands are mainly used to upload or download data.

-   Upload: Supports file or directory \(level-one\) uploading. Data can only be uploaded to a single table or table partition each time. For partitioned tables, the destination partition must be specified.

    ```
    tunnel upload log.txt test_project.test_table/p1="b1",p2="b2";
    -- Uploads data in log.txt to the test_project project's test_table table, partitions: p1="b1",p2="b2".
    tunnel upload log.txt test_table --scan=only;
    -- Uploads data from log.txt to the test_table table.--The scan parameter indicates that the data in log.txt must be scanned to determine if it complies with the test_table definitions.If it does not, the system reports an error and the upload is stopped.
    ```

-   Download: You can only download data to a single file. Only data in one table or partition can be downloaded to one file each time. For partitioned tables, the source partition must be specified.

    ```
    tunnel download test_project.test_table/p1="b1",p2="b2" test_table.txt;
    -- Download data from the table to the test_table.txt file.
    ```

-   Resume: If an error occurs because of network or the Tunnel service, you can resume transmission of the file or directory after interruption. This command allows you to resume the previous data upload operation, but does not support download operations.

    ```
    tunnel resume;
    ```

-   Show: Displays the history of the commands used.

    ```
    tunnel show history -n 5;
    -- Displays details for the last five data upload/download commands.
    tunnel show log;
    --Displays the log for the last data upload/download.
    ```

-   Purge: Clears the session directory. Use this command to clear history for last three days.

    ```
    tunnel purge 5;
    --Clears logs from the previous five days.
    ```


## Tunnel upload and download limits {#section_an5_42t_j2b .section}

Tunnel command does not support uploading and downloading data of the ARRAY, MAP, and STRUCT types.

Each session has a 24-hour life cycle on the server. It can be used within 24 hours after being created, and can be shared among processes or threads. The block ID of each session must be unique.

## Use of Tunnel commands {#section_g2l_1wf_vdb .section}

Tunnel commands allow you to obtain help information using the Help sub-command on the client. Each command and selection supports short command format.

```
odps@ project_name>tunnel help;
    Usage: tunnel <subcommand> [options] [args]
    Type 'tunnel help <subcommand>' for help on a specific subcommand.
Available subcommands:
    upload (u)
    download (d)
    resume (r)
    show (s)
    purge (p)
    help (h)
tunnel is a command for uploading data to / downloading data from MaxCompute.
```

**Parameters** 

-   upload: Uploads the data to a MaxCompute table.
-   download: Downloads the data from a MaxCompute table.
-   resume: If data fails to be uploaded, use the Resume command to resume the upload from where it was interrupted. Do not use this command for download operations. Each data upload or download operation is called as a session. Run the Resume command and specify the session ID to be resumed.
-   show: Displays the history of the commands used.
-   purge: Clears the session directory. Use this command to clear history for last three days.
-   help: Provides 'help' information regarding questions related to Tunnel.

## Upload {#section_uwr_pwf_vdb .section}

Import data of local files to MaxCompute tables in the append mode. The sub-commands are used as follows:

```
odps@ project_name>tunnel help upload;
usage: tunnel upload [options] <path> <[project.]table[/partition]>
              upload data from local file
 -acp,-auto-create-partition <ARG> auto create target partition if not
                                     exists, default false
 -bs,-block-size <ARG> block size in MiB, default 100
 -c,-charset <ARG> specify file charset, default ignore.
                                     set ignore to download raw data
 -cp,-compress <ARG> compress, default true
 -dbr,-discard-bad-records <ARG> specify discard bad records
                                     action(true|false), default false
 -dfp,-date-format-pattern <ARG> specify date format pattern, default
                                     yyyy-MM-dd HH:mm:ss; 
 -fd,-field-delimiter <ARG> specify field delimiter, support
                                     unicode, eg \u0001. default ","
 -h,-header <ARG> if local file should have table
                                     header, default false
 -mbr,-max-bad-records <ARG> max bad records, default 1000
 -ni,-null-indicator <ARG> specify null indicator string,
                                     default ""(empty string)
 -rd,-record-delimiter <ARG> specify record delimiter, support
                                     unicode, eg \u0001. default "\r\n"
 -s,-scan <ARG> specify scan file
                                     action(true|false|only), default false
 -sd,-session-dir <ARG> set session dir, default
                                     D:\software\odpscmd_public\plugins\ds
                                     hip
 -ss,-strict-schema <ARG> specify strict schema mode. If false,
                                     extra data will be abandoned and
                                     insufficient field will be filled
                                     with null. Default true
 -te,-tunnel_endpoint <ARG> tunnel endpoint
    -threads <ARG> number of threads, default 1
 -tz,-time-zone <ARG> time zone, default local timezone:
                                     Asia/Shanghai
For example:
    tunnel upload log.txt test_project.test_table/p1="b1",p2="b2"
```

**Parameters** 

-   -acp: Determines if the operation automatically creates the destination partition if it does not exist. This one is disabled by default.
-   -bs: Specifies the size of each data block uploaded using Tunnel. Default value: 100MiB \(1MiB=1024\*1024B\) .
-   -c: Specifies the local data file encoding. Default value: I. When not set, the encoding of the downloaded source data is used by default.
-   -cp: Determines whether the local file is compressed before being uploaded, reducing traffic usage. It is enabled by default.
-   -dbr: Determines whether to ignore corrupted data \(including extra, missing columns or mismatched column data types\).
    -   If this value is true, all the data that does not satisfy table definitions is ignored.
    -   When the parameter is set to false, the system displays error messages in case of corrupted data, but the raw data in the destination table remains unaffected.
-   -dfp: Specifies the format of DateTime data. Default value: yyyy-MM-dd HH:mm:ss. If you want to specify the time format to the level of milliseconds, use tunnel upload -dfp 'yyyy-MM-dd HH:mm:ss.SSS', for more information, see [Data types](../../../../reseller.en-US/User Guide/Definitions/Data types.md#).
-   -fd: Specifies the column delimiter of the local data file. The default value is comma \(,\).
-   -h: Determines whether the data file contains the header. If it is set to true, Dship skips the header and starts uploading from the next row.
-   -mbr: By default, if more than 1,000 rows of corrupted data is uploaded, the upload is terminated. This parameter allows you to adjust the tolerated volume of the corrupted data.
-   -ni: Specifies the NULL data identifier. Default value: “ “\(blank string\).
-   -rd: Specifies the row delimiter of the local data file. Default value: \\r\\n.
-   -s: Determines whether to scan the local data file. Default value: false.
    -   If set to true, the system scans the data first, and then imports the data if the format is correct.
    -   If set to false, the system imports the data directly without scanning.
    -   If the value is 'only', then only the local data is scanned. No data is imported after scanning.
-   -sd: Sets the session directory.
-   -te: Specifies the tunnel endpoint.
-   -threads: Specifies the number of threads. Default value: 1.
-   -tz: Specifies the time zone. The default value is the local time zone: Asia/Shanghai.

**Example** 

-   Create a destination table:

```
CREATE TABLE IF NOT EXISTS sale_detail(
      shop_name STRING,
      customer_id STRING,
      total_price DOUBLE)
PARTITIONED BY (sale_date STRING,region STRING);
```

-   Add a partition:

    ```
    alter table sale_detail add partition (sale_date='201312', region='hangzhou');
    ```

-   Prepare the data file data.txt with the following content:

    ```
    shop9,97,100
    shop10,10,200
    shop11,11
    ```

    The data of the third row of this file is not consistent with the definition in Table sale\_detail. The three columns are defined by sale\_detail, but this row only has two.

-   Import data:

    ```
    odps@ project_name>tunnel u d:\data.txt sale_detail/sale_date=201312,region=hangzhou -s false
    Upload session: 20150610************************60c
    Start upload:d:\data.txt
    Total bytes:41 Split input to 1 blocks
    2015-06-10 16:39:22 upload block: '1'
    ERROR: column mismatch -,expected 3 columns, 2 columns found, please check data or delimiter
    ```

    Because data.txt contains corrupted data, data import fails. The system displays the session ID and error message.

-   Verify data:

    ```
    odps@ odpstest_ay52c_ay52> select * from sale_detail where sale_date='201312';
    ID = 20150610084135370gyvc61z5
    +-----------+-------------+-------------+-----------+--------+
    | shop_name | customer_id | total_price | sale_date | region |
    +-----------+-------------+-------------+-----------+--------+
    +-----------+-------------+-------------+-----------+--------+
    ```

    The data import failed because of dirty data and hence the table is empty.


## Show {#section_wzb_xxf_vdb .section}

Displays historical records. The sub-commands are used as follows:

```
odps@ project_name>tunnel help show;
usage: tunnel show history [options]
              show session information
 -n,-number <ARG> lines
For example:
    tunnel show history -n 5
    tunnel show log
```

**Parameter**

-n: Specifies the number of rows to be displayed.

**Example** 

```
odps@ project_name>tunnel show history;
20150610************************60c failed 'u --config-file /D:/console/conf/odps_config.ini --project odpstest_ay52c_ay52 --endpoint http://service.odps.aliyun.com/api --id Ul*******rI1 --key 2m*************R d:\data.txt sale_detail/sale_date=201312,region=hangzhou -s false'
```

**Note:** With reference to the preceding example, 20150610\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*60c is the session ID of the failed data importing in the previous section.

## Resume {#section_wgq_jyf_vdb .section}

Repairs and re-executes historical records \(only valid for data uploads\). The sub-commands are used as follows:

```
odps@ project_name>tunnel help resume;
usage: tunnel resume [session_id] [-force]
              resume an upload session
 -f,-force force resume
For example:
    tunnel resume
```

**Example**

Modify the data.txt file as follows:

```
shopx,x_id,100
shopy,y_id,200
```

Re-upload the repaired data:

```
odps@ project_name>tunnel resume 20150610************************60c --force;
start resume
20150610************************60c
Upload session: 20150610************************60c
Start upload:d:\data.txt
Resume 1 blocks 
2015-06-10 16:46:42 upload block: '1'
2015-06-10 16:46:42 upload block complete, blockid=1
upload complete, average speed is 0 KB/s
OK
```

**Note:** With reference to the preceding example, 20150610\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*60c is session ID.

Verify data:

```
odps@ project_name>select * from sale_detail where sale_date='201312';
 ID = 20150610xxxxxxxxxxxa741z5
 +-----------+-------------+-------------+-----------+--------+
 | shop_name | customer_id | total_price | sale_date | region |
 +-----------+-------------+-------------+-----------+--------+
 | shopx     | x_id        | 100.0       | 201312    | hangzhou|
 | shopy     | y_id        | 200.0       | 201312    | hangzhou|
 +-----------+-------------+-------------+-----------+--------+
```

## Download {#section_qxw_2zf_vdb .section}

The sub-commands are used as follows:

```
odps@ project_name>tunnel help download;
usage: tunnel download [options] <[project.]table[/partition]> <path>
              download data to local file
 -c,-charset <ARG> specify file charset, default ignore.
                                   set ignore to download raw data
 -ci,-columns-index <ARG> specify the columns index(starts from
                                   0) to download, use comma to split each
                                   index
 -cn,-columns-name <ARG> specify the columns name to download,
                                   use comma to split each name
 -cp,-compress <ARG> compress, default true
 -dfp,-date-format-pattern <ARG> specify date format pattern, default
                                   yyyy-MM-dd HH:mm:ss
 -e,-exponential <ARG> When download double values, use
                                   exponential express if necessary.
                                   Otherwise at most 20 digits will be
                                   reserved. Default false
 -fd,-field-delimiter <ARG> specify field delimiter, support
                                   unicode, eg \u0001. default ","
 -h,-header <ARG> if local file should have table header,
                                   default false
    -limit <ARG> specify the number of records to
                                   download
 -ni,-null-indicator <ARG> specify null indicator string, default
                                   ""(empty string)
 -rd,-record-delimiter <ARG> specify record delimiter, support
                                   unicode, eg \u0001. default "\r\n"
 -sd,-session-dir <ARG> set session dir, default
                                   D:\software\odpscmd_public\plugins\dshi
                                   p
 -te,-tunnel_endpoint <ARG> tunnel endpoint
    -threads <ARG> number of threads, default 1
 -tz,-time-zone <ARG> time zone, default local timezone:
                                   Asia/Shanghai
usage: tunnel download [options] instance://<[project/]instance_id> <path>
              download instance result to local file
 -c,-charset <ARG> specify file charset, default ignore.
                                   set ignore to download raw data
 -ci,-columns-index <ARG> specify the columns index(starts from
                                   0) to download, use comma to split each
                                   index
 -cn,-columns-name <ARG> specify the columns name to download,
                                   use comma to split each name
 -cp,-compress <ARG> compress, default true
 -dfp,-date-format-pattern <ARG> specify date format pattern, default
                                   yyyy-MM-dd HH:mm:ss
 -e,-exponential <ARG> When download double values, use
                                   exponential express if necessary.
                                   Otherwise at most 20 digits will be
                                   reserved. Default false
 -fd,-field-delimiter <ARG> specify field delimiter, support
                                   unicode, eg \u0001. default ","
 -h,-header <ARG> if local file should have table header,
                                   default false
    -limit <ARG> specify the number of records to
                                   download
 -ni,-null-indicator <ARG> specify null indicator string, default
                                   ""(empty string)
 -rd,-record-delimiter <ARG> specify record delimiter, support
                                   unicode, eg \u0001. default "\r\n"
 -sd,-session-dir <ARG> set session dir, default
                                   D:\software\odpscmd_public\plugins\dshi
                                   p
 -te,-tunnel_endpoint <ARG> tunnel endpoint
    -threads <ARG> number of threads, default 1
 -tz,-time-zone <ARG> time zone, default local timezone:
                                   Asia/Shanghai
For example:
    tunnel download test_project.test_table/p1="b1",p2="b2" log.txt
    tunnel download instance://test_project/test_instance log.txt
```

**Parameters** 

-   -c: Specifies the local data file encoding. Default value: Ignore.
-   -ci: Specifies the column index \(starts from 0\) for downloading. Separate multiple entries with commas \(,\).
-   -cn: Specifies the names of the columns to download. Separate multiple entries with commas \(,\).
-   -cp, -compress: Determines whether the data is compressed before it is downloaded, reducing traffic usage. It is enabled by default.
-   -dfp: Specifies the format of DateTime data. Default value: yyyy-MM-dd HH:mm:ss.
-   -e: When downloading Double type data, use this parameter to express the values as exponential functions. Otherwise, a maximum of 20 digits can be retained.
-   -fd: Specifies the column delimiter of the local data file. The default value is comma \(,\).
-   -h: Determines whether the data file contains the header. If set to ‘true’, Dship skips the header and starts downloading from the second row.

    **Note:** `-h=true` and `threads>1` cannot be used together.

-   -limit: Specifies the number of files to be downloaded.
-   -ni: Specifies the NULL data identifier. Default value: “ “\(blank string\).
-   -rd: Specifies the row delimiter of the local data file. Default value: \\r\\n.
-   -sd: Sets the session directory.
-   -te: Specifies the tunnel endpoint.
-   -threads: Specifies the number of threads. Default value: 1.
-   -tz: Specifies the time zone. The default value is the local time zone: Asia/Shanghai.

**Example**

Download data to the result.txt:

```
$ ./tunnel download sale_detail/sale_date=201312,region=hangzhou result.txt;
    Download session: 201506101658245283870a002ed0b9
    Total records: 2
    2015-06-10 16:58:24 download records: 2
    2015-06-10 16:58:24 file size: 30 bytes
    OK
```

Verify the content of the result.txt:

```
shop9,97,100.0
shop10,10,200.0
```

**Download the data of an instance**

**Method 1**

Run the tunnel download command to download the query result of a specified instance to your computer.

```
tunnel download instance://<[project_name/]instance_id> <path>
```

**Parameters** 

-   project\_name: the name of the project where the instance resides.
-   instance\_id: the ID of the instance whose data is to be downloaded.

**Example**

1.  Run a `SELECT` statement to query the data of a specified instance:

    ``` {#codeblock_b8h_zzb_0to}
    odps@ odps_test_project>select * from wc_in;
    ID = 20170724071705393ge3csfb8
    ... ...
    ```

2.  Use InstanceTunnel to download the query result to your computer:

    ``` {#codeblock_odm_7to_6v9}
    odps@ odps_test_project>tunnel download instance://20170724071705393ge3csfb8 result;
    2017-07-24 15:18:47  -  new session: 2017072415184785b6516400090ca8    total lines: 8
    2017-07-24 15:18:47  -  file [0]: [0, 8), result
    downloading 8 records into 1 file
    2017-07-24 15:18:47  -  file [0] start
    2017-07-24 15:18:48  -  file [0] OK. total: 44 bytes
    download OK
    ```

3.  View the query result:

    ``` {#codeblock_64s_b6q_rw6}
    cat result
    slkdfj
    hellp
    apple
    tea
    peach
    apple
    tea
    teaa
    ```


**Method 2**

Set the `use_instance_tunnel` parameter to true.

After you set this parameter to true on the MaxCompute console, SQL uses InstanceTunnel to download the query results that are returned by `SELECT` statements. This helps to ensure that the query results can be properly downloaded in the case that your session times out or the data to be downloaded exceeds the maximum size that is permitted. You can use one of the following two methods to enable this function:

-   Log on to the latest version of [Console](../../../../reseller.en-US/Tools and Downloads/Client.md#). Find the odps\_config.ini file. Then set the `use_instance_tunnel` parameter to true and the `instance_tunnel_max_record` parameter to 10000.

    ```
    # download sql results by instance tunnel
    use_instance_tunnel=true
    # the max records when download sql results by instance tunnel
    instance_tunnel_max_record=10000
    ```

    **Note:** The `instance_tunnel_max_record` parameter specifies the maximum number of query result records that SQL can download by using InstanceTunnel. If this parameter is unspecified, the number of query result records that can be downloaded is not limited by this parameter.

-   Run the following command to set the `console.sql.result.instancetunnel` parameter to true:

    ``` {#codeblock_5oo_1hw_8gj}
    odps@ odps_test_tunnel_project>set console.sql.result.instancetunnel=true;
    OK
    ```

    You can run a `SELECT` statement to query the data of a specified instance:

    ``` {#codeblock_e8t_7n7_nzm}
    odps@ odps_test_tunnel_project>select * from wc_in;
    ID = 20170724081946458g14csfb8
    Log view:
    http://logview/xxxxx.....
    +------------+
    | key        |
    +------------+
    | slkdfj     |
    | hellp      |
    | apple      |
    | tea        |
    | peach      |
    | apple      |
    | tea        |
    | teaa       |
    +------------+
    A total of 8 records fetched by instance tunnel.
    ```

    **Note:** When InstanceTunnel is enabled, a message is displayed that states that a total of eight records is returned, after you run a `SELECT` statement to query the data of an instance. You can run the `set console.sql.result.instancetunnel` command to disable InstanceTunnel.


## Purge {#section_l3y_51g_vdb .section}

Purge the session directory. By default, sessions for last three days are purged. The sub-commands are used as follows:

```
odps@ project_name>tunnel help purge;
usage: tunnel purge [n]
              force session history to be purged.([n] days before, default
              3 days)
For example:
    tunnel purge 5
```

**Data types**:

|Type|Required|
|:---|:-------|
|STRING|String type data. The length cannot exceed 8MB.|
|BOOLEAN|Upload values only support true, false, 0, and 1. Only the values true or false \(not case-sensitive\) are supported for downloading.|
|BIGINT|Value range: \[-9223372036854775807, 9223372036854775807\].|
|DOUBLE| -   16-bit valid.
-   Uploads support expression in scientific notation.
-   Supports only numerical expression for downloading.
-   Max value: 1.7976931348623157E308.
-   Min value: 4.9E-324.
-   Positive infinity: Infinity.
-   Negative infinity: -Infinity.

 |
|DATETIME|By default, Datetime data supports the UTC+8 time zone for data upload. Use the command to specify the format pattern for the date in your data.|

If you upload DATETIME type data, specify the time and date format. For more information about specific formats, see SimpleDateFormat.

```
"yyyyMMddHHmmss": data format "20140209101000"
"yyyy-MM-dd HH:mm:ss" (default): data format "2014-02-09 10:10:00"
"MM/dd/yyyy": data format "09/01/2014"
```

**Example** 

```
tunnel upload log.txt test_table -dfp "yyyy-MM-dd HH:mm:ss"
```

**Null**: All data types can be Null.

-   By default, a blank string indicates a Null value.
-   The parameter -null-indicator can be used in the command line to specify a Null string.

```
tunnel upload log.txt test_table -ni "NULL"
```

**Character encoding**: You can specify the character encoding of the file. Default value: UTF-8.

```
tunnel upload log.txt test_table -c "gbk"
```

**Delimiter**: The Tunnel commands support custom file delimiters. The row delimiter is ‘-record-delimiter’, and the column delimiter is -field-delimiter.

Description:

-   Row and column delimiters of multiple characters are supported.
-   A column delimiter cannot contain a row delimiter.
-   Only the follow escape character delimiters are supported in the command line: \\r, \\n, and \\t.

**Example** 

```
tunnel upload log.txt test_table -fd "||" -rd "\r\n"
```

