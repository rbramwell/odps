# PyODPS节点实现结巴中文分词 {#task_1079380 .task}

本文为您介绍如何使用DataWorks的PyODPS类型节点，借助开源结巴中文分词包实现对中文字段的分词并写入新的表。

-   请首先确保您已经完成DataWorks工作空间的创建，本例使用**简单模式**的工作空间，详情请参见[创建工作空间](../../../../cn.zh-CN/准备工作/管理员使用云账号/创建工作空间.md#)。完成工作空间创建后，单击**进入数据开发**。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/868788/156376562551113_zh-CN.png)

-   请在GitHub[下载](https://github.com/fxsjy/jieba)开源结巴分词中文包。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/868788/156376562551112_zh-CN.png)


PyODPS集成了Maxcompute的Python SDK。您可以在DataWorks的PyODPS节点上直接编辑Python代码并使用Maxcompute的Python SDK。关于PyODPS节点的详情请参见[PyODPS节点](../../../../cn.zh-CN/使用指南/数据开发/节点类型/PyODPS节点.md#)。

1.  创建业务流程。 
    1.  右键单击**业务流程**，选择**新建业务流程**。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/868788/156376562551114_zh-CN.png)


    2.  输入您的业务流程名称后，单击**新建**。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/868788/156376562551115_zh-CN.png)


2.  上传jieba-master.zip包。 
    1.  右键单击**资源**，选择**Archive**。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/868788/156376562651121_zh-CN.png)


    2.  上传您已下载到本地的jieba-master.zip，勾选**上传为ODPS资源**，单击**确定**。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/868788/156376562651122_zh-CN.png)


    3.  提交资源。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/868788/156376562651211_zh-CN.png)


3.  创建测试数据表。 
    1.  右键单击**表**，选择**新建表**。输入表名jieba\_test。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/868788/156376562651124_zh-CN.png)


    2.  单击**DDL模式**，输入建表DDL语句如下。 本教程准备了两列测试数据，您在后续开发过程中可以选择一列进行分词。

        ``` {#codeblock_ruk_28g_nht}
        CREATE TABLE `jieba_test` (
            `chinese` string,
            `content` string
        );
        ```

    3.  单击**提交到生产环境**。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/868788/156376562651128_zh-CN.png)


4.  创建测试结果存放表。 本例仅对测试数据的chinese列做分词处理，因此结果表仅有一列，创建方法同上。DDL语句如下所示。

    ``` {#codeblock_t3s_ia1_64l}
    CREATE TABLE `jieba_result` (
        `chinese` string
    ) ;
    ```

5.  上传测试数据。 本例已为您准备好分词测试数据，请点击此处[下载](http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/attach/124882/cn_zh/1562725154626/jieba_test.csv)。
    1.  单击**导入**。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/868788/156376562651152_zh-CN.png)


    2.  输入测试数据表名jieba\_test，单击**下一步**。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/868788/156376562751153_zh-CN.png)


    3.  单击**浏览**，上传您下载到本地的jieba\_test.csv文件，单击**下一步**。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/868788/156376562751154_zh-CN.png)


    4.  勾选**按名称匹配**，单击**导入数据**。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/868788/156376562751155_zh-CN.png)


6.  创建PyODPS节点。 
    1.  在业务流程中右键单击**数据开发**，选择**新建数据开发节点** \> **PyODPS**。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/868788/156376562751205_zh-CN.png)


    2.  输入节点名称word\_split。
    3.  输入您的PyODPS代码。 输入代码如下，释义请见代码注释。

        ``` {#codeblock_l9x_n4s_c5e .language-python}
        def test(input_var):
            import jieba
            import sys
            reload(sys)
            sys.setdefaultencoding('utf-8')
            result=jieba.cut(input_var, cut_all=False)
            return "/ ".join(result)
        hints = {
            'odps.isolation.session.enable': True
        }
        libraries =['jieba-master.zip']  #引用您的jieba-master.zip压缩包。
        iris = o.get_table('jieba_test').to_df()  #引用您的jieba_test表中的数据。
        example = iris.chinese.map(test).execute(hints=hints, libraries=libraries)
        print(example)  #查看分词结果，分词结构为MAP类型数据。
        abci=list(example)   #将分词结果转为list类型数据。
        i = 0
        for i in range(i,len(abci)):
            pq=str(abci[i])
            o.write_table('jieba_result',[pq])  #通过循环，将数据逐条写入您的结果表jieba_result中。
            i+=1
        else:
            print("done")
        ```

    4.  运行代码进行测试。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/868788/156376562851216_zh-CN.png)


    5.  您可以在**运行日志**查看结巴分词的程序运行结果。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/868788/156376562851217_zh-CN.png)


7.  在**数据开发**创建一个**ODPS SQL**类型节点，输入select \* from jieba\_result，单击运行。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/868788/156376562851218_zh-CN.png)


8.  查看运行结果，验证数据是否已写入结果表中。![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/868788/156376562851219_zh-CN.png)



