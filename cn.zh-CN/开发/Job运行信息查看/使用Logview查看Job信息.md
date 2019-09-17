# 使用Logview查看Job信息 {#concept_kkg_xgn_vdb .concept}

Logview是MaxCompute Job提交后查看和Debug任务的工具。

通过Logview可看到一个Job的如下内容：

-   任务的运行状态。
-   任务的运行结果。
-   任务的细节和每个步骤的进度。

Job提交到MaxCompute后，会生成Logview的链接。您可以直接在浏览器上打开Logview链接，进入查看Job的信息。

![logview 获取截图](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12086/156870291460802_zh-CN.png)

**说明：** 每个Job的Logview页面的有效期为一周。

## Logview功能组件 {#section_a35_tln_vdb .section}

下面结合具体的Logview Web UI界面，为您介绍每个组件的含义。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12086/15687029142353_zh-CN.png)

Logview的首页分成上下两部分：

-   Instance信息
-   Task信息

**Instance信息**

在Logview页面中，上半部分是您提交的SQL任务对应的MaxCompute Instance信息，包含URL链接、项目、InstanceID、提交人、开始时间、结束时间和状态（Status）等。

-   Instance信息部分，您可以单击Status查看当前队列的Status详细信息，包含四种状态：

    -   Waiting：说明当前作业正在MaxCompute中处理，并没有提交到分布式调度系统（Fuxi）中运行。
    -   Waiting List : n：说明作业已提交至分布式调度系统（Fuxi）排队等待运行，当前在队列中处于第n位。
    -   Running：作业在分布式调度系统（Fuxi）中运行。
    **说明：** 处于Terminated状态的Instance由于作业已处理完成，因此不存在队列信息。

-   单击Status查看队列详细信息：
    -   Sub Status：表示当前子状态信息。
    -   WaitPos：表示排队位置，如果是`0`表示正在运行，如果为`-`表示尚未到Fuxi。
    -   QueueLength：表示Fuxi中总的队列长度。
    -   Total Priority：表示作业运行时经过系统判断后授予的优先级。
    -   SubStatus History：单击后，可以查看作业执行的详细历史状态，包含状态码、状态描述、开始时间、持续时间等（某些版本暂时无历史信息）。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12086/15687029152356_zh-CN.png)


**Task 信息**

在Logview页面中，下半部分为该Instance对应的task信息，包括任务名称、类型、状态、运行结果（Result）、详细信息（Detail）、开始时间、结束时间、总运行时间（Latency）、进度。其中，Latency指运行总共消耗的时间，与其他页面的Latency含义类似。

Result：

在Job运行结束后，可以看到运行结果。例如一条select SQL的结果如下图所示。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12086/15687029152365_zh-CN.png)

Detail：

一个Job在运行中和结束后，均可以单击**Detail**来查看任务运行的具体情况。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12086/15687029152366_zh-CN.png)

在MaxCompute Task详细信息对话框中，您可以查看以下信息：

-   一个MaxCompute Task可以由一个或多个Fuxi Job组成。例如，如果您提交的SQL任务较为复杂，MaxCompute将自动向分布式调度系统（Fuxi）提交多个Fuxi Job。
-   每个Fuxi Job可以由一个或多个Fuxi Task组成。例如，简单的MapReduce任务通常会产生两个Fuxi Task，即Map Task（M1）和Reduce Task（R2）。而一个比较复杂的SQL任务也可能产生多个Fuxi Task。
-   查看所产生的Fuxi Task的任务名称。任务名称一般由字母和数字组成，其中字母表示任务类型（例如，M代表Map Task），数字标识任务编号以及依赖关系（例如，R5\_4表示该Reduce Task需要J4任务执行结束后才会开始执行；J4\_1\_2\_3则表示该Join Task需要M1、M2、M3这三个任务都执行完成后才会开始执行）。

    **说明：** I/O Records表示该Fuxi Task的输入和输出的记录数。

-   I/O Records表示这个task的输入和输出的records数。

单击Fuxi Task最右侧的查看列中的按钮或双击该Fuxi Task，您可以查看该Fuxi Task对应的Instance信息。

**说明：** 每个Fuxi Task由一个或多个Fuxi Instance组成。当Fuxi Task的输入数据量持续增大时，MaxCompute将自动为该Task启动更多的节点来处理数据，每个节点即对应一个Fuxi Instance。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12086/15687029152367_zh-CN.png)

在页面下方的Fuxi Instance信息部分，Logview将处于不同阶段的Instance进行分组展示。例如，您可以选择Failed组，查看运行出错的节点。同时，您可以单击StdOut和StdErr列中的按钮查看标准输出信息和标准错误信息。

**说明：** 您在所提交的MaxCompute任务中编写的打印信息也将在标准输出信息和标准错误信息中展示。

## 使用Logview排查问题 {#section_xf2_1pn_vdb .section}

-   **出错的任务** 

    当有任务出错时，您可以在Logview页面的Result中看到错误的提示信息，也可以在Detail页面中通过Fuxi Instance的stderr，查看具体某个Instance出错的信息。

-   **数据倾斜** 

    运行缓慢有时是由于在某个Fuxi Task的所有Fuxi Instance中，有个别Instance形成长尾造成的。长尾的现象由同一个Task内任务分配不均导致。这时可以在任务运行完后，在Summay标签页中看运行结果。在每个Task中都可以看到如下输出：

    ``` {#codeblock_3sw_81x_f5b}
    output records:
    R2_1_Stg1: 199998999  (min: 22552459, max: 177446540, avg: 99999499)
    ```

    此处如果min和max值相差较大，不在一个数量级，说明在这一阶段出现了数据倾斜。例如在执行字段Join时，某个字段中有一个值出现的比例很高，则在这一字段上执行Join就会出现数据倾斜。


