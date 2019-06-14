# PySpark开发示例 {#concept_263640 .concept}

本文为您介绍PySpark开发示例。

若需要访问MaxCompute表，则需要编译datasource包。具体步骤请参见[配置依赖](cn.zh-CN/开发/Spark/搭建开发环境.md#section_oey_mqy_cq1)。

## SparkSQL应用示例（Spark1.6） {#section_839_eh0_s8n .section}

详细代码

``` {#codeblock_5gl_bhq_d9p}
from pyspark import SparkContext, SparkConf
from pyspark.sql import OdpsContext
if __name__ == '__main__':
    conf = SparkConf().setAppName("odps_pyspark")
    sc = SparkContext(conf=conf)
    sql_context = OdpsContext(sc)
    sql_context.sql("DROP TABLE IF EXISTS spark_sql_test_table")
    sql_context.sql("CREATE TABLE spark_sql_test_table(name STRING, num BIGINT)")
    sql_context.sql("INSERT INTO TABLE spark_sql_test_table SELECT 'abc', 100000")
    sql_context.sql("SELECT * FROM spark_sql_test_table").show()
    sql_context.sql("SELECT COUNT(*) FROM spark_sql_test_table").show()
```

提交运行

``` {#codeblock_gxv_h64_avi}
./bin/spark-submit \
--jars cupid/odps-spark-datasource_xxx.jar \
example.py
```

## SparkSQL应用示例（Spark2.3） {#section_9rs_9yt_o5a .section}

详细代码

``` {#codeblock_7l5_jae_wgu}
from pyspark.sql import SparkSession
if __name__ == '__main__':
    spark = SparkSession.builder.appName("spark sql").getOrCreate()
    spark.sql("DROP TABLE IF EXISTS spark_sql_test_table")
    spark.sql("CREATE TABLE spark_sql_test_table(name STRING, num BIGINT)")
    spark.sql("INSERT INTO spark_sql_test_table SELECT 'abc', 100000")
    spark.sql("SELECT * FROM spark_sql_test_table").show()
    spark.sql("SELECT COUNT(*) FROM spark_sql_test_table").show()
```

Cluster模式提交运行

``` {#codeblock_9q8_mgk_ppc}
spark-submit --master yarn-cluster \
--jars cupid/odps-spark-datasource_xxx.jar \
example.py
```

Local模式运行

``` {#codeblock_0bq_n58_nfe}
cd $SPARK_HOME
./bin/spark-submit --master local[4] \
--driver-class-path cupid/odps-spark-datasource_xxx.jar \
/path/to/odps-spark-examples/spark-examples/src/main/python/spark_sql.py
```

**说明：** 

-   Local模式访问表依赖Tunnel，需要在Spark-defaults.conf配置`tunnel_end_point`配置项的内容。
-   Local模式要用--driver-class-path而非--jars。

## Package依赖 {#section_0q7_5da_5vt .section}

由于MaxCompute集群无法自由安装Python库，PySpark依赖其它Python库/插件/项目时，通常需在本地打包后通过Spark-submit上传。对于特定依赖，打包环境需与线上环境保持一致。您可根据依赖复杂度使用以下两种打包方式。

-   本地打egg包

    例如mllib需要numpy和setuptool两个插件，需要制作对应的egg包，通过`--py-files`上传。 具体步骤如下：

    **说明：** 由于运行Spark作业的线上环境是Python2.7，所以本地必须使用Python2.7打包。

    1.  制作numpy、setuptools egg包。具体操作步骤如下：
        1.  [下载](https://pypi.org/)numpy和setuptools安装包。
        2.  进入setuptools源码路径，执行Python setup.py bdist\_egg后，会在dist目录生成egg文件。
        3.  进入numpy源码路径，执行Python setupeggs.py bdist\_egg后，会在dist目录生成egg文件。
    2.  提交Spark作业

        ``` {#codeblock_o3l_n9x_uh4}
        cd $SPARK_HOME
        ./bin/spark-submit --master yarn-cluster \
        --jars cupid/odps-spark-datasource_2.11-3.3.2-hotfix1.jar \
        --py-files /path/to/numpy-1.7.1-py2.7-lunux-x85_64.egg,/path/to/setuptools-33.1.1-py2.7.egg \
        app.py
        ```

-   本地打Python包

    如果依赖链太长，导致打egg包无法满足需求，或依赖包含so文件等无法通过zipimport引入的文件，则需要把所有依赖下载后打Python包，再把整个Python包上传。 具体步骤如下：

    1.  添加配置

        ``` {#codeblock_cd6_3pg_02b}
        spark.pyspark.python=./public.python-2.7-ucs4.zip/python-2.7-ucs4/bin/python2.7
        ```

    2.  提交Spark作业

        ``` {#codeblock_a9l_jnn_9oh}
        cd $SPARK_HOME
        ./bin/spark-submit --master yarn-cluster \
        --jars cupid/odps-spark-datasource_2.11-3.3.2-hotfix1.jar \
        --archives ./python-2.7-ucs4.zip app.py
        ```

    如果您不想通过`--archives`的方式提交，也可以通过公共资源的方式提交。

    1.  添加配置

        ``` {#codeblock_j0d_tug_ted}
        spark.hadoop.odps.cupid.resources=public.python-2.7-ucs4.zip
        spark.pyspark.python=./public.python-2.7-ucs4.zip/python-2.7-ucs4/bin/python2.7
        ```

    2.  提交Spark作业

        ``` {#codeblock_fy8_zr0_hbt}
        cd $SPARK_HOME
        ./bin/spark-submit --master yarn-cluster \
        --jars cupid/odps-spark-datasource_2.11-3.3.2-hotfix1.jar app.py
        ```

    如果有其他依赖，可参考以下脚本进行自主打包。

    ``` {#codeblock_h1l_448_wgd}
    work_root=`dirname $0`
    work_root=`cd ${work_root}; pwd`
    # Step 1 compile python
    # 1.1 python source code
    cd ${work_root}
    if [ ! -f Python-2.7.13.tgz ]; then
        wget https://www.python.org/ftp/python/2.7.13/Python-2.7.13.tgz
    fi
    # 1.2 configure && make && make install
    if [ ! -d ${work_root}/Python-2.7.13 ]; then
        cd ${work_root}
        tar xf ${work_root}/Python-2.7.13.tgz
    fi
    if [ -d ${work_root}/python-2.7-ucs4 ]; then
        rm -rf ${work_root}/python-2.7-ucs4
    fi
    cd ${work_root}/Python-2.7.13
    ./configure --prefix=${work_root}/python-2.7-ucs4 --enable-unicode=ucs4
    sed -i 's/#.*zlib zlibmodule.c/zlib zlibmodule.c/g' Modules/Setup
    make -j20
    make install
    # 1.3 install pip
    cd ${work_root}
    if [ ! -f get-pip.py ]; then
        curl -s https://bootstrap.pypa.io/get-pip.py -o ${work_root}/get-pip.py
    fi
    ${work_root}/python-2.7-ucs4/bin/python ${work_root}/get-pip.py
    # 1.4 install numpy
    ${work_root}/python-2.7-ucs4/bin/pip install numpy
    # 1.6 make python zip
    if [ -f ${work_root}/python-2.7-ucs4.zip ]; then
        rm -rf ${work_root}/python-2.7-ucs4.zip
    fi
    cd ${work_root}
    zip -r ${work_root}/python-2.7-ucs4.zip python-2.7-ucs4
    ```


