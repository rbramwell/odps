# 设置SQL的Flag {#concept_388480 .concept}

本文为您介绍如何使用MaxCompute Java SDK设置SQL的Flag。

使用DataWorks或MaxCompute Console提交SQL时，通常需要设置SQL的Flag。如果需要使用MaxCompute新数据类型，通过Session级别方式开启，则需要在涉及新数据类型的SQL前加Set Flag语句：`set odps.sql.type.system.odps2=true;`。

使用SDK提交SQL时，不能简单地把Set Flag语句直接放到SQL Query中执行。以Java SDK为例，设置Flag的正确方式如下。

``` {#codeblock_utj_je6_zjf}
// 构造SQLTask对象。
SQLTask task = new SQLTask();
task.setName("foobar");
task.setQuery("select ...");
// 设置flag。
Map<String, String> settings = new HashMap<>();
settings.put("odps.sql.type.system.odps2", "true");
...  // 设置其它flags。
task.setProperty("settings", new JSONObject(settings).toString());  // 这里是关键：将flags对应的json string设置到settings property中。
Instance instance = odps.instances().create(task);  // 执行。
```

