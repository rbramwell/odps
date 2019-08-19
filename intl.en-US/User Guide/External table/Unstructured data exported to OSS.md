# Unstructured data exported to OSS {#concept_wsw_nnb_wdb .concept}

The unstructured framework of MaxCompute supports output of MaxCompute data directly to OSS via insert, MaxCompute also associates OSS with external tables for data output.

[Accessing OSS unstructured data](intl.en-US/User Guide/External table/Access OSS unstructured data.md) shows you how MaxCompute can be accessed and processed through the associations of External tables unstructured data stored in OSS.

Output Data to OSS is typically two cases:

-   The MaxCompute internal table is output to the External table that is associated with the OSS.
-   After MaxCompute processes the external tables, the results are output directly to the external tables that are associated with the OSS.

Like accessing OSS data, MaxCompute supports output via built-in storagehandler and custom storagehandler.

## Output to OSS via built-in StorageHandler {#section_xcn_snb_wdb .section}

Using the built-in StorageHandler in MaxCompute And can be very convenient to output data in the agreed format to the OSS for storage. All we need to do is create an external table that indicates the built-in StorageHandler, it can be associated with this table, and the related logic is implemented by the system.

Currently MaxCompute supports 2 built-in StorageHandler:

-   com.aliyun.odps.CsvStorageHandler , Defines how to read and write CSV format data, data format Conventions: English comma, column separator`,` line Break is `\n`.
-   com.aliyun.odps.TsvStorageHandler, defines how to read and write CSV format data, data format Conventions: `\t`is a column separator, line Break is `\n`.

-   Creating an External table 

    ``` {#codeblock_oop_v9i_lxk}
    CREATE EXTERNAL TABLE [IF NOT EXISTS] <external_table> 
    (<column schemas>) 
    [PARTITIONED BY (partition column schemas)]
     STORED BY '<StorageHandler>' 
    [WITH SERDEPROPERTIES ( 'odps.properties.rolearn'='${roleran}') ] 
    LOCATION 'oss://${endpoint}/${bucket}/${userfilePath}/';
    						
    ```

    -   STORED By, if the data file that is required to be exported to OSS is a TSV file, then built-in `com.aliyun.odps.TsvStorageHandler`； if the data file that is required to be exported to OSS is a CSV file, a built-in `com.aliyun.odps.CsvStorageHandler`.
    -   WITH SERDEPROPERTIES, when associating OSS privileges with "Custom Authorization" of "STS Mode Authorization", this parameter needs to specify the'odps.properties.rolearn'attribute, whose value is the information of the Custom Role specifically used in RAM.

        **Note:** STS mode authorization can be seen in [Accessing the unstructured data of OSS](intl.en-US/User Guide/External table/Access OSS unstructured data.md).

    -   Location that specifies the path to the file that corresponds to the OSS storage. If the 'odps.properties.rolearn' attribute is not set in WITH SERDEPROPERTIES and the plaintext AK is used for authorization, then LOCATION is

        ``` {#codeblock_8vv_7z5_icy}
        LOCATION
            'oss://${accessKeyId}:${accessKeySecret}@${endpoint}/${bucket}/${userPath}/'
        ```

-   The data is output to OSS through the INSERT operation of External Table.

    **Note:** The insert-to-OSS single file size cannot exceed 5g.

    When associated with an OSS storage path via external table,you can do a standard SQL INSERT OVERWRITE/INSERT INTO operation on the External table to output both data to OSS.

    ``` {#codeblock_7rn_osk_qiq}
    INSERT OVERWRITE|INTO TABLE <external_tablename> [PARTITION (partcol1=val1, partcol2=val2 ...)]
    select_statement
    FROM <from_tablename> 
    [WHERE where_condition];
    							
    ```

    -   from\_tablename: it can be an internal table, it can also be an external table \(including an external table for the associated OSS or OTS \).
    -   INSERT specifies the format of 'StorageHandler'\(that is, TSV or CSV\) according to the External table 'stored '\) write to OSS.
    When the INSERT operation is completed successfully, you can see that the corresponding LOCATION on the OSS produces a series of files.

    For example: the location corresponding to External table is the oss://oss-cn-hangzhou-zmf.aliyuncs.com/oss-odps-test/tsv\_output\_folder/, you can see the generation of a series of files in the OSS corresponding path:

    ``` {#codeblock_7dl_bia_0gq}
    osscmd ls oss://oss-odps-test/tsv_output_folder/
    2017-01-14 06:48:27 39.00B Standard oss://oss-odps-test/tsv_output_folder/.odps/.meta
    2017-01-14 06:48:12 4.80MB Standard oss://oss-odps-test/tsv_output_folder/.odps/20170113224724561g9m6csz7/M1_0_0-0.tsv
    2017-01-14 06:48:05 4.78MB Standard oss://oss-odps-test/tsv_output_folder/.odps/20170113224724561g9m6csz7/M1_1_0-0.tsv
    2017-01-14 06:47:48 4.79MB Standard oss://oss-odps-test/tsv_output_folder/.odps/20170113224724561g9m6csz7/M1_2_0-0.tsv
    ...
    ```

    The folder tsv\_output\_folder under the OSS bucket oss-odps-test specified by LOCATION contains the .odps folder which includes some .tsv files and a .meta file. Similar file structures are specific to MaxCompute's output to OSS:

    -   When you use MaxCompute to execute INSERT INTO/OVERWRITE on an external table and write to an OSS address, all of the data is writen to a .odps folder in the specified LOCATION.
    -   The.meta file in the .odps folder is an extra macro data file written by MaxCompute to record the valid data in the current folder. Typically, if the INSERT operation is successful, all the data in the current folder is valid. The macro data only needs to be parsed when a job fails. If an operation fails midway or is killed, you can simply re-execute the INSERT OVERWRITE statement.
    -   If it is a partition table, A corresponding partition sub-directory is generated based on the partition value specified by the insert statement under the fig folder and then the partition sub-directory inside is '.odps 'folder. For example, `test/tsv_output_folder/first-level partition name = partition value/.odps/20170113224724561g9m6csz7/M1_2_0-0.tsv`.
    For the TSV/CSV storagehandler processing built in by MaxCompute, the number of files generated is corresponding to the corresponding SQL

    If the `INSER OVERWITE ... SELECT ... FROM ... ;`operation allocates 1000 mappers on the source data table \(from\_tablename\), 1000 TSV/CSV files will be generated.


## Output to OSS via custom storagehandler {#section_tbq_44b_wdb .section}

In addition to using the built-in StorageHandler to implement the output TSV/CSV common text format on the OSS, the MaxCompute unstructured framework provides a general-purpose SDK that supports external output of custom data format files.

As well as the built-in StorageHandler, you need to "Create an External TABLE" And then output data to OSS through INSERT operation of External Table. The difference is that when creating external tables, STORED BY is required to specify custom StorageHandler.

**Note:** The MaxCompute unstructured framework describes the processing of varieties of data storage formats through an interface called StorageHandler. Specifically, the StorageHandler acts as a Wrapper class, allowing you to specify a custom Exatractor\(for data reading, parsing, processing, etc\) And Outputer\(for data processing and output, etc \). Custom StorageHandler should inherit Odps StorageHandler and implement getExtractorClass and getOutputerClass interfaces.

Next, we use the example of [Access OSS data](intl.en-US/User Guide/External table/Access OSS unstructured data.md). OSS unstructured data of "Custom Extractor accesses OSS", to show how MaxCompute can output data to OSS txt file by customizing StorageHandler, and use'|'as column delimiter and'\\ n' as line breaker.

**Note:** [After the MaxCompute Studio](../../../../intl.en-US/Tools and Downloads/MaxCompute Studio/What is Studio.md)is configured with [MaxCompute Java module](../../../../intl.en-US/Tools and Downloads/MaxCompute Studio/Developing Java/Create MaxCompute Java Module.md), you can see the corresponding sample code in examples. Or click [here](https://github.com/aliyun/aliyun-odps-java-sdk/tree/master/odps-sdk-impl/odps-udf-example/src/main/java/com/aliyun/odps/udf/example/text) to see the complete code.

-   Define Outputer 

    Both output logic must implement the outputer interface:

    ``` {#codeblock_yt7_brd_gtp}
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
            System.out.println("Extractor using delimiter [" + this.delimiter + "].");
        }
        @Override
        public void close() {
            // no-op
        }
        private String recordToString(Record record){
            StringBuilder sb = new StringBuilder();
            for (int i = 0; i &lt; record.getColumnCount(); i++)
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

    All output logic must call the Outputer API. There are three outputer APIs \(setup, output, and close\) which all correspond to the three extractor APIs \(setup, extract, and close\). Where setup\(\) and close\(\) are called only once in an outputer. The user may perform initialization preparation in setup. Furthermore, the three parameters returned by setup\(\) must be saved as outputer class variables to be used in the output\(\) or close\(\) APIs. The interface, close \(\), is used to sweep the end of the Code.

    Typically, most of the data processing occurs in the output \(Record\) interface. The MaxCompute system calls output \(Record\) Once based on each input record processed by the current outputer assignment \). Assuming that when an output \(Record\) call returns, the Code has already consumed the Record, so after the current output \(Record\) returns, the system uses the memory used by the record for it, so when the information in Record is used across multiple output\(\) function calls, the record for the current process needs to be invoked.clone\(\) method to save the current record.

-   Define Extractor 

    Exatrractor is used for Data Reading, parsing, processing, and so on, if the output tables eventually do not need to be read by MaxCompute and so on, you do not need to define them.

    ``` {#codeblock_9rz_9po_x95}
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
            if (this.attributes.getRecordColumns().length ! = 0 ){
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
                for(int i = 0; i &lt; parts.length; i++){
                    // only parse data in columns indexed by output indexes
                    if (index &lt; outputIndexes.length && i == outputIndexes[index]){
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
                            Default:
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

    For more information, see [Accessing the OSS unstructured data](intl.en-US/User Guide/External table/Access OSS unstructured data.md) documentation.

-   Define StorageHandler 

    ``` {#codeblock_8z3_rcc_s3q}
    package com.aliyun.odps.examples.unstructured.text;
    import com.aliyun.odps.udf.Extractor;
    import com.aliyun.odps.udf.OdpsStorageHandler;
    import com.aliyun.odps.udf.Outputer;
    public class TextStorageHandler extends OdpsStorageHandler {
        @Override
        public Class&lt;? extends Extractor&gt; getExtractorClass() {
            return TextExtractor.class;
        }
        @Override
        public Class&lt;? extends Outputer> getOutputerClass() {
            return TextOutputer.class;
        }
    }
    ```

    If the table does not need to be read, you do not need to specify an Extractor interface.

-   Compile and package 

    Compile your custom code into a package and upload it to MaxCompute. If the jar package is named 'odps-TextStorageHandler.jar', upload to MaxCompute

    ``` {#codeblock_58m_2ad_3r6}
    add jar odps-TextStorageHandler.jar;
    ```

-   Creating External tables 

    Like using the built-in StorageHandler, an External table needs to be created, the difference is that this time you need to specify that the data is output to an external table, using a custom StorageHandler.

    ``` {#codeblock_kmf_h5j_psx}
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

    **Note:** If you need `odps.properties.rolearn` property, for more information, see custom authorization for STS mode authorization of [Access the OSS unstructured data](intl.en-US/User Guide/External table/Access OSS unstructured data.md). If not, you can refer to one-click authorization or use clear-text AK on top of location.

-   Write unstructured files into External Table using INSERT 

    After creating an external table Association on the OSS storage path by customizing the storagehandler, you can do a standard SQL insert override/insert into operation on the External table to output both data to OSS, in the same way as the built-in storagehandler:

    ``` {#codeblock_l5v_8ua_9zb}
    INSERT OVERWRITE|INTO TABLE &lt;external_tablename&gt; [PARTITION (partcol1=val1, partcol2=val2 ...)]
    Select_statement
    FROM &lt;from_tablename&gt; 
    [WHERE where_condition];
    							
    ```

    When the insert operation is successful,as the built-in StorageHandler, you can see a series of files generated in the OSS corresponding LOCATION path .odps folder.


