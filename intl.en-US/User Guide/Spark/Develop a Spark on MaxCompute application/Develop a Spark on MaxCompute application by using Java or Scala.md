# Develop a Spark on MaxCompute application by using Java or Scala {#concept_263587 .concept}

This topic describes how to develop a Spark on MaxCompute application by using Java or Scala.

## Download an example project {#section_5ee_u7b_69f .section}

You can run the following commands to download an example project:

``` {#codeblock_cdl_1q8_s97}
git clone git@github.com:aliyun/aliyun-cupid-sdk.git
cd aliyun-cupid-sdk
git checkout 3.3.3-public
# Download an example project for Spark-2.x.
cd spark/spark-2.x/spark-examples
# Download an example project for Spark-1.x.
cd spark/spark-1.x/spark-examples
# Package data to create a shaded JAR package in the target directory.
mvn clean package
```

## Configure dependencies for Spark-1.x {#section_eup_qmp_ec4 .section}

If you want to submit your Spark-1.x application by using the Spark on MaxCompute client, you must add the following dependencies to the pom.xml file:

``` {#codeblock_djj_tvb_2k2}
<properties>
    <spark.version>1.6.3</spark.version>
    <cupid.sdk.version>3.3.3-public</cupid.sdk.version>
    <scala.version>2.10.4</scala.version>
    <scala.binary.version>2.10</scala.binary.version>
</properties>
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_${scala.binary.version}</artifactId>
    <version>${spark.version}</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-sql_${scala.binary.version}</artifactId>
    <version>${spark.version}</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-mllib_${scala.binary.version}</artifactId>
    <version>${spark.version}</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming_${scala.binary.version}</artifactId>
    <version>${spark.version}</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>cupid-sdk</artifactId>
    <version>${cupid.sdk.version}</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>hadoop-fs-oss</artifactId>
    <version>${cupid.sdk.version}</version>
</dependency>
<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>odps-spark-datasource_${scala.binary.version}</artifactId>
    <version>${cupid.sdk.version}</version>
</dependency>
<dependency>
    <groupId>org.scala-lang</groupId>
    <artifactId>scala-library</artifactId>
    <version>${scala.version}</version>
</dependency>
<dependency>
    <groupId>org.scala-lang</groupId>
    <artifactId>scala-actors</artifactId>
    <version>${scala.version}</version>
</dependency>
```

**Note:** You need to set the scope parameter as follows:

-   Set it to provided for all packages that are released in the Apache Spark community, such as spark-core and spark-sql.
-   Set it to compile for the odps-spark-datasource module.

## Develop a Spark-1.x application {#section_r1l_jix_3z0 .section}

-   Develop the WordCount application.

    For this application, you will need to download [aliyun-cupid-sdk](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.3-public/spark/spark-1.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/WordCount.scala).

    To submit the code, follow these steps:

    1.  Build the aliyun-cupid-sdk module.
    2.  Configure the spark.defaults.conf file.
    3.  Run the following script:

        ``` {#codeblock_r4y_om0_guw}
        bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.WordCount \
        ${path to aliyun-cupid-sdk}/spark/spark-1.x/spark-examples/target/spark-examples_2.10-version-shaded.jar
        ```

-   Develop the Spark SQL application on MaxCompute tables.

    For this application, you will need to download [aliyun-cupid-sdk](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.3-public/spark/spark-1.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/sparksql/SparkSQL.scala).

    To submit the code, follow these steps:

    1.  Build the aliyun-cupid-sdk module.
    2.  Configure the spark.defaults.conf file.
    3.  Run the following script:

        ``` {#codeblock_jek_5ri_6ma}
        bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.sparksql.SparkSQL \
        ${path to aliyun-cupid-sdk}/spark/spark-1.x/spark-examples/target/spark-examples_2.10-version-shaded.jar
        ```

    **Note:** 

    -   If the "Table Not Found" error is returned, the table you specify in the code cannot be found in the MaxCompute project.
    -   You can develop a Spark SQL application for the target table with reference to various APIs in the code.
-   Develop the GraphX PageRank application.

    For this application, you will need to download [aliyun-cupid-sdk](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.3-public/spark/spark-1.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/graphx/PageRank.scala).

    To submit the code, follow these steps:

    1.  Build the aliyun-cupid-sdk module.
    2.  Configure the spark.defaults.conf file.
    3.  Run the following script:

        ``` {#codeblock_vpt_jw5_adv}
        bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.graphx.PageRank \
        ${path to aliyun-cupid-sdk}/spark/spark-1.x/spark-examples/target/spark-examples_2.10-version-shaded.jar
        ```

-   Develop the MLlib Kmeans-ON-OSS application.

    For this application, you will need to download [aliyun-cupid-sdk](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.3-public/spark/spark-1.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/mllib/KmeansModelSaveToOss.scala).

    **Note:** Before you submit the code, make sure that you enter the following OSS account information in the code:

    ``` {#codeblock_p7x_6x8_pba}
    conf.set("spark.hadoop.fs.oss.accessKeyId", "***")
    conf.set("spark.hadoop.fs.oss.accessKeySecret", "***")
    conf.set("spark.hadoop.fs.oss.endpoint", "oss-cn-hangzhou-zmf.aliyuncs.com")
    ```

    To submit the code, follow these steps:

    1.  Build the aliyun-cupid-sdk module.
    2.  Configure the spark.defaults.conf file.
    3.  Run the following script:

        ``` {#codeblock_o0l_hkm_lqc}
        bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.mllib.KmeansModelSaveToOss \
        ${path to aliyun-cupid-sdk}/spark/spark-1.x/spark-examples/target/spark-examples_2.10-version-shaded.jar
        ```

-   Devel the OSS UnstructuredData application.

    For this application, you will need to download [aliyun-cupid-sdk](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.3-public/spark/spark-1.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/oss/SparkUnstructuredDataCompute.scala).

    **Note:** Before you submit the code, make sure that you enter the following OSS account information in the code:

    ``` {#codeblock_38w_dnf_5yf}
    conf.set("spark.hadoop.fs.oss.accessKeyId", "***")
    conf.set("spark.hadoop.fs.oss.accessKeySecret", "***")
    conf.set("spark.hadoop.fs.oss.endpoint", "oss-cn-hangzhou-zmf.aliyuncs.com")
    ```

    To submit the code, follow these steps:

    1.  Build the aliyun-cupid-sdk module.
    2.  Configure the spark.defaults.conf file.
    3.  Run the following script:

        ``` {#codeblock_6q1_48c_ovk}
        bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.oss.SparkUnstructuredDataCompute \
        ${path to aliyun-cupid-sdk}/spark/spark-1.x/spark-examples/target/spark-examples_2.10-version-shaded.jar
        ```


## Configure dependencies for Spark-2.x {#section_ttc_w11_fd3 .section}

If you want to submit your Spark-2.x application by using the Spark on MaxCompute client, you must add the following dependencies to the pom.xml file:

``` {#codeblock_t50_tzp_5t7}
<properties>
    <spark.version>2.3.0</spark.version>
    <cupid.sdk.version>3.3.3-public</cupid.sdk.version>
    <scala.version>2.11.8</scala.version>
    <scala.binary.version>2.11</scala.binary.version>
</properties>
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_${scala.binary.version}</artifactId>
    <version>${spark.version}</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-sql_${scala.binary.version}</artifactId>
    <version>${spark.version}</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-mllib_${scala.binary.version}</artifactId>
    <version>${spark.version}</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming_${scala.binary.version}</artifactId>
    <version>${spark.version}</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>cupid-sdk</artifactId>
    <version>${cupid.sdk.version}</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>hadoop-fs-oss</artifactId>
    <version>${cupid.sdk.version}</version>
</dependency>
<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>odps-spark-datasource_${scala.binary.version}</artifactId>
    <version>${cupid.sdk.version}</version>
</dependency>
<dependency>
    <groupId>org.scala-lang</groupId>
    <artifactId>scala-library</artifactId>
    <version>${scala.version}</version>
</dependency>
<dependency>
    <groupId>org.scala-lang</groupId>
    <artifactId>scala-actors</artifactId>
    <version>${scala.version}</version>
</dependency>
```

**Note:** You need to set the scope parameter as follows:

-   Set it to provided for all packages that are released in the Apache Spark community, such as spark-core and spark-sql.
-   Set it to compile for the odps-spark-datasource module.

## Develop a Spark-2.x application {#section_dx2_04s_i2u .section}

-   Develop the WordCount application.

    For this application, you will need to download [aliyun-cupid-sdk](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.3-public/spark/spark-2.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/WordCount.scala).

    To submit the code, follow these steps:

    1.  Build the aliyun-cupid-sdk module.
    2.  Configure the spark.defaults.conf file.
    3.  Run the following script:

        ``` {#codeblock_myw_ieg_flm}
        bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.WordCount \
        ${path to aliyun-cupid-sdk}/spark/spark-2.x/spark-examples/target/spark-examples_2.11-version-shaded.jar
        ```

-   Develop the Spark SQL application on MaxCompute tables.

    For this application, you will need to download [aliyun-cupid-sdk](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.3-public/spark/spark-2.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/sparksql/SparkSQL.scala).

    **Note:** 

    -   If the "Table Not Found" error is returned, the table you specify in the code cannot be found in the MaxCompute project.
    -   You can develop a Spark SQL application for the target table with reference to various APIs in the code.
    To submit the code, follow these steps:

    1.  Build the aliyun-cupid-sdk module.
    2.  Configure the spark.defaults.conf file.
    3.  Run the following script:

        ``` {#codeblock_cqc_nnc_dru}
        bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.sparksql.SparkSQL \
        ${path to aliyun-cupid-sdk}/spark/spark-2.x/spark-examples/target/spark-examples_2.11-version-shaded.jar
        ```

-   Develop the GraphX PageRank application.

    For this application, you will need to download [aliyun-cupid-sdk](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.3-public/spark/spark-2.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/graphx/PageRank.scala).

    To submit the code, follow these steps:

    1.  Build the aliyun-cupid-sdk module.
    2.  Configure the spark.defaults.conf file.
    3.  Run the following script:

        ``` {#codeblock_voy_9ft_kkc}
        bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.graphx.PageRank \
        ${path to aliyun-cupid-sdk}/spark/spark-2.x/spark-examples/target/spark-examples_2.11-version-shaded.jar
        ```

-   Develop the MLlib Kmeans-ON-OSS application.

    For this application, you will need to download [aliyun-cupid-sdk](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.3-public/spark/spark-2.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/mllib/KmeansModelSaveToOss.scala).

    **Note:** Before you submit the code, make sure that you enter the following OSS account information in the code:

    ``` {#codeblock_78k_oas_u5s}
    val spark = SparkSession
          .builder()
          .config("spark.hadoop.fs.oss.accessKeyId", "***")
          .config("spark.hadoop.fs.oss.accessKeySecret", "***")
          .config("spark.hadoop.fs.oss.endpoint", "oss-cn-hangzhou-zmf.aliyuncs.com")
          .appName("KmeansModelSaveToOss")
          .getOrCreate()
    ```

    To submit the code, follow these steps:

    1.  Build the aliyun-cupid-sdk module.
    2.  Configure the spark.defaults.conf file.
    3.  Run the following script:

        ``` {#codeblock_xij_upd_atx}
        bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.mllib.KmeansModelSaveToOss \
        ${path to aliyun-cupid-sdk}/spark/spark-2.x/spark-examples/target/spark-examples_2.11-version-shaded.jar
        ```

-   Develop the OSS UnstructuredData application.

    For this application, you will need to download [aliyun-cupid-sdk](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.3-public/spark/spark-2.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/oss/SparkUnstructuredDataCompute.scala).

    **Note:** Before you submit the code, make sure that you enter the following OSS account information in the code:

    ``` {#codeblock_lb1_yg0_ahc}
    val spark = SparkSession
          .builder()
          .config("spark.hadoop.fs.oss.accessKeyId", "***")
          .config("spark.hadoop.fs.oss.accessKeySecret", "***")
          .config("spark.hadoop.fs.oss.endpoint", "oss-cn-hangzhou-zmf.aliyuncs.com")
          .appName("SparkUnstructuredDataCompute")
          .getOrCreate()
    ```

    To submit the code, follow these steps:

    1.  Build the aliyun-cupid-sdk module.
    2.  Configure the spark.defaults.conf file.
    3.  Run the following script:

        ``` {#codeblock_7t4_nd2_kcd}
        bin/spark-submit --master yarn-cluster --class \
        com.aliyun.odps.spark.examples.oss.SparkUnstructuredDataCompute \
        ${path to aliyun-cupid-sdk}/spark/spark-2.x/spark-examples/target/spark-examples_2.11-version-shaded.jar
        ```


