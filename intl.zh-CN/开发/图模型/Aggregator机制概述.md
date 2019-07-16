# Aggregator机制概述 {#concept_cv5_ndn_vdb .concept}

本文将为您介绍Aggregator的执行机制、相关API，并以Kmeans Clustering为例，说明Aggregator的具体用法。

Aggregator是MaxCompute Graph作业中常用的feature之一，特别适用于解决机器学习问题。MaxCompute Graph中，Aggregator用于汇总并处理全局信息。

## Aggregator机制 {#section_scb_rdn_vdb .section}

Aggregator的逻辑分为两部分：

-   一部分在所有Worker上执行，即分布式执行。
-   另一部分只在AggregatorOwner所在的Worker上执行，即单点。

其中在所有Worker上执行的操作包括创建初始值及局部聚合，然后将局部聚合结果发送给AggregatorOwner所在的Worker上。AggregatorOwner所在的Worker上聚合普通Worker发送过来的局部聚合对象，得到全局聚合结果，然后判断迭代是否结束。全局聚合的结果会在下一轮超步分发给所有Worker，供下一轮迭代使用。如下图所示：

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12062/15632453372317_zh-CN.png)

## Aggregator的API {#section_st2_xdn_vdb .section}

Aggregator共提供了五个API供您实现。API 的调用时机及常规用途，如下所示：

-   createStartupValue\(context\)

    该API在所有Worker上执行一次，调用时机是所有超步开始之前，通常用以初始化AggregatorValue。在第0轮超步中，调用WorkerContext.getLastAggregatedValue\(\)或ComputeContext.getLastAggregatedValue\(\)可以获取该API初始化的AggregatorValue对象。

-   createInitialValue\(context\)

    该API在所有Worker上每轮超步开始时调用一次，用以初始化本轮迭代所用的AggregatorValue。通常操作是通过 WorkerContext.getLastAggregatedValue\(\)得到上一轮迭代的结果，然后执行部分初始化操作。

-   aggregate\(value, item\)

    该API同样在所有Worker上执行，与上述API不同的是，该API由用户显示调用ComputeContext\#aggregate\(item\)来触发，而上述两个API，则由框架自动调用。该API用以执行局部聚合操作，其中第一个参数value是本Worker在该轮超步已经聚合的结果（初始值是createInitialValue返回的对象），第二个参数是您的代码调用ComputeContext\#aggregate\(item\)传入的参数。该API中通常用item来更新value实现聚合。所有aggregate执行完后，得到的value就是该Worker的局部聚合结果，然后由框架发送给AggregatorOwner所在的Worker。

-   merge\(value, partial\)

    该API执行于AggregatorOwner所在Worker，用以合并各Worker局部聚合的结果，达到全局聚合对象。与Aggregate类似，value是已经聚合的结果，而partial待聚合的对象，同样用partial更新value。

    假设有3个worker，分别是w0、w1、w2，其局部聚合结果是p0、p1、p2。假设发送到AggregatorOwner所在Worker的顺序为p1、p0、p2。那么merge执行次序为：

    1.  首先执行merge\(p1, p0\)，这样p1和p0就聚合为p1。
    2.  然后执行merge\(p1, p2\)，p1和p2聚合为p1，而p1即为本轮超步全局聚合的结果。
    由上述示例可见，当只有一个worker时，不需要执行merge方法，也就是说merge\(\)不会被调用。

-   terminate\(context, value\)

    当AggregatorOwner所在Worker执行完merge\(\)后，框架会调用terminate\(context, value\)执行最后的处理。其中第二个参数value，即为merge\(\)最后得到全局聚合，在该方法中可以对全局聚合继续修改。执行完terminate\(\)后，框架会将全局聚合对象分发给所有Worker，供下一轮超步使用。terminate\(\)方法的一个特殊之处在于，如果返回true，则整个作业就结束迭代，否则继续执行。在机器学习场景中，通常判断收敛后返回true以结束作业。


## Kmeans Clustering 示例 {#section_x53_c2n_vdb .section}

下面以典型的KmeansClustering为例，为您介绍Aggregator的具体用法。

**说明：** 完整代码见附件，此处为解析代码。

-   **GraphLoader 部分**

    GraphLoader部分用以加载输入表，并转换为图的点或边。这里我们输入表的每行数据为一个样本，一个样本构造一个点，并用Vertex的value来存放样本。

    首先定义一个Writable类KmeansValue作为Vertex的value类型：

    ```
    public static class KmeansValue implements Writable {
      DenseVector sample;
      public KmeansValue() { 
      }
      public KmeansValue(DenseVector v) {
        this.sample = v;
      }
      @Override
      public void write(DataOutput out) throws IOException {
        wirteForDenseVector(out, sample);
      }
      @Override
      public void readFields(DataInput in) throws IOException {
        sample = readFieldsForDenseVector(in);
      }
    }
    ```

    KmeansValue中封装一个DenseVector对象来存放一个样本，这里DenseVector类型来自[matrix-toolkits-java](https://github.com/fommil/matrix-toolkits-java/)，而wirteForDenseVector\(\)及readFieldsForDenseVector\(\)用以实现序列化及反序列化，可参见附件中的完整代码。

    自定义的KmeansReader代码，如下所示：

    ```
    public static class KmeansReader extends 
      GraphLoader<LongWritable, KmeansValue, NullWritable, NullWritable> {
      @Override
      public void load(
          LongWritable recordNum,
          WritableRecord record,
          MutationContext<LongWritable, KmeansValue, NullWritable, NullWritable> context)
          throws IOException {
        KmeansVertex v = new KmeansVertex();
        v.setId(recordNum);
        int n = record.size();
        DenseVector dv = new DenseVector(n);
        for (int i = 0; i < n; i++) {
          dv.set(i, ((DoubleWritable)record.get(i)).get());
        }
        v.setValue(new KmeansValue(dv));
        context.addVertexRequest(v);
      }
    }
    
    ```

    KmeansReader中，每读入一行数据（一个 Record）创建一个点，这里用recordNum作为点的ID，将record内容转换成DenseVector对象并封装进VertexValue中。

-   **Vertex部分**

    自定义的KmeansVertex代码如下。逻辑非常简单，每轮迭代要做的事情就是将自己维护的样本执行局部聚合。具体逻辑参见下面Aggregator的实现。

    ```
    public static class KmeansVertex extends
      Vertex<LongWritable, KmeansValue, NullWritable, NullWritable> {
      @Override
      public void compute(
          ComputeContext<LongWritable, KmeansValue, NullWritable, NullWritable> context,
          Iterable<NullWritable> messages) throws IOException {
        context.aggregate(getValue());
      }
    }
    
    ```

-   **Aggregator部分**

    整个Kmeans的主要逻辑集中在Aggregator中。首先是自定义的KmeansAggrValue，用以维护要聚合及分发的内容。

    ```
    public static class KmeansAggrValue implements Writable {
      DenseMatrix centroids;
      DenseMatrix sums; // used to recalculate new centroids
      DenseVector counts; // used to recalculate new centroids
      @Override
      public void write(DataOutput out) throws IOException {
        wirteForDenseDenseMatrix(out, centroids);
        wirteForDenseDenseMatrix(out, sums);
        wirteForDenseVector(out, counts);
      }
      @Override
      public void readFields(DataInput in) throws IOException {
        centroids = readFieldsForDenseMatrix(in);
        sums = readFieldsForDenseMatrix(in);
        counts = readFieldsForDenseVector(in);
      }
    }
    ```

    KmeansAggrValue中维护了三个对象，其中centroids是当前的K个中心点，假定样本是m维的话，centroids就是一个K\*m的矩阵。sums是和centroids大小一样的矩阵，每个元素记录了到特定中心点最近的样本特定维之和，例如sums\(i,j\)是到第i个中心点最近的样本的第j维度之和。

    counts是个K维的向量，记录到每个中心点距离最短的样本个数。sums和counts一起用以计算新的中心点，也是要聚合的主要内容。

    接下来是自定义的Aggregator实现类KmeansAggregator，我们按照上述API的顺序逐个分析其实现。

    1.  createStartupValue\(\)的实现。

        ```
        public static class KmeansAggregator extends Aggregator<KmeansAggrValue> {
        public KmeansAggrValue createStartupValue(WorkerContext context) throws IOException {
        KmeansAggrValue av = new KmeansAggrValue();
        byte[] centers = context.readCacheFile("centers");
        String lines[] = new String(centers).split("\n");
        int rows = lines.length;
        int cols = lines[0].split(",").length; // assumption rows >= 1 
        av.centroids = new DenseMatrix(rows, cols);
        av.sums = new DenseMatrix(rows, cols);
        av.sums.zero();
        av.counts = new DenseVector(rows);
        av.counts.zero();
        for (int i = 0; i < lines.length; i++) {
          String[] ss = lines[i].split(",");
          for (int j = 0; j < ss.length; j++) {
            av.centroids.set(i, j, Double.valueOf(ss[j]));
          }
        }
        return av;
        }
        
        ```

        我们在该方法中初始化一个KmeansAggrValue对象，然后从资源文件centers中读取初始中心点，并赋值给centroids。而sums和counts初始化为0。

    2.  createInitialValue\(\)的实现。

        ```
        @Override
        public void aggregate(KmeansAggrValue value, Object item)
          throws IOException {
        DenseVector sample = ((KmeansValue)item).sample;
        // find the nearest centroid
        int min = findNearestCentroid(value.centroids, sample);
        // update sum and count
        for (int i = 0; i < sample.size(); i ++) {
          value.sums.add(min, i, sample.get(i));
        }
        value.counts.add(min, 1.0d);
        }
        ```

        该方法中调用findNearestCentroid\(\)（实现见附件）找到样本item距离最近的中心点索引，然后将其各个维度加到sums上，最后counts计数加1。


以上三个方法执行于所有worker上，实现局部聚合。接下来看下在AggregatorOwner所在Worker执行的全局聚合相关操作：

1.  merge的实现：

    ```
    @Override
    public void merge(KmeansAggrValue value, KmeansAggrValue partial)
      throws IOException {
    value.sums.add(partial.sums);
    value.counts.add(partial.counts);
    }
    ```

    merge的实现逻辑很简单，就是把各个worker聚合出的sums和counts相加即可。

2.  terminate\(\)的实现。

    ```
    @Override
    public boolean terminate(WorkerContext context, KmeansAggrValue value)
       throws IOException {
     // Calculate the new means to be the centroids (original sums)
     DenseMatrix newCentriods = calculateNewCentroids(value.sums, value.counts, value.centroids);
     // print old centroids and new centroids for debugging
     System.out.println("\nsuperstep: " + context.getSuperstep() + 
         "\nold centriod:\n" + value.centroids + " new centriod:\n" + newCentriods);
     boolean converged = isConverged(newCentriods, value.centroids, 0.05d);
     System.out.println("superstep: " + context.getSuperstep() + "/" 
         + (context.getMaxIteration() - 1) + " converged: " + converged);
     if (converged || context.getSuperstep() == context.getMaxIteration() - 1) {
       // converged or reach max iteration, output centriods
       for (int i = 0; i < newCentriods.numRows(); i++) {
         Writable[] centriod = new Writable[newCentriods.numColumns()];
         for (int j = 0; j < newCentriods.numColumns(); j++) {
           centriod[j] = new DoubleWritable(newCentriods.get(i, j));
         }
         context.write(centriod);
       }
       // true means to terminate iteration
       return true;
     }
     // update centriods
     value.centroids.set(newCentriods);
     // false means to continue iteration
     return false;
    }
    ```

    teminate\(\)中首先根据sums和counts调用calculateNewCentroids\(\)求平均计算出新的中心点。然后调用isConverged\(\)根据新老中心点欧拉距离判断是否已经收敛。如果收敛或迭代次数达到最大数，则将新的中心点输出并返回true，以结束迭代。否则更新中心点并返回false以继续迭代。其中calculateNewCentroids\(\)和isConverged\(\)的实现见附件。


-   **main方法**

    main方法用以构造GraphJob，然后设置相应配置，并提交作业。代码如下：

    ```
    public static void main(String[] args) throws IOException {
      if (args.length < 2)
        printUsage();
      GraphJob job = new GraphJob();
      job.setGraphLoaderClass(KmeansReader.class);
      job.setRuntimePartitioning(false);
      job.setVertexClass(KmeansVertex.class);
      job.setAggregatorClass(KmeansAggregator.class);
      job.addInput(TableInfo.builder().tableName(args[0]).build());
      job.addOutput(TableInfo.builder().tableName(args[1]).build());
      // default max iteration is 30
      job.setMaxIteration(30);
      if (args.length >= 3)
        job.setMaxIteration(Integer.parseInt(args[2]));
      long start = System.currentTimeMillis();
      job.run();
      System.out.println("Job Finished in "
          + (System.currentTimeMillis() - start) / 1000.0 + " seconds");
    }
    ```

    **说明：** job.setRuntimePartitioning\(false\)设置为false后，各个worker加载的数据不再根据Partitioner重新分区，即谁加载的数据谁维护。


## 总结 {#section_sgj_wfn_vdb .section}

本文介绍了MaxCompute Graph中的Aggregator机制，API含义以及Kmeans Clustering示例。总的来说，Aggregator的基本步骤，如下所示：

1.  每个worker启动时执行createStartupValue用以创建AggregatorValue。
2.  每轮迭代开始前，每个worker执行createInitialValue来初始化本轮的AggregatorValue。
3.  一轮迭代中每个点通过context.aggregate\(\)来执行aggregate\(\)实现worker内的局部迭代。
4.  每个Worker将局部迭代结果发送给AggregatorOwner所在的Worker。
5.  AggregatorOwner所在worker执行多次merge，实现全局聚合。
6.  AggregatorOwner所在Worker执行terminate用以对全局聚合结果做处理并决定是否结束迭代。

## 附件 {#section_wrq_yfn_vdb .section}

本文所需附件，请参见[Kmeans](https://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/cn/odps/0.0.90/assets/graph/Kmeans.gz)。

