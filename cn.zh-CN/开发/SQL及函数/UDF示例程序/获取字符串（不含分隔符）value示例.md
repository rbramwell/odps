# 获取字符串（不含分隔符）value示例 {#concept_qby_2zw_tgb .concept}

本文为您介绍如何获取不包含分隔符的字符串中指定key对应的value值。

## 获取字符串（不含分隔符）value的UDF说明 {#section_eqm_rkc_5gb .section}

-   函数名：UDFKeyValue
-   功能：从字符串中获得指定`key`的`value`值。

    **说明：** 该UDF不适用于字符串本身包含分隔符的情况，如果需要处理该情况请使用[UDFKeyValueEx](cn.zh-CN/开发/SQL及函数/UDF示例程序/获取字符串（含有分隔符）value示例.md#)。

-   参数说明

    `UDFKeyValue(String str, String splitor1, String splitor2, String key)`：

    -   `str`：字符串。
    -   `splitor1`：通过`splitor1`分割出`key:value`对。
    -   `splitor2`：对分割出来的`key:value`对，使用`splitor2`进行分割。
    -   `key`：要获取的参数值。
    `UDFKeyValue(String keyValues, String split)`：

    -   `str`：字符串。
    -   `key`：要获取的参数值。`splitor1`为`;`，`splitor2`为`:`。

## UDF使用示例 {#section_pq4_rzc_5gb .section}

1.  注册函数

    UDFKeyValue.java测试通过后，将其注册函数。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/124620/156317069738827_zh-CN.png)

    **说明：** 一个UDF从发布到服务端供生产使用，需要经过打包、上传、注册这三个步骤。您可以使用一键发布功能一次性完成这些步骤（Studio会依次执行`mvn clean package`、上传Jar和注册UDF这三个步骤）。详情请参见[打包、上传和注册](../../../../cn.zh-CN/工具及下载/MaxCompute Studio/开发Java程序/打包、上传和注册.md#)。

2.  使用示例

    成功注册UDF后，执行以下命令：

    -   示例一

        ``` {#codeblock_jdi_umd_1kf .language-sql}
        select UDFKeyValue('a:1;b:2;','a') from dual;
        ```

        运行结果如下。

        ``` {#codeblock_mpx_iml_y7l .language-sql}
        +-----+
        | _c0 |
        +-----+
        | 1   |
        +-----+
        ```

    -   示例二

        ``` {#codeblock_fnu_6co_d8z .language-sql}
        select UDFKeyValue('a:1;b:2;','\;',':','a') from dual;
        ```

        运行结果如下。

        ``` {#codeblock_4bu_3zv_hme .language-sql}
        +-----+
        | _c0 |
        +-----+
        | 1   |
        +-----+
        ```


## UDF代码示例 {#section_zh3_mqc_5gb .section}

``` {#codeblock_5oe_bvr_fzx .language-java}
//package名称，可以根据您的情况定义。
package com.aliyun.odps.examples.udf;
import com.aliyun.odps.io.Text;
import com.aliyun.odps.udf.UDF;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class UDFKeyValue extends UDF{
        // 存储所有键：值对。
    private Map<String, String> mKeyValueache = new ConcurrentHashMap<String, String>();
    private Text result = new Text();

    public UDFKeyValue() {
    }

    public String evaluate(String str, String keyname){
        // 默认split1=; split2=:。
        return evaluate(str, ";", ":", keyname);
    }

    public String evaluate(String str, String split1, String split2, String keyname) {
        try {   
                        // 通过splitor1分割出key:value对。
            if (str == null || "".equals(str))
                return null;
            String[] values1 = str.split(split1);
            mKeyValueache.clear();
            int i = 0;
                        // 对分割出来的key:value对，使用splitor2进行分割。
            while (i < values1.length) {
                storeKeyValue(values1[i], split2);
                i++;
            }
            String resultValue = getKeyValue(keyname);
            if (resultValue == null)
                return null;
            result.set(new Text(resultValue));
                        // 获取结果值。
            return result.toString();
        } catch (Exception e) {
            return null;
        }
    }

    private boolean storeKeyValue(String keyValues, String split) {
        if (keyValues == null || "".equals(keyValues))
            return false;
        if (mKeyValueache == null)
            mKeyValueache = new ConcurrentHashMap<String, String>();
        String[] keyValueArr = keyValues.split(split);
        if (keyValueArr.length == 2) {
            mKeyValueache.put(keyValueArr[0], keyValueArr[1]);
            return true;
        }
        return false;
    }

    private String getKeyValue(String keyName) {
        if (keyName == null || 
            "".equals(keyName) || 
            mKeyValueache == null || 
            mKeyValueache.size() == 0)
            return null;
        return mKeyValueache.get(keyName);
    }
}
```

