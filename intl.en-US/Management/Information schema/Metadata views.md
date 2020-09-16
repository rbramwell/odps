---
keyword: [metadata view, Information Schema]
---

# Metadata views

The Information Schema service of MaxCompute contains the metadata of key objects in a project and provides historical information about job execution and data upload and download.

**Note:** For more information about how to query metadata views, see [t1631189.md\#section\_l3k\_0v2\_zwm](/intl.en-US/Management/Information schema/Overview.md).

The following table lists the metadata views.

|Category|View|Timeliness and retention period|Delay|
|--------|----|-------------------------------|-----|
|Metadata information|TABLES|Quasi-real-time view|Online data is displayed in metadata views with a delay of about three hours.|
|PARTITIONS|Quasi-real-time view|
|COLUMNS|Quasi-real-time view|
|UDFS|Quasi-real-time view|
|RESOURCES|Quasi-real-time view|
|UDF\_RESOURCES|Quasi-real-time view|
|USERS|Quasi-real-time view|
|ROLES|Quasi-real-time view|
|USER\_ROLES|Quasi-real-time view|
|PACKAGE\_OBJECTS|Quasi-real-time view|
|INSTALLED\_PACKAGES|Quasi-real-time view|
|SCHEMA\_PRIVILEGES|Quasi-real-time view|
|TABLE\_PRIVILEGES|Quasi-real-time view|
|COLUMN\_PRIVILEGES|Quasi-real-time view|
|UDF\_PRIVILEGES|Quasi-real-time view|
|RESOURCE\_PRIVILEGES|Quasi-real-time view|
|TABLE\_LABELS|Quasi-real-time view|
|COLUMN\_LABELS|Quasi-real-time view|
|TABLE\_LABEL\_GRANTS|Quasi-real-time view|
|COLUMN\_LABEL\_GRANTS|Quasi-real-time view|
|Historical information|TASKS\_HISTORY|Quasi-real-time view. Historical data is stored in a partitioned table, and data from the last 14 days is retained.|Online data is displayed in metadata views with a delay of about three hours.|
|TUNNELS\_HISTORY|Quasi-real-time view. Historical data is stored in a partitioned table, and data from the last 14 days is retained.|

## TABLES

Displays information about a table in a project.

|Parameter|Type|Description|
|---------|----|-----------|
|table\_catalog|STRING|Set the value to `odps`.|
|table\_schema|STRING|The name of the project.|
|table\_name|STRING|The name of the table.|
|table\_type|STRING|The type of the table. Valid values:-   MANAGED\_TABLE
-   VIRTUAL\_VIEW
-   EXTERNAL\_TABLE |
|is\_partitioned|BOOLEAN|Specifies whether the table is a partitioned table.|
|owner\_id|STRING|The ID of the table owner.|
|owner\_name|STRING|Optional. The Alibaba Cloud account of the table owner.|
|create\_time|DATETIME|The time when the table was created.|
|last\_modified\_time|DATETIME|The last time when the table was modified.|
|data\_length|BIGINT|The size of data in the table. Unit: bytes.|
|table\_comment|STRING|The comments on the table.|
|life\_cycle|BIGINT|Optional. The lifecycle of the table.|
|is\_archived|BOOLEAN|Specifies whether to archive data.|
|table\_exstore\_type|STRING|Optional. Specifies whether the table is a logical or physical table of the extreme storage table. Valid values: EXSTORE\_TABLE\_VIRTUAL and EXSTORE\_TABLE\_PHYSICAL.|
|cluster\_type|STRING|The clustering type of the table. Valid values: HASH and RANGE.|
|number\_buckets|BIGINT|Optional. The number of buckets in the clustering table. The value 0 indicates that the number dynamically changes during job execution.|
|view\_original\_text|STRING|The view text in the table if the table is of the VIRTUAL\_VIEW type.|

## PARTITIONS

Displays information about a table partition in a project.

|Parameter|Type|Description|
|---------|----|-----------|
|table\_catalog|STRING|Set the value to `odps`.|
|table\_schema|STRING|The name of the project.|
|table\_name|STRING|The name of the table.|
|partition\_name|STRING|The name of the partition. Example: `ds='20190130'`.|
|create\_time|DATETIME|The time when the partition was created.|
|last\_modified\_time|DATETIME|The last time when the table was modified.|
|data\_length|BIGINT|N/A.|
|is\_archived|BOOLEAN|Specifies whether to archive data.|
|is\_exstore|BOOLEAN|Specifies whether the partition is an extreme storage partition. If it is an extreme storage partition, data is stored in physical partitions.|
|cluster\_type|STRING|Optional. The clustering type of the table. Valid values: HASH and RANGE.|
|number\_buckets|BIGINT|Optional. The number of buckets in the clustering table. The value 0 indicates that the number dynamically changes during job execution.|

## COLUMNS

Displays information about a table column in a project.

|Parameter|Type|Description|
|---------|----|-----------|
|table\_catalog|STRING|Set the value to `odps`.|
|table\_schema|STRING|The name of the project.|
|table\_name|STRING|The name of the table.|
|column\_name|STRING|The name of the column.|
|ordinal\_position|BIGINT|The serial number of the column.|
|column\_default|STRING|The default value of the column.|
|is\_nullable|STRING|Optional. Set the value to YES.|
|data\_type|STRING|The data type of the column.|
|column\_comment|STRING|The comments on the column.|
|is\_partition\_key|BOOLEAN|Specifies whether the column is a partition key.|

## UDFS

Displays information about a user-defined function \(UDF\) in a project.

|Parameter|Type|Description|
|---------|----|-----------|
|udf\_catalog|STRING|Set the value to `odps`.|
|udf\_schema|STRING|The name of the project.|
|udf\_name|STRING|The name of the UDF.|
|owner\_id|STRING|The ID of the UDF owner.|
|owner\_name|STRING|Optional. The Alibaba Cloud account of the UDF owner.|
|create\_time|DATETIME|The time when the UDF was created.|
|last\_modified\_time|DATETIME|The last time when the UDF was modified.|

## RESOURCES

Displays information about a resource in a project.

|Parameter|Type|Description|
|---------|----|-----------|
|resource\_catalog|STRING|Set the value to `odps`.|
|resource\_schema|STRING|The name of the project.|
|resource\_name|STRING|The name of the resource.|
|resource\_type|STRING|The type of the resource. Valid values: Py and Jar.|
|owner\_id|STRING|The ID of the resource owner.|
|owner\_name|STRING|Optional. The Alibaba Cloud account of the resource owner.|
|create\_time|DATETIME|The time when the resource was created.|
|last\_modified\_time|DATETIME|The last time when the resource was modified.|
|size|BIGINT|The storage space used by the resource.|
|comment|STRING|The comments on the resource.|
|is\_temp\_resource|BOOLEAN|Specifies whether the resource is a temporary resource.|

## UDF\_RESOURCES

Displays information about a dependent resource of a UDF in a project.

|Parameter|Type|Description|
|---------|----|-----------|
|udf\_catalog|STRING|Set the value to `odps`.|
|udf\_schema|STRING|The name of the project to which the UDF belongs.|
|udf\_name|STRING|The name of the UDF.|
|resource\_schema|STRING|The name of the project to which the dependent resource belongs.|
|resource\_name|STRING|The name of the dependent resource.|

## USERS

Displays information about a user in a project.

|Parameter|Type|Description|
|---------|----|-----------|
|user\_catalog|STRING|Valid values: ALIYUN and RAM.|
|user\_schema|STRING|The name of the project.|
|user\_name|STRING|Optional. The name of the user.|
|user\_id|STRING|The ID of the user.|
|user\_label|STRING|The label of the user.|

## ROLES

Displays information about a role in a project.

|Parameter|Type|Description|
|---------|----|-----------|
|role\_catalog|STRING|Set the value to `odps`.|
|role\_schema|STRING|The name of the project.|
|role\_name|STRING|The name of the role.|
|role\_label|STRING|The label of the role.|
|comment|STRING|The comments on the role.|

## USER\_ROLES

Displays information about a role that a user assumes in a project.

|Parameter|Type|Description|
|---------|----|-----------|
|user\_role\_catalog|STRING|Set the value to `odps`.|
|user\_role\_schema|STRING|The name of the project.|
|role\_name|STRING|The name of the role.|
|user\_name|STRING|The name of the user.|
|user\_id|STRING|The ID of the user.|

## PACKAGE\_OBJECTS

Displays information about a package object in a project.

|Parameter|Type|Description|
|---------|----|-----------|
|package\_catalog|STRING|Set the value to `odps`.|
|package\_schema|STRING|The name of the project.|
|package\_name|STRING|The name of the package.|
|object\_type|STRING|The type of the package object.|
|object\_name|STRING|The name of the package object.|
|column\_name|STRING|The name of the table column.|
|allowed\_privileges|VECTOR<STRING\>|The shared permissions.|
|allowed\_label|STRING|The shared label.|

## INSTALLED\_PACKAGE

Displays information about an installed package in a project.

|Parameter|Type|Description|
|---------|----|-----------|
|installed\_package\_catalog|STRING|Set the value to `odps`.|
|installed\_package\_schema|STRING|The name of the project in which the package was installed.|
|package\_project|STRING|The name of the project in which the package was created.|
|package\_name|STRING|The name of the package.|
|installed\_time|DATETIME|Reserved. The time when the package was installed.|
|allowed\_label|STRING|The shared label.|

## SCHEMA\_PRIVILEGES

Displays information about a schema permission in a project.

|Parameter|Type|Description|
|---------|----|-----------|
|user\_catalog|STRING|Set the value to `odps`.|
|user\_schema|STRING|The name of the project.|
|grantee|STRING|The name of the user.|
|user\_id|STRING|The ID of the Alibaba Cloud account.|
|grantor|STRING|The account of the permission grantor. The current value is NULL.|
|privilege\_type|STRING|The type of the permission.|

## TABLE\_PRIVILEGES

Displays information about a table permission in a project.

|Parameter|Type|Description|
|---------|----|-----------|
|table\_catalog|STRING|Set the value to `odps`.|
|table\_schema|STRING|The name of the project to which the table belongs.|
|table\_name|STRING|The name of the table.|
|grantee|STRING|The name of the user.|
|user\_id|STRING|The ID of the Alibaba Cloud account.|
|grantor|STRING|The account of the permission grantor. The current value is NULL.|
|privilege\_type|STRING|The type of the permission.|
|user\_schema|STRING|The name of the project to which the user belongs.|

## COLUMN\_PRIVILEGES

Displays information about a column permission in a project.

|Parameter|Type|Description|
|---------|----|-----------|
|table\_catalog|STRING|Set the value to `odps`.|
|table\_schema|STRING|The name of the project to which the table belongs.|
|table\_name|STRING|The name of the table.|
|column\_name|STRING|The name of the column.|
|grantee|STRING|The name of the user.|
|user\_id|STRING|The ID of the Alibaba Cloud account.|
|grantor|STRING|Optional. The account of the permission grantor. The current value is NULL.|
|privilege\_type|STRING|The type of the permission.|
|user\_schema|STRING|The name of the project to which the user belongs.|

## UDF\_PRIVILEGE

Displays information about a UDF permission in a project.

|Parameter|Type|Description|
|---------|----|-----------|
|udf\_catalog|STRING|Set the value to `odps`.|
|udf\_schema|STRING|The name of the project to which the UDF belongs.|
|udf\_name|STRING|The name of the UDF.|
|user\_schema|STRING|The name of the project to which the user belongs.|
|grantee|STRING|The name of the user.|
|user\_id|STRING|The ID of the Alibaba Cloud account.|
|grantor|STRING|The account of the permission grantor. The current value is NULL.|
|privilege\_type|STRING|The type of the permission.|

## RESOURCE\_PRIVILEGES

Displays information about a resource permission in a project.

|Parameter|Type|Description|
|---------|----|-----------|
|resource\_catalog|STRING|Set the value to `odps`.|
|resource\_schema|STRING|The name of the project to which the resource belongs.|
|resource\_name|STRING|The name of the resource.|
|user\_schema|STRING|The name of the project to which the user belongs.|
|grantee|STRING|The name of the user.|
|user\_id|STRING|The ID of the Alibaba Cloud account.|
|grantor|STRING|The account of the permission grantor. The current value is NULL.|
|privilege\_type|STRING|The type of the permission.|

## TABLE\_LABELS

Displays information about a table label in a project.

|Parameter|Type|Description|
|---------|----|-----------|
|table\_catalog|STRING|Set the value to `odps`.|
|table\_schema|STRING|The name of the project.|
|table\_name|STRING|The name of the table.|
|label\_type|STRING|The type of the label. Set the value to NULL.|
|label\_level|STRING|The level of the label.|

## COLUMN\_LABELS

Displays information about a table column label in a project.

|Parameter|Type|Description|
|---------|----|-----------|
|table\_catalog|STRING|Set the value to `odps`.|
|table\_schema|STRING|The name of the project.|
|table\_name|STRING|The name of the table.|
|column\_name|STRING|The name of the column.|
|label\_type|STRING|The type of the label. Set the value to NULL.|
|label\_level|STRING|The level of the label.|

## TABLE\_LABEL\_GRANTS

Displays authorization information of a table label in a project.

|Parameter|Type|Description|
|---------|----|-----------|
|table\_label\_grant\_catalog|STRING|Set the value to `odps`.|
|table\_label\_grant\_schema|STRING|The name of the project to which the user belongs.|
|user|STRING|The name of the user.|
|user\_id|STRING|The ID of the user.|
|table\_schema|STRING|The name of the project to which the table belongs.|
|table\_name|STRING|The name of the table.|
|grantor|STRING|The account of the permission grantor. The current value is NULL.|
|label\_level|STRING|The granted level of the label.|
|expired|DATETIME|The time when the authorization expires.|

## COLUMN\_LABEL\_GRANTS

Displays the authorization information of a table column label in a project.

|Parameter|Type|Description|
|---------|----|-----------|
|column\_label\_grant\_catalog|STRING|Set the value to `odps`.|
|column\_label\_grant\_schema|STRING|The name of the project to which the user belongs.|
|user|STRING|The name of the user.|
|user\_id|STRING|The ID of the user.|
|table\_schema|STRING|The name of the project to which the table belongs.|
|table\_name|STRING|The name of the table.|
|column\_name|STRING|The name of the column.|
|grantor|STRING|The account of the permission grantor. The current value is NULL.|
|label\_level|STRING|The granted level of the label.|
|expired|DATETIME|The time when the authorization expires.|

## TASKS\_HISTORY

Displays the job execution history in a MaxCompute project. Data from the last 14 days is retained.

|Parameter|Type|Description|
|---------|----|-----------|
|task\_catalog|STRING|Set the value to `odps`.|
|task\_schema|STRING|The name of the project.|
|task\_name|STRING|The name of the job.|
|task\_type|STRING|The type of the job. Valid values: SQL, MAPREDUCE, and GRAPH.|
|inst\_id|STRING|The ID of the instance.|
|status|STRING|The running state of the job at the time when data is collected. This is not a real-time state.|
|owner\_id|STRING|The ID of the Alibaba Cloud account.|
|owner\_name|STRING|The name of the Alibaba Cloud account.|
|result|STRING|The error information displayed if an error occurs in an SQL job.|
|start\_time|DATETIME|The start time of the job.|
|end\_time|DATETIME|The end time of the job. If the job has not ended on the current day, this value is NULL.|
|input\_records|BIGINT|The number of records read by the job.|
|output\_records|BIGINT|The number of records generated by the job.|
|input\_bytes|BIGINT|The actual amount of data scanned, which is the same as that of Logview.|
|output\_bytes|BIGINT|The number of output bytes.|
|input\_tables|STRING|The job input tables in the \[project.table1,project.table2\] format.|
|output\_tables|STRING|The job output tables in the \[project.table1,project.table2\] format.|
|operation\_text|STRING|source\_xml of the query statement. If the source\_xml value exceeds 256 KB, set the value to NULL.|
|signature|STRING|Optional. The job signature.|
|complexity|DOUBLE|Optional. The job complexity. This parameter is available only for SQL jobs.|
|cost\_cpu|DOUBLE|The CPU utilization of the job. The value 100 indicates 1 CPU core per second. For example, if 10 CPU cores run for five seconds, cost\_cpu is 5000, which is calculated by using the following formula: 10 × 100 × 5.|
|cost\_mem|DOUBLE|The memory consumed by the job. Unit: MB/s.|
|settings|STRING|The information that is scheduled by the upper layer or passed by users. The information is saved in JSON format. The information includes the useragent, bizid, skynet\_id, and skynet\_nodename fields.|
|ds|STRING|The data collection date. Example: 20190101.|

## TUNNELS\_HISTORY

Displays historical data uploaded and downloaded in batches over a data tunnel. Data from the last 14 days is retained.

|Parameter|Type|Description|
|---------|----|-----------|
|tunnel\_catalog|STRING|Set the value to `odps`.|
|tunnel\_schema|STRING|The name of the project.|
|session\_id|STRING|The session ID, which is saved in the format of `TIMESTAMP (YYYYMMDDHHmmss, 14 characters) + IP address (8 characters) + numHex (8 characters)`. Example: 2013060414484474e5e60a00000002.|
|operate\_type|STRING|The type of the operation. Valid values:-   UPLOADLOG
-   DOWNLOADLOG
-   FILEDOWNLOADLOG
-   FILEUPLOADLOGDLOG
-   FILEUPLOADLOG |
|tunnel\_type|STRING|The type of the tunnel. Valid values: TUNNEL LOG and TUNNEL FILE LOG.|
|request\_id|STRING|The ID of the request.|
|object\_type|STRING|The type of the object on which the operation was performed. Valid values: TABLE and INSTANCE.|
|object\_name|STRING|The table name or instance ID.|
|partition\_spec|STRING|The name of the partition field. Example: time = 20130222, loc = beijing.|
|data\_size|BIGINT|The number of bytes of data. Unit: bytes. This parameter is valid only when the operation type is UPLOADLOG, DOWNLOADLOG, or FILEDOWNLOADLOG. Otherwise, this parameter is left empty.|
|block\_id|BIGINT|The ID of the block uploaded by the tunnel. This parameter is valid only when the operation type is UPLOADLOG, FILEUPLOADLOGDLOG, or FILEUPLOADLOG. Otherwise, this parameter is left empty.|
|offset|BIGINT|The number of records to skip before data is downloaded. The download starts from record 0 by default.|
|length|BIGINT|The number of records to download or upload in the current session. The number of records to download is the specified number of rows.|
|owner\_id|STRING|N/A.|
|owner\_name|STRING|Optional.|
|start\_time|DATETIME|N/A.|
|end\_time|DATETIME|N/A.|
|client\_ip|STRING|N/A.|
|user\_agent|STRING|Optional.|
|ds|STRING|The data collection date. Example: 20190101.|

