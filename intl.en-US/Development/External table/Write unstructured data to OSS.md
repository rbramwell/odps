---
keyword: unstructured OSS data
---

# Write unstructured data to OSS

The unstructured data processing framework of MaxCompute enables you to write data from MaxCompute to Object Storage Service \(OSS\) by using INSERT statements. You can also associate MaxCompute foreign tables with OSS to write data from MaxCompute to OSS.

MaxCompute writes data to OSS by using foreign tables in one of the following scenarios:

-   MaxCompute writes data from an internal table to OSS by using a foreign table that is associated with OSS.
-   After MaxCompute processes data in a data source by using a foreign table that is associated with the data source, MaxCompute writes the processed data to OSS by using another foreign table that is associated with OSS.

Similar to the way that you access OSS data, you can write data to OSS by using a built-in or custom storage handler in MaxCompute.

## Write data to OSS by using a built-in storage handler

When you create a foreign table, you can specify a built-in storage handler in MaxCompute to associate the foreign table with OSS. Then, you can write data from MaxCompute to OSS in the specified format by using the foreign table.

MaxCompute supports two built-in storage handlers:

-   `com.aliyun.odps.CsvStorageHandler` defines how MaxCompute reads and writes data in the comma-separated values \(CSV\) format. When this storage handler is used, the column delimiter is a comma \(`,`\) and the line break is `\n`.
-   `com.aliyun.odps.TsvStorageHandler` defines how MaxCompute reads and writes data in the tab-separated values \(TSV\) format. When this storage handler is used, the column delimiter is `\t` and the line break is `\n`.

**Examples**

1.  Execute the following statement to create a foreign table:

    ```
    CREATE EXTERNAL TABLE [IF NOT EXISTS] <external_table>
    (<column schemas>)
    [PARTITIONED BY (partition column schemas)]
    STORED BY '<StorageHandler>'  
    [WITH SERDEPROPERTIES ('odps.properties.rolearn'='${roleran}') ]
    LOCATION 'oss://${endpoint}/${bucket}/${userfilePath}/';                       
    ```

    -   STORED BY: specifies the name of the built-in storage handler. To write data to OSS as a TSV file, use `com.aliyun.odps.TsvStorageHandler`. To write data to OSS as a CSV file, use `com.aliyun.odps.CsvStorageHandler`.
    -   WITH SERDEPROPERTIES: specifies the properties of the foreign table. To allow MaxCompute to assume a RAM role to access OSS by using temporary security credentials provided by Security Token Service \(STS\), specify the odps.properties.rolearn property in this clause. Set the property value to the Alibaba Cloud Resource Name \(ARN\) of the specific RAM role.

        **Note:** For more information about STS authorization, see [STS authorization](/intl.en-US/Development/External table/STS authorization.md).

    -   LOCATION: specifies the file storage path in OSS.
        -   If the odps.properties.rolearn property is specified in the WITH SERDEPROPERTIES clause and MaxCompute is authorized to use STS to access OSS, set LOCATION in the following format:

            ```
            LOCATION 'oss://${endpoint}/${bucket}/${userfilePath}/
            ```

        -   If the odps.properties.rolearn property is not specified in the WITH SERDEPROPERTIES clause and MaxCompute is authorized to use a plaintext AccessKey pair to access OSS, set LOCATION in the following format:

            ```
            LOCATION 'oss://${accessKeyId}:${accessKeySecret}@${endpoint}/${bucket}/${userPath}/'
            ```

2.  Perform an INSERT operation on a foreign table to write data to OSS.

    **Note:** When you perform an INSERT operation on a foreign table to write data to files in OSS, the size of each file in OSS cannot exceed 5 GB.

    After you create a foreign table that is associated with a storage path in OSS, you can perform an INSERT OVERWRITE or INSERT INTO operation on the foreign table by using the following syntax to write data to OSS:

    ```
    INSERT OVERWRITE|INTO TABLE <external_tablename> [PARTITION (partcol1=val1, partcol2=val2 ...)]
    select_statement
    FROM <from_tablename> 
    [WHERE where_condition];                         
    ```

    -   You can specify from\_tablename to an internal table or a foreign table, for example, a foreign table that is associated with OSS or Tablestore.
    -   The INSERT operation writes data to OSS in the TSV or CSV format based on the storage handler you specified in the STORED BY clause when you created the foreign table. After the INSERT operation is complete, a series of files are generated in the corresponding OSS storage path specified by LOCATION. For example, if LOCATION is set to oss://oss-cn-hangzhou-zmf.aliyuncs.com/oss-odps-test/tsv\_output\_folder/ for a foreign table, you can run the following ls command to view the files generated in the corresponding storage path in OSS:

        ```
        osscmd ls oss://oss-odps-test/tsv_output_folder/
        2017-01-14 06:48:27 39.00B Standard oss://oss-odps-test/tsv_output_folder/.odps/.meta
        2017-01-14 06:48:12 4.80MB Standard oss://oss-odps-test/tsv_output_folder/.odps/20170113224724561g9m****/M1_0_0-0.tsv
        2017-01-14 06:48:05 4.78MB Standard oss://oss-odps-test/tsv_output_folder/.odps/20170113224724561g9m****/M1_1_0-0.tsv
        2017-01-14 06:47:48 4.79MB Standard oss://oss-odps-test/tsv_output_folder/.odps/20170113224724561g9m****/M1_2_0-0.tsv
        ...
        ```

        In this example, a .odps folder is generated under the tsv\_output\_folder folder in the OSS bucket oss-odps-test specified by LOCATION. The .odps folder contains a series of .tsv files and a .meta file. Similar file structures are dedicated for the data that is written to OSS from MaxCompute.

        -   When you perform an INSERT INTO or INSERT OVERWRITE operation on a MaxCompute foreign table to write data from MaxCompute to an OSS storage path specified by LOCATION, all data is written to files in the .odps folder in the OSS storage path.
        -   The .meta file in the .odps folder is an extra macro data file written by MaxCompute to record valid data in the current folder. If the INSERT operation is successful, all data in the current folder is valid. You are required to parse the macro data file only when the operation fails. If an INSERT OVERWRITE operation fails or is terminated, you can perform the operation again.
        -   If the foreign table is a partitioned table, a corresponding partitioned subfolder is generated under the tsv\_output\_folder folder based on the partition key value that is specified by the INSERT statement. The partitioned subfolder contains the .odps folder. Example: `test/tsv_output_folder/Name of the level-1 partition=Partition key value/Name of the partition at a specific level=Partition key value/.odps/20170113224******/M1_2_0-0.tsv`.
    If you use the built-in TSV or CSV storage handler in MaxCompute, the number of files generated in the storage path in OSS is the same as the number of concurrent jobs in the corresponding SQL stage.

    If the `INSERT OVERWITE ... SELECT ... FROM ...;` operation allocates 1,000 mappers to the source table specified by from\_tablename, 1,000 TSV or CSV files will be generated.

    You can use the flexible semantics and configurations of MaxCompute to limit the number of TSV files. If an outputer is run in a mapper, you can adjust the number of generated files by using the following method: Change the value of `odps.stage.mapper.split.size` to adjust the number of concurrent mappers. If an outputer is run in a reducer or joiner, you can adjust the number of generated files by changing the value of `odps.stage.reducer.num` or `odps.stage.joiner.num` respectively.


## Write data to OSS by using a custom storage handler

In addition to the built-in storage handlers that are used to write data to OSS in the TSV or CSV format, the unstructured data processing framework of MaxCompute provides a common SDK for you to customize storage handlers to write data to external data sources in custom formats.

You can use a custom storage handler in a similar way that you use a built-in storage handler. To write data to OSS in a custom format, specify the corresponding custom storage handler in the STORED BY clause when you create a foreign table that is associated with OSS and then perform an INSERT operation on the foreign table.

**Note:** The unstructured data processing framework of MaxCompute uses a StorageHandler class to describe the processing of various data storage formats. StorageHandler acts as a wrapper class that enables you to specify custom extractors and outputers. An extractor reads, parses, and extracts data, whereas an outputer processes and writes data. A custom storage handler inherits the OdpsStorageHandler class to implement the getExtractorClass and getOutputerClass methods.

By using the TextStorageHandler class that customizes an extractor to access OSS as an example, the following section demonstrates how MaxCompute can use a custom storage handler to write data to OSS as text files, with \| as the column delimiter and \\n as the line break. For more information about the TextStorageHandler class, see [Access OSS unstructured data](/intl.en-US/Development/External table/Access OSS data by using the built-in extractor.md).

**Note:** After a [MaxCompute Java module](/intl.en-US/Tools and Downloads/MaxCompute Studio/Developing Java/Create MaxCompute Java Module.md) is created in [MaxCompute Studio](/intl.en-US/Tools and Downloads/MaxCompute Studio/What is Studio.md), you can view sample code files in the examples folder. To view complete code files, click [here](https://github.com/aliyun/aliyun-odps-java-sdk/tree/master/odps-sdk-impl/odps-udf-example/src/main/java/com/aliyun/odps/udf/example/text).

1.  Define an outputer.

    An Outputer class must be implemented based on the output logic.

    ```
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

    The setup\(\), output\(\), and close\(\) methods of an Outputer class correspond to the setup\(\), extract\(\), and close\(\) methods of an Extractor class respectively. You can call the `setup()` and `close()` methods only once in an Outputer class. You can perform initialization tasks in the `setup()` method. You must save the three parameters returned by the `setup()` method as the `variables` of the Outputer class for later usage in the `output()` or `close()` method. The `close()` method is used to mark the end of the code.

    Data processing occurs in the `output(Record)` method. MaxCompute calls the `output(Record)` method once based on each input record processed by the current outputer. Assume that the code has consumed a record when the corresponding `output(Record)` call returns a response. In this case, MaxCompute uses the memory that was used by the record for other purposes.`` Therefore, if the information in a record is called across multiple `output()` methods, you must call the `record.clone()` method to save the current record.

    **Note:** When you use a custom storage handler for a foreign table to implement an Outputer class, the record generated by the previous operation of the Outputer class is passed to the `Outputer.output(Record record)` method. In this case, the column name has changed. Column names are not fixed. For example, the column name generated by the `some_function(column_a)` method is a temporary column name.

    Therefore, we recommend that you use `record.get(index)` instead of `record.get(column name)` to obtain the content of a column.

2.  Define an extractor.

    An extractor is used to read, parse, and process data. If MaxCompute does not need to read data from output tables, you do not need to define an extractor.

    ```
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

    For more information, see [Access OSS unstructured data](/intl.en-US/Development/External table/Access OSS data by using the built-in extractor.md).

3.  Define a storage handler.

    ```
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

    If MaxCompute does not need to read data from a table, you do not need to specify an extractor.

4.  Compile and package the code.

    Compile your custom code, package the code as a JAR package, and upload the JAR package to MaxCompute. If the JAR package is named odps-TextStorageHandler.jar, you can use the following code to upload the package as a MaxCompute resource:

    ```
    add jar odps-TextStorageHandler.jar;
    ```

5.  Create a foreign table.

    You can use a custom storage handler in a similar way that you use a built-in storage handler. To write data to OSS in a custom format, you must specify the corresponding custom storage handler in the STORED BY clause when you create a foreign table that is associated with OSS.

    ```
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

6.  Perform an INSERT operation on the foreign table to write the data to OSS.

    A custom storage handler works in the same way as a built-in storage handler. After you create a foreign table and associate it with a storage path in OSS by using the custom storage handler, you can perform a standard INSERT OVERWRITE or INSERT INTO operation on the foreign table to write the data to OSS.

    ```
    INSERT OVERWRITE|INTO TABLE <external_tablename> [PARTITION (partcol1=val1, partcol2=val2 ...)]
    select_statement
    FROM <from_tablename> 
    [WHERE where_condition];       
    ```

    After the INSERT operation is complete, a series of files are generated in the .odps folder in the corresponding OSS storage path specified by LOCATION.


