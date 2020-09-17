---
keyword: [自定义Extractor, OSS非结构化数据]
---

# 自定义Extractor访问OSS

本文为您介绍如何使用自定义Extractor访问OSS非结构化数据。

## 自定义Extractor访问OSS上的文本数据

当OSS中的数据格式比较复杂，内置的Extractor无法满足需求时，需要自定义Extractor来读取OSS文件中的数据。

以TXT数据文件为例，记录之间的列通过`|`分隔。存储路径为`/demo/SampleData/CustomTxt/AmbulanceData/vehicle.csv`，数据如下。

```
1|1|51|1|46.81006|-92.08174|9/14/2014 0:00|S
1|2|13|1|46.81006|-92.08174|9/14/2014 0:00|NE
1|3|48|1|46.81006|-92.08174|9/14/2014 0:00|NE
1|4|30|1|46.81006|-92.08174|9/14/2014 0:00|W
1|5|47|1|46.81006|-92.08174|9/14/2014 0:00|S
1|6|9|1|46.81006|-92.08174|9/14/2014 0:00|S
1|7|53|1|46.81006|-92.08174|9/14/2014 0:00|N
1|8|63|1|46.81006|-92.08174|9/14/2014 0:00|SW
1|9|4|1|46.81006|-92.08174|9/14/2014 0:00|NE
1|10|31|1|46.81006|-92.08174|9/14/2014 0:00|N
```

1.  定义Extractor。

    编写一个通用的Extractor，传入分隔符参数，可以处理所有类似格式的TXT文件。如下所示。

    ```
    /**
     * Text extractor that extract schematized records from formatted plain-text(csv, tsv etc.)
     **/
    public class TextExtractor extends Extractor {
      private InputStreamSet inputs;
      private String columnDelimiter;
      private DataAttributes attributes;
      private BufferedReader currentReader;
      private boolean firstRead = true;
      public TextExtractor() {
        // default to ",", this can be overwritten if a specific delimiter is provided (via DataAttributes)
        this.columnDelimiter = ",";
      }
      // no particular usage for execution context in this example
      @Override
      public void setup(ExecutionContext ctx, InputStreamSet inputs, DataAttributes attributes) {
        this.inputs = inputs; //inputs是一个InputStreamSet，每次调用next()返回一个InputStream，这个InputStream可以读取一个OSS文件的所有内容。
        this.attributes = attributes;
        // check if "delimiter" attribute is supplied via SQL query
        String columnDelimiter = this.attributes.getValueByKey("delimiter"); //delimiter通过DDL语句传参。
        if ( columnDelimiter != null)
        {
          this.columnDelimiter = columnDelimiter;
        }
        // note: more properties can be inited from attributes if needed
      }
      @Override
      public Record extract() throws IOException {//extactor()调用返回一条Record，代表外部表中的一条记录。
        String line = readNextLine();
        if (line == null) {
          return null; // 返回NULL来表示这个表中已经没有记录可读。
        }
        return textLineToRecord(line); // textLineToRecord将一行数据按照delimiter分割为多个列。
      }
      @Override
      public void close(){
        // no-op
      }
    }
    ```

    **说明：** TextLineToRecord将数据分割的完整实现请参见[TextExtractor.java](https://github.com/aliyun/aliyun-odps-java-sdk/blob/master/odps-sdk-impl/odps-udf-example/src/main/java/com/aliyun/odps/udf/example/text/TextExtractor.java)。

2.  定义StorageHandler。

    StorageHandler是外部表自定义逻辑的统一入口。

    ```
    package com.aliyun.odps.udf.example.text;
    public class TextStorageHandler extends OdpsStorageHandler {
      @Override
      public Class<? extends Extractor> getExtractorClass() {
        return TextExtractor.class;
      }
      @Override
      public Class<? extends Outputer> getOutputerClass() {
        return TextOutputer.class;
      }
    }
    ```

3.  编译打包。

    将自定义代码编译打包，并上传到MaxCompute。

    ```
    add jar odps-udf-example.jar;
    ```

4.  创建外部表。

    与使用内置Extractor相似，首先需要创建一张外部表，不同的是在指定外部表访问数据的时候，需要使用自定义的StorageHandler。

    创建外部表语句如下，其中delimeter是您自定义的分割方法名称。

    ```
    CREATE EXTERNAL TABLE IF NOT EXISTS ambulance_data_txt_external
    (
    vehicleId int,
    recordId int,
    patientId int,
    calls int,
    locationLatitute double,
    locationLongtitue double,
    recordTime string,
    direction string
    )
    STORED BY 'com.aliyun.odps.udf.example.text.TextStorageHandler' 
      with SERDEPROPERTIES (
    'delimiter'='\\|',  
    'odps.properties.rolearn'='acs:ram::xxxxxxxxxxxxx:role/aliyunodpsdefaultrole'
    )
    LOCATION 'oss://oss-cn-shanghai-internal.aliyuncs.com/oss-odps-test/Demo/SampleData/CustomTxt/AmbulanceData/'
    USING 'odps-udf-example.jar'; 
    ```

    -   STORED BY：指定自定义StorageHandler的类名。
    -   WITH SERDEPROPERTIES：指定外表相关参数。
        -   delimiter：指定分隔符。
        -   odps.properties.rolearn：指定RAM中AliyunODPSDefaultRole的ARN信息。您可以通过RAM访问控制台中的[角色详情](https://ram.console.aliyun.com/#/role/detailAliyunODPSDefaultRole/info)获取。
    -   LOCATION：必须指定一个OSS目录，系统默认会读取这个目录下所有的文件。
        -   建议您使用OSS提供的内网域名，否则将产生OSS流量费用。
        -   建议您OSS数据存放的区域对应您开通MaxCompute的区域。由于MaxCompute只在部分区域部署，我们不承诺跨区域的数据连通性。
        -   OSS的连接格式为`oss://oss-cn-shanghai-internal.aliyuncs.com/Bucket名称/目录名称/`。目录后不要加文件名称，以下为错误用法。

            ```
            http://oss-odps-test.oss-cn-shanghai-internal.aliyuncs.com/Demo/     //不支持http连接。
            https://oss-odps-test.oss-cn-shanghai-internal.aliyuncs.com/Demo/    //不支持https连接。
            oss://oss-odps-test.oss-cn-shanghai-internal.aliyuncs.com/Demo       //连接地址错误。
            oss://oss://oss-cn-shanghai-internal.aliyuncs.com/oss-odps-test/Demo/vehicle.csv  //不必指定文件名。
            ```

    -   USING：指定类定义所在的JAR包。
5.  执行如下SQL语句查询外部表。

    ```
    SELECT recordId, patientId, direction FROM ambulance_data_txt_external WHERE patientId > 25;
    ```


## 自定义Extractor访问OSS上的非文本数据

以语音数据（WAV格式文件）为例，为您介绍如何通过自定义的Extractor访问并处理OSS上的非文本文件。

1.  执行如下SQL语句创建外部表。通过外部表的Schema定义从语音文件中抽取出如下信息：

    -   语音文件中的语句信噪比（SNR）：sentence\_snr。
    -   语音文件的名字：ID。
    ```
    CREATE EXTERNAL TABLE IF NOT EXISTS speech_sentence_snr_external
    (
    sentence_snr double,
    id string
    )
    STORED BY 'com.aliyun.odps.udf.example.speech.SpeechStorageHandler'
    WITH SERDEPROPERTIES (
        'mlfFileName'='sm_random_5_utterance.text.label' ,
        'speechSampleRateInKHz' = '16'
    )
    LOCATION 'oss://oss-cn-shanghai-internal.aliyuncs.com/oss-odps-test/dev/SpeechSentenceTest/'
    USING 'odps-udf-example.jar,sm_random_5_utterance.text.label';
    ```

    -   com.aliyun.odps.udf.example.speech.SpeechStorageHandler：封装的Extractor可计算语音文件的平均有效语句信噪比，并将抽取出来的结构化数据直接进行SQL运算。
    -   LOCATION：`oss://oss-cn-hangzhou-zmf.aliyuncs.com/oss-odps-test/dev/SpeechSentenceTest/`存储了多个原始的WAV格式的语音文件，MaxCompute框架会读取该地址上的所有文件，并按照文件级别分片，自动将文件分配给多个计算节点处理。
2.  执行如下语句查询结果。

    ```
    select sentence_snr, id from speech_sentence_snr_external where sentence_snr > 10.0; 
    ```

    获得计算结果，如下所示。

    ```
    --------------------------------------------------------------
    | sentence_snr |                     id                      |
    --------------------------------------------------------------
    |   34.4703    |          J310209090013_H02_K03_042          |
    --------------------------------------------------------------
    |   31.3905    | tsh148_seg_2_3013_3_6_48_80bd359827e24dd7_0 |
    --------------------------------------------------------------
    |   35.4774    | tsh148_seg_3013_1_31_11_9d7c87aef9f3e559_0  |
    --------------------------------------------------------------
    |   16.0462    | tsh148_seg_3013_2_29_49_f4cb0990a6b4060c_0  |
    --------------------------------------------------------------
    |   14.5568    |   tsh_148_3013_5_13_47_3d5008d792408f81_0   |
    --------------------------------------------------------------
    ```


通过自定义Extractor，可以在SQL语句上分布式地处理多个OSS上的语音数据文件。您可以使用同样的方法利用MaxCompute的大规模计算能力，处理图像、视频等各种类型的非结构化数据。

## 注意事项

本文中自定义Extractor的方法不支持访问OSS文本文件中的DateTime类型数据。如果有此需求，请参见[MaxCompute自定义Extractor访问OSS文本文件DateTime类型数据](https://yq.aliyun.com/articles/725544)。

