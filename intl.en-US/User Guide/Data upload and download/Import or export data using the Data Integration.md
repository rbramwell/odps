# Import or export data using the Data Integration {#concept_k5w_ybg_vdb .concept}

Use [Data Integration](https://www.alibabacloud.com/help/doc-detail/47677.html) function of DataWorks to create data synchronization tasks and import and export MaxCompute data.

## Prerequisites {#section_y1n_bcg_vdb .section}

Before importing or exporting data, complete the required operations first. For more information, see [Prepare an Alibaba Cloud account](../../../../reseller.en-US/Prepare/Create an Alibaba Cloud account.md) and [Purchase and create a project](../../../../reseller.en-US/Prepare/Create a project.md).

## Add MaxCompute data source {#section_drp_ccg_vdb .section}

**Note:** 

-   Only the project administrator can create a data source. Other roles can only view the data source.
-   **If the data source you want to add is a current MaxCompute project, skip this operation**. After this project is created and appears as a Data Integration data source, this project is added as a MaxCompute data source named odps\_first by default.

**Procedure**

1.  Log on to the [DataWorks console](https://workbench.data.aliyun.com/console?spm=a2c4g.11186623.2.7.bBZHDz) as an administrator and click **Enter Workspace** from the Actions column of the relevant project in the **Project List**.
2.  Select **Data Integration** from the upper navigation pane. Click **Data Source** from the left-side navigation pane.
3.  Click **New Source**.Select MaxCompute \(ODPS\) from the Large Data Storage section.
4.  Enter required configurations in the data dialog box.

    Parameters

    -   Name: Contains letters, numbers, and underscores \(\_\). It must begin with a letter or an underscore \(\_\) , and cannot exceed 60 characters.
    -   Data source description: Provides a brief description of the data source, and cannot exceed 80 characters.
    -   Data source type: Currently, it is ODPS.
    -   ODPS Endpoint: Read-only by default. The value is automatically read from the system configuration.
    -   ODPS Item name: Name of the project, helps to identify the corresponding MaxCompute project.
    -   Access ID: The Access ID associated with the account of the MaxCompute project owner.
    -   AccessKey: The AccessKey associated with the account of the MaxCompute project owner, used in pairs with the Access ID.
5.  \(Optional\). Click **Test Connectivity** to test the connectivity after entering all the required information in the relevant fields.
6.  If the connectivity test is successful, click **Save**.

**Note:** For more information about the other data sources configurations, see [data source configuration](https://www.alibabacloud.com/help/doc-detail/60416.html).

## Import data through Data Integration {#section_zdw_1dg_vdb .section}

Take importing MySQL data to MaxCompute as an example, you can configure a synchronization task using **Wizard Mode** or **Script Mode**.

**Configure a synchronization task in Wizard mode** 

1.  Create a Wizard Mode synchronization task.
2.  Select the source.

    Select the MySQL data source and the source table “mytest”. The data browsing area is collapsed by default. Click **Next**.

3.  Select a Target.

    The target must be a previously created MaxCompute table. You can also create a new table by clicking **Quick Table Creation**.

    Parameters

    -   Partition information: Specify every level of partition. When writing data to a table with three levels of partitions, you must configure the last partition level, for example, pt=20150101, type=1, biz=2. This item is unavailable for non-partitioned tables.
    -   Data clearing rules:
        -   Clear existing data before writing: Before data is imported to a table or partition, all data in the table or partition is cleared, which is equivalent to Insert Overwrite.
        -   Retain existing data before writing: Existing data is not cleared before new data is imported. Each operation appends new data, which is equivalent to Insert Into.
4.  Map the fields.

    Select the mapping between fields. Configure the field mapping relationships. The **Source Table Fields** on the left correspond one to one with the **Target Table Fields** on the right.

5.  Control the channel.

    Click **Next** to configure the maximum job rate and dirty data check rules.

    Parameters

    -   Maximum job rate: Determines the highest rate possible for data synchronization jobs. The actual rate of the job may vary with the network environment, database configuration, and other factors.
    -   Concurrent job count: For a single synchronization job, Concurrent job count \* Individual job transmission rate = Total job transmission rate.
    **When a maximum job rate is specified, how do you select the concurrent job count?** 

    -   If your data source is an online business database, we recommend that you refrain from setting a large value for the concurrent job count to avoid interference with the online database.
    -   If you require a high data synchronization rate, we recommend that you select the highest job rate and a large concurrent job count.
6.  Preview and store.

    Make sure the configuration of the task is correct, and click **Save**.


## Run a synchronization task {#section_lfd_z2g_vdb .section}

**Run a synchronization task directly**

If system variable parameters are set in the synchronization task, the variable parameter configuration window is displayed during task operation.

After saving the task, click **Run** to run the task immediately. Click **Submit** and the synchronization task will be submitted to the scheduling system of the DataWorks. The scheduling system automatically and periodically runs the task from the second day according to the configuration attributes. For more information on scheduling configurations, see [Scheduling configuration description](https://www.alibabacloud.com/help/doc-detail//50130.html).

## Configure a synchronization task in Script mode {#section_ut3_mfg_vdb .section}

Use the following script to configure synchronization tasks. Other configurations and job operation are the same as **Wizard Mode**.

``` {#codeblock_00j_rh0_iju}
{
  "type": "job",
  "version": "1.0",
  "configuration": {
    "reader": {
      "plugin": "mysql",
      "parameter": {
        "datasource": "mysql",
        "where": "",
        "splitPk": "id",
        "connection": [
          {
            "table": [
              "person"
            ],
            "datasource": "mysql"
          }
        ],
        "connectionTable": "person",
        "Column ":[
          "id",
          "name"
        ]
      }
    },
    "writer": {
      "plugin": "odps",
      "parameter": {
        "datasource": "odps_first",
        "table": "a1",
        "truncate": true,
        "partition": "pt=${bdp.system.bizdate}",
        "Column ":[
          "id",
          "col1"
        ]
      }
    },
    "Setting ":{
      "speed": {
        "mbps": "1",
        "concurrent": "1"
      }
    }
  }
}
```

## Referenes {#section_hzj_qf1_fe6 .section}

-   For the Reader configurations about different types of data sources, see [Configure Reader Plug-ins](https://www.alibabacloud.com/help/faq-list/74300.htm).
-   For the Writer configurations about different types of data sources, see [Configure Writer Plug-ins](https://www.alibabacloud.com/help/faq-list/74301.htm).

