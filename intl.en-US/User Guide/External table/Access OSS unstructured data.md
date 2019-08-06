# Access OSS unstructured data {#concept_tzd_tlg_vdb .concept}

This topic describes how to access OSS unstructured data in MaxCompute by using different methods.

**Note:** You can use DataWorks and MaxCompute to create, search, query, configure, process, and analyze external tables. For more information, see [External tables](../../../../reseller.en-US/User Guide/Data development/External tables.md#). For information about how to use an external table to process unstructured data, see [External table overview](reseller.en-US/User Guide/External table/Overview of External tables.md#).

## Authorize OSS data access permission with STS mode {#section_uv5_1fb_wdb .section}

Authorize OSS data access permission to MaxCompute account in advance so that MaxCompute can directly access the OSS. You can authorize permissions in one of the following two ways:

-   When the MaxCompute owner and OSS owner use the same account, you can use this account to log on to the [Alibaba Cloud](https://partners-intl.aliyun.com/login-required#/ram) and authorize the role to access OSS resources.
-   Customize authorization.
    1.  Log on to the [RAM console](https://partners-intl.aliyun.com/login-required#/ram). If the MaxCompute owner and OSS owner use different accounts, navigate to the RAM Roles page and use the OSS owner account to create a role such as `AliyunODPSDefaultRole` or `AliyunODPSRoleForOtherUser` to grant MaxCompute the permissions to access OSS.
    2.  Modify the policy content of role as follows:

        ``` {#codeblock_2ki_vgs_1ec}
        --If MaxCompute and OSS owners use the same account, do the following:
        {
        "Statement": [
        {
         "Action": "sts:AssumeRole",
         "Effect": "Allow",
         "Principal": {
           "Service": [
             "odps.aliyuncs.com"
           ]
         }
        }
        ],
        "Version": "1"
        }
        --If MaxCompute and OSS owners use different accounts, do the following:
        {
        "Statement": [
        {
         "Action": "sts:AssumeRole",
         "Effect": "Allow",
         "Principal": {
           "Service ":[
             "MaxCompute's Owner account: id@odps.aliyuncs.com"
           ]
         }
        }
        ],
        "Version": "1"
        }
        ```

    3.  Grant the necessary permission `AliyunODPSRolePolicy` to the role to access OSS as follows:

        ``` {#codeblock_pta_fc9_0sx}
        {
        "Version": "1",
        "Statement": [
        {
         "Action": [
           "oss:ListBuckets",
           "oss:GetObject",
           "oss:ListObjects",
           "oss:PutObject",
           "oss:DeleteObject",
           "Oss: maid ",
           "oss:ListParts"
         ],
         "Resource": "*",
         "Effect": "Allow"
        }
        ]
        }
        --You can customize other permissions.
        ```

    4.  Authorize the permission **AliyunODPSRolePolicy** to this role.

## Read OSS text data by using a built-in extractor {#section_ukx_nfb_wdb .section}

When accessing external data sources, you must use different custom extractor. You can also use MaxCompute’s built-in extractor to read conventionally-formatted data stored in [OSS](https://www.alibabacloud.com/product/oss). You only need to create an external table and use this table as the source table for query operations.

In this example, assume that you have a CSV data file in [OSS](https://www.alibabacloud.com/product/oss). The endpoint is `oss-cn-shanghai-internal.aliyuncs.com`, the bucket is `oss-odps-test`, and the data file is stored in /demo/vehicle.csv.

-   **Create an external table** 

    Use the following statements to create an external table:

    ``` {#codeblock_qrd_66q_itb}
    CREATE EXTERNAL TABLE IF NOT EXISTS ambulance_data_csv_external
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
    STORED BY 'com.aliyun.odps.CsvStorageHandler' -- (1)
    WITH SERDEPROPERTIES (
     'odps.properties.rolearn'='acs:ram::xxxxx:role/aliyunodpsdefaultrole'
    ) -- (2)
    LOCATION 'oss://oss-cn-shanghai-internal.aliyuncs.com/oss-odps-test/Demo/'; -- (3)(4)
    ```

    The above statement is described below:

    -   `com.aliyun.odps.CsvStorageHandler` is the built-in `StorageHandler` for processing CSV-format files. It defines how CSV files are read and written. You only need to specify this name. The relevant logic is implemented by the system.
    -   The information in `odps.properties.rolearn` comes from the `Arn` information of `AliyunODPSDefaultRole` in RAM. You can get it through the [role details](https://ram.console.aliyun.com/#/role/detailAliyunODPSDefaultRole/info) in the RAM console.
    -   You must specify an OSS directory for LOCATION. By default, the system reads all the files in this directory.
        -   We recommend you to use the domain name of the intranet, to avoid incurring fees for the OSS data-flow.
        -   We recommend that the region you store the OSS data is the same as the region you open MaxCompute. Because MaxCompute can only be deployed in some regions, cross-regional data connectivity cannot be guaranteed.
        -   OSS connection format is `oss://oss-cn-shanghai-internal.aliyuncs.com/bucketname/directoryname/`. You do not have to add a file name after the directory. Some common errors are shown as follows:

            ``` {#codeblock_g8r_2yi_7mq}
            http://oss-odps-test.oss-cn-shanghai-internal.aliyuncs.com/Demo/ -- HTTP connection is not supported.
            https://oss-odps-test.oss-cn-shanghai-internal.aliyuncs.com/Demo/  -- HTTPS connection is not supported.
            oss://oss-odps-test.oss-cn-shanghai-internal.aliyuncs.com/Demo  -- The connection address is incorrect.
            oss://oss://oss-cn-shanghai-internal.aliyuncs.com/oss-odps-test/Demo/vehicle.csv  -- You do not need to specify the file name.
            ```

    -   In the MaxCompute system, external tables only record the associated OSS directory. If you DROP \(delete\) this table, the corresponding `LOCATION` data is not deleted.
    In the returned information, `Extended Info` contains external tables information such as `StorageHandler` and `Location`.

-   **Query an external table** 

    After creating an external table, you can use it as a normal table. Assume the data in `/demo/vehicle.csv` is as follows:

    ``` {#codeblock_b22_lyt_t8b}
    1,1,51,1,46.81006,-92.08174,9/14/2014 0:00,S
    1,2,13,1,46.81006,-92.08174,9/14/2014 0:00,NE
    1,3,48,1,46.81006,-92.08174,9/14/2014 0:00,NE
    1,4,30,1,46.81006,-92.08174,9/14/2014 0:00,W
    1,5,47,1,46.81006,-92.08174,9/14/2014 0:00,S
    1,6,9,1,46.81006,-92.08174,9/14/2014 0:00,S
    1,7,53,1,46.81006,-92.08174,9/14/2014 0:00,N
    1,8,63,1,46.81006,-92.08174,9/14/2014 0:00,SW
    1,9,4,1,46.81006,-92.08174,9/14/2014 0:00,NE
    1,10,31,1,46.81006,-92.08174,9/14/2014 0:00,N
    ```

    Run the following SQL statement:

    ``` {#codeblock_cf5_s0e_o41}
    select recordId, patientId, direction from ambulance_data_csv_external where patientId > 25;
    ```

    **Note:** Currently, external table can only be operated through MaxCompute SQL. MaxCompute MapReduce cannot operate the external table.

    This statement submits a job, scheduling the built-in CSV extractor to read and process data from OSS. The result is as follows:

    ``` {#codeblock_02a_005_1u6}
    +------------+------------+-----------+
    | recordId   | patientId  | direction |
    +------------+------------+-----------+
    | 1 | 51 | S |
    | 3          | 48         | NE        |
    | 4          | 30         | W         |
    | 5          | 47         | S         |
    | 7          | 53         | N         |
    | 8          | 63         | SW        |
    | 10         | 31         | N         |
    +------------+------------+-----------+
    ```


## Read OSS text data by using a custom extractor {#section_lbc_ngb_wdb .section}

When OSS data is in a complex format, and the built-in extractor cannot meet your requirements, you must use a custom extractor to read data from OSS files.

For example, assume you have a TXT data file that is not in CSV format, and `|` is used as the column delimiter between records. For example, the data in `/demo/SampleData/CustomTxt/AmbulanceData/vehicle.csv` is as follows:

``` {#codeblock_zf2_3me_szc}
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

-   Define an extractor.

    Write a common extractor by using the delimiter as the parameter. This allows you to process all text files with similar formats.

    ``` {#codeblock_bq6_eqp_blp}
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
        this.inputs = inputs; // inputs is an InputStreamSet, each call to next() returns an InputStream. This InputStream can read all the content in an OSS file.
        this.attributes = attributes;
        // check if "delimiter" attribute is supplied via SQL query
        String columnDelimiter = this.attributes.getValueByKey("delimiter"); //The delimiter parameter is supplied by a DDL statement.
        if ( columnDelimiter ! = NULL)
        {
          this.columnDelimiter = columnDelimiter;
        }
        // note: more properties can be inited from attributes if needed
      }
      @Override
      public Record extract() throws IOException {//extractor() calls return one record, corresponding to one record in an external table.
        String line = readNextLine();
        if (line == null) {
          return null;  // A return value of NULL indicates that this table has no readable records.
        }
        return textLineToRecord(line);  // textLineToRecord splits a row of data into multiple columns according to the delimiter.
      }
      @Override
      public void close(){
        // no-op
      }
    }
    ```

    See [here](https://github.com/aliyun/aliyun-odps-java-sdk/blob/master/odps-sdk-impl/odps-udf-example/src/main/java/com/aliyun/odps/udf/example/text/TextExtractor.java) for a complete implementation of the textLineToRecord splitting data.

-   Define StorageHandler.

    A StorageHandler acts as a centralized portal for custom external table logic.

    ``` {#codeblock_7xk_m3v_ejy}
    package com.aliyun.odps.udf.example.text;
    public class TextStorageHandler extends OdpsStorageHandler {
      @Override
      public Class<? extends Extractor> getExtractorClass() {
        return TextExtractor.class;
      }
      @Override
      public Class<? extends Outputer>getOutputerClass() {
        return TextOutputer.class;
      }
    }
    ```

-   Compile and package custom code.

    Compile your custom code into a package and upload it to MaxCompute.

    ``` {#codeblock_cbp_l3v_kid}
    add jar odps-udf-example.jar;
    ```

-   Create an external table.

    Similar to using the built-in extractor, first, you must create an external table. The difference is that, when specifying the external table access data, you must use a custom StorageHandler.

    Use the following statements to create an external table:

    ``` {#codeblock_a4c_9wc_o04}
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
    STORED BY 'com.aliyun.odps.udf.example.text.TextStorageHandler'   --STORED BY specifies the custom StorageHandler class name.
      with SERDEPROPERTIES (
    'delimiter'='\\|', -- SERDEPROPERITES can specify parameters, these parameters are passed through the DataAttributes to the Extractor code.
    'odps.properties.rolearn'='acs:ram::xxxxxxxxxxxxx:role/aliyunodpsdefaultrole'
    )
    LOCATION 'oss://oss-cn-shanghai-internal.aliyuncs.com/oss-odps-test/Demo/SampleData/CustomTxt/AmbulanceData/'
    USING 'odps-udf-example.jar'; --You must also specify the jar package containing the class definition.
    ```

-   Query an external table.

    Run the following SQL statement:

    ``` {#codeblock_gzy_u6o_1vw}
    select recordId, patientId, direction from ambulance_data_txt_external where patientId > 25;
    ```


## Read OSS non-text data by using a custom extractor {#section_kqm_bhb_wdb .section}

The preceding sections show how the built-in extractor or a custom extractor can conveniently process CSV and other text data stored in OSS. Following sections using audio data \(WAV format files\) as an example show how to use a custom extractor to access and process non-text files in OSS.

**However, this section describes how to use MaxCompute SQL as a portal to process audio files stored in OSS.**

Create the external table SQL as follows:

``` {#codeblock_pru_381_bqk}
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

As in the preceding example, you must create an external table. Then, use the schema of this table to define the information that you want to extract from the audio file:

-   The statement signal-to-noise ratio \(SNR\) in an audio file: sentence\_snr.
-   The name of the audio file: id.

After creating the external table, use a standard Select statement to perform a query. This operation triggers the extractor to perform computation. When reading and processing OSS data, in addition to simple deserialization on text files, you can use custom extractor to perform more complex data processing and extraction logic.

In this example, use the custom extractor encapsulated in `com.aliyun.odps.udf.example.speech.SpeechStorageHandler` to calculate the average SNR of valid statements in the audio file, and extract structured data for SQL operations \(WHERE sentence\_snr \> 10\). After this operation is completed, the operation returns all audio files with an SNR that are greater than 10 along with the corresponding SNR values.

Multiple WAV format files are stored at the OSS address `oss://oss-cn-hangzhou-zmf.aliyuncs.com/oss-odps-test/dev/SpeechSentenceTest/`. The MaxCompute framework reads all the files at this address and, when necessary, performs file-level sharing. The framework automatically allocates the file to multiple computing nodes for processing.

On each computing node, the extractor is responsible for processing the file set allocated to the node by InputStreamSet. The special processing logic is similar to a single-host program. The algorithm is implemented by using the single host method according to its class.

Details about the `SpeechSentenceSnrExtractor` formulation logic are as follows:

First, read the parameters in the `setup` interface to perform initialization and import the audio processing model \(using resource introduction\):

``` {#codeblock_rga_a2y_oz2}
public SpeechSentenceSnrExtractor(){
    this.utteranceLabels = new HashMap<String, UtteranceLabel>();
  }
  @Override
  public void setup(ExecutionContext ctx, InputStreamSet inputs, DataAttributes attributes){
    this.inputs = inputs;
    This. Attributes = attributes;
    this.mlfFileName = this.attributes.getValueByKey(MLF_FILE_ATTRIBUTE_KEY);
    String sampleRateInKHzStr = this.attributes.getValueByKey(SPEECH_SAMPLE_RATE_KEY);
    this.sampleRateInKHz = Double.parseDouble(sampleRateInKHzStr);
    try {
      // read the speech model file from resource and load the model into memory
      BufferedInputStream inputStream = ctx.readResourceFileAsStream(mlfFileName);
      loadMlfLabelsFromResource(inputStream);
      inputStream.close();
    } catch (IOException e) {
      throw new RuntimeException("reading model from mlf failed with exception " + e.getMessage());
    }
  }
				
```

The extract\(\) interface implements reading and processing logic of the voice file, computes the signal-to-noise ratio \(SNR\) of the data based on the voice model, and fills Record with the result in the format of \[snr, id\].

The preceding example simplifies the implementation process and does not include the relevant audio processing algorithm logic. See the [example code](https://github.com/aliyun/aliyun-odps-java-sdk/blob/master/odps-sdk-impl/odps-udf-example/src/main/java/com/aliyun/odps/udf/example/speech/SpeechSentenceSnrExtractor.java) provided by the MaxCompute SDK from the open-source community.

``` {#codeblock_xhr_jt9_s71}
@Override
  public Record extract() throws IOException {
    SourceInputStream inputStream = inputs.next();
    if (inputStream == null){
      return null;
    }
    // process one wav file to extract one output record [snr, id]
    String fileName = inputStream.getFileName();
    fileName = fileName.substring(fileName.lastIndexOf('/') + 1);
    logger.info("Processing wav file " + fileName);
    String id = fileName.substring(0, fileName.lastIndexOf('.'));
    // read speech file into memory buffer
    long fileSize = inputStream.getFileSize();
    byte[] buffer = new byte[(int)fileSize];
    int readSize = inputStream.readToEnd(buffer);
    inputStream.close();
    // compute the avg sentence snr
    double snr = computeSnr(id, buffer, readSize);
    // construct output record [snr, id]
    Column[] outputColumns = this.attributes.getRecordColumns();
    ArrayRecord record = new ArrayRecord(outputColumns);
    record.setDouble(0, snr);
    record.setString(1, id);
    return record;
  }
  private void loadMlfLabelsFromResource(BufferedInputStream fileInputStream)
          throws IOException {
    //  skipped here
  }
  // compute the snr of the speech sentence, assuming the input buffer contains the entire content of a wav file
  private double computeSnr(String id, byte[] buffer, int validBufferLen){
    // computing the snr value for the wav file (supplied as byte buffer array), skipped here
  }
```

Run the query:

``` {#codeblock_mbv_0xj_pah}
select sentence_snr, id
    from speech_sentence_snr_external
where sentence_snr > 10.0;
```

Results:

``` {#codeblock_uuk_n0b_pwl}
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

By using the custom extractor, you can process multiple voice data files stored on OSS on the SQL statement in a distributed way. You can also use the same method given the large-scale computing power of MaxCompute to process various types of unstructured data such as image or video files.

## Partition data {#section_avv_rhb_wdb .section}

In earlier sections, data linked to an external table is implemented through a designated OSS Directory on LOCATION. During this process, MaxCompute reads all data under the OSS Directory, including all files in sub-directory. For directories with largest amounts of data, especially those with data that is continually accumulating over time, scanning the entire directory may cause unnecessary IO and data processing time. In which case, you can typically implement one of the following two solutions:

-   Reduce access data volume: You can implement changes in terms of the data storage organization, use multiple external tables to describe different parts of the data, or have each EXTERNAL TABLE LOCATION refer to a subset of the data.
-   Partition data: An external table is the same as an internal table, it supports functions of partition table, you are available to manage data systemization based on partition function.

-   Standard Organization and Path Format of Partitioned Data in OSS 

    Unlike its internal tables, MaxCompute does not have the authority to manage data stored in external memory \(such as OSS\). As such, if using the partition table function on your system is a requirement, the storage path for data files in OSS must conform to a certain format. This format is as follows.

    ``` {#codeblock_s82_5oa_b1l}
    partitionKey1=value1\partitionKey2=value2\...
    ```

    Related examples are as follows 

    Assume that you save your daily LOG files on OSS and want to access part of the data when processed with MaxCompute, based on the granularity of Day. Assuming that these LOG files are CSV files \(usage of complicated and customized format is similar\), you can define data using the following partitioned external table.

    ``` {#codeblock_mvh_ca2_x7e}
    CREATE EXTERNAL TABLE log_table_external (
        click STRING,
        ip STRING,
        url STRING,
      )
      PARTITIONED BY (
        year STRING,
        month STRING,
        day STRING
      )
      STORED BY 'com.aliyun.odps.CsvStorageHandler'
      WITH SERDEPROPERTIES (
     'odps.properties.rolearn'='acs:ram::xxxxx:role/aliyunodpsdefaultrole'
    ) 
      LOCATION 'oss://oss-cn-hangzhou-zmf.aliyuncs.com/oss-odps-test/log_data/';
    ```

    Like the previous table statement, the difference with the previous example is that when you define an external table, the external table is specified as a partition table through the PARTITIONED BY syntax, and the example is a three-tier partition table, the key for the partition is year, month, and day.

    In order for a partition like this to work effectively, you must comply with the aforementioned path format when storing data on OSS. The following is an example of a valid path storage layout.

    ``` {#codeblock_wqy_9gx_c16}
    osscmd ls oss://oss-odps-test/log_data/
    2017-01-14 08:03:35 128MB Standard oss://oss-odps-test/log_data/year=2016/month=06/day=01/logfile
    2017-01-14 08:04:12 127MB Standard oss://oss-odps-test/log_data/year=2016/month=06/day=01/logfile. 1
    2017-01-14 08:05:02 118MB Standard oss://oss-odps-test/log_data/year=2016/month=06/day=02/logfile
    2017-01-14 08:06:45 123MB Standard oss://oss-odps-test/log_data/year=2016/month=07/day=10/logfile
    2017-01-14 08:07:11 115MB Standard oss://oss-odps-test/log_data/year=2016/month=08/day=08/logfile
    ...
    ```

    **Note:** If you have prepared the offline data, that is, if you have uploaded the offline data to the OSS storage service with **osscmd** or other OSS tools, you then define the data path format.

    You can introduce the partition information into MaxCompute by using the ALTER TABLE ADD PARTITIONDDL pant statement.

    An example of the corresponding DDL statement is as follows.

    ``` {#codeblock_r9g_igs_t1u}
    ALTER TABLE log_table_external ADD PARTITION (year = '2016', month = '06', day = '01')
    ALTER TABLE log_table_external ADD PARTITION (year = '2016', month = '06', day = '02')
    ALTER TABLE log_table_external ADD PARTITION (year = '2016', month = '07', day = '10')
    ALTER TABLE log_table_external ADD PARTITION (year = '2016', month = '08', day = '08')
    ...
    ```

    **Note:** These actions are the same as the standard maxcompute internal table operation, and for more information about the partition, see [Partition](../../../../reseller.en-US/Product Introduction/Definitions/Partition.md#). When the data is ready and the PARTITION information has been imported into the system, the partitioning of the external table data on OSS can be performed by means of an SQL statement.

    Assuming that you only want to analyze how many different IPs there are in LOG on June 1, 2016, the following command can be used:

    ``` {#codeblock_dod_j07_nn9}
    SELECT count(distinct(ip)) FROM log_table_external WHERE year = '2016' AND month = '06' AND day = '01';
    ```

    At this point, for log\_table\_external, the directory that corresponds to the external table will only access the files under the `log_data/year=2016/month=06/day=01` subdirectory \(logfile and logfile 1\), not on the whole log\_data/ to avoid a large number of useless I/O operations.

    Similarly, if you only want to analyze the data for the second half of 2016, you can use the following command:

    ``` {#codeblock_du6_p15_ryh}
    SELECT count(distinct(ip)) FROM log_table_external 
    WHERE year = '2016' AND month > '06';
    ```

    At this point, only access the second half of the LOG stored on OSS.

-   Customized Path of Partitioned Data in OSS 

    If you have historical data stored in OSS but this data is not stored by using the `partitionKey1=value1\partitionKey2=value2\...` path format, you can still access it using MaxCompute’s partition mode. MaxCompute also provides a way to import partitions through a customized path.

    Assume that only a simple partition value is on your data path \(and no partition key information\). The following is an example of the data path storage layout:

    ``` {#codeblock_khz_pgm_kgh}
    osscmd ls oss://oss-odps-test/log_data_customized/
    2017-01-14 08:03:35 128MB Standard oss://oss-odps-test/log_data_customized/2016/06/01/logfile
    2017-01-14 08:04:12 127MB Standard oss://oss-odps-test/log_data_customized/2016/06/01/logfile. 1
    2017-01-14 08:05:02 118MB Standard oss://oss-odps-test/log_data_customized/2016/06/02/logfile
    2017-01-14 08:06:45 123MB Standard oss://oss-odps-test/log_data_customized/2016/07/10/logfile
    2017-01-14 08:07:11 115MB Standard oss://oss-odps-test/log_data_customized/2016/08/08/logfile
    ...
    ```


The external table builder DDL can see the previous example and also specify the partition key in the clause.

To bind different subdirectories to different partitions, you can do so by using a command similar to the following customized partition path:

``` {#codeblock_vqg_qlq_7ij}
ALTER TABLE log_table_external ADD PARTITION (year = '2016', month = '06', day = '01')
LOCATION 'oss://oss-cn-hangzhou-zmf.aliyuncs.com/oss-odps-test/log_data_customized/2016/06/01/';
```

When LOCATION information is added in `ADD PARTITION` to customize a partition data path. Even if the data is not stored in the recommended format of `partitionKey1=value1\partitionKey2=value2\...`, you can still access the partition data of the subdirectory.

