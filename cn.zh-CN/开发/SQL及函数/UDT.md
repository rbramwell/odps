# UDT {#concept_drb_xk4_hfb .concept}

MaxCompute基于新一代的SQL引擎再推出新功能User Defined Type（UDT）。MaxCompute的UDT功能允许您在SQL中直接引用第三方语言的类或对象，获取其数据内容或调用其方法。

## 应用场景 {#section_wbd_xp4_hfb .section}

UDT的常用场景如下：

-   场景1：某些可以用其它语言实现的功能，例如只需调用一次Java内置类的方法即可实现的功能，MaxCompute的内置函数却没有简单的方法实现那些功能。如果使用UDF实现，整个过程会过于繁杂。
-   场景2：SQL中需要调用第三方库实现相关功能。希望能够在SQL中直接调用，而不需要再Wrap一层UDF。
-   场景3：[Select Transform](cn.zh-CN/开发/SQL及函数/Select语句/Select Transform语法.md#)支持把脚本写到SQL语句中，提升可读性和代码易维护性。但是某些语言无法这样使用，例如Java源代码必须经过编译才能执行，希望类似这些语言也可以直接写到SQL中。

## UDT概述 {#section_drd_zp4_hfb .section}

其它的SQL引擎中UDT的概念不同于MaxCompute UDT的概念。很多SQL引擎中的概念比较像MaxCompute的STRUCT复杂类型，而某些语言提供了调用第三方库的功能，例如Oracle的Create Type。相比之下，MaxCompute的UDT更像Create Type的概念，Type中不仅包含数据域，还包含方法。而且MaxCompute不需要用特殊的DDL语法来定义类型的映射，而是在SQL中直接使用。

示例如下：

``` {#codeblock_qqw_u25_b9t}
set odps.sql.type.system.odps2=true;    
-- 打开新类型，因为下面的操作会用到INTEGER，即INT类型。
SELECT java.lang.Integer.MAX_VALUE;
```

上述语句的输出结果如下。

``` {#codeblock_mwc_6p8_wgm}
+-----------+
| max_value |
+-----------+
| 2147483647 |
+-----------+
```

和Java语言一样，java.lang包可以省略，所以上述示例可以简写为如下语句。

``` {#codeblock_g8v_54a_9lz}
set odps.sql.type.system.odps2=true;
SELECT Integer.MAX_VALUE;
```

可以看到，上述示例在Select列表中直接写上了类似于Java表达式的表达式，而这个表达式按照Java的语义来执行，这个表达式表现出的能力就是MaxCompute的UDT。

UDT所提供的所有扩展能力可以用UDF实现。如果您想使用UDF实现上述示例，请进行以下操作。

1.  定义一个UDF的类。

    ``` {#codeblock_4na_qt9_gnj}
    package com.aliyun.odps.test;
    public class IntegerMaxValue extends com.aliyun.odps.udf.UDF {
      public Integer evaluate() {
         return Integer.MAX_VALUE;
      } 
    }
    ```

2.  将上面的UDF编译，并打成Jar包。然后上传Jar包，并创建Function。

    ``` {#codeblock_qqq_dg0_qvu}
    add jar odps-test.jar;
    create function integer_max_value as 'com.aliyun.odps.test.IntegerMaxValue' using 'odps-test.jar';
    ```

3.  在SQL中使用。

    ``` {#codeblock_6f4_oxe_1kr}
    select integer_max_value();
    ```

    UDT简化了上述一系列的过程，方便您使用其它语言扩展SQL的功能。


## 实现原理 {#section_hgw_cn4_hfb .section}

概述中的示例体现了Java静态域访问的能力，UDT的能力远不限于此。以下示例为您介绍UDT的执行过程和具体功能。

``` {#codeblock_dju_67w_1kz}
-- 示例数据。
@table1 := select * from values ('100000000000000000000') as t(x);
@table2 := select * from values (100L) as t(y);
-- 代码逻辑。
@a := select new java.math.BigInteger(x) x from @table1;          -- new创建对象。
@b := select java.math.BigInteger.valueOf(y) y from @table2;      -- 静态方法调用。
select /*+mapjoin(b)*/ x.add(y).toString() from @a a join @b b;   -- 实例方法调用。
```

输出结果如下所示。

``` {#codeblock_3yx_dq8_i8w}
100000000000000000100
```

上述示例还体现了一种用UDF比较不好实现的功能：子查询的结果允许UDT类型的列。例如上面变量a的x列是`java.math.BigInteger`类型，而不是内置类型。UDT类型的数据可以被带到下一个Operator中再调用其它方法，甚至可以参与数据Shuffle。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/22183/156137027813239_zh-CN.png)

如上图可知，该UDT共有三个Stage：M1、R2和J3。如果您熟悉MapReduce原理便会知道，由于`join`的存在需要做数据Reshuffle，所以会出现多个Stage。通常，不同的Stage是在不同的进程、不同的物理机器上运行的。

查看M1，可以看到M1只执行了`new java.math.BigInteger(x)`操作。

查看J3，可以看到J3在不同阶段执行了`java.math.BigInteger.valueOf(y)`和`x.add(y).toString()`操作。这几个操作不仅分阶段执行，而且在不同的进程、不同的物理机器上执行。UDT把这个过程封装起来，将这个过程变得看起来和在同一个JVM中执行的效果几乎一样。

详细功能说明：

-   目前UDT仅支持Java语言，后续版本会加入对其它语言的支持。
-   UDT同样允许您上传自己的Jar包，并且直接引用。当前提供了一些Flag以方便您使用。
    -   `set odps.sql.session.resources`指定引用的资源，可以指定多个，用逗号隔开，例如`set odps.sql.session.resources=foo.sh,bar.txt;`。

        **说明：** 这个Flag和[Select Transform](https://help.aliyun.com/document_detail/73719.html)中指定资源的Flag相同，所以这个Flag会同时影响两个功能。 例如UDT概述中UDF的Jar包，用UDT来使用。

        ``` {#codeblock_rie_cq3_ciw}
        set odps.sql.type.system.odps2=true;
        set odps.sql.session.resources=odps-test.jar; 
        --指定要引用的Jar，这些Jar一定要事先上传到Project，并且需要是Jar类型的资源。
        select new com.aliyun.odps.test.IntegerMaxValue().evaluate();
        ```

    -   `odps.sql.session.java.imports`指定默认的Java Package，可以指定多个，用逗号隔开。和Java的`import`语句类似，可以提供完整类路径，例如`java.math.BigInteger`，也可以使用`*`。暂不支持`static import`。

        UDT概述中UDF的Jar包，用UDT使用还可以有如下写法。

        ``` {#codeblock_p9y_msz_a2y}
        set odps.sql.type.system.odps2=true;
        set odps.sql.session.resources=odps-test.jar;
        set odps.sql.session.java.imports=com.aliyun.odps.test.*;  
        -- 指定默认的Package。
        select new IntegerMaxValue().evaluate();
        ```

-   UDT支持的操作如下：

    -   实例化对象的`new`操作。
    -   实例化数组的`new`操作，包括使用初始化列表创建数组，例如`new Integer[] { 1, 2, 3 }`。
    -   方法调用，包括静态方法调用（因此能用工厂方法构建对象）。
    -   域访问，包括静态域。
    **说明：** 

    -   仅支持公有方法和公有域的访问。
    -   UDT中的标识符是大小写敏感的，包括Package、类名、方法名和域名。
    -   UDT支持类型转换，支持SQL的风格，例如`cast(1 as java.lang.Object)`。但不支持Java风格的类型转换，例如`(Object)1`。
    -   暂不支持匿名类和Lambda表达式（后续版本可能会支持）。
    -   暂不支持无返回值的函数调用（因为UDT均出现在Expression中，没有返回值的函数调用无法嵌入到Expression中，这个问题在后续的版本中会有解决方案）。
-   Java SDK的类都是默认可用的。

    **说明：** 目前Runtime使用的JDK版本是JDK1.8，可能不支持更新版本的JDK。

-   所有的运算符都是MaxCompute SQL的语义，不是UDT的语义。例如`String.valueOf(1) + String.valueOf(2)`的结果是3（STRING隐式转换为DOUBLE，并且DOUBLE相加） ，而不是12（Java中STRING相加是Concatenate的语义）。

    除了STRING的相加操作比较容易混淆外，另一个比较容易混淆的是`=`操作。SQL中的`=`不是赋值而是判断相等。而对于Java对象来说，判断相等应该用Equals方法，而非`=`操作（在UDT场景下，同一对象的概念是不能保证的，详情请参见下文的说明）。

-   内置类型与特定Java类型有一一映射关系，详情请参见[Java UDF](cn.zh-CN/开发/SQL及函数/UDF/Java UDF.md#)中的数据类型映射表，这个映射关系在UDT也有效。

    -   内置类型的数据能够直接调用其映射到的Java类型的方法，例如`'123'.length() , 1L.hashCode()`。
    -   UDT类型能够直接参与内置函数或者UDF的运算， 例如`chr(Long.valueOf('100'))`，其中`Long.valueOf`返回的是`java.lang.Long`类型的数据，而内置函数`Chr`接受的数据类型是内置类型BIGINT。
    -   Java的PRIMITIVE类型可以自动转化为其BOXING类型，并应用前两条规则。
    **说明：** 部分内置的[新数据类型](../../../../cn.zh-CN/开发/数据类型.md#)需要先设置`set odps.sql.type.system.odps2=true;`才可使用，否则会报错。

-   UDT对泛型有比较完整的支持，例如`java.util.Arrays.asList(new java.math.BigInteger('1'))`，编译器可以根据参数类型判断出该方法的返回值是`java.util.List<java.math.BigInteger>`类型。

    **说明：** 构造函数需要指定类型参数，否则使用`java.lang.Object`，这一点和Java保持一致。`new java.util.ArrayList(java.util.Arrays.asList('1', '2'))`的结果是`java.util.ArrayList<Object>`类型，而`new java.util.ArrayList<String>(java.util.Arrays.asList('1', '2'))`的结果是`java.util.ArrayList<String>`类型。

-   UDT对同一对象的概念是模糊的。这是由数据的Reshuffle导致的。从第一部分`join`的示例可以看出，对象有可能会在不同进程、不同物理机器之间传输，在传输过程中同一个对象的两个引用后面可能分别引用了不同的对象（例如对象先被Shuffle到两台机器，然后下次又Shuffle回一起）。

    所以在使用UDT时，应该使用`equals`方法判断相等，避免使用`=`判断相等。

    某行某列的对象，其内部包含的各个数据对象的相关性是可以保证的。不同行或者不同列的对象的数据相关性是不保证的。

-   目前UDT不能用作Shuffle Key，包括`join`、`group by`、`distribute by`、`sort by`、`order by`、cluster by等结构的Key。

    并不是说UDT不能用在这些结构中，UDT可以在Expression中间的任意阶段使用，但不能作为最终输出。例如，不可以使用语句`group by new java.math.BigInteger('123')`，但可以使用语句`group by new java.math.BigInteger('123').hashCode()`。因为`hashCode`方法的返回值是`int.class`类型，可以当做内置类型INT来使用（应用上述内置类型与特定Java类型规则）。

-   UDT扩展了类型转换规则：
    -   UDT对象可以被隐式类型转换为其基类对象。
    -   UDT对象可以被强制类型转换为其基类或子类对象。
    -   没有继承关系的两个对象之间遵守原来的类型转换规则，注意这时可能会导致内容的变化。例如`java.lang.Long`类型的数据是可以强制转换为`java.lang.Integer`的，应用的是内置类型的BIGINT强制转换为INT的过程，而这个过程会导致数据内容的变化，甚至可能导致精度的损失。
-   目前UDT对象不能落盘，这意味着不能将UDT对象insert到表中（实际上DDL不支持UDT，创建不出这样的表），当然，隐式类型转换变成了内置类型的除外。同时，屏显的最终结果也不能是UDT类型，对于屏显的场景，由于所有的Java类都有`toString()`方法，而`java.lang.String`类型是合法的。所以Debug时，可以用这种方法观察UDT的内容。

    您也可以设置`set odps.sql.udt.display.tostring=true;`，这样MaxCompute会自动帮您把所有以UDT为最终输出的列Wrap上`java.util.Objects.toString(...)`，以方便调试。这个Flag只对屏显语句生效，对Insert语句不生效，所以它只用在调试的过程中。

    内置类型支持BINARY，因此支持自己实现序列化的过程，将byte\[\]的数据落盘。下次读出时再还原回来。

    某些类可能自带序列化和反序列化的方法，例如`Protobuffer`。目前UDT依旧不支持落盘，需要您自己调用序列化反序列化方法，变成BINARY数据类型来落盘。

-   UDT不仅可以实现Scalar函数的功能，配合内置函数[collect list](cn.zh-CN/开发/SQL及函数/内建函数/聚合函数.md#)和[explode](cn.zh-CN/开发/SQL及函数/内建函数/其他函数.md#)，UDT还可以实现Aggregator和Table Function功能。
-   UDT支持资源（Resource）的访问，您可以在SQL中通过`com.aliyun.odps.udf.impl.UDTExecutionContext.get()`静态方法获取ExecutionContext对象，从而访问当前的Execution Context，进而访问资源（例如文件资源和表格资源）。

## 使用Java数组示例 {#section_0d3_sbx_khg .section}

``` {#codeblock_t28_5b0_pnu}
set odps.sql.type.system.odps2=true;
set odps.sql.udt.display.tostring=true;
select
    new Integer[10],    -- 创建一个10个元素的数组。
    new Integer[] {c1, c2, c3},  -- 通过初始化列表创建一个长度为3的数组。
    new Integer[][] { new Integer[] {c1, c2}, new Integer[] {c3, c4} },  -- 创建多维数组。
    new Integer[] {c1, c2, c3} [2], -- 通过下标操作访问数组元素。
    java.util.Arrays.asList(c1, c2, c3);    -- 这个创建了一个List<Integer>，这个也能当做array<int>来用，所以这是另一个创建内置ARRAY数据的方法。
from values (1,2,3,4) as t(c1, c2, c3, c4);
```

## 使用JSON示例 {#section_35n_gm2_zxe .section}

UDT的Runtime自带一个JSON的依赖（2.2.4），因此可以直接使用JSON。

``` {#codeblock_ahw_n1p_sn9}
set odps.sql.type.system.odps2=true;
set odps.sql.session.java.imports=java.util.*,java,com.google.gson.*; -- 同时import多个Package时用逗号隔开。
@a := select new Gson() gson;   -- 构建gson对象。
select 
gson.toJson(new ArrayList<Integer>(Arrays.asList(1, 2, 3))), -- 将任意对象转成JSON字符串。
cast(gson.fromJson('["a","b","c"]', List.class) as List<String>) -- 反序列化JSON字符串, 注意gson的接口，直接反序列化出来是List<Object>类型，所以这里强转成了List<String>，方便后续使用。
from @a;
```

相比于内置函数[get\_json\_object](cn.zh-CN/开发/SQL及函数/内建函数/字符串函数.md#)，上述用法不仅使用方便，在需要对JSON字符串多个部分做内容提取时，先将JSON字符串反序列成格式化数据，提升工作效率。

除JSON外，MaxCompute Runtime自带的依赖还包括：`commons-logging（1.1.1）`、`commons-lang（2.5）`、`commons-io（2.4）`、`protobuf-java（2.4.1）`。

## 复杂类型操作示例 {#section_yxt_p98_96x .section}

内置类型ARRAY和MAP与`java.util.List`和`java.util.Map`存在映射关系。结果如下：

-   Java中实现了`java.util.List`或者`java.util.Map`接口类的对象，都可参与MaxCompute SQL的复杂类型操作。
-   MaxCompute中ARRAY、MAP的数据，能够直接调用List或者MAP的接口。

``` {#codeblock_uwz_is6_g09}
set odps.sql.type.system.odps2=true;
set odps.sql.session.java.imports=java.util.*;
select
    size(new ArrayList<Integer>()),        -- 对ArrayList数据调用内置函数Size。
    array(1,2,3).size(),                   -- 对内置类型ARRAY调用List的方法。
    sort_array(new ArrayList<Integer>()),  -- 对ArrayList的数据进行排序。
    al[1],                                 -- 虽然Java的List不支持下标操作，但ARRAY支持。
    Objects.toString(a),        -- 之前不支持将ARRAY类型cast成STRING，现在有绕过方法了。
    array(1,2,3).subList(1, 2)             -- 求出subList。
from (select new ArrayList<Integer>(array(1,2,3)) as al, array(1,2,3) as a) t;
```

## 聚合操作的实现示例 {#section_irn_2y9_mty .section}

UDT实现聚合的原理是，先用内置函数[collect\_set](cn.zh-CN/开发/SQL及函数/内建函数/聚合函数.md#)或[collect\_list](cn.zh-CN/开发/SQL及函数/内建函数/聚合函数.md#)函数将数据转变成List，之后对该List应用UDT的标量方法求得这一组数据的聚合值。

下述示例实现对BigInteger求中位数（由于数据是`java.math.BigInteger`类型的，所以不能直接用内置的[median](cn.zh-CN/开发/SQL及函数/内建函数/聚合函数.md#)函数）。

``` {#codeblock_32v_850_sqo}
set odps.sql.session.java.imports=java.math.*;
@test_data := select * from values (1),(2),(3),(5) as t(value);
@a := select collect_list(new BigInteger(value)) values from @test_data;  -- 先把数据聚合成List。
@b := select sort_array(values) as values, values.size() cnt from @a;  -- 求中位数的逻辑，先将数据排序。
@c := select if(cnt % 2 == 1, new BigDecimal(values[cnt div 2]), new BigDecimal(values[cnt div 2 - 1].add(values[cnt div 2])).divide(new BigDecimal(2))) med from @b;
-- 最终结果。
select med.toString() from @c;
```

由于`collect_list`会先把所有数据都收集到一块，是没有办法实现Partial Aggregate的，所以这个做法的效率会比内置的Aggregator或者UDAF低，所以在内置Aggregator能实现的情况下，应尽量使用内置的Aggregator。同时把一个Group的所有数据都收集到一起，会增加数据倾斜的风险。

但是另一方面，如果UDAF本身的逻辑就是要将所有数据收集到一块（例如类似内置函数`wm_concat`的功能），此时使用上述方法，反而可能比UDAF（注意不是内置Aggregator）高。

## 表值函数的实现示例 {#section_lmh_m8r_pxj .section}

表值函数允许输入多行多列数据，输出多行多列数据。可以按照下述原理实现：

1.  对于输入多行多列数据，可以参考聚合函数实现的示例。
2.  要实现多行的输出，可以让UDT方法输出一个Collection类型的数据（List或者MAP\)，然后调用explode函数，将Collections展开成多行。
3.  UDT本身就可以包含多个数据域，通过调用不同的getter方法获取各个域的内容即可展开成多列。

下述示例实现将一个JSON字符串的内容展开的功能。

``` {#codeblock_kxe_fz7_d00}
@a := select '[{"a":"1","b":"2"},{"a":"1","b":"2"}]' str; -- 示例数据。
@b := select new com.google.gson.Gson().fromJson(str, java.util.List.class) l from @a; -- 反序列化JSON。
@c := select cast(e as java.util.Map<Object,Object>) m from @b lateral view explode(l) t as e;  -- 用explode打成多行。
@d := select m.get('a') as a, m.get('b') as b from @c; -- 展开成多列。
select a.toString() a, b.toString() b from @d; -- 最终结果输出（注意变量d的输出中a, b两列是Object类型）。
```

## 函数重载实现示例 {#section_uap_og3_qgs .section}

MaxCompute的UDF使用重载`evaluate`方法的方式来重载函数。这种方式不支持泛型，所以当您需要定义一个可以接受任何数据类型的函数时，必须为每种类型都写一个`evaluate`函数。但是，这种方法依然无法实现个别输入类型（例如ARRAY）的重载函数。在没有提供Resolve注解（Annotation）的情况下，Python UDF或UDTF会根据参数个数决定输入参数，同时支持变长参数，但这种过于灵活的机制也会导致编译器无法静态找到某些错误。

通过UDT实现函数重载，可以很好地解决以上问题。 UDT支持泛型、类继承、变长参数，为您提供灵活的函数定义方式，示例如下。

``` {#codeblock_zhr_iyc_ybb}
public class UDTClass {
    // 这个函数接受一个数值类型（可以是TINYINT、SMALLINT、INT、BIGINT、FLOAT、DOUBLE以及任何以Number为基类的UDT)，返回DOUBLE。
    public static Double doubleValue(Number input) {
        return input.doubleValue();
    }
    // 这个方法，接受一个数值类型参数和一个任意类型的参数，返回值类型与第二个参数的类型相同。
    public static <T extends Number, R> R nullOrValue(T a, R b) {
        return a.doubleValue() > 0 ? b : null;
    }
    // 这个方法接受一个任意元素类型的ARRAY或List，返回BIGINT。
    public static Long length(java.util.List<? extends Object> input) {
        return input.size();
    }
    // 注意这个在不做强制转换的情况下参数只能接受UDT的java.util.Map<Object, Object>对象。如果需要传入任何MAP对象，例如map<bigint,bigint>可以考虑:
    // 1. 定义函数时使用java.util.Map<? extends Object, ? extends Object>。
    // 2. 调用时强转，例如UDTClass.mapSize(cast(mapObj as java.util.Map<Object, Object>))。
    public static Long mapSize(java.util.Map<Object, Object> input) {
        return input.size();
    }
}
```

特定场景下，UDF需要通过`com.aliyun.odps.udf.ExecutionContext`（在`setup`方法中传入）获取一些上下文；UDT也可以通过`com.aliyun.odps.udt.UDTExecutionContext.get()`方法获取这样的一个`ExecutionContext`对象。

## 功能/性能/安全性 {#section_mxl_mq4_hfb .section}

UDT在功能方面的优势如下：

-   使用简单，无需定义任何Function。
-   支持JDK的所有功能，扩展了SQL的能力。
-   代码可与SQL放于同一文件，便于管理。
-   可直接使用其它类库，代码重用率高。
-   可以使用面向对象的思想设计某些功能。

后续待完善功能如下：

-   支持无返回值的函数调用，或支持（有返回值但忽略返回值）直接取操作数本身的函数调用。例如，调用List的`add`方法会返回执行完`add`操作的List。
-   支持匿名类和Lambda表达式。
-   支持用作Shuffle Key。
-   支持Java外的其他语言，例如Python。

因为UDT和UDF的执行过程非常接近，所以UDT与UDF的性能几乎一致。优化后的计算引擎使得UDT在特定场景下的性能更高。

-   UDT对象只有在跨进程时才需要做序列化和反序列化，因此在执行不需要数据Reshuffle的操作（如`join`或`ggregate`）时，UDT可节省序列化和反序列化的开销。
-   因为UDT的Runtime基于Codegen而非反射实现的，所以不存在反射带来的性能损失。在您使用过程中，连续多个UDT操作会合并在一个FunctionCall里一起执行。例如在之前的示例中，`values[x].add(values[y]).divide(java.math.BigInteger.valueOf(2))`实际上只会发生一次的UDT方法调用。所以，UDT操作的单元虽然较小，却并不会因多次函数调用而造成接口额外开销。

在安全控制方面，UDT和UDF完全一样，都会受到[Java沙箱](cn.zh-CN/开发/Java沙箱.md#)Policy的限制。所以如果要使用受限的操作，需要打开沙箱隔离，或者申请沙箱白名单。

