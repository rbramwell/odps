---
keyword: [custom extractor, unstructured data in OSS]
---

# Access unstructured data in OSS by using a custom extractor

This topic describes how to access unstructured data in Object Storage Service \(OSS\) by using a custom extractor.

## Access text data in OSS by using a custom extractor

If OSS data is in a complicated format and cannot be processed by the built-in extractor, you must customize an extractor to read the OSS data.

Assume that a TXT file that contains the following data is stored in OSS. The columns of records in the file are separated by vertical bars \(`|`\). The file path is `/demo/SampleData/CustomTxt/AmbulanceData/vehicle.csv`.

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

1.  Define an extractor.

    Define a common extractor with a specified delimiter. The extractor can process all TXT files with data in the similar format. Example:

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
        this.inputs = inputs; // inputs specifies an InputStreamSet. An InputStream is returned each time next() is called. This InputStream can read all the content of an OSS object.
        this.attributes = attributes;
        // check if "delimiter" attribute is supplied via SQL query
        String columnDelimiter = this.attributes.getValueByKey("delimiter"); // The delimiter can be used as a parameter in DDL statements.
        if ( columnDelimiter ! = null)
        {
          this.columnDelimiter = columnDelimiter;
        }
        // note: more properties can be inited from attributes if needed
      }
      @Override
      public Record extract() throws IOException { // The extractor returns a record that is extracted from the foreign table.
        String line = readNextLine();
        if (line == null) {
          return null; // If NULL is returned, all records in the table have been read.
        }
        return textLineToRecord(line); // textLineToRecord splits a row into multiple columns by using the delimiter.
      }
      @Override
      public void close(){
        // no-op
      }
    }
    ```

    **Note:** For more information about the implementation process, see [TextExtractor.java](https://github.com/aliyun/aliyun-odps-java-sdk/blob/master/odps-sdk-impl/odps-udf-example/src/main/java/com/aliyun/odps/udf/example/text/TextExtractor.java).

2.  Define a storage handler.

    A storage handler is a unified entry to your custom logic for processing foreign tables.

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

3.  Compile and pack the code.

    Compile the custom code and pack it into a JAR package. Then, upload the JAR package to MaxCompute.

    ```
    add jar odps-udf-example.jar;
    ```

4.  Create a foreign table.

    Before you use a custom extractor, you must create a foreign table with the specified custom storage handler.

    Execute the following SQL statement to create a foreign table, in which the delimiter parameter specifies the delimiter for splitting each row into columns:

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

    -   STORED BY: specifies the class name of a custom storage handler.
    -   WITH SERDEPROPERTIES: specifies the following parameters for the foreign table.
        -   delimiter: specifies the delimiter.
        -   odps.properties.rolearn: specifies the Alibaba Cloud Resource Name \(ARN\) for the AliyunODPSDefaultRole role in Resource Access Management \(RAM\). You can obtain the ARN from the role details page in the [RAM console](https://ram.console.aliyun.com/#/role/detailAliyunODPSDefaultRole/info).
    -   LOCATION: specifies an OSS folder. By default, the system reads all files in the folder.
        -   We recommend that you use the internal same-region endpoint of OSS to avoid fees that are incurred by OSS data flows.
        -   We recommend that you store OSS data in the region where you activate MaxCompute. MaxCompute can be deployed only in specific regions, so cross-region data connectivity is not ensured.
        -   The OSS connection format is `oss://oss-cn-shanghai-internal.aliyuncs.com/Bucket name/Folder name/`. Do not add a file name after the Folder name parameter. The following examples are negative:

            ```
            http://oss-odps-test.oss-cn-shanghai-internal.aliyuncs.com/Demo/     // HTTP connections are not supported.
            https://oss-odps-test.oss-cn-shanghai-internal.aliyuncs.com/Demo/    // HTTPS connections are not supported.
            oss://oss-odps-test.oss-cn-shanghai-internal.aliyuncs.com/Demo       // The connection address is invalid.
            oss://oss://oss-cn-shanghai-internal.aliyuncs.com/oss-odps-test/Demo/vehicle.csv  // You do not need to specify the file name.
            ```

    -   USING: specifies the JAR package that defines the specified class.
5.  Execute the following SQL statement to query the foreign table:

    ```
    SELECT recordId, patientId, direction FROM ambulance_data_txt_external WHERE patientId > 25;
    ```


## Access non-text data in OSS by using a custom extractor

This section describes how to use a custom extractor to access and process non-text data in OSS. WAV files are used in this example.

1.  Execute the following SQL statement to create a foreign table. Define the schema of the foreign table for the custom extractor to extract the following information from the WAV files:

    -   sentence\_snr: the signal-to-noise ratio \(SNR\) of sentences in each WAV file.
    -   id: the name of each WAV file.
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

    -   com.aliyun.odps.udf.example.speech.SpeechStorageHandler: encapsulates an extractor to calculate the average SNR of valid sentences in each WAV file, and run SQL jobs to process the structured data that is extracted.
    -   LOCATION: specifies the folder, `oss://oss-cn-hangzhou-zmf.aliyuncs.com/oss-odps-test/dev/SpeechSentenceTest/` in this example, that stores multiple original WAV files. MaxCompute reads all the files and allocates these files to multiple compute nodes for processing.
2.  Execute the following statement to query the foreign table:

    ```
    select sentence_snr, id from speech_sentence_snr_external where sentence_snr > 10.0; 
    ```

    The following data is returned:

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


A custom extractor allows you to execute SQL statements to process multiple audio files in OSS in a distributed manner. You can process other unstructured data such as images and videos in the same way to make full use of the powerful computing capabilities of MaxCompute.

## Considerations

A custom extractor that is defined by using the method described in this topic cannot access data of the DATETIME type in OSS text files. For more information about how to define a custom extractor to access data of the DATETIME type in OSS text files, see [Access data of the DATETIME type in OSS text files by using a MaxCompute custom extractor](https://yq.aliyun.com/articles/725544).

