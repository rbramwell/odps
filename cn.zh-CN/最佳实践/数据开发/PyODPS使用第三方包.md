# PyODPS使用第三方包 {#task_1340288 .task}

本文为您介绍如何在PyODPS中使用第三方包。

请提前开通MaxCompute和DataWorks服务。

1.  下载资源包。 
    1.  单击[此处](https://help.aliyun.com/document_detail/90716.html#title-20s-f7m-o1c)找到图示画面，并下载这6个文件。![资源包](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1068732/156403717652827_zh-CN.jpg)


    2.  下载后的文件如下。![本地资源包](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1068732/156403717752837_zh-CN.png)


2.  在DataWorks中进行资源的创建与提交。 
    1.  新建Archive资源。![新建资源](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1068732/156403717752886_zh-CN.jpg)


    2.  单击**点击上传**，选择python-dateutil-2.6.0.zip文件。![点击上传资源](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1068732/156403717752923_zh-CN.jpg)


    3.  将资源名称重命名为python.dateutil.zip，单击**确定**。![资源重命名](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1068732/156403717752940_zh-CN.jpg)


    4.  单击提交按钮。![提交资源](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1068732/156403717852944_zh-CN.jpg)


    5.  按照上述步骤完成资源pytz.zip、six.tar.gz、pandas.zip、sklearn.zip、scipy.zip的创建与提交。![上传完毕](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1068732/156403717852945_zh-CN.jpg)


3.  新建PyODPS节点，并将节点命名为第三方包的使用。 ![新建节点](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1068732/156403717852863_zh-CN.jpg)

 代码如下。

    ``` {#codeblock_ckc_7px_7x5}
    import sys
    import numpy as np
    import pandas as pd
    from pandas import Series,DataFrame
    from dateutil.parser import parse
    reload(sys)
    #修改系统默认编码。
    sys.setdefaultencoding('utf8')
    
    #第一个示例:对Series操作。
    array_01=Series([2,4,7,8,9,3])
    print array_01.values
    print array_01.index
    
    # 创建对数据点进行标记的索引。
    array_02=Series([2,4,7,8,9,3],index=['a','b','c','d','e','f'])
    print array_02['c']
    print array_02[['a','c','f']]
    
    #第二个示例:对pandas的DataFrame操作。
    data = {'name': ['Bob', 'Anny', 'Jack', 'Tom'], 'sex': ['M', 'F', 'M', 'F'], 'age': [11, 14, 15, 17],'birthday':['2015-06-13','2009-07-12','2018-04-02','1998-09-09']}
    frame = DataFrame(data)
    print frame
    frame1=DataFrame(data,columns=['name','sex','age'])
    print frame1
    
    #指定index为行的索引。
    frame2=DataFrame(data,columns=['name','sex','age','birthday'],index=['a','b','c','d'])
    print frame2
    print frame2['sex']
    print frame1.age
    
    #第三个示例，日期的截取。
    def get_year(t):
        return parse(t).strftime('%Y')
    print frame2.birthday.map(get_year)
    ```

4.  单击运行按钮。![运行](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1068732/156403717952956_zh-CN.jpg)


5.  在**运行日志**中查看运行结果。![运行日志](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/1068732/156403717952957_zh-CN.jpg)



