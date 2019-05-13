# Learn to write SQL statements quickly {#concept_jxd_4qw_5db .concept}

This article introduces you to maxcompute SQL through the way of course practice, allows you to quickly understand how SQL is written, and you know the difference between maxcompute SQL and standard SQL, please read in conjunction with the [MaxCompute SQL base documentation](../../../../reseller.en-US/User Guide/SQL/SQL summary.md).

## Prepare a dataset {#section_cxt_3rw_5db .section}

In the example the Emp/Dept table is used as the dataset. For the convenience of everybody's operation, provides the relevant maxcompute build table statements and data files \([emp table data files](http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/attach/51009/cn_zh/1489374509403/emp%E6%95%B0%E6%8D%AE.txt), [dept table data file](http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/attach/51009/cn_zh/1488265664915/dept%E8%A1%A8%E6%95%B0%E6%8D%AE%E6%96%87%E4%BB%B6.txt)\), where you can create a table and upload the data yourself on the maxcompute project.

The DDL statements for creating an emp table are as follows:

```


CREATE TABLE IF NOT EXISTS emp (
  EMPNO string,
  ENAME string ,
  JOB string ,
  MGR bigint ,
  HIREDATE datetime ,
  SAL double ,
  COMM double ,
  DEPTNO bigint );
```

The DDL statements for creating a Dept table are as follows:

```


CREATE TABLE IF NOT EXISTS dept (
  DEPTNO bigint ,
  DNAME string,
  LOC string);
```

## SQL {#section_hcy_nvw_5db .section}

**Notes for SQL beginners** 

-   If you use Group by, the Select part must be either group items or aggregate functions.

-   Order by must be followed by Limit n.

-   The Select expression does not support subqueries. You can rewrite the code to the Join clause to use subqueries.

-   The Join clause does not support Cartesian projects and usage of MapJoin.

-   Union all must use the format of subqueries.

-   The In/Not in statements can correspond to only one column of subquery, and the number of rows of the returned results cannot exceed 1,000. Otherwise, rewrite the statements using Join.


 **Compile SQL programs to solve problems** 

**Example 1: List all departments that have at least one employee.**

To avoid a situation where the amount of data is too large, you often encounter 6th points in the **problem point** , you need to override using join. For example:

```


SELECT d. *
FROM dept d
JOIN (
    SELECT the DISTINCT deptno AS no
    FROM emp
) e
on d.deptno = e.no;
```

**Example 2: List all employees who have higher salaries than Smith.**

This example is a typical scenario of MapJoin, as shown in the following code:

```


SELECT /*+ MapJoin(a) */ e.empno
    , e.ename
    , e.sal
FROM emp e
JOIN (
    SELECT MAX(sal) AS sal
    FROM 'emp'
    WHERE `ENAME` = 'SMITH'
) a
ON e.sal> a.sal;
```

**Example 3: List the name and the immediate superior’s name of all employees.**

Use non-equi join, as shown in the following code:

```


SELECT a.ename
    , b.ename
FROM emp a
LEFT OUTER JOIN emp b
ON b.empno = a.mgr;
```

**Example 4: List all jobs of which salaries are higher than 1500 yuan.**

Use Having, as shown in the following code:

```


SELECT emp. 'JOB'
    , MIN(emp.sal) AS sal
FROM 'emp'
GROUP BY emp. 'JOB'
HAVING MIN(emp.sal) > 1500;
```

**Example 5: List the number of employees of each department, and their average salary and average service year.**

You can use many built-in functions for time processing, as shown in the following code:

```


SELECT COUNT(empno) AS cnt_emp
    , ROUND(AVG(sal), 2) AS avg_sal
    , ROUND(AVG(datediff(getdate(), hiredate, 'dd')), 2) AS avg_hire
FROM 'emp'
GROUP BY 'DEPTNO';
```

**Example 6: List the names of first three employees who have the highest salaries and their ranks \(Top n is frequently used\).**

The SQL statements are as follows:

```


SELECT *
FROM (
  SELECT deptno
    , ename
    ,sal
    , ROW_NUMBER() OVER (PARTITION BY deptno ORDER BY sal DESC) AS nums
  FROM emp
) emp1
WHERE emp1.nums < 4;
```

**Title 7: write down the number of people in each department in one SQL, clerk \(clerk\) the total number of people in the department accounted**.

The SQL statement is as follows:

```


SELECT deptno
    , COUNT(empno) AS cnt
    , ROUND(SUM(CASE 
      WHEN job = 'CLERK' THEN 1
      ELSE 0
    END) / COUNT(empno), 2) AS rate
FROM `EMP`
GROUP BY deptno;
```

