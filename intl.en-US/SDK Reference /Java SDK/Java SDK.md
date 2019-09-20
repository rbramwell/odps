# Java SDK {#concept_utw_vvc_5db .concept}

This article introduces most commonly used MaxCompute core interfaces. For more information, see [SDK Java Doc](http://www.javadoc.io/doc/com.aliyun.odps/odps-sdk-core/).

You can configure the version of the new SDK through maven management. The configuration information of Maven is as follows\(The latest version can be searched for odps-sdk-core at any time at [search.maven.org](https://search.maven.org/search?q=g:com.aliyun.odps%20AND%20a:odps-sdk-udf&core=gav).

```
<dependency>
  <groupId>com.aliyun.odps</groupId>
  <artifactId>odps-sdk-core</artifactId>
  <version>0.26.2-public</version>
</dependency>
```

**Note:** 0.27.2-public version and above support MaxCompute 2.0 [New data type](../../../../intl.en-US/User Guide/Data types.md#).

The overall information of the SDK package provided by MaxCompute is shown in the following table:

|Package Name|Description|
|:-----------|:----------|
|odps-sdk-core|The basic functions of MaxCompute, such as the operation of tables, Project, and Tunnel, are all included in this package.|
|odps-sdk-commons|Some Util packages.|
|odps-sdk-udf|Main interface of UDF.|
|odps-sdk-mapred|MapReduce Java SDK.|
|odps-sdk-graph|Graph Java SDK, the keyword. used to search is “odps-sdk-graph”.|

## AliyunAccount {#section_os1_f5b_wdb .section}

AlibabaCloudAccount. The primary account created with Alibaba Cloud. It generally has an AccessKey that comprises of an AccessKeyId and an AccessKeySecret, used to initialize MaxCompute.

## MaxCompute {#section_i25_f5b_wdb .section}

It is the entry of MaxCompute SDK. You can get set of all objects under the project shell by such endpoint, including [Projects](../../../../intl.en-US/Product Introduction/Definitions/Project.md), [Tables](../../../../intl.en-US/Product Introduction/Definitions/Table.md), [Resources](../../../../intl.en-US/Product Introduction/Definitions/Resource.md), [Functions](../../../../intl.en-US/Product Introduction/Definitions/Function.md), and [Instances](../../../../intl.en-US/Product Introduction/Definitions/Instance.md).

**Note:** MaxCompute was formerly called ODPS, so the portal class is still named as ODPS in the current SDK version.

User can construct MaxCompute object by entering the AliyunAccount instance. The code example is shown as follows:

```
    Account account = new AliyunAccount("my_access_id", "my_access_key");
    Odps odps = new Odps(account);
    String odpsUrl = "<your odps endpoint>";
    odps.setEndpoint(odpsUrl);
    odps.setDefaultProject("my_project");
    for (Table t : odps.tables()) {
        ....
    }
				
```

## Tunnel {#section_ohh_wy5_qfb .section}

The MaxCompute Tunnel data channel is written based on the Tunnel SDK, and you can upload or download data to MaxCompute through the Tunnel. For more information, see[Tunnel SDK](../../../../intl.en-US/User Guide/Data upload and download/Tunnel SDK/Tunnel overview.md#).At present, Tunnel supports only tables \(excluding views View\) and uploading and downloading data.

## MapReduce {#section_pjl_pnz_sfb .section}

See[MapReduce SDK](../../../../intl.en-US/User Guide/MapReduce/Java SDK/Java SDK.md#) for more information.

## Projects {#section_eg4_k5b_wdb .section}

It is the set of all projects in MaxCompute. The element of this set is Projects. The code example is shown as follows:

```
    Account account = new AliyunAccount("my_access_id", "my_access_key");
    Odps odps = new Odps(account);
    String odpsUrl = "<your odps endpoint>";
    odps.setEndpoint(odpsUrl);
    Project p = odps.projects().get("my_exists");
    p.reload();
    Map<String, String> properties = prj.getProperties();
    ...
				
```

## Project {#section_lg2_n5b_wdb .section}

It refers to the description of project and corresponding project, and can be acquired from Projects.

## SQLTask {#section_fpg_45b_wdb .section}

It refers to an interface to run and process SQL task. SQL can run directly through the interface ‘run’.\(**Note: only one SQL statement can be submitted at a time.**\)

The run interface returns the Instance instance and obtains the SQL running status and result through Instance.

**Example:** 

```
    import java.util.List;
    import com.aliyun.odps.Instance;
    mport com.aliyun.odps.Odps;
    import com.aliyun.odps.OdpsException;
    import com.aliyun.odps.account.Account;
    import com.aliyun.odps.account.AliyunAccount;
    import com.aliyun.odps.data.Record;
    import com.aliyun.odps.task.SQLTask;
    public class testSql {
    private static final String accessId = "";
    private static final String accessKey = "";
    private static final String endPoint = "http://service.odps.aliyun.com/api";
    private static final String project = "";
    private static final String sql = "select category from iris;";
    public static void
    main(String[] args) {
      Account account = new AliyunAccount(accessId, accessKey);
       Odps odps = new Odps(account);
       odps.setEndpoint(endPoint);
       odps.setDefaultProject(project);
       Instance i;
      try {
         i = SQLTask.run(odps, sql);
         i.waitForSuccess();
         List<Record> records = SQLTask.getResult(i); 
         for(Record r:records){
            System.out.println(r.get(0).toString());
         }
      } catch (OdpsException e) {
         e.printStackTrace();
      }
   }
  }
				
```

**Note:** To create a table, use SQLTask interface instead of the interface Table. You must introduce the statement of [Table Operation](../../../../intl.en-US/User Guide/SQL/DDL SQL/Table operations.md#) into SQLTask.

## Instances {#section_xtz_s5b_wdb .section}

This class refers to the set of all \(instances\) in MaxCompute and the element of this set is Instance. The code example is shown as follows:

```
    Account account = new AliyunAccount("my_access_id", "my_access_key");
    Odps odps = new Odps(account);
    String odpsUrl = "<your odps endpoint>";
    odps.setEndpoint(odpsUrl);
    odps.setDefaultProject("my_project");
    for (Instance i : odps.instances()) {
        ....
    }
				
```

## Instance {#section_rqp_v5b_wdb .section}

It refers to the description of instance and corresponding instance,and can be acquired from Instances. The code example is as follows:

```
    Account account = new AliyunAccount("my_access_id", "my_access_key");
    Odps odps = new Odps(account);
    String odpsUrl = "<your odps endpoint>";
    odps.setEndpoint(odpsUrl);
    Instance instance= odps.instances().get("instance id");
    Date startTime = instance.getStartTime();
    Date endTime = instance.getEndTime();
    ...
    Status instanceStatus = instance.getStatus();
    String instanceStatusStr = null;
    if (instanceStatus == Status.TERMINATED) {
      instanceStatusStr = TaskStatus.Status.SUCCESS.toString();
      Map<String, TaskStatus> taskStatus = instance.getTaskStatus();
      for (Entry<String, TaskStatus> status : taskStatus.entrySet()) {
        if (status.getValue().getStatus() ! = TaskStatus.Status.SUCCESS) {
          instanceStatusStr = status.getValue().getStatus().toString();
          break;
        }
      }
    } else {
      instanceStatusStr = instanceStatus.toString();
    }
    ...
    TaskSummary summary = instance.getTaskSummary("task name");
    String s = summary.getSummaryText();
				
```

## Tables {#section_bwd_z5b_wdb .section}

This class refers to the set of all tables in MaxCompute. The element of this set is Table. The code example is shown as follows:

```
    Account account = new AliyunAccount("my_access_id", "my_access_key");
    Odps odps = new Odps(account);
    String odpsUrl = "<your odps endpoint>";
    odps.setEndpoint(odpsUrl);
    odps.setDefaultProject("my_project");
    for (Table t : odps.tables()) {
        ....
    }
				
```

## Table {#section_c13_bvb_wdb .section}

It refers to the description of table and corresponding table and can be acquired through Tables. The code example is shown as follows:

```
    Account account = new AliyunAccount("my_access_id", "my_access_key");
    Odps odps = new Odps(account);
    String odpsUrl = "<your odps endpoint>";
    odps.setEndpoint(odpsUrl);
    Table t = odps.tables().get("table name");
    t.reload();
    Partition part = t.getPartition(new PartitionSpec(tableSpec[1]));
    part.reload();
    ...
				
```

## Resources {#section_tdn_dvb_wdb .section}

The class refers to the set of all resources in MaxCompute. The element of this set is Resource. The code example is as follows:

```
    Account account = new AliyunAccount("my_access_id", "my_access_key");
    Odps odps = new Odps(account);
    String odpsUrl = "<your odps endpoint>";
    odps.setEndpoint(odpsUrl);
    odps.setDefaultProject("my_project");
    for (Resource r : odps.resources()) {
        ....
    }
				
```

## Resource {#section_p3x_fvb_wdb .section}

It refers to the resource description and the corresponding resource and can be acquired through Resources. The code example is as follows:

```
    Account account = new AliyunAccount("my_access_id", "my_access_key");
    Odps odps = new Odps(account);
    String odpsUrl = "<your odps endpoint>";
    odps.setEndpoint(odpsUrl);
    Resource r = odps.resources().get("resource name");
    r.reload();
    if (r.getType() == Resource.Type.TABLE) {
     TableResource tr = new TableResource(r);
     String tableSource = tr.getSourceTable().getProject() + "."
         + tr.getSourceTable().getName();
     if (tr.getSourceTablePartition() ! = null) {
       tableSource += " partition(" + tr.getSourceTablePartition().toString()
           + ")";
     }
     ....   
    }
				
```

File resource creation example is as follows:

```
    String projectName = "my_porject";
    String source = "my_local_file.txt";
    File file = new File(source);
    InputStream is = new FileInputStream(file);
    FileResource resource = new FileResource();
    String name = file.getName();
    resource.setName(name);
    odps.resources().create(projectName, resource, is);
```

Table resource creation example is as follows:

```
    TableResource resource = new TableResource(tableName, tablePrj, partitionSpec);
    //resource.setName(INVALID_USER_TABLE);
    resource.setName("table_resource_name");
    odps.resources().update(projectName, resource);
```

## Functions {#section_b4d_lvb_wdb .section}

This class refers to the set of all functions in MaxCompute. The element of this set is Function. An example is as follows:

```
    Account account = new AliyunAccount("my_access_id", "my_access_key");
    Odps odps = new Odps(account);
    String odpsUrl = "<your odps endpoint>";
    odps.setEndpoint(odpsUrl);
    odps.setDefaultProject("my_project");
    for (Function f : odps.functions()) {
        ....
    }
				
```

## Function {#section_xbw_nvb_wdb .section}

It refers to the function description and corresponding function and can be acquired through Functions. The code example is as follows:

```
    Account account = new AliyunAccount("my_access_id", "my_access_key");
    Odps odps = new Odps(account);
    String odpsUrl = "<your odps endpoint>";
    odps.setEndpoint(odpsUrl);
    Function f = odps.functions().get("function name");
    List<Resource> resources = f.getResources();
				
```

Function creation example:

```
    String resources = "xxx:xxx";
    String classType = "com.aliyun.odps.mapred.open.example.WordCount";
    ArrayList<String> resourceList = new ArrayList<String>();
    for (String r : resources.split(":")) {
      resourceList.add(r);
    }
    Function func = new Function();
    func.setName(name);
    func.setClassType(classType);
    func.setResources(resourceList);
    odps.functions().create(projectName, func);
				
```

