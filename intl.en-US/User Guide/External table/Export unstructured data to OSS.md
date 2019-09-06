# Export unstructured data to OSS {#concept_wsw_nnb_wdb .concept}

The unstructured data processing framework of MaxCompute allows you to export MaxCompute data to OSS through the INSERT method. MaxCompute also associates OSS with external tables to export data to OSS.

You can create, search for, configure, and process external tables through a visual interface in the DataWorks console. You can also query, compute, and analyze data in external tables. DataWorks is powered by MaxCompute. For more information, see [External tables](../../../../reseller.en-US/User Guide/Data development/External tables.md#).

[Access OSS unstructured data](reseller.en-US/User Guide/External table/Access OSS unstructured data.md) describes how MaxCompute can access and process unstructured data stored in OSS through external tables.

The two scenarios for exporting data to OSS are as follows:

-   Data in MaxCompute internal tables is exported to external tables that are associated with OSS.
-   After MaxCompute processes data stored in OSS through external tables, the results are directly exported to the external tables.

Similar to how you access OSS data, you can export data to OSS through a built-in or custom StorageHandler in MaxCompute.

## Export data to OSS through a built-in StorageHandler {#section_xcn_snb_wdb .section}

Using a built-in StorageHandler in MaxCompute, you can export data to OSS in the specified format. You only need to create an external table and specify the built-in StorageHandler to associate OSS with the table. The system will then implement the related logic.

MaxCompute supports two built-in StorageHandlers:

-   `com.aliyun.odps.CsvStorageHandler` defines how to read and write data in CSV format. With this specification, the column delimiter is a comma \(`,`\) and the line break is `\n`.
-   `com.aliyun.odps.TsvStorageHandler` defines how to read and write data in TSV format. With this specification, the column delimiter is `\t` and the line break is `\n`.

-   Create an external table

    ``` {#codeblock_x9k_v9t_1b5}
    CREATE EXTERNAL TABLE [IF NOT EXISTS] <external_table>
    (<column schemas>)
    [PARTITIONED BY (partition column schemas)]
    STORED BY '<StorageHandler>'  
    [WITH SERDEPROPERTIES (  'odps.properties.rolearn'='${roleran}') ]
    LOCATION 'oss://${endpoint}/${bucket}/${userfilePath}/';                       
    ```

    -   The <StorageHandler\> parameter following STORED BY specifies the built-in StorageHandler corresponding to CSV or TSV files. If the data file to be exported to OSS is a TSV file, use `com.aliyun.odps.TsvStorageHandler`. If the data file to be exported to OSS is a CSV file, use `com.aliyun.odps.CsvStorageHandler`.
    -   When granting OSS data access permissions through Custom Authorization of STS Mode Authorization, you must use WITH SERDEPROPERTIES to specify the odps.properties.rolearn property. The property value is the Alibaba Cloud Resource Name \(ARN\) of a custom role used in RAM.

        **Note:** For more information about STS mode authorization, see [Access OSS unstructured data](reseller.en-US/User Guide/External table/Access OSS unstructured data.md).

    -   LOCATION specifies the file storage path in OSS. If the odps.properties.rolearn property is not set in the WITH SERDEPROPERTIES clause and a plaintext AccessKey pair is used for authorization, LOCATION is set as follows:

        ``` {#codeblock_9ge_0mv_1ji}
        LOCATION 'oss://${accessKeyId}:${accessKeySecret}@${endpoint}/${bucket}/${userPath}/'
        ```

-   Perform an INSERT operation on the external table to export data to OSS.

    **Note:** The size of a single file inserted into OSS cannot exceed 5 GB.

    After the external table is associated with the OSS storage path, you can perform standard SQL INSERT OVERWRITE/INSERT INTO operations on the external table to export data to OSS.

    ``` {#codeblock_8x1_ubt_zqz}
    INSERT OVERWRITE|INTO TABLE <external_tablename> [PARTITION (partcol1=val1, partcol2=val2 ...)]
    select_statement
    FROM <from_tablename> 
    [WHERE where_condition];                         
    ```

    -   from\_tablename: This table can either be an internal or external table, including an external table stored in OSS or Table Store.
    -   INSERT writes external table data to OSS based on the StorageHandler format \(TSV or CSV\) specified by the STORED BY clause.
    After the INSERT operation is complete, a series of files will be generated in the corresponding OSS storage path specified by LOCATION.

    For example, if the OSS storage location of the external table is oss://oss-cn-hangzhou-zmf.aliyuncs.com/oss-odps-test/tsv\_output\_folder/, a series of files will be generated in tsv\_output\_folder:

    ``` {#codeblock_zp1_2wf_r49}
    osscmd ls oss://oss-odps-test/tsv_output_folder/
    2017-01-14 06:48:27 39.00B Standard oss://oss-odps-test/tsv_output_folder/.odps/.meta
    2017-01-14 06:48:12 4.80MB Standard oss://oss-odps-test/tsv_output_folder/.odps/20170113224724561g9m6csz7/M1_0_0-0.tsv
    2017-01-14 06:48:05 4.78MB Standard oss://oss-odps-test/tsv_output_folder/.odps/20170113224724561g9m6csz7/M1_1_0-0.tsv
    2017-01-14 06:47:48 4.79MB Standard oss://oss-odps-test/tsv_output_folder/.odps/20170113224724561g9m6csz7/M1_2_0-0.tsv
    ...
    ```

    The tsv\_output\_folder folder in OSS bucket oss-odps-test specified by LOCATION contains a .odps folder that includes some .tsv files and a .meta file. Similar file structures are specific to export of data from MaxCompute to OSS:

    -   When you use MaxCompute to execute INSERT INTO/OVERWRITE on an external table and write data to an OSS storage path, all data is written to a .odps folder in the specified LOCATION.
    -   The .meta file in the .odps folder is an extra macro data file written by MaxCompute to record valid data in the current folder. If the INSERT operation is successful, all data in the current folder is valid. You are only required to parse the macro data if a job fails. If a job fails or is terminated, you can re-execute the INSERT OVERWRITE statement.
    -   If the external table is a partitioned table, a corresponding partition subdirectory will be generated under the tsv\_output\_folder folder based on the partition value specified by the INSERT statement. The partition subdirectory contains the .odps folder. Example: `test/tsv_output_folder/first-level partition name = partition value/n-level partition name = partition value/.odps/20170113224******/M1_2_0-0.tsv`
    The number of files generated by a built-in StorageHandler in MaxCompute is the same as the number of concurrent jobs in the corresponding SQL stage.

    If the `INSER OVERWITE ... SELECT ... FROM ... ;` operation allocates 1,000 mappers on source data table from\_tablename, 1,000 TSV or CSV files will be generated.


## Export data to OSS through a custom StorageHandler {#section_tbq_44b_wdb .section}

In addition to built-in StorageHandlers used to export TSV and CSV files to OSS, MaxCompute also provides an SDK using an unstructured data processing framework to export data files in custom formats.

To export data to OSS, you must first create an external table and then perform an INSERT operation on the external table in a similar manner as using a built-in StorageHandler. However, you must specify a custom StorageHandler in the STORED BY clause when creating the external table.

**Note:** The MaxCompute unstructured data processing framework uses the StorageHandler API to describe the processing of various data storage formats. Specifically, StorageHandler acts as a wrapper class that allows you to specify a custom Extractor and Outputer. An Extractor reads, parses, and extracts data, while an Outputer processes and exports data. A custom StorageHandler must inherit OdpsStorageHandler to implement the getExtractorClass and getOutputerClass APIs.

Using the TextStorageHandler example that customizes an Extractor to access OSS as an example, the following section demonstrates how MaxCompute can export data to OSS text files through a custom StorageHandler, and use | as the column delimiter and \\n as the line break. For more information, see [Access OSS unstructured data](reseller.en-US/User Guide/External table/Access OSS unstructured data.md).

**Note:** After [MaxCompute Studio](../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/What is Studio.md) is configured with [MaxCompute Java Module](../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Developing Java/Create MaxCompute Java Module.md), you can view the corresponding sample code in examples. Alternatively, click [here](https://github.com/aliyun/aliyun-odps-java-sdk/tree/master/odps-sdk-impl/odps-udf-example/src/main/java/com/aliyun/odps/udf/example/text) to view the complete code.

-   Define an Outputer

    The output logic must implement an Outputer API.

    ``` {#codeblock_ppc_buo_0va}
    package com.aliyun.odps.examples.unstructured.text;
    import com.aliyun.odps.data.Record;
    import com.aliyun.odps.io.OutputStreamSet;
    import com.aliyun.odps.io.SinkOutputStream;
    import com.aliyun.odps.udf.DataAttributes;
    import com.aliyun.odps.udf.ExecutionContext;
    import com.aliyun.odps.udf.Outputer;
    import java.io.IOException;
    public class TextOutputer extends Outputer {
        private SinkOutputStream outputStream;
        private DataAttributes attributes;
        private String delimiter;
        public TextOutputer (){
            // default delimiter, this can be overwritten if a delimiter is provided through the attributes.
            this.delimiter = "|";
        }
        @Override
        public void output(Record record) throws IOException {
            this.outputStream.write(recordToString(record).getBytes());
        }
        // no particular usage of execution context in this example
        @Override
        public void setup(ExecutionContext ctx, OutputStreamSet outputStreamSet, DataAttributes attributes) throws IOException {
            this.outputStream = outputStreamSet.next();
            this.attributes = attributes;
            this.delimiter = this.attributes.getValueByKey("delimiter");
            if ( this.delimiter == null)
            {
                this.delimiter=",";
            }
            System.out.println("Extractor using delimiter [" + this.delimiter + "].") ;
        }
        @Override
        public void close() {
            // no-op
        }
        private String recordToString(Record record){
            StringBuilder sb = new StringBuilder();
            for (int i = 0; i < record.getColumnCount(); i++)
            {
                if (null == record.get(i)){
                    sb.append("NULL");
                }
                else{
                    sb.append(record.get(i).toString());
                }
                if (i ! = record.getColumnCount() - 1){
                    sb.append(this.delimiter);
                }
            }
            sb.append("\n");
            return sb.toString();
        }
    }
    ```

    There are three Outputer APIs: setup, output, and close, which correspond to three Extractor APIs: setup, extract, and close. setup \(\) and close \(\) are called only once in an Outputer. You can perform initialization in the setup API. The three parameters returned by setup\(\) must be saved as the Outputer class variable to be used in the output\(\) or close\(\) API. The close \(\) API is used to mark the end of the code.

    Data processing occurs in the output\(Record\) API. The MaxCompute system calls output\(Record\) once based on each input record processed by the current Outputer. Assume that when an output\(Record\) call returns, the code has already consumed the Record. After the current output\(Record\) call returns, the system will use the memory used by the Record for other purposes. Therefore, when the information in Record is used across multiple output\(\) function calls, you must call the record.clone\(\) method to save the current record.

    **Note:** When you use a custom StorageHandler for an external table to implement an Outputer API, the record passed into Outputer.output\(Record record\) is the record generated by the previous operator of the Outputer. In this case, the column name has changed. Column names are not fixed. For example, the column name generated by the some\_function\(column\_a\) expression is a temporary column name.

    Therefore, we recommend that you use record.get \(index\) instead of record.get \(column name\) to obtain the content of a column.

-   Define an Extractor

    An Extractor is used to read, parse, and process data. If the output tables do not need to be read through MaxCompute, you do not need to define an Extractor.

    ``` {#codeblock_5vj_kph_5mc}
    package com.aliyun.odps.examples.unstructured.text;
    import com.aliyun.odps.Column;
    import com.aliyun.odps.data.ArrayRecord;
    import com.aliyun.odps.data.Record;
    import com.aliyun.odps.io.InputStreamSet;
    import com.aliyun.odps.udf.DataAttributes;
    import com.aliyun.odps.udf.ExecutionContext;
    import com.aliyun.odps.udf.Extractor;
    import java.io.BufferedReader;
    import java.io.IOException;
    import java.io.InputStream;
    import java.io.InputStreamReader;
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
            this.inputs = inputs;
            this.attributes = attributes;
            // check if "delimiter" attribute is supplied via SQL query
            String columnDelimiter = this.attributes.getValueByKey("delimiter");
            if ( columnDelimiter ! = null)
            {
                this.columnDelimiter = columnDelimiter;
            }
            System.out.println("TextExtractor using delimiter [" + this.columnDelimiter + "].");
            // note: more properties can be inited from attributes if needed
        }
        @Override
        public Record extract() throws IOException {
            String line = readNextLine();
            if (line == null) {
                return null;
            }
            return textLineToRecord(line);
        }
        @Override
        public void close(){
            // no-op
        }
        private Record textLineToRecord(String line) throws IllegalArgumentException
        {
            Column[] outputColumns = this.attributes.getRecordColumns();
            ArrayRecord record = new ArrayRecord(outputColumns);
            if (this.attributes.getRecordColumns().length ! = 0){
                // string copies are needed, not the most efficient one, but suffice as an example here
                String[] parts = line.split(columnDelimiter);
                int[] outputIndexes = this.attributes.getNeededIndexes();
                if (outputIndexes == null){
                    throw new IllegalArgumentException("No outputIndexes supplied.");
                }
                if (outputIndexes.length ! = outputColumns.length){
                    throw new IllegalArgumentException("Mismatched output schema: Expecting "
                            + outputColumns.length + " columns but get " + parts.length);
                }
                int index = 0;
                for(int i = 0; i < parts.length; i++){
                    // only parse data in columns indexed by output indexes
                    if (index < outputIndexes.length && i == outputIndexes[index]){
                        switch (outputColumns[index].getType()) {
                            case STRING:
                                record.setString(index, parts[i]);
                                break;
                            case BIGINT:
                                record.setBigint(index, Long.parseLong(parts[i]));
                                break;
                            case BOOLEAN:
                                record.setBoolean(index, Boolean.parseBoolean(parts[i]));
                                break;
                            case DOUBLE:
                                record.setDouble(index, Double.parseDouble(parts[i]));
                                break;
                            case DATETIME:
                            case DECIMAL:
                            case ARRAY:
                            case MAP:
                            default:
                                throw new IllegalArgumentException("Type " + outputColumns[index].getType() + " not supported for now.");
                        }
                        index++;
                    }
                }
            }
            return record;
        }
        /**
         * Read next line from underlying input streams.
         * @return The next line as String object. If all of the contents of input
         * streams has been read, return null.
         */
        private String readNextLine() throws IOException {
            if (firstRead) {
                firstRead = false;
                // the first read, initialize things
                currentReader = moveToNextStream();
                if (currentReader == null) {
                    // empty input stream set
                    return null;
                }
            }
            while (currentReader ! = null) {
                String line = currentReader.readLine();
                if (line ! = null) {
                    return line;
                }
                currentReader = moveToNextStream();
            }
            return null;
        }
        private BufferedReader moveToNextStream() throws IOException {
            InputStream stream = inputs.next();
            if (stream == null) {
                return null;
            } else {
                return new BufferedReader(new InputStreamReader(stream));
            }
        }
    }
    ```

    For more information, see [Access OSS unstructured data](reseller.en-US/User Guide/External table/Access OSS unstructured data.md).

-   Define a StorageHandler

    ``` {#codeblock_3xe_s57_yzp}
    package com.aliyun.odps.examples.unstructured.text;
    import com.aliyun.odps.udf.Extractor;
    import com.aliyun.odps.udf.OdpsStorageHandler;
    import com.aliyun.odps.udf.Outputer;
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

    If the table does not need to be read, you do not need to specify an Extractor API.

-   Compile the code into a package

    Compile your custom code into a JAR package and upload it to MaxCompute. If the JAR package is named odps-TextStorageHandler.jar, the code used to upload the package as a MaxCompute resource is as follows:

    ``` {#codeblock_5wr_u66_qhv}
    add jar odps-TextStorageHandler.jar;
    ```

-   Create an external table

    To export data to OSS, you must first create an external table in a similar manner as using a built-in StorageHandler. However, you must specify a custom StorageHandler to export data to the external table.

    ``` {#codeblock_d95_nsi_854}
    CREATE EXTERNAL TABLE IF NOT EXISTS output_data_txt_external
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
    STORED BY 'com.aliyun.odps.examples.unstructured.text.TextStorageHandler' 
    WITH SERDEPROPERTIES(
        'delimiter'='|'
        [,'odps.properties.rolearn'='${roleran}'])
    LOCATION 'oss://${endpoint}/${bucket}/${userfilePath}/'
    USING 'odps-TextStorageHandler.jar';
    ```

    **Note:** If the `odps.properties.rolearn` property is required, see [Access OSS unstructured data](reseller.en-US/User Guide/External table/Access OSS unstructured data.md) for more information about custom authorization based on STS. If the property is not required, refer to one-click authorization or use a plaintext AccessKey pair on LOCATION.

-   Perform an INSERT operation on an external table to export data to OSS

    After you create an external table and associate it with an OSS storage path through a custom StorageHandler, you can perform standard SQL operations such as INSERT OVERWRITE/INTO on the external table to export data to OSS in the same way as a built-in StorageHandler.

    ``` {#codeblock_svg_toe_eja}
    INSERT OVERWRITE|INTO TABLE <external_tablename> [PARTITION (partcol1=val1, partcol2=val2 ...)]
    select_statement
    FROM <from_tablename> 
    [WHERE where_condition];       
    ```

    After the INSERT operation is complete, a series of files will be generated in the .odps folder in the OSS storage path specified by LOCATION. This is similar to what happens when you use a built-in StorageHandler.


