# SELECT TRANSFORM语法 {#concept_a45_b1c_wdb .concept}

SELECT TRANSFORM功能允许您指定启动一个子进程，将输入数据按照一定的格式通过stdin输入子进程，并且通过parse子进程的stdout输出获取输出数据。您无需编写UDF，便可以实现MaxCompute SQL不具备的功能。

命令格式如下。

``` {#codeblock_3ky_k3y_cxq}
SELECT TRANSFORM(arg1, arg2 ...) 
(ROW FORMAT DELIMITED (FIELDS TERMINATED BY field_delimiter (ESCAPED BY character_escape)?)? 
(LINES SEPARATED BY line_separator)? 
(NULL DEFINED AS null_value)?)?
USING 'unix_command_line' 
(RESOURCES 'res_name' （',' 'res_name'）*)? 
( AS col1, col2 ...)?
(ROW FORMAT DELIMITED (FIELDS TERMINATED BY field_delimiter (ESCAPED BY character_escape)?)? 
(LINES SEPARATED BY line_separator)? (NULL DEFINED AS null_value)?)?
```

说明：

-   `SELECT TRANSFORM`关键字可以用`MAP`或`REDUCE`关键字替换，无论使用哪个关键字，语义是完全一样的。为使语法更清晰，推荐您使用`SELECT TRANSFORM`。
-   `arg1,arg2...`是`transform`的参数，其格式和`select`子句的Item类似。默认格式下，参数的各个表达式的结果会在隐式转换成STRING后，用`\t`拼起来，输入到子进程中（该格式可以进行配置，详情请参见下文对ROW FORMAT的说明）。
-   `USING`：指定要启动的子进程的命令。

    **说明：** 

    -   大多数的MaxCompute SQL命令Using子句指定的是资源（Resources），但此处是为了和Hive的语法兼容。
    -   Using中的格式和Shell的语法非常类似，但并非真的启动Shell来执行，而是直接根据命令的内容来创建了子进程。因此，很多Shell的功能不能使用，例如输入输出重定向、管道、循环等。若有需要，Shell本身也可以作为子进程命令来使用。
-   `RESOURCES`子句：允许指定子进程能够访问的资源，支持以下两种方式指定资源。
    -   支持使用`resources`子句，例如`using 'sh foo.sh bar.txt' resources 'foo.sh','bar.txt'`。
    -   支持在SQL语句前使用`set odps.sql.session.resources=foo.sh,bar.txt;`来指定。注意这种配置是全局的，意味着整个SQL中所有的`select transform`都可以访问这个设置配置的资源。
-   `ROW FORMAT`子句：允许自定义输入输出的格式。

    语法中有两个Row Format子句，第一个子句指定输入的格式，第二个指定输出的格式。 默认情况下使用`\t`来作为列的分隔符，`\n`作为行的分隔符，使用`\N`（注意是两个字符，反斜杠字符和字符N）表示NULL。

    **说明：** 

    -   `field_delimiter`、`character_escape`和`line_separator`只接受一个字符。如果指定的是字符串，则以第一个字符为准。
    -   MaxCompute支持Hive指定格式的语法，例如`inputRecordReader`、`outputRecordReader`、`SerDe`等，但您需要打开Hive兼容模式才能使用。打开方式为在SQL语句前加set语句`set odps.sql.hive.compatible=true;`。Hive支持的语法详情请参见[Hive文档](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Transform#LanguageManualTransform-TRANSFORMExamples)。
    -   如果使用Hive的`inputRecordReader`、`outputRecordReader`等自定义类，可能会降低执行性能。
-   `AS`子句：指定输出列。
    -   输出列可以不指定类型，默认为STRING类型，例如`as(col1, col2)`。您也可以指定类型，例如`as(col1:bigint, col2:boolean)`。
    -   由于输出实际是`parse`子进程`stdout`获取的，如果指定的类型不是STRING，系统会隐式调用`cast`函数，而`cast`有可能出现Runtime Exception。
    -   输出列类型不支持部分指定部分不指定，例如`as(col1, col2:bigint)`。
    -   `as`可以省略，此时默认`stdout`的输出中第一个`\t`之前的字段为Key，后面的部分全部为Value，相当于`as(key, value)`。

## 调用Shell脚本示例 {#section_q4t_mbc_wdb .section}

假设通过Shell脚本生成50行数据，值是从1到50，对应`data`字段输出如下。

``` {#codeblock_o5j_flk_h6m}
SELECT TRANSFORM(script) USING 'sh' AS (data) 
FROM (
        SELECT  'for i in `seq 1 50`; do echo $i; done' AS script
      ) t
;
```

直接将Shell命令作为`transform`数据输入。

`SELECT TRANSFORM`不仅仅是语言支持的扩展。一些简单的功能，例如AWK、Python、Perl、Shell都支持直接在命令中写脚本，而不需要专门编写脚本文件、上传资源等，开发过程更简单。对于复杂的功能，您可以上传脚本文件来执行，例如下文将介绍的调用Python脚本示例。

## 调用Python脚本示例 {#section_fx2_qbc_wdb .section}

准备Python文件，脚本文件名为myplus.py，如下所示。

``` {#codeblock_3lt_duk_jrq}
#!/usr/bin/env python
import sys
line = sys.stdin.readline()
while line:
    token = line.split('\t')
    if (token[0] == '\\N') or (token[1] == '\\N'):
        print '\\N'
    else:
        print int(token[0]) + int(token[1])
    line = sys.stdin.readline()
```

将该Python脚本文件添加为MaxCompute资源（Resource）。

``` {#codeblock_fcr_148_791}
add py ./myplus.py -f;
```

**说明：** 您也可通过DataWorks控制台进行新增资源操作。

使用`select transform`语法调用资源。

``` {#codeblock_rij_7hw_k1c}
Create table testdata(c1 bigint,c2 bigint); --创建测试表。
insert into Table testdata values (1,4),(2,5),(3,6); --测试表中插入测试数据。

--接下来执行select transform。 
SELECT 
TRANSFORM (testdata.c1, testdata.c2) 
USING 'python myplus.py'resources 'myplus.py' 
AS (result bigint) 
FROM testdata;

--或执行以下语句。
set odps.sql.session.resources=myplus.py;
SELECT TRANSFORM (testdata.c1, testdata.c2) 
USING 'python myplus.py' 
AS (result bigint) 
FROM testdata;
```

执行结果如下所示。

``` {#codeblock_wqa_o42_x31}
+-----+
| cnt |
+-----+
| 5   |
| 7   |
| 9   |
+-----+
```

Python脚本无需依赖MaxCompute的Python框架，也没有格式要求。

MaxCompute也支持直接将Python命令作为`transform`数据输入。例如调用Shell脚本示例也可以用Python命令实现。

``` {#codeblock_gec_r8i_dnk}
SELECT TRANSFORM('for i in xrange(1, 50):  print i;') USING 'python' AS (data);
```

## 调用Java脚本示例 {#section_dsx_ccc_wdb .section}

与之前调用Python脚本类似，在本例中，您需要编辑好Java文件，导出Jar包，再通过`add file`方式将Jar包添加为MaxCompute资源，最后用于`select transform`调用。

1.  准备好Jar文件，脚本文件名为Sum.jar。Java代码如下所示。

    ``` {#codeblock_7i3_0vh_x7h}
    package com.aliyun.odps.test;
    import java.util.Scanner
    public class Sum {
        public static void main(String[] args) {
            Scanner sc = new Scanner(System.in);
            while (sc.hasNext()) {
                String s = sc.nextLine();
                String[] tokens = s.split("\t");
                if (tokens.length < 2) {
                    throw new RuntimeException("illegal input");
                }
                if (tokens[0].equals("\\N") || tokens[1].equals("\\N")) {
                    System.out.println("\\N");
                }
                System.out.println(Long.parseLong(tokens[0]) + Long.parseLong(tokens[1]));
            }
        }
    }
    ```

2.  将Jar文件添加为MaxCompute的Resource。

    ``` {#codeblock_2jc_vbq_qmo}
    add jar ./Sum.jar -f;
    ```

3.  使用`select transform`语法调用资源。

    ``` {#codeblock_xdt_32n_xvz}
    Create table testdata(c1 bigint,c2 bigint); --创建测试表。
    insert into Table testdata values (1,4),(2,5),(3,6); --测试表中插入测试数据。
    --接下来执行Select Transform。
    SELECT TRANSFORM(testdata.c1, testdata.c2) USING 'java -cp Sum.jar com.aliyun.odps.test.Sum' resources 'Sum.jar' from testdata;
    --或执行以下语句。
    set odps.sql.session.resources=Sum.jar;
    SELECT TRANSFORM(testdata.c1, testdata.c2) USING 'java -cp Sum.jar com.aliyun.odps.test.Sum' FROM testdata;
    ```

    执行结果如下所示。

    ``` {#codeblock_q7c_qq1_2nt}
    +-----+
    | cnt |
    +-----+
    | 5   |
    | 7   |
    | 9   |
    +-----+
    ```


大多数Java的Utility可以直接使用本方法运行。

**说明：** Java和Python虽然有现成的UDTF框架，但是用`select transform`编写更简单，并且不需要额外依赖，也没有格式要求，甚至可以实现直接使用离线脚本（Java和Python离线脚本的实际路径，可以从`JAVA_HOME`和`PYTHON_HOME`环境变量中得到）。

## 调用其他脚本语言 {#section_p4m_pcc_wdb .section}

`select transform`不仅仅支持上述语言扩展，还支持其它的常用Unix命令或脚本解释器，例如AWK、Perl等。

-   调用AWK，将第二列原样输出，示例如下。

    ``` {#codeblock_g6t_61g_3sh}
    SELECT TRANSFORM(*) USING "awk '//{print $2}'" as (data) from testdata;
    ```

-   调用Perl，示例如下。

    ``` {#codeblock_308_411_m8u}
    SELECT TRANSFORM (testdata.c1, testdata.c2) USING "perl -e 'while($input = <STDIN>){print $input;}'" FROM testdata;
    ```


**说明：** 目前由于MaxCompute计算集群上未部署PHP和Ruby，所以不支持调用这两种脚本。

## 串联使用示例 {#section_sfq_ejl_zo1 .section}

`select transform`还可以串联使用。例如使用`distribute by`和`sort by`对输入数据做预处理。

``` {#codeblock_64v_bqg_m3o}
SELECT TRANSFORM(key, value) USING 'cmd2' from 
(
    SELECT TRANSFORM(*) USINg 'cmd1' from 
    (
        SELECt * FROM data distribute by col2 sort by col1
    ) t distribute by key sort by value
) t2;
```

或使用`map`、`reduce`关键字。

``` {#codeblock_0ji_g0d_rag}
@a := select * from data distribute by col2 sort by col1;
@b := map * using 'cmd1' distribute by col1 sort by col2 from @a;
reduce * using 'cmd2' from @b;
```

## SELECT TRANSFORM性能介绍 {#section_ymd_1dc_wdb .section}

`select transform`与UDTF在不同场景下性能不同。经过多种场景对比测试，数据量较小时，大多数场景下`select transform`有优势，而数据量大时UDTF有优势。因为`select transform`的开发更加简便，所以`select transform`更适合做Ad Hoc的数据分析。

UDTF的优势

-   UDTF的输出结果和输入参数是有类型的，而`transform`的子进程基于`stdin`和`stdout`传输数据，所有数据都当做STRING处理，因此`select transform`多了一步类型转换。
-   `select transform`数据传输依赖于操作系统的管道，而目前管道的Buffer仅有4KB，且不能设置。`select transform`读空或Pipe写满会导致进程被挂起。
-   UDTF的常量参数可以不用传输，而`select transform`无法利用这个优化。

`select transform`的优势

-   子进程和父进程是两个进程，而UDTF是单线程的。如果计算占比较高，数据吞吐量较小，`select transform`可以利用服务器的多核特性。
-   数据的传输通过更底层的系统调用来读写，效率比Java高。
-   `select transform`支持的部分工具，例如AWK是Native代码实现的。理论上，与Java相比，使用`select transform`会更有性能优势。

