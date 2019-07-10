# HAVING子句 {#concept_zgy_5kb_wdb .concept}

MaxCompute SQL的WHERE关键字无法与合计函数一起使用，此时您可以使用HAVING子句来实现。

命令格式如下。

``` {#codeblock_dt5_pip_rar}
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name
HAVING aggregate_function(column_name) operator value
```

例如有一张订单表`Orders`，包括客户名称（`Customer`），订单金额（`OrderPrice`），订单日期（`Order_date`），订单号（`Order_id`）四个字段。现在需要查找订单总额少于2000的客户，SQL语句如下所示。

``` {#codeblock_hsm_2qs_8x9}
SELECT Customer,SUM(OrderPrice) FROM Orders
GROUP BY Customer
HAVING SUM(OrderPrice)<2000
```

