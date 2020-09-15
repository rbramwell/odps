---
keyword: [复制表, CLONE TABLE]
---

# CLONE TABLE

CLONE TABLE可以复制表数据到另一个表中，极大的提高了数据迁移的效率。本文为您介绍如何使用CLONE TABLE。

## 使用限制

-   目标表与源表的Schema需要兼容。
-   支持分区表和非分区表，暂不支持对Hash Clutered Table使用CLONE TABLE命令进行复制。
-   目标表已存在时，一次性复制分区的数量上限为10000个。
-   目标表不存在时，无分区数量限制，满足原子性。
-   对同一个非分区表或者分区表的同一个分区，执行CLONE TABLE命令的次数不能超过7次。
-   不支持在跨区域的项目之间使用CLONE TABLE命令进行复制。

## 命令格式

```
CLONE TABLE <[src_project_name.]src_table_name> [PARTITION(spec), ...]
 TO <[dest_project_name.]desc_table_name> [IF EXISTS (OVERWRITE | IGNORE)] ;
```

## 命令说明

将源表src\_table\_name中的数据复制到目标表desc\_table\_name中。

**说明：** 将数据复制到目标表之后，建议您执行数据验证（例如，执行`SELECT COUNT`命令查看表的行数、执行`DESC`命令查看表大小）以确保复制后数据的准确性。

## 参数说明

-   src\_table\_name：源表名称。
-   src\_project\_name：源表所在项目空间名称。不指定时，默认为当前项目空间。
-   desc\_table\_name：目标表名称。
    -   当目标表不存在时，CLONE TABLE会创建目标表（创建目标表使用的是CREATE TABLE LIKE语义）。
    -   当目标表已存在并指定IF EXISTS OVERWRITE，则覆盖目标表对应分区的数据。
    -   当目标表已存在并指定IF EXISTS IGNORE，则跳过已存在分区，不覆盖目标表已有分区的数据。
-   dest\_project\_name：目标表所在项目空间名称。不指定时，默认为当前项目空间。

## 示例

假设源表为分区表srcpart\_copy和非分区表src\_copy，表的元数据信息如下。

```
odps@ multi>READ srcpart_copy;
+------------+------------+------------+------------+
| key        | value      | ds         | hr         |
+------------+------------+------------+------------+
| 1          | ok49       | 2008-04-09 | 11         |
| 1          | ok48       | 2008-04-08 | 12         |
+------------+------------+------------+------------+
odps@ multi>READ src_copy;
+------------+------------+
| key        | value      |
+------------+------------+
| 1          | ok         |
+------------+------------+
```

-   全量复制非分区表src\_copy的数据至目标表src\_clone。

    ```
    odps@ multi>CLONE TABLE src_copy TO src_clone;
    ID = 2019102303024544g2540cdv2
    OK
    //查看复制后目标表src_clone信息，验证数据准确性。
    odps@ multi>SELECT * FROM src_clone;
    ```

-   复制分区表srcpart\_copy指定分区的数据至目标表srcpart\_clone。

    ```
    odps@ multi>CLONE TABLE srcpart_copy PARTITION(ds="2008-04-09", hr='11') TO srcpart_clone IF EXISTS OVERWRITE;
    ID = 20191023030534986g4540cdv2
    OK
    //查看复制后目标表srcpart_clone信息，验证数据准确性。
    odps@ multi>SELECT * FROM srcpart_clone;
    ```

-   全量复制分区表srcpart\_copy的数据至目标表srcpart\_clone并跳过目标表中已存在的分区。

    ```
    odps@ multi>CLONE TABLE srcpart_copy TO srcpart_clone IF EXISTS IGNORE;
    ID = 20191023030619196g5540cdv2
    OK
    //查看复制后目标表srcpart_clone信息，验证数据准确性。
    odps@ multi>SELECT * FROM srcpart_clone;
    ```

-   全量复制分区表srcpart\_copy的数据至目标表srcpart\_clone2。

    ```
    odps@ multi>CLONE TABLE srcpart_copy TO srcpart_clone2;
    ID = 20191023030825186g6540cdv2
    OK
    //查看复制后目标表srcpart_clone2信息，验证数据准确性。
    odps@ multi>CLONE TABLE srcpart_clone2;
    ```


