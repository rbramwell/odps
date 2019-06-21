# UDAF/UDTF动态参数说明 {#concept_778239 .concept}

MaxCompute的UDAF和UDTF曾经使用Resolve注解决定函数的Signature，示例如下。这种方式的局限性在于输入参数和输出参数是固定的，无法实现方法重载。

``` {#codeblock_pd8_wpt_id6}
@com.aliyun.odps.udf.annotation.Resolve("BIGINT->DOUBLE")
public class UDTFClass extends UDTF {
    ...
}
```

**说明：** 示例定义了一个UDTF，该UDTF接受的参数类型为BIGINT，返回值类型为DOUBLE。

现在MaxCompute对Resolve注解的语法做了如下扩展：

-   您可以在参数列表中使用`*`，表示接受任意长度、任意类型的输入参数。例如`@Resolve('double,*->String')`表示接受第一个是DOUBLE类型，后接任意长度、任意类型的参数列表。此时，您需要自己编写代码判断输入的个数和参数类型，然后对它们进行相应的操作（您可以对比C语言里面的`printf`函数来理解此操作）。

    **说明：** 星号用在返回值列表中时，表示的是不同的含义，详见下文。

-   您可以在参数列表中使用`any`关键字，表示任意类型的参数。例如`@Resolve('double,any->string')`表示接受第一个是DOUBLE类型，后接任意类型的参数列表。

    **说明：** `any`不能用在返回值列表中，也不能用在复杂类型的子类型中（例如ARRAY）。

-   UDTF的返回值可以使用星号，表示返回任意个STRING类型。返回值的个数与调用函数时设置的alias个数有关。例如`@Resolve("ANY,ANY->DOUBLE,*")`，调用方式是`UDTF(x, y) as (a, b, c)`，此处`as`后面设置了三个alias，即a、b、c。编辑器会认定a为DOUBLE类型（Annotation中返回值第一列的类型是给定的），b和c为STRING类型。因为这里给出了三个返回值，所以UDTF在调用`forward`时，必须`forward`长度为3的数组，否则会出现运行时报错。

    **说明：** 这种错误无法在编译时报出，因此UDTF的调用者在SQL中设置alias个数时，必须遵循该UDTF定义的规则。由于Aggregator的返回值个数固定是1，所以这个功能对UDAF来说并无意义。


UDTF示例

``` {#codeblock_wxq_gl9_jq5}
import com.aliyun.odps.udf.UDFException;
import com.aliyun.odps.udf.UDTF;
import com.aliyun.odps.udf.annotation.Resolve;
import org.json.JSONException;
import org.json.JSONObject;
@Resolve("STRING,*->STRING,*")
public class JsonTuple extends UDTF {
  private Object[] result = null;
  @Override
  public void process(Object[] input) throws UDFException {
    if (result == null) {
      result = new Object[input.length];
    }
    try {
      JSONObject obj = new JSONObject((String)input[0]);
      for (int i = 1; i < input.length; i++) {
        // 返回值要求变长部分都是STRING。
        result[i] = String.valueOf(obj.get((String)(input[i])));
      }
      result[0] = null;
    } catch (JSONException ex) {
      for (int i = 1; i < result.length; i++) {
        result[i] = null;
      }
      result[0] = ex.getMessage();
    }
    forward(result);
  }
}
```

以上UDTF示例中，返回值个数会根据输入参数的个数来决定。输出参数中的第一个参数是一个JSON文本，后面的参数是需要从JSON中解析的Key。返回值中的第一个返回值是解析JSON过程中的出错信息，如果没有出错，则会根据输入的Key依次输出从JSON中解析出来的内容。使用示例如下。

``` {#codeblock_xyn_v7y_wmm}
-- 根据输入参数的个数定制输出alias个数。
SELECT my_json_tuple(json, ’a‘, 'b') as exceptions, a, b FROM jsons;

-- 变长部分可以一列都没有。
SELECT my_json_tuple(json) as exceptions, a, b FROM jsons;

-- 下面这个SQL会出现运行时错误，因为alias个数与实际输出个数不符。
-- 注意编译时无法发现这个错误。
SELECT my_json_tuple(json, 'a', 'b') as exceptions, a, b, c FROM jsons;
```

当本文介绍的这些扩展无法满足您的业务需求时，建议您使用UDT实现Aggregator和UDTF的功能。详情请参见[UDT](cn.zh-CN/开发/SQL及函数/UDT.md#)文档中的[聚合操作的实现示例](cn.zh-CN/开发/SQL及函数/UDT.md#section_irn_2y9_mty)和[表值函数的实现示例](cn.zh-CN/开发/SQL及函数/UDT.md#section_lmh_m8r_pxj)。

