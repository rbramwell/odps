---
keyword: code-embedded UDF
---

# Code-embedded UDFs

This topic describes how to use code-embedded user-defined functions \(UDFs\) to embed Java or Python code into SQL scripts.

## Background information

Code-embedded UDFs of MaxCompute resolve the following issues in code implementation and maintenance:

-   Complicated code implementation: After you create UDFs and develop code, you must compile the code in Java and create resources and functions.
-   Inconvenient code maintenance: You cannot directly view the implementation logic of UDFs that are referenced in SQL scripts or obtain the source code of JAR packages.
-   Poor code readability: To implement Java library functions by using user-defined type \(UDT\), you must write Java code as expressions in long code lines. In addition, you may fail to write some Java code as expressions. Example:

    ```
    Foo f = new Foo();
    f.execute();
    f.getResult();
    ```


## Features

Code-embedded UDFs allow you to embed Java or Python code into SQL scripts. When you compile a script, Janino-compiler identifies and extracts the embedded code, compiles the code in Java, and then dynamically generates resources and creates temporary functions.

You can place SQL scripts and third-party code lines in the same source code file. This simplifies the usage of UDT or UDFs and facilitates daily development and maintenance.

## Limits

You can use only Janino-compiler to compile embedded Java code, and the syntax of the embedded Java code must be a subset of the standard JDK syntax. Embedded Java code has the following limits:

-   It does not support lambda expressions.
-   You cannot specify multiple types of exceptions in a single catch block. For example, `catch(Exception1 | Exception2 e)` is not allowed.
-   It cannot automatically infer generic arguments. For example, `Map map = new HashMap<>();` is not supported.
-   Expressions for type argument inference are ignored. You must use cast expressions to specify the argument type, for example, `(String) myMap.get(key)`.
-   Assertions are forcibly enabled, even if the -ea option of the Java virtual machine \(JVM\) is used.
-   Code that is programmed in Java 8 or later versions is not supported.

## Reference embedded code in UDT

You must submit and execute the script in Script Mode SQL. For more information, see [Script Mode SQL](/intl.en-US/Development/SQL/Script Mode SQL.md). Sample code:

```
SELECT 
  s, 
  com.mypackage.Foo.extractNumber(s) 
FROM VALUES ('abc123def'),('apple') AS t(s);

#CODE ('lang'='JAVA')
package com.mypackage;
import java.util.regex.Matcher;
import java.util.regex.Pattern;
public class Foo {
  final static Pattern compile = Pattern.compile(". *?([ 0-9]+).*");
  public static String extractNumber(String input) {
    final Matcher m = compile.matcher(input);
    if (m.find()) {
      return m.group(1);
    }
    return null;
  }
}
#END CODE;
```

-   `#CODE` and `#END CODE` indicate the beginning and end of the embedded code block, respectively. In this example, the embedded code block is placed at the end of the script, and applies to the whose script.
-   `'lang'='JAVA'` indicates that the embedded code is in Java. JAVA can be replaced with `PYTHON`.
-   You can use UDT syntax in the SQL script to call `Foo.extractNumber`.

## Define and call a Java code-embedded UDF

Sample code:

```
CREATE TEMPORARY FUNCTION foo AS 'com.mypackage.Reverse' USING
#CODE ('lang'='JAVA')
package com.mypackage;
import com.aliyun.odps.udf.UDF;
public class Reverse extends UDF {
  public String evaluate(String input) {
    if (input == null) return null;
    StringBuilder ret = new StringBuilder();
    for (int i = input.toCharArray().length - 1; i >= 0; i--) {
      ret.append(input.toCharArray()[i]);
    }
    return ret.toString();
  }
}
#END CODE;

SELECT foo('abdc');
```

-   You can place the embedded code block next to `USING` or at the end of the script. If you place it next to `USING`, it applies only to the `CREATE TEMPORARY FUNCTION` statement.
-   The function created by `CREATE TEMPORARY FUNCTION` is a temporary function. This temporary function is executed only during the current execution process and is not stored in the MaxCompute meta system.

## Define and call a Java code-embedded UDTF

Sample code:

```
CREATE TEMPORARY FUNCTION foo AS 'com.mypackage.Reverse' USING 
#CODE ('lang'='JAVA', 'filename'='embedded.jar')
package com.mypackage;

import com.aliyun.odps.udf.UDTF;
import com.aliyun.odps.udf.UDFException;
import com.aliyun.odps.udf.annotation.Resolve;

@Resolve({"string->string,string"})
public class Reverse extends UDTF {
  @Override
  public void process(Object[] objects) throws UDFException {
    String str = (String) objects[0];
    String[] split = str.split(",");
    forward(split[0], split[1]);
  }
}

#END CODE;

SELECT foo('ab,dc') AS (a,b);
```

The return value of `@Resolve` must be of the `string[]` type. However, Janino-compiler cannot identify `"string->string,string"` in embedded code as `string[]`. To enable Janino-compiler to identify "string-\>string,string" as string\[\], "string-\>string,string" must be enclosed in braces `{}`.`` When you create a Java UDTF by using a common method, braces `{}` are not required.

## Define and call a Python code-embedded UDF

Sample code:

```
CREATE TEMPORARY FUNCTION foo AS 'embedded.UDFTest' USING
#CODE ('lang'='PYTHON', 'filename'='embedded')
from odps.udf import annotate
@annotate("bigint->bigint")
class UDFTest(object):
  def evaluate(self, a):
    return a * a
#END CODE;

SELECT foo(4);
```

-   The indentation of Python code must comply with Python language specifications.
-   When you register a Python UDF, the class name that follows the `AS` clause must contain the file name of the Python source code. You can use `'filename'='embedded'` to specify a virtual file name.

