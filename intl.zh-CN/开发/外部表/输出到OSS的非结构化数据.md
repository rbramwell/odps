---
keyword: OSS的非结构化数据
---

# 输出到OSS的非结构化数据

MaxCompute的非结构化框架支持您通过INSERT方式将MaxCompute的数据直接输出到OSS。MaxCompute也支持通过外部表关联OSS，进行数据输出。

输出数据到OSS分为两种情况：

-   MaxCompute内部表输出到关联OSS的外部表。
-   MaxCompute处理外部表后，结果直接输出到关联OSS的外部表。

与访问OSS数据一样，MaxCompute支持通过内置StorageHandler和自定义StorageHandler进行数据输出。

## 通过内置StorageHandler输出到OSS

使用MaxCompute内置的StorageHandler时，通过创建外部表可以非常方便的按照约定格式输出数据到OSS进行存储。

MaxCompute支持2个内置StorageHandler：

-   `com.aliyun.odps.CsvStorageHandler`定义如何读写CSV格式数据，数据格式中英文逗号`,`为列分隔符，换行符为`\n`。
-   `com.aliyun.odps.TsvStorageHandler`定义如何读写CSV格式数据，数据格式中`\t`为列分隔符，换行符为`\n`。

**示例**

1.  参考如下语句创建外部表。

    ```
    CREATE EXTERNAL TABLE [IF NOT EXISTS] <external_table>
    (<column schemas>)
    [PARTITIONED BY (partition column schemas)]
    STORED BY '<StorageHandler>'  
    [WITH SERDEPROPERTIES ('odps.properties.rolearn'='${roleran}') ]
    LOCATION 'oss://${endpoint}/${bucket}/${userfilePath}/';                       
    ```

    -   STORED BY：指定内置StorageHandler。如果需要输出到OSS上的数据文件是TSV文件，则用`com.aliyun.odps.TsvStorageHandler`；如果需要输出到OSS上的数据文件是CSV文件，则用`com.aliyun.odps.CsvStorageHandler`。
    -   WITH SERDEPROPERTIES：当使用STS模式授权的自定义授权关联OSS权限时，该参数需要指定为’odps.properties.rolearn’属性，属性值为RAM中具体的自定义角色的ARN信息。

        **说明：** 关于STS模式授权的介绍，请参见[STS模式授权](/intl.zh-CN/开发/外部表/STS模式授权.md)。

    -   LOCATION：指定对应OSS存储的文件路径。
        -   如果WITH SERDEPROPERTIES中设置了odps.properties.rolearn属性，且授权方式是STS模式授权，则LOCATION如下。

            ```
            LOCATION 'oss://${endpoint}/${bucket}/${userfilePath}/
            ```

        -   如果WITH SERDEPROPERTIES中未设置odps.properties.rolearn属性，且授权方式是采用明文AccessKey，则LOCATION如下。

            ```
            LOCATION 'oss://${accessKeyId}:${accessKeySecret}@${endpoint}/${bucket}/${userPath}/'
            ```

2.  对外部表执行INSERT操作实现数据输出到OSS。

    **说明：** 执行INSERT操作将数据输出至OSS时，单个文件的大小不能超过5 GB。

    通过外部表关联OSS存储路径后，可以对外部表进行INSERT OVERWRITE或INSERT INTO操作，即可将数据输出到OSS。SQL语法如下。

    ```
    INSERT OVERWRITE|INTO TABLE <external_tablename> [PARTITION (partcol1=val1, partcol2=val2 ...)]
    select_statement
    FROM <from_tablename> 
    [WHERE where_condition];                         
    ```

    -   from\_tablename：可以是内部表，也可以是外部表（包括关联的OSS或OTS的外部表）。
    -   INSERT结果将按照外部表STORED BY指定的StorageHandler格式（即TSV或CSV）写入OSS。INSERT操作成功后，您可以看到OSS上的对应LOCATION产生了一系列文件。例如：外部表对应的LOCATION是oss://oss-cn-hangzhou-zmf.aliyuncs.com/oss-odps-test/tsv\_output\_folder/，则在OSS对应路径中可以看到生成一系列文件，示例如下。

        ```
        osscmd ls oss://oss-odps-test/tsv_output_folder/
        2017-01-14 06:48:27 39.00B Standard oss://oss-odps-test/tsv_output_folder/.odps/.meta
        2017-01-14 06:48:12 4.80MB Standard oss://oss-odps-test/tsv_output_folder/.odps/20170113224724561g9m****/M1_0_0-0.tsv
        2017-01-14 06:48:05 4.78MB Standard oss://oss-odps-test/tsv_output_folder/.odps/20170113224724561g9m****/M1_1_0-0.tsv
        2017-01-14 06:47:48 4.79MB Standard oss://oss-odps-test/tsv_output_folder/.odps/20170113224724561g9m****/M1_2_0-0.tsv
        ...
        ```

        LOCATION指定的名为oss-odps-test的OSS bucket下的tsv\_output\_folder文件夹下产生了一个.odps文件夹，包含部分.tsv与.meta文件。类似的文件结构是MaxCompute往OSS上输出所特有的：

        -   通过MaxCompute对一个OSS地址，使用INSERT INTO或INSERT OVERWRITE外部表来做输出操作，所有的数据将在指定的LOCATION下的.odps文件夹产生。
        -   .odps文件夹中的.meta文件为MaxCompute额外写出的宏数据文件，用于记录当前文件夹中有效的数据。正常情况下，如果INSERT操作成功，可以认为当前文件夹的所有数据均是有效数据。只有在有作业失败的情况下，需要对该宏数据进行解析。即使是在作业中途失败或被中止的情况下，对于INSERT OVERWRITE操作，再成功运行一次即可。
        -   如果是分区表，将在tsv\_output\_folder文件夹下根据INSERT语句指定的分区值生成对应的分区子目录。分区子目录中包括.odps文件夹。例如`test/tsv_output_folder/一级分区名=分区值/n级分区名=分区值/.odps/20170113224******/M1_2_0-0.tsv`。
    对于MaxCompute内置的TSV或CSV StorageHandler，处理产生的文件数目与对应SQL的并发度相同。

    如果`INSER OVERWITE ... SELECT ... FROM ...;`操作在源数据表from\_tablename上分配了1000个Mapper，则最后将产生1000个TSV或CSV文件。

    如果您需要控制TSV文件的数目，可以通过MaxCompute的各种灵活语义和配置来实现。如果Outputer在Mapper里，可以通过调整`odps.stage.mapper.split.size`的大小来控制Mapper的并发数，从而调整产生的文件数目。如果Outputer在Reducer或Joiner里，也可以分别通过`odps.stage.reducer.num`和`odps.stage.joiner.num`来调整。


## 通过自定义StorageHandler输出到OSS

除了使用内置的StorageHandler来实现在OSS上输出TSV或CSV常见文本格式，MaxCompute非结构化框架还提供了通用的SDK自定义StorageHandler，支持对外输出自定义数据格式文件。

与内置StorageHandler一样，您需要先创建外部表，再通过对外部表的INSERT操作实现数据输出到OSS。不同点在于创建外部表时，STORED BY需要指定自定义的StorageHandler。

**说明：** MaxCompute非结构化框架通过StorageHandler接口，来描述对各种数据存储格式的处理。StorageHandler作为一个wrapper class，允许您指定自定义的Exatractor（用于数据的读入、解析、处理等）和Outputer（用于数据的处理和输出等）。自定义的StorageHandler应该继承OdpsStorageHandler，实现getExtractorClass和getOutputerClass接口。

下面引用[访问OSS非结构化数据](/intl.zh-CN/开发/外部表/内置Extractor访问OSS.md)中自定义Extractor访问OSS的TextStorageHandler例子，为您介绍MaxCompute如何通过自定义StorageHandler将数据输出到OSS的TXT文件，以\|为列分隔符，以\\n为换行符。

**说明：** [MaxCompute Studio](/intl.zh-CN/工具及下载/MaxCompute Studio/认识MaxCompute Studio.md)配置好[MaxCompute Java Module](/intl.zh-CN/工具及下载/MaxCompute Studio/开发Java程序/创建MaxCompute Java Module.md)后，您可以在Examples中看到对应的示例代码，或单击[此处](https://github.com/aliyun/aliyun-odps-java-sdk/tree/master/odps-sdk-impl/odps-udf-example/src/main/java/com/aliyun/odps/udf/example/text)也可以看到完整代码。

1.  定义Outputer。

    输出逻辑都必须实现Outputer接口。

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
            System.out.println("Extractor using delimiter [" + this.delimiter + "].");
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
                if (i != record.getColumnCount() - 1){
                    sb.append(this.delimiter);
                }
            }
            sb.append("\n");
            return sb.toString();
        }
    }
    ```

    Outputer接口有三个：setup、output和close，与Extractor的setup、extract和close三个接口基本上对称。其中：`setup()`和`close()`在一个Outputer中只会调用一次，可以在`setup()`里做初始化准备工作。通常，您需要把`setup()`传递进来的这三个参数保存成Outputer的`class variable`，方便在之后`output()`或`close()`接口中使用。而`close()`接口用于代码的扫尾工作。

    通常，数据处理发生在`output(Record)`接口内。MaxCompute根据当前Outputer分配处理的每个输入Record调用一次`output(Record)`。假设，在一个`output(Record)`调用返回的时候，代码已经消费完该Record，则在当前`output(Record)`返回后，系统会将Record所使用的内存作它用。因此，当Record中的信息在跨多个`output()`函数调用时，需要调用当前处理的`record.clone()`方法，将当前Record保存下来。

    **说明：** 使用外表自定义storagehandler实现Outputer接口时，`Outputer.output(Record record)`中传入的Record是Outputer的上一个操作产生的记录，这个列名发生了变化。这些列名不保证固定，例如表达式`some_function(column_a)`产生的列名是一个临时列名。

    因此，当您使用`record.get(列名)`方式来获取列的内容时需特别注意，建议改用`record.get(index)`方式获取。

2.  定义Extractor。

    Exatractor用于数据的读入、解析、处理等，如果输出的表最终不需要再通过MaxCompute进行读取等，无需定义Extractor。

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
            if ( columnDelimiter != null)
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
            if (this.attributes.getRecordColumns().length != 0){
                // string copies are needed, not the most efficient one, but suffice as an example here
                String[] parts = line.split(columnDelimiter);
                int[] outputIndexes = this.attributes.getNeededIndexes();
                if (outputIndexes == null){
                    throw new IllegalArgumentException("No outputIndexes supplied.");
                }
                if (outputIndexes.length != outputColumns.length){
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
            while (currentReader != null) {
                String line = currentReader.readLine();
                if (line != null) {
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

    详情请参见[访问OSS非结构化数据](/intl.zh-CN/开发/外部表/内置Extractor访问OSS.md)。

3.  定义StorageHandler。

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

    若表无需读取，无需指定Extractor接口。

4.  编译打包。

    将自定义代码编译打包，并作为JAR资源上传到MaxCompute。例如JAR包名为odps-TextStorageHandler.jar，则上传为MaxCompute Resource的示例如下。

    ```
    add jar odps-TextStorageHandler.jar;
    ```

5.  创建外部表。

    与使用内置StorageHandler类似，需要建立一个外部表，不同的是指定数据输出到外部表时，使用自定义的StorageHandler。

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

6.  通过对External Table的INSERT操作实现数据输出到OSS。

    通过自定义StorageHandler创建External Table并关联OSS存储路径后，您可以对External Table进行标准的SQL INSERT OVERWRITE或INSERT INTO操作，将数据输出到OSS的方式与内置StorageHandler相同。

    ```
    INSERT OVERWRITE|INTO TABLE <external_tablename> [PARTITION (partcol1=val1, partcol2=val2 ...)]
    select_statement
    FROM <from_tablename> 
    [WHERE where_condition];       
    ```

    INSERT操作执行成功后，与内置StorageHandler一样，可以在OSS对应LOCATION路径看到.odps文件夹中生成了一系列文件。


