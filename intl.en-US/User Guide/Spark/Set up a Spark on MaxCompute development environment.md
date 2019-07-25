# Set up a Spark on MaxCompute development environment {#concept_263311 .concept}

This topic describes how to set up a Spark on MaxCompute development environment. To do so, you need to download the Spark on MaxCompute client, set environment variables, configure the Spark-defaults.conf file, and configure dependencies.

## Download the Spark on MaxCompute client {#section_732_97j_dh3 .section}

The Spark on MaxCompute software packages are interoperable with the authentication function of MaxCompute. This allows Spark on MaxCompute to serve as a client that submits jobs with the spark-submit script that is encrypted. The following two Spark on MaxCompute software packages are provided to meet different needs:

-   [spark-1.6.3](http://odps-repo.oss-cn-hangzhou.aliyuncs.com/spark/1.6.3-public/spark-1.6.3-public.tar.gz): Develops Spark1.x applications.
-   [spark-2.3.0](http://odps-repo.oss-cn-hangzhou.aliyuncs.com/spark/2.3.0-odps0.30.0/spark-2.3.0-odps0.30.0.tar.gz): Develops Spark2.x applications.

## Set environment variables {#section_h50_2mv_513 .section}

-   Set the JAVA\_HOME environment variable as follows:

    ``` {#codeblock_f02_ahy_eis .language-java}
    # We recommend that you use JDK 1.8 or later.
    export JAVA_HOME=/path/to/jdk
    export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    export PATH=$JAVA_HOME/bin:$PATH
    ```

-   Set the SPARK\_HOME environment variable as follows:

    ``` {#codeblock_sr8_wy9_7ni .language-java}
    export SPARK_HOME=/path/to/spark_extracted_package
    export PATH=$SPARK_HOME/bin:$PATH
    ```

-   If you use PySpark, install Python 2.7 and set the PATH environment variable as follows:

    ``` {#codeblock_67c_ym0_s5i}
    export PATH=/path/to/python/bin/:$PATH
    ```


## Configure the Spark-defaults.conf file {#section_9i8_ixj_ntx .section}

You can use the spark-defaults.conf.template file in the $SPARK\_HOME/conf directory as a template to prepare your own spark-defaults.conf file. However, before you submit Spark on MaxCompute jobs to MaxCompute, you must set the required MaxCompute account and region information in your spark-defaults.conf file.

In the spark-defaults.conf file, you can retain the default settings and enter the MaxCompute account information as follows:

``` {#codeblock_3ur_xq5_zy1}
# Set the MaxCompute project and account information:
spark.hadoop.odps.project.name =
spark.hadoop.odps.access.id =
spark.hadoop.odps.access.key =

# Configure the endpoint through which the Spark on MaxCompute client accesses MaxCompute projects (this endpoint varies depending on the network conditions and region):
spark.hadoop.odps.end.point = http://service.cn.maxcompute.aliyun.com/api
# Configure the endpoint that runs Spark on MaxCompute (this endpoint runs in the MaxCompute VPC in your region):
spark.hadoop.odps.runtime.end.point = http://service.cn.maxcompute.aliyun-inc.com/api

# Retain the following default settings:
spark.sql.catalogImplementation=odps
spark.hadoop.odps.task.major.version = cupid_v2
spark.hadoop.odps.cupid.container.image.enable = true
spark.hadoop.odps.cupid.container.vm.engine.type = hyper
```

## Configure dependencies {#section_oey_mqy_cq1 .section}

-   Configure the dependencies for Spark on MaxCompute jobs to access MaxCompute tables.

    Spark on MaxCompute jobs use the odps-spark-datasource module to access MaxCompute tables. The Maven coordinates of this module are as follows:

    ``` {#codeblock_xc0_vd1_ga2}
    <!-- Spark-2.x uses the following module:-->
    <dependency>
        <groupId>com.aliyun.odps</groupId>
        <artifactId>odps-spark-datasource_2.11</artifactId>
        <version>3.3.3-public</version>
    </dependency>
    
    <!-- Spark-1.x uses the following module:-->
    <dependency>
      <groupId>com.aliyun.odps</groupId>
      <artifactId>odps-spark-datasource_2.10</artifactId>
      <version>3.3.3-public</version>
    </dependency>
    ```

-   Configure the dependencies for Spark on MaxCompute jobs to access OSS.

    If Spark on MaxCompute jobs need to access OSS, add the following dependencies:

    ``` {#codeblock_2eh_eqv_llt}
    <dependency>
        <groupId>com.aliyun.odps</groupId>
        <artifactId>hadoop-fs-oss</artifactId>
        <version>3.3.3-public</version>
    </dependency>
    ```


