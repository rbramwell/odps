# 开发Java UDF（可选） {#concept_edn_3yy_5db .concept}

MaxCompute的UDF包括UDF、UDAF和UDTF三种函数。通常情况下，这三种函数被统称为UDF。

**说明：** 当前MaxCompute已支持[Java UDF](../../../../intl.zh-CN/开发/SQL及函数/UDF/Java UDF.md#)、[Python UDF](../../../../intl.zh-CN/开发/SQL及函数/UDF/Python UDF.md#)、[UDJ](../../../../intl.zh-CN/开发/SQL及函数/UDJ.md#)、[UDT](../../../../intl.zh-CN/开发/SQL及函数/UDT.md#)，详细信息可参见[Java UDF](../../../../intl.zh-CN/开发/SQL及函数/UDF/Java UDF.md#)。

如果您使用Maven实现Java UDF，可以从[Maven库](http://search.maven.org/)中搜索odps-sdk-udf获取不同版本的Java SDK。例如，使用以下配置添加指定版本的Java SDK依赖。

``` {#codeblock_uui_k9r_xs1}
<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>odps-sdk-udf</artifactId>
    <version>0.20.7</version>
</dependency>
```

通常，Java UDF的开发可以通过以下几种方式实现：

-   使用[MaxCompute Studio完成Java UDF开发整个流程](../../../../intl.zh-CN/工具及下载/MaxCompute Studio/开发Java程序/开发和调试UDF.md)。
-   使用[Eclipse插件开发和调试Java UDF](../../../../intl.zh-CN/工具及下载/Eclipse开发插件/UDF开发插件介绍.md)，导出Jar包，然后通过命令或者DataWorks[添加资源](../../../../intl.zh-CN/开发/常用命令/资源操作.md)后再[注册函数](../../../../intl.zh-CN/开发/常用命令/函数操作.md)。

本文中分别提供UDF、UDAF、UDTF的代码示例，并通过提供两种开发UDF完整流程的步骤示例。UDAF、UDTF与UDF的操作步骤一致。

**说明：** 

-   关于自定义函数注册、注销、查看函数列表的相关命令请参见[函数操作](../../../../intl.zh-CN/开发/常用命令/函数操作.md)。
-   Java和MaxCompute的数据类型对应关系，请参见[参数与返回值类型](../../../../intl.zh-CN/开发/SQL及函数/UDF/Java UDF.md)。

## UDF示例 {#section_q5g_1zy_5db .section}

下面将为您介绍使用MaxCompute Studio或Eclipse开发字符小写转换功能的UDF实现示例。

-   **使用MaxCompute Studio开发** 
    1.  **准备工具环境并创建Java Module** 

        假设已经完成环境准备，包括[安装Studio](../../../../intl.zh-CN/工具及下载/MaxCompute Studio/工具安装与版本信息/IntelliJ IDEA安装步骤.md)并在Studio上[创建MaxCompute项目链接](../../../../intl.zh-CN/工具及下载/MaxCompute Studio/项目空间连接管理.md)以及[创建MaxCompute Java Module](../../../../intl.zh-CN/工具及下载/MaxCompute Studio/开发Java程序/创建MaxCompute Java Module.md)。

    2.  **编写代码** 

        在配置好的Java Module下创建Java文件。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11953/15639390111573_zh-CN.png)

        直接选择MaxCompute Java，然后在name一栏里输入`package名称.文件名`，Kind选择UDF。 之后编辑如下代码：

        ``` {#codeblock_b4t_np2_2ln}
        package <package名称>;
        import com.aliyun.odps.udf.UDF;
        public final class Lower extends UDF {
            public String evaluate(String s) {
                if (s == null) { 
                   return null; 
                }
                   return s.toLowerCase();
            }
        }
        ```

        **说明：** 若需本地调试Java UDF，请参见[开发和调试UDF](../../../../intl.zh-CN/工具及下载/MaxCompute Studio/开发Java程序/开发和调试UDF.md)。

    3.  **注册MaxCompute UDF** 

        如下图所示，右键单击UDF的Java文件，选择**Deploy to server**，弹框里选择注册到哪个`MaxCompute project`，输入`function name`，`Resource name`也可以修改。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11953/15639390121574_zh-CN.png)

        填写好后，单击**OK**即可。注册成功后会有提示。

    4.  **试用UDF** 

        打开SQL脚本，执行代码如`select Lower_test(‘ABC’);`。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11953/15639390121575_zh-CN.png)

        **说明：** Studio中编写SQL脚本请参见[编写SQL脚本](../../../../intl.zh-CN/工具及下载/MaxCompute Studio/开发SQL程序/编写SQL脚本.md)。

-   **使用Eclipse插件开发** 
    1.  **创建工程** 

        请首先在Eclipse插件创建一个MaxCompute（原名ODPS）工程，详情请参见[创建MaxCompute工程](../../../../intl.zh-CN/工具及下载/Eclipse开发插件/创建MaxCompute工程.md)。

    2.  **编写代码** 

        按照MaxCompute UDF框架的规定，实现函数功能，并进行编译。

        ``` {#codeblock_0vp_4v7_sia}
        package <package名称>;
        import com.aliyun.odps.udf.UDF;
        public final class Lower extends UDF {
            public String evaluate(String s) {
                if (s == null) { 
                    return null; 
                }
                    return s.toLowerCase();
            }
        }
        ```

        将这个Jar包命名为**my\_lower.jar**。

        **说明：** 

        -   更详细的开发调试代码的介绍请参见[UDF开发插件介绍](../../../../intl.zh-CN/工具及下载/Eclipse开发插件/UDF开发插件介绍.md)。
        -   SDK的使用信息请参见[UDF SDK](../../../../intl.zh-CN/开发/SQL及函数/UDF/UDF概述.md) 。
    3.  **添加资源** 

        在运行UDF函数前，必须在MaxCompute中指定引用的UDF代码。您需要将编写的Java UDF编译成Jar包，并以Jar资源的形式添加至MaxCompute中。MaxCompute的UDF框架将自动加载所添加的Jar包，从而运行您自定义的UDF函数。

        **说明：** MaxCompute MapReduce也用到了资源这一特有概念，[MapReduce](../../../../intl.zh-CN/开发/MapReduce/概要/MapReduce概述.md)文档中对资源的使用也有阐述。

        例如，执行以下命令在MaxCompute中添加上一步骤中编译完成的UDF函数Jar包。

        ``` {#codeblock_8w2_k1j_tzt}
        add jar my_lower.jar;
        -- 如果存在同名的资源请将这个jar包重命名
        -- 并注意修改下面示例命令中相关jar包的名字
        -- 又或者直接使用-f选项覆盖原有的jar资源
        ```

    4.  **注册UDF函数** 

        在MaxCompute中添加Jar资源后，MaxCompute可以自动获取该UDF函数的代码。但在MaxCompute中使用该UDF函数前，您还需要注册该UDF函数，指定该函数名与jar资源的对应关系。

        **命令格式** 

        ``` {#codeblock_38m_bdv_uv9}
        CREATE FUNCTION <function_name> AS <package_to_class> USING <resource_list>;
        ```

         **参数说明：** 

        -   **function\_name**：UDF函数名，这个名字就是SQL中引用该函数所使用的名字。
        -   **package\_to\_class**：如果是Java UDF，这个名字就是从顶层包名一直到实现UDF类名的fully qualified class name。如果是python UDF，这个名字就是python脚本名.类名。并且这个名字必须使用引号。
        -   **resource\_list**：UDF所用到的资源列表。
            -   此资源列表必须包括UDF代码所在的资源。
            -   如果您的代码中通过distributed cache接口读取资源文件，此列表中还要包括UDF所读取的资源文件列表。
            -   资源列表由多个资源名组成，资源名之间由逗号分隔，且资源列表必须用引号引起来。
            -   如果需要指定资源所在的project，写法为`<project_name>/resources/<resource_name>`。
        例如，执行以下命令注册test\_lower函数并将其与my\_lower.jar资源关联。

        ``` {#codeblock_ki6_5sn_4n7}
        CREATE FUNCTION test_lower AS 'org.alidata.odps.udf.examples.Lower' USING 'my_lower.jar';
        ```

        **说明：** 

        -   与资源文件一样，同名函数只能注册一次。
        -   通常，您的自建函数无法覆盖系统内建函数。只有项目空间的Owner才有权利覆盖内建函数。如果您使用了覆盖内建函数的自定义函数，在SQL执行结束后，会在Summary中打印出Warning信息。
    5.  在SQL中使用此函数进行验证。

        ``` {#codeblock_xy6_4bc_389}
        select test_lower('A') from my_test_table;
        ```


## UDAF示例 {#section_zvj_3bz_5db .section}

UDAF的注册方式与UDF基本相同，使用方式与内建函数中的[聚合函数](../../../../intl.zh-CN/开发/SQL及函数/内建函数/聚合函数.md)相同。计算平均值的UDAF的代码示例如下所示。

``` {#codeblock_d42_5sp_ngv .language-java}
package org.alidata.odps.udf.examples;
import com.aliyun.odps.io.LongWritable;
import com.aliyun.odps.io.Text;
import com.aliyun.odps.io.Writable;
import com.aliyun.odps.udf.Aggregator;
import com.aliyun.odps.udf.UDFException;
/**
 * project: example_project
 * table: wc_in2
 * partitions: p2=1,p1=2
 * columns: colc,colb,cola
 */
public class UDAFExample extends Aggregator {
  @Override
  public void iterate(Writable arg0, Writable[] arg1) throws UDFException {
    LongWritable result = (LongWritable) arg0;
    for (Writable item : arg1) {
      Text txt = (Text) item;
      result.set(result.get() + txt.getLength());
    }
  }
  @Override
  public void merge(Writable arg0, Writable arg1) throws UDFException {
    LongWritable result = (LongWritable) arg0;
    LongWritable partial = (LongWritable) arg1;
    result.set(result.get() + partial.get());
  }
  @Override
  public Writable newBuffer() {
    return new LongWritable(0L);
  }
  @Override
  public Writable terminate(Writable arg0) throws UDFException {
    return arg0;
  }
}
```

## UDTF示例 {#section_a1j_lbz_5db .section}

UDTF的注册和使用方式与UDF相同，代码示例如下。

``` {#codeblock_x22_ono_p8l}
package org.alidata.odps.udtf.examples;
import com.aliyun.odps.udf.UDTF;
import com.aliyun.odps.udf.UDTFCollector;
import com.aliyun.odps.udf.annotation.Resolve;
import com.aliyun.odps.udf.UDFException;
// TODO define input and output types, e.g., "string,string->string,bigint".
@Resolve({"string,bigint->string,bigint"})
public class MyUDTF extends UDTF {
  @Override
  public void process(Object[] args) throws UDFException {
    String a = (String) args[0];
    Long b = (Long) args[1];
    for (String t: a.split("\\s+")) {
      forward(t, b);
    }
  }
}
```

