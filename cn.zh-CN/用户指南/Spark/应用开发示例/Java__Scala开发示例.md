# Java/Scala开发示例 {#concept_263587 .concept}

本文为您介绍Java/Scala的开发示例。

## 配置Spark-1.x的依赖 {#section_eup_qmp_ec4 .section}

pom.xml须知：用户构建Spark应用时，通过MaxCompute提供的Spark客户端提交应用，因此需要添加以下依赖。

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

**说明：** scope的定义：

-   spark-core、spark-sql等所有Spark社区发布的包，使用providedscope。
-   odps-spark-datasource使用默认的compilescope。

## Spark-1.x案例说明 {#section_r1l_jix_3z0 .section}

-   WordCount

    [详细代码](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.3-public/spark/spark-1.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/WordCount.scala)

    提交方式

    ``` {#codeblock_jhn_84m_pky}
    Step 1. build aliyun-cupid-sdk
    Step 2. properly set spark.defaults.conf
    Step 3. bin/spark-submit --master yarn-cluster --class \
          com.aliyun.odps.spark.examples.WordCount \
          ${path to aliyun-cupid-sdk}/spark/spark-1.x/spark-examples/target/spark-examples_2.10-version-shaded.jar
    ```

-   Spark-SQL on MaxCompute Table

    [详细代码](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.3-public/spark/spark-1.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/sparksql/SparkSQL.scala)

    提交方式

    ``` {#codeblock_2b9_ros_gup}
    # 运行可能会报Table Not Found的异常，这是因为用户的MaxCompute Project中没有代码中指定的表。
    # 可参考代码中的各种接口，实现对应Table的SparkSQL应用。
    Step 1. build aliyun-cupid-sdk
    Step 2. properly set spark.defaults.conf
    Step 3. bin/spark-submit --master yarn-cluster --class \
          com.aliyun.odps.spark.examples.sparksql.SparkSQL \
          ${path to aliyun-cupid-sdk}/spark/spark-1.x/spark-examples/target/spark-examples_2.10-version-shaded.jar
    ```

-   GraphX PageRank

    [详细代码](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.3-public/spark/spark-1.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/graphx/PageRank.scala)

    提交方式

    ``` {#codeblock_ikz_341_n1i}
    Step 1. build aliyun-cupid-sdk
    Step 2. properly set spark.defaults.conf
    Step 3. bin/spark-submit --master yarn-cluster --class \
          com.aliyun.odps.spark.examples.graphx.PageRank \
          ${path to aliyun-cupid-sdk}/spark/spark-1.x/spark-examples/target/spark-examples_2.10-version-shaded.jar
    ```

-   Mllib Kmeans-ON-OSS

    [详细代码](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.3-public/spark/spark-1.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/mllib/KmeansModelSaveToOss.scala)

    提交方式

    ``` {#codeblock_3xa_k3j_369}
    # 需要填上代码中的OSS账号信息，再编译提交。
    conf.set("spark.hadoop.fs.oss.accessKeyId", "***")
    conf.set("spark.hadoop.fs.oss.accessKeySecret", "***")
    conf.set("spark.hadoop.fs.oss.endpoint", "oss-cn-hangzhou-zmf.aliyuncs.com")
    Step 1. build aliyun-cupid-sdk
    Step 2. properly set spark.defaults.conf
    Step 3. bin/spark-submit --master yarn-cluster --class \
          com.aliyun.odps.spark.examples.mllib.KmeansModelSaveToOss \
          ${path to aliyun-cupid-sdk}/spark/spark-1.x/spark-examples/target/spark-examples_2.10-version-shaded.jar
    ```

-   OSS UnstructuredData

    [详细代码](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.3-public/spark/spark-1.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/oss/SparkUnstructuredDataCompute.scala)

    提交方式

    ``` {#codeblock_syf_sbr_422}
    # 填上代码中的OSS账号信息，再编译提交。
    conf.set("spark.hadoop.fs.oss.accessKeyId", "***")
    conf.set("spark.hadoop.fs.oss.accessKeySecret", "***")
    conf.set("spark.hadoop.fs.oss.endpoint", "oss-cn-hangzhou-zmf.aliyuncs.com")
    Step 1. build aliyun-cupid-sdk
    Step 2. properly set spark.defaults.conf
    Step 3. bin/spark-submit --master yarn-cluster --class \
          com.aliyun.odps.spark.examples.oss.SparkUnstructuredDataCompute \
          ${path to aliyun-cupid-sdk}/spark/spark-1.x/spark-examples/target/spark-examples_2.10-version-shaded.jar
    ```


## 配置Spark-2.x的依赖 {#section_ttc_w11_fd3 .section}

pom.xml须知：用户构建Spark应用时，通过MaxCompute提供的Spark客户端提交应用，因此需要添加以下依赖。

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

**说明：** scope的定义：

-   spark-core、spark-sql等所有Spark社区发布的包，使用providedscope。
-   odps-spark-datasource使用默认的compilescope。

## Spark-2.x案例说明 {#section_dx2_04s_i2u .section}

-   WordCount

    [详细代码](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.3-public/spark/spark-2.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/WordCount.scala)

    提交方式

    ``` {#codeblock_gxb_056_kf7}
    Step 1. build aliyun-cupid-sdk
    Step 2. properly set spark.defaults.conf
    Step 3. bin/spark-submit --master yarn-cluster --class \
          com.aliyun.odps.spark.examples.WordCount \
          ${path to aliyun-cupid-sdk}/spark/spark-2.x/spark-examples/target/spark-examples_2.11-version-shaded.jar
    ```

-   Spark-SQL on MaxCompute Table

    [详细代码](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.3-public/spark/spark-2.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/sparksql/SparkSQL.scala)

    提交方式

    ``` {#codeblock_02u_5ul_eul}
    # 运行可能会报Table Not Found的异常，因为用户的MaxCompute Project中没有代码中指定的表。
    # 可以参考代码中的各种接口，实现对应Table的SparkSQL应用。
    Step 1. build aliyun-cupid-sdk
    Step 2. properly set spark.defaults.conf
    Step 3. bin/spark-submit --master yarn-cluster --class \
          com.aliyun.odps.spark.examples.sparksql.SparkSQL \
          ${path to aliyun-cupid-sdk}/spark/spark-2.x/spark-examples/target/spark-examples_2.11-version-shaded.jar
    ```

-   GraphX PageRank

    [详细代码](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.3-public/spark/spark-2.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/graphx/PageRank.scala)

    提交方式

    ``` {#codeblock_ghl_ybc_qe3}
    Step 1. build aliyun-cupid-sdk
    Step 2. properly set spark.defaults.conf
    Step 3. bin/spark-submit --master yarn-cluster --class \
          com.aliyun.odps.spark.examples.graphx.PageRank \
          ${path to aliyun-cupid-sdk}/spark/spark-2.x/spark-examples/target/spark-examples_2.11-version-shaded.jar
    ```

-   Mllib Kmeans-ON-OSS

    [详细代码](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.3-public/spark/spark-2.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/mllib/KmeansModelSaveToOss.scala)

    提交方式

    ``` {#codeblock_8oj_a07_0dz}
    # 需要填上代码中的OSS账号信息，再编译提交。
    val spark = SparkSession
          .builder()
          .config("spark.hadoop.fs.oss.accessKeyId", "***")
          .config("spark.hadoop.fs.oss.accessKeySecret", "***")
          .config("spark.hadoop.fs.oss.endpoint", "oss-cn-hangzhou-zmf.aliyuncs.com")
          .appName("KmeansModelSaveToOss")
          .getOrCreate()
    Step 1. build aliyun-cupid-sdk
    Step 2. properly set spark.defaults.conf
    Step 3. bin/spark-submit --master yarn-cluster --class \
          com.aliyun.odps.spark.examples.mllib.KmeansModelSaveToOss \
          ${path to aliyun-cupid-sdk}/spark/spark-2.x/spark-examples/target/spark-examples_2.11-version-shaded.jar
    ```

-   OSS UnstructuredData

    [详细代码](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.3-public/spark/spark-2.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/oss/SparkUnstructuredDataCompute.scala)

    提交方式

    ``` {#codeblock_r7k_28b_u1y}
    # 需要填上代码中的OSS账号信息，再编译提交。
    val spark = SparkSession
          .builder()
          .config("spark.hadoop.fs.oss.accessKeyId", "***")
          .config("spark.hadoop.fs.oss.accessKeySecret", "***")
          .config("spark.hadoop.fs.oss.endpoint", "oss-cn-hangzhou-zmf.aliyuncs.com")
          .appName("SparkUnstructuredDataCompute")
          .getOrCreate()
    Step 1. build aliyun-cupid-sdk
    Step 2. properly set spark.defaults.conf
    Step 3. bin/spark-submit --master yarn-cluster --class \
          com.aliyun.odps.spark.examples.oss.SparkUnstructuredDataCompute \
          ${path to aliyun-cupid-sdk}/spark/spark-2.x/spark-examples/target/spark-examples_2.11-version-shaded.jar
    ```


