# MapOnly示例 {#concept_wz1_4yg_vdb .concept}

对于MapOnly的作业，Map直接将<Key,Value\>信息输出到MaxCompute的表中。您只需要指定输出表即可，无需指定Map输出的Key/Value元信息。

## 测试准备 {#section_e3n_syg_vdb .section}

1.  准备好测试程序的Jar包，假设名字为mapreduce-examples.jar，本地存放路径为data\\resources。
2.  准备好MapOnly的测试表和资源。
    -   创建测试表。

        ``` {#codeblock_lil_w1j_0b4}
        create table wc_in (key string, value string);
        create table wc_out(key string, cnt bigint);
        ```

    -   添加测试资源。

        ``` {#codeblock_td2_ucy_eek}
        add jar data\resources\mapreduce-examples.jar -f;
        ```

3.  使用Tunnel导入数据。

    ``` {#codeblock_wbz_96d_gse}
    tunnel upload data wc_in;
    ```

    导入wc\_in表的数据文件data的内容。

    ``` {#codeblock_jxi_e45_vts}
     hello,odps
     hello,odps
    ```


## 测试步骤 {#section_rlv_bzg_vdb .section}

在odpscmd中执行MapOnly。

``` {#codeblock_2p6_1o0_q6j}
jar -resources mapreduce-examples.jar -classpath data\resources\mapreduce-examples.jar
com.aliyun.odps.mapred.open.example.MapOnly wc_in wc_out map
```

## 预期结果 {#section_hzz_dzg_vdb .section}

作业成功结束后，输出表wc\_out中的内容如下。

``` {#codeblock_0m8_15r_xqe}
+------------+------------+
| key        | cnt        |
+------------+------------+
| hello      | 1          |
| hello      | 1          |
+------------+------------+
```

## 代码示例 {#section_jgb_gzg_vdb .section}

``` {#codeblock_zf2_otv_16t}
package com.aliyun.odps.mapred.open.example;
import java.io.IOException;
import com.aliyun.odps.data.Record;
import com.aliyun.odps.mapred.JobClient;
import com.aliyun.odps.mapred.MapperBase;
import com.aliyun.odps.mapred.conf.JobConf;
import com.aliyun.odps.mapred.utils.SchemaUtils;
import com.aliyun.odps.mapred.utils.InputUtils;
import com.aliyun.odps.mapred.utils.OutputUtils;
import com.aliyun.odps.data.TableInfo;
public class MapOnly {
    public static class MapperClass extends MapperBase {
        @Override
            public void setup(TaskContext context) throws IOException {
            boolean is = context.getJobConf().getBoolean("option.mapper.setup", false);
            //Main函数在jobconf里设置了option.mapper.setup为true，才会执行下面的逻辑。
            if (is) {
                Record result = context.createOutputRecord();
                result.set(0, "setup");
                result.set(1, 1L);
                context.write(result);
            }
        }
        @Override
            public void map(long key, Record record, TaskContext context) throws IOException {
            boolean is = context.getJobConf().getBoolean("option.mapper.map", false);
            //Main函数在jobconf里设置了option.mapper.map为true，才会执行下面的逻辑。
            if (is) {
                Record result = context.createOutputRecord();
                result.set(0, record.get(0));
                result.set(1, 1L);
                context.write(result);
            }
        }
        @Override
            public void cleanup(TaskContext context) throws IOException {
            boolean is = context.getJobConf().getBoolean("option.mapper.cleanup", false);
            //Main函数在jobconf里设置了option.mapper.cleanup为true，才会执行下面的逻辑。
            if (is) {
                Record result = context.createOutputRecord();
                result.set(0, "cleanup");
                result.set(1, 1L);
                context.write(result);
            }
        }
    }
    public static void main(String[] args) throws Exception {
        if (args.length != 2 && args.length != 3) {
            System.err.println("Usage: OnlyMapper <in_table> <out_table> [setup|map|cleanup]");
            System.exit(2);
        }
        JobConf job = new JobConf();
        job.setMapperClass(MapperClass.class);
        //对于MapOnly的作业，必须显式设置reducer的个数为0。
        job.setNumReduceTasks(0);
        //设置输入输出的表信息。
        InputUtils.addTable(TableInfo.builder().tableName(args[0]).build(), job);
        OutputUtils.addTable(TableInfo.builder().tableName(args[1]).build(), job);
        if (args.length == 3) {
            String options = new String(args[2]);
            //jobconf中可以设置自定义的key,value值，在mapper中通过context的getJobConf可以获取到相关的设置。
            if (options.contains("setup")) {
                job.setBoolean("option.mapper.setup", true);
            }
            if (options.contains("map")) {
                job.setBoolean("option.mapper.map", true);
            }
            if (options.contains("cleanup")) {
                job.setBoolean("option.mapper.cleanup", true);
            }
        }
        JobClient.runJob(job);
    }
}
```

