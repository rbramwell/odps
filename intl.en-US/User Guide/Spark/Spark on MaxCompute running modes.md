# Spark on MaxCompute running modes {#concept_e1h_35c_kgb .concept}

This topic describes the running modes of Spark on MaxCompute, including the local, cluster, and DataWorks modes.

## Local mode {#section_1tq_j9h_c6m .section}

The local mode allows you to easily debug application code, through a similar method to that of native Spark, and allows you to read and write MaxCompute tables by running the undefinedtunnelcommand. You need to add the required tunnel configuration items and endpoint information in the Spark-defaults.conf file. The endpoint varies depending on the network conditions and the region where the MaxCompute project is located. For more information, see [Configure Endpoint](../../../../reseller.en-US/Prepare/Configure Endpoint.md#). You can enable the local mode in an integrated development environment \(IDE\) or command line interface \(CLI\). To do so, you need to add the `spark.master=local[N]` configuration item, where `N` indicates the CPU resources required for running in the local mode. The following is an example of enabling the local mode in the CLI:

``` {#codeblock_029_y8r_40c .language-php}
1.bin/spark-submit --master local[4] \
--class com.aliyun.odps.spark.examples.SparkPi \
${path to aliyun-cupid-sdk}/spark/spark-2.x/spark-examples/target/spark-examples_2.11-version-shaded.jar
```

## Cluster mode {#section_anp_rmp_mpx .section}

In cluster mode, you need to specify Main, which is a custom program entry and a crucial part of this mode. In this mode, the Main must succeed or fail for the Spark job to end. This mode is suitable to offline jobs and can be used in combination with Alibaba Cloud DataWorks for job scheduling. The method of submitting the CLI is as follows:

``` {#codeblock_ri0_mv6_38o .language-java}
1.bin/spark-submit --master yarn-cluster \
–class SparkPi \
${ProjectRoot}/spark/spark-2.x/spark-examples/target/spark-examples_2.11-version-shaded.jar
```

## DataWorks mode {#section_uqe_uwt_q0b .section}

**Note:** Supported Region: China \(Hong Kong\), West USA 1, Central Europe 1, Asia Pacific SOU 1, Asia Pacific SE 1.

In DataWorks, you can run offline Spark on MaxCompute jobs in cluster mode. This helps to integrate Spark on MaxCompute nodes with other types of nodes and better schedule these nodes. To run an offline Spark on MaxCompute job, follow these steps:undefined

1.  Log on to DataWorks, and then upload and submit resources as a file in the target workflow.
2.  In the left-side navigation, click a workflow and select **ODPS Spark**.
3.  Drag a Spark on MaxCompute node to the workflow and double-click the node to define a Spark on MaxCompute job.
4.  Select a Spark version, language, and resource file \(the file you have uploaded and submitted in Step 1\), specify configuration items \(for example, the number of executors and memory size\), and set the endpoint configuration item `spark.hadoop.odps.cupid.webproxy.endpoint` for the Spark on MaxCompute service \(the value of the endpoint configuration item is the connection address of the endpoint serving the region where your Spark on MaxCompute project is located\).
5.  Manually run the Spark on MaxCompute node. You can view the logs of this job and based on the logs you can obtain the LogView data of the job and the URL of LogView. In addition, you can compile the data to diagnose the job.

    **Note:** After the Spark on MaxCompute job is defined, you can orchestrate and centrally schedule and run different types of services in the corresponding workflow.


