# Operators {#concept_bll_pfl_vdb .concept}

Operators are used to perform program code operations. This article introduces four types of operators: relational operator, arithmetic operator, bit operator and logical operator.

## Relational operators {#section_dlk_lgl_vdb .section}

|Operator|Description|
|:-------|:----------|
|A=B|If A or B is NULL, NULL is returned. If A is equal to B, TRUE is returned; otherwise FALSE is returned.|
|A<\>B|If A or B is NULL, NULL is returned. If A is not equal to B, TRUE is returned; otherwise FALSE is returned.|
|A<B|If A or B is NULL, NULL is returned. If A is less than B, TRUE is returned; otherwise FALSE is returned.|
|A<=B|If A or B is NULL, NULL is returned. If A is not greater than B, TRUE is returned; otherwise FALSE is returned.|
|A\>B|If A or B is NULL, NULL is returned. If A is greater than B, TRUE is returned; otherwise FALSE is returned.|
|A\>=B|If A or B is NULL, NULL is returned; if A is not less than B, TRUE is returned; otherwise, FALSE is returned.|
|A IS NULL|If A is NULL, TRUE is returned; otherwise, FALSE is returned.|
|A IS NOT NULL|If A is NULL, TRUE is returned; otherwise FALSE is returned.|
|A LIKE B|If A or B is NULL, NULL is returned. If String A matches the SQL simple regular B TRUE is returned; otherwise FALSE is returned. The \( %\) character in B matches an arbitrary number of characters and the \(\_\) character in B matches any character in A. To match \(%\) or\(\_\), use by the escape characters \('%'\)'and \('\_'\). ``` {#codeblock_qc3_lh1_56j}
‘aaa’ like‘a_’= TRUE 
‘aaa’ like‘a%’ = TRUE
‘aaa’ like‘aab’= FALSE 
‘a%b’ like‘a\\%b’= TRUE 
‘axb’ like ‘a\\%b’= FALSE               
```

 |
|A RLIKE B|A is a string, and B is a string constant regular expression. If any substring of A matches the Java regular expression B, TRUE is returned; otherwise FALSE is returned. If expression B is empty, report an error and exit. If expression A or B is NULL, NULL is returned.|
|A IN B|B is a set. If expression A is NULL, NULL is returned. If expression A is in expression B, TRUE is returned; otherwise FALSE is returned. If expression B has only one element NULL, that is, A IN \(NULL\), return NULL. If expression B contains NULL element, take NULL as the type of other elements in B set. B must be a constant and at least has one element; all types must be consistent.|
|BETWEEN AND|The expression is `A [NOT] BETWEEN B AND C`. Empty if A, B, or C is empty. TRUE if A is larger than or equal to B and less than or equal to C; otherwise FALSE is returned.|

The common use:

``` {#codeblock_q9s_w9s_wum}
select * from user where user_id = '0001'; 
select * from user where user_name <> 'maggie'; 
select * from user where age > ‘50’; 
select * from user where birth_day >= '1980-01-01 00:00:00'; 
select * from user where is_female is null; 
select * from user where is_female is not null; 
select * from user where user_id in (0001,0010); 
select * from user where user_name like 'M%';
```

The Double values in MaxCompute are different in precision. For this reason, we do not recommend using the equal sign for comparison between two Double data. You can subtract two Double types, and then take the absolute value into consideration. When the absolute value is small enough, the two double values are considered equal.

**Example:** 

``` {#codeblock_eqv_sba_jw4}
abs(0.9999999999 - 1.0000000000) < 0.000000001
 -- 0.9999999999 and 1.0000000000 have the precision of 10 decimal digits, while 0.000000001 has the precision of 9 decimal digits.
 -- It is considered that 0.9999999999 is equal to 1.0000000000.
```

**Note:** 

-   ABS is a built-in function provided by MaxCompute to take absolute value. For more information, see [ABS](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md).
-   In general, the DOUBLE type in MaxCompute can retain 14-bit decimal.

## Arithmetic operators {#section_ycc_chl_vdb .section}

|Operator|Description|
|:-------|:----------|
|A + B|If expression A or B is NULL, NULL is returned; otherwise the result of A+B is returned.|
|A – B|If expression A or B is NULL, NULL is returned; otherwise the result of A – B is returned.|
|A \* B|If expression A or B is NULL, NULL is returned; otherwise result of A \* B is returned.|
|A / B|If expression A or B is NULL, NULL is returned; otherwise the result of A / B is returned. If Expression A and B are BIGINT types, the result is DOUBLE type.|
|A % B|If expression A or B is NULL, NULL is returned; otherwise the reminder result from dividing A by B is returned.|
|+A|Result A is returned.|
|-A|If expression A is NULL, NULL is returned; otherwise –A is returned.|

The common use:

``` {#codeblock_g0r_vos_bk3}
select age+10, age-10, age%10, -age, age*age, age/10 from user;
```

**Note:** 

-   You can only use STRING, BIGINT, and DOUBLE to perform arithmetic operations. \(Using Datetime type and Boolean type is restricted.\)
-   Before you begin these operations, the type STRING is converted into DOUBLE by implicit type conversion.
-   If BIGINT and DOUBLE both are involved in arithmetic operation, the type BIGINT is converted into DOUBLE by implicit type conversion.
-   When A and B are BIGINT types, the return result of A/B will be a DOUBLE type. For other arithmetic operations, the return value is also a BIGINT type.

## Bitwise operators {#section_abw_cs4_gof .section}

|Operator|Description|
|:-------|:----------|
|A & B|Return the result of bitwise AND of A and B. For example: 1&2, return 0; 1&3, return 1; Bitwise AND of NULL and other values, all return NULL. Expression A and B must be BIGINT .|
|A | B|Return the result of bitwise OR of A and B. For example: 1|2, return3. 1|3, return 3. Bitwise OR of NULL and other values, all return NULL. Expression A and B must be BIGINT type.|

**Note:** Bitwise operator does not support implicit conversions, only supports the type BIGINT.

## Logical operators {#section_qcm_g6o_o17 .section}

|Operator|Description|
|--------|-----------|
|A and B|TRUE and TRUE=TRUE|
|TRUE and FALSE=FALSE|
|FALSE and TRUE=FALSE|
|FALSE and NULL=FALSE|
|NULL and FALSE=FALSE|
|TRUE and NULL=NULL|
|NULL and TRUE=NULL|
|NULL and NULL=NULL|
|A or B|TRUE or TRUE=TRUE|
|TRUE or FALSE=TRUE|
|FALSE or TRUE=TRUE|
|FALSE or NULL=NULL|
|NULL or FALSE=NULL|
|TRUE or NULL=TRUE|
|NULL or TRUE=TRUE|
|NULL or NULL=NULL|
|NOT A|If A is NULL, NULL is returned.|
|If A is TRUE, FALSE is returned.|
|If A is FALSE, TRUE is returned.|

**Note:** Only the type Boolean can be involved in logic operations and the implicit type conversion is not supported.

