# SQL Common Errors {#concept_c4b_42w_tdb .concept}

|Error message|Trigger condition|
|:------------|:----------------|
|ODPS-0110005:Unknown exception from metadata operation|Catch all unknown exceptions on the top layer.|
|ODPS-0110011:Authorization exception|You are unauthorized.|
|ODPS-0110021:Invalid parameters|The parameter is invalid.|
|ODPS-0110031:Invalid object type|The object type is invalid.|
|ODPS-0110041:Invalid meta operation - AlreadyExistsException\(message:Partition already exists, existed values|Currently, MaxCompute does not lock the operated table. This error is caused by competition between META tags, that is, multiple read/write actions on the same partition at the same time. We recommend that you do not perform multiple actions on the same table at the same time when the lock mechanism of MaxCompute is not enabled.|
|ODPS-0110061: Failed to run ddltask - AlreadyExistsException\(message:Partition already exists, existed values\).|Currently, MaxCompute does not lock the operated table. This error is caused by competition between META tags, that is, multiple read/write actions on the same partition at the same time. We recommend that you do not perform multiple actions on the same table at the same time when the lock mechanism of MaxCompute is not enabled.|
|ODPS-0110061: Failed to run ddltask - SimpleLock conflict failure, add partition is already on-going|This error occurs when you add the same partition in batches. MaxCompute runs only the first command to add a partition and ignores the subsequent ones.|
|ODPS-0110071：OTS initialization exception|Table Store initialization is abnormal.|
|ODPS-0110081：OTS transaction exception|Table Store transaction is abnormal.|
|ODPS-0110091：OTS filtering exception|The conditional filtering of Table Store is abnormal.|
|ODPS-01100101：OTS processing exception|Table Store execution is abnormal.|
|ODPS-01100111：OTS invalid data object|Table Store data is faulty.|
|ODPS-01100121： Unknown OTS exception|An unknown Table Store error occurs.|
|Fig: compression exception|Storage descriptor compression is abnormal.|
|ODPS-0110141:Data version exception|The data version is abnormal.|
|ODPS-0110999:Critical! Internal error happened in commit operation and rollback failed, possible breach of atomicity|Roll-back fails.|
|ODPS-0120005:Unknown exception from processor|Catch all unknown exceptions on the top layer.|
|ODPS-0120011:Authorization exception|You are unauthorized.|
|ODPS-0120021:the delimitor must be the same in wm\_concat|The separators in the same group are different.|
|ODPS-0120031: Instance has been canceled|The instance is canceled for multiple times.|
|ODPS-0121011:Invalid regular expression pattern|The regular expression processing function of the [built-in functions](../../../../reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md) receive an unrecognizable regular expression.|
|ODPS-0121021:Regexec call failed|An error occurs when the regular expression is matched.|
|ODPS-0120031：Instance has been cancelled|The instance is canceled for multiple times.|
|ODPS-0121035:Illegal implicit type cast| The type conversion is invalid. Generally, the error is caused by implicit conversions of unsupported types, which is against the rules of implicit type conversion. [Built-in functions](../../../../reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md) also have different implicit conversion rules.

 |
|ODPS-0121045:Unsupported return type|The return value is unsupported.|
|ODPS-0121055:Empty argument value|The parameter value is an empty string or null.|
|ODPS-0121065:Argument value out of range|The parameter value is out of the range.|
|ODPS-0121075:Invalid number of arguments|The number of parameters is invalid.|
|ODPS-0121081:Illegal argument type|The basic type of the parameter is invalid.|
|ODPS-0121095:Invalid arguments|Another parameter error occurs.|
|ODPS-0121105:Constant argument value expected|Constant must be input, but columns names are input.|
|ODPS-0121115:Column reference expected|Column names must be input, but constants are input.|
|ODPS-0121125:Unsupported function or operation|The UDF or other action is unsupported.|
|ODPS-0121135:Malloc memory failed|The memory allocation is abnormal.|
|ODPS-0121145:Data overflow|The data overflows. The value is out of the [value range of the data type](../../../../reseller.en-US/User Guide/Data types.md). This error is often caused by a division by zero or aggregate function. For example, sum causes data overflow.|
|ODPS-0123019:Distributed file operation exception|The disk read/write operation is abnormal.|
|ODPS-0123023:Unsupported reduce type|The reduce type is unsupported.|
|ODPS-0123031:Partition exception|A partition exception occurs.|
|ODPS-0123043:buffer overflow|The cache overflows.|
|Fig: script exception|The script is abnormal.|
|ODPS-0123065:Join exception|A Join exception occurs.|
|ODPS-0123075:Hash exception|A hash exception occurs.|
|ODPS-0123081:Invalid datetime string|The datetime string is abnormal.|
|ODPS-0123091:Illegal type cast| The type conversion is invalid. This error is often caused by explicit conversions.

 |
|ODPS-0123105： Job got killed|The job is killed.|
|ODPS-0123111:Format string does not match datetime string|The format string does not match the date string. The format of the date manually input in the SQL statement does not meet the MaxCompute format requirements, or DataTime built-in functions are improperly used. For the display formats of the DateTime type in MaxCompute, see examples in [Data types](../../../../reseller.en-US/User Guide/Data types.md) or introduction on date functions in [Built-in functions](../../../../reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md).|
|ODPS-0123121:Mapjoin exception|A MapJoin exception occurs. This error often appears because the size of the MapJoin small table exceeds the upper limit, which is 512 MB.|
|ODPS-0123131:User defined function exception|The UDF is abnormal.|
|ODPS-0123141： Exstore exception|The extreme storage is abnormal.|
|ODPS-0130005:Unknown exception from parser|Catch all unknown exceptions on the top layer.|
|ODPS-0130013:Authorization exception|You are unauthorized and cannot pass the security check.|
|ODPS-0130025:Failed to I/O|An input/output exception occurs.|
|ODPS-0130031:Failed to drop table|The source table does not exist when you delete the table.|
|ODPS-0130041:Statistics exception|A statistics exception occurs.|
|ODPS-0130051:Exception in sub query|A subquery exception occurs.|
|ODPS-0130061:Invalid table|The table is invalid.|
|ODPS-0130071:Semantic analysis exception - Invalid table alias or column reference|The syntax parsing is abnormal. The column name is incorrect, and the corresponding column cannot be found.|
|ODPS-0130071:Semantic analysis exception - Invalid column reference|The syntax parsing is abnormal. The column reference is incorrect, and the corresponding column cannot be found.|
|ODPS-0130071:Semantic analysis exception - Expression not in GROUP BY key|The syntax parsing is abnormal. The read columns in the Select clause are different from those in the Group By clause.|
|ODPS-0130071:Semantic analysis exception - Partition not found|The syntax parsing is abnormal. Partitions of the specified partition values cannot be found.|
|ODPS-0130071:Semantic analysis exception - SELECT DISTINCT and GROUP BY can not be in the same query|Distinct and Group By cannot be used in the same Select clause.|
|ODPS-0130071:Semantic analysis exception - Cannot insert into target table because column number/types are different|The number of type of the columns in the source table and target table is different when you insert data to the target table.|
|ODPS-0130071:Semantic analysis exception - physical plan generation failed: java.lang.RuntimeException: Table\(xxxx\) is full scan with all partitions, please specify partition predicates.|The project to which the table belongs forbids full table scan on the partition table. You must specify the partition conditions.|
|ODPS-0130071:Semantic analysis exception - xxxx type is not enabled in current mode|The new data type setting is not turned on. To use the new data types \(TINYINT, SMALLINT integer, float, VARCHAR TIMESTAMP, and BINARY\), you need to add a setting statement before the table statement. Session level:`set odps.sql.type.system.odps2=true;`

 project level:`setproject odps.sql.type.system.odps2=true;`

 |
|ODPS-0130081:Invalid UDF reference|UDF method signature.|
|ODPS-0130091:Invalid parameters|The UDF parameter is invalid.|
|ODPS-0130101:Ambiguous data type|The data type is invalid.|
|ODPS-0130111:Subquery partition pruning exception|The dynamic partition optimization is abnormal in the subquery in the IN condition determine statement.|
|ODPS-0130121:Invalid argument type|The parameter type is invalid. This error is often caused by incorrect parameter type received by the [built-in functions](../../../../reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md).|
|ODPS-0130131:Table not found|The table does not exist. When you run the DDL or DML statements, the operated table does not exist.|
|ODPS-0130141:Illegal implicit type cast|The implicit type conversion is not allowed.|
|ODPS-0130151:Illegal data type|The data type is invalid.|
|ODPS-0130161:Parse exception|The syntax parsing fails.|
|ODPS-0130171:Creating view exception|An exception occurs when you create a view.|
|ODPS-0130181:Window function exception|The window function is abnormal.|
|ODPS-0130191:Invalid column or partition key|A column or partition key is invalid.|
|ODPS-0130201:View not found|The view does not exist.|
|ODPS-0130211:Table or view already exists|The table or view already exists. This error is often caused by repeated table or view names. When you run the Create Table/View statement, the table or view to be created already exists.|
|ODPS-0130221:Invalid number of arguments|The number of parameters is invalid.|
|ODPS-0130231:Invalid view|The view status is invalid.|
|ODPS-0130241:Illegal union operation|The Union action is invalid. This error is often caused by inconsistent number or type of columns at the two sides of Union.|
|ODPS-0130252:Cartesian product is not allowed|Cartesian products are not supported. MaxCompute does not support non-equality expressions in the ON connection in the join clause.|
|ODPS-0130261:Invalid schema|The metadata is invalid.|
|ODPS-0130271:Partition does not exist|The partition does not exist.|
|ODPS-0140005:Unknown exception from planner|Catch all unknown exceptions on the top layer.|
|ODPS-0140011:Illegal type cast|The explicit type conversion is not allowed.|
|ODPS-0140021:Illegal implicit type cast|The implicit type conversion is not allowed.|
|ODPS-0140031:Invalid column reference|The column name or table name reference is invalid.|
|ODPS-0140041:Invalid UDF reference|The UDF name does not exist.|
|ODPS-0140051:Invalid function|The function is invalid.|
|ODPS-0140061:Invalid parameters|The input parameter is invalid.|
|ODPS-0140071:Unsupported operator|The operator is not supported.|
|ODPS-0140081:Unsupported join type|The small table performs the \(left\) outer join action on the big table, or the big table performs the \(right\) outer join action on the small table.|
|ODPS-0140091:Unsupported stage type|The execution plan type is not supported.|
|ODPS-0140105:Invalid multiple I/O|Multiple outputs cause conflict.|
|ODPS-0140111:Unsupported col type in EXTRACT now|Currently, EXTRACT does not support the column type.|
|ODPS-0140125:The path need to read has different storage type| |
|ODPS-0140133：Invalid structure|The data structure cannot be recognized.|
|ODPS-0140141：Can not remove operator with more than one children| |
|ODPS-0140151：Can not do topologic sort, the stages is not a DAG|The sorting algorithm is faulty.|
|ODPS-0140161：Invalid stage| |
|ODPS-0140171:Sandbox violation exception|The secure sandbox model is abnormal.|
|ODPS-0140181:Sql plan exception|The SQL job cannot generate the execution plan. If this error occurs, you can try resubmitting the job. If the job fails to be submitted for several times, you can open a ticket to contact us.|
|ODPS-0123049: buffer overflow|The memory overflows. If the same key data in the join clause is too much, and the memory is insufficient to store the data, you must check whether the data is faulty.|
|ODPS-0140178: Internal system failure|The system is abnormal. You can retry to solve the problem.|
|ODPS-0420061: Invalid parameter in HTTP request - Fetched data is larger than the rendering limitation. Please try to reduce your limit size or column number.| |

