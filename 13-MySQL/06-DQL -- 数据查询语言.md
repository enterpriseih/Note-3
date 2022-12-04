# DQL -- 数据查询语言

 查询不会修改数据库表记录！

## 基本查询

### 字段(列)控制

#### 查询所有列

```
 SELECT * FROM 表名;

 SELECT * FROM emp;
```

 --> 其中“*”表示查询所有列

<img src="img/wpsjErqFU.jpg" alt="img" style="zoom: 15%;" /> 

#### 查询指定列

```
 SELECT 列1 [, 列2, ... 列N] FROM 表名;

 SELECT empno, ename, sal, comm FROM 表名;
```

<img src="img/wps8okSy8.jpg" alt="img" style="zoom: 15%;" /> 

#### 完全重复的记录只一次

 当查询结果中的多行记录一模一样时，只显示一行。一般查询所有列时很少会有这种情况，但只查询一列（或几列）时，这种可能就大了！

```
 SELECT DISTINCT * | 列1 [, 列2, ... 列N] FROM 表名;

 SELECT DISTINCT sal FROM emp;
```

 --> 保查询员工表的工资，如果存在相同的工资只显示一次！

<img src="img/wpsBc8eSb.jpg" alt="img" style="zoom:10%;" /> 

 

<img src="img/wpsTady42.jpg" alt="img" style="zoom:10%;" /> 

 

<img src="img/wpsKHV5bn.jpg" alt="img" style="zoom:10%;" /> 

#### 列运算

#####  I 数量类型的列可以做加、减、乘、除运算

```
 SELECT sal*1.5 FROM emp;

 SELECT sal+comm FROM emp;
```

<img src="img/wps3YmxFp.jpg" alt="img" style="zoom:15%;" /> 

 

<img src="img/wpsFOdA1R.jpg" alt="img" style="zoom:15%;" /> 

 

当需要运算的列无法进行整数运算的时候会自动当做 0 处理

<img src="img/wpsFC05cX.jpg" alt="img" style="zoom:15%;" /> 

 

#####  II 字符串类型可以做连续运算

```
SELECT CONCAT('$', sal) FROM emp;
```

<img src="img/wpsJjji42.jpg" alt="img" style="zoom:15%;" />  

SQL中不能使用 + 来链接字符串，会当做无法转换的东西用 0 代替 

<img src="img/wpsLdPTgu.jpg" alt="img" style="zoom:15%;" /> 

 

<img src="img/wps9w1FsS.jpg" alt="img" style="zoom:15%;" /> 

 

<img src="img/wpsqqvaF6.jpg" alt="img" style="zoom:15%;" /> 

#####  III 转换NULL值

  有时需要把NULL转换成其它值，例如com+1000时，如果com列存在NULL值，那么NULL+1000还是NULL，而我们这时希望把NULL当前0来运算。 

<img src="img/wps5wrihx.jpg" alt="img" style="zoom:15%;" /> 

任何列和 null 相加运算都为 null 

<img src="img/wpsbsN3Hn.jpg" alt="img" style="zoom:15%;" /> 

```
 SELECT IFNULL(comm, 0)+1000 FROM emp;
```

  --> IFNULL(comm, 0)：如果comm中存在NULL值，那么当成0来运算。

<img src="img/wpsYM9f9Q.jpg" alt="img" style="zoom:15%;" /> 

 

<img src="img/wpsO3WVgK.jpg" alt="img" style="zoom:15%;" /> 

#####  IV 给列起别名

  你也许已经注意到了，当使用列运算后，查询出的结果集中的列名称很不好看，这时我们需要给列名起个别名，这样在结果集中列名就显示别名了

```
SELECT IFNULL(comm, 0)+1000 AS 奖金 FROM emp;
```

<img src="img/wpsg0iHqW.jpg" alt="img" style="zoom:15%;" /> 

 

<img src="img/wpsydm5vo.jpg" alt="img" style="zoom:15%;" /> 

 

  --> 其中AS可以省略

<img src="img/wps64KSc9.jpg" alt="img" style="zoom:15%;" /> 

 

 

### 条件控制

#### 条件查询

 与前面介绍的UPDATE和DELETE语句一样，SELECT语句也可以使用WHERE子句来控制记录。

 * ```
    * SELECT empno,ename,sal,comm FROM emp WHERE sal > 10000 AND comm IS NOT NULL;
   
    * SELECT empno,ename,sal FROM emp WHERE sal BETWEEN 20000 AND 30000;
   
    * SELECT empno,ename,job FROM emp WHERE job IN ('经理', '董事长');
   ```

   


<img src="img/wps5gE1fq.jpg" alt="img" style="zoom:15%;" /> 

 

<img src="img/wpsYanFVt.jpg" alt="img" style="zoom:15%;" /> 

 

<img src="img/wpsp9fYhl.jpg" alt="img" style="zoom:15%;" /> 

 

<img src="img/wpsNKFhA7.jpg" alt="img" style="zoom:15%;" /> 

 

<img src="img/wps4mnFpP.jpg" alt="img" style="zoom:15%;" /> 

 

 

#### 模糊查询

 当你想查询姓张，并且姓名一共两个字的员工时，这时就可以使用模糊查询

 * ```
   SELECT * FROM emp WHERE ename LIKE '张_';
   ```

   

<img src="img/wpsuSsKIf.jpg" alt="img" style="zoom:15%;" /> 

 --> 模糊查询需要使用运算符：LIKE，其中_匹配一个任意字符，注意，只匹配一个字符而不是多个。

 --> 上面语句查询的是姓张，名字由两个字组成的员工。

 * ```
   SELECT * FROM emp WHERE ename LIKE '___'; /*姓名由3个字组成的员工*/
   ```

三个下滑线 ename  like '_ _ _'  ,表示名字是三个字的员工

<img src="img/wpsqEzTnl.jpg" alt="img" style="zoom:15%;" /> 

 如果我们想查询姓张，名字几个字可以的员工时就要使用“%”了。

```
SELECT * FROM emp WHERE ename LIKE '张%';
```

<img src="img/wps9mO4dr.jpg" alt="img" style="zoom:15%;" />  

<img src="img/wpsCh6NGm.jpg" alt="img" style="zoom:15%;" /> 

 --> 其中%匹配0~N个任意字符，所以上面语句查询的是姓张的所有员工。

```
SELECT * FROM emp WHERE ename LIKE '%阿%';
```

<img src="img/wpsImagkr.jpg" alt="img" style="zoom:15%;" /> 

 --> 千万不要认为上面语句是在查询姓名中间带有阿字的员工，因为%匹配0~N个字符，所以姓名以阿开头和结尾的员工也都会查询到。 

```
SELECT * FROM emp WHERE ename LIKE '%';
```

 --> 这个条件等同与不存在，但如果姓名为NULL的查询不出来！

## 排序

### 升序

```
 SELECT * FROM emp ORDER BY sal ASC;
```

 --> 按sal排序，升序！

<img src="img/wpsRZUw2S.jpg" alt="img" style="zoom:15%;" /> 

 --> 其中ASC是可以省略的，建议打出来

<img src="img/wpspLliYq.jpg" alt="img" style="zoom:15%;" /> 

### 降序

```
SELECT * FROM emp ORDER BY comm DESC;
```

 --> 按comm排序，降序！

 --> 其中DESC不能省略

<img src="img/wpsOrEUHH.jpg" alt="img" style="zoom:15%;" /> 

### 使用多列作为排序条件

```
SELECT * FROM emp ORDER BY sal ASC, comm DESC;
```

 --> 使用sal升序排，如果sal相同时，使用comm的降序排

<img src="img/wpsKzqe5N.jpg" alt="img" style="zoom:15%;" /> 

 

<img src="img/wpsgde28A.jpg" alt="img" style="zoom:15%;" /> 

## 聚合函数

 聚合函数用来做某列的纵向运算。

### COUNT

```
 SELECT COUNT(*) FROM emp;
```

 --> 计算emp表中所有列都不为NULL的记录的行数

<img src="img/wpsDpXwNL.jpg" alt="img" style="zoom:10%;" /> 

 SELECT COUNT(comm) FROM emp;

 --> 计算emp表中comm列不为NULL的记录的行数

<img src="img/wpsd5kB7A.jpg" alt="img" style="zoom:10%;" />  

<img src="img/wps720pvl.jpg" alt="img" style="zoom:10%;" /> 

数字和 * 效果完全相同

<img src="img/wpsXco0X9.jpg" alt="img" style="zoom:10%;" /> 

### MAX

```
 SELECT MAX(sal) FROM emp;
```

 --> 查询最高工资

<img src="img/wpscb4ItG.jpg" alt="img" style="zoom:10%;" /> 

### MIN

```
 SELECT MIN(sal) FROM emp;
```

 --> 查询最低工资

<img src="img/wps6FIqeS.jpg" alt="img" style="zoom:10%;" /> 

### SUM

```
 SELECT SUM(sal) FROM emp;
```

 --> 查询工资合

<img src="img/wps2u2OMm.jpg" alt="img" style="zoom:10%;" /> 

 

自动把 null 当成 0 计算了

<img src="img/wpsPzgjYd.jpg" alt="img" style="zoom:10%;" /> 

把不能运算的当做 0 计算

<img src="img/wpsRTYEgm.jpg" alt="img" style="zoom:10%;" /> 

### AVG

```
SELECT AVG(sal) FROM emp;
```

 --> 查询平均工资

<img src="img/wpsgquLbA.jpg" alt="img" style="zoom:7%;"/>  

<img src="img/wpsG2JGEq.jpg" alt="img" style="zoom:15%;" /> 

 

## 分组查询

 分组查询是把记录使用某一列进行分组，然后查询组信息。

 例如：查看所有部门的记录数。

```
SELECT deptno, COUNT(*) FROM emp GROUP BY deptno;
```

 --> 使用deptno分组，查询部门编号和每个部门的记录数，分组列必须在查询列中出现，其他的必须是聚合函数 ，如下图中的 job 列

<img src="img/wpswfILa8.jpg" alt="img" style="zoom:10%;" /> 

```
 SELECT job, MAX(SAL) FROM emp GROUP BY job;
```

 --> 使用job分组，查询每种工作的最高工资

<img src="img/wps4W0QFx.jpg" alt="img" style="zoom:10%;" /> 

 

<img src="img/wpsk7tfiN.jpg" alt="img" style="zoom:10%;" /> 

 

<img src="img/wpsumVQQQ.jpg" alt="img" style="zoom:10%;" /> 

 

###  组条件

 以部门分组，查询每组记录数。条件为记录数大于3

```
 SELECT deptno, COUNT(*) FROM emp GROUP BY deptno HAVING COUNT(*) > 3;
```

<img src="img/wps3flpCW.jpg" alt="img" style="zoom:10%;" /> 

## limit子句(方言)

 LIMIT用来限定查询结果的起始行，以及总行数。

 例如：查询起始行为第5行，一共查询3行记录

```
 SELECT * FROM emp LIMIT 4, 3;
```

 --> 其中4表示从第5行开始，其中3表示一共查询3行。即第5、6、7行记录。

![img](img/wpsUPgJbP.jpg)  

总共 15 行，查询出几行显示几行

<img src="img/wpsu3vwBn.jpg" alt="img" style="zoom:10%;" /> 

```
select * from emp limit 0, 5;
```

 

 表示从 0 行记录开始，查询 5 行记录 

 1. 一页的记录数：10行

 2. 查询第3页 

```
select * from emp limit 20, 10; 
```

 (当前页-1) * 每页记录数

 (3-1) * 10

 (17-1) * 8, 8 

==============================

书写的顺序和执行顺序如下：

```
select

from

where

group by

having

order by

desc

limit
```

# 练习

1. 查询出部门编号为30的所有员工

2. 所有销售员的姓名、编号和部门编号。

3. 找出奖金高于工资的员工。

<img src="img/wps7LVKMA.jpg" alt="img" style="zoom:10%;" /> 

4. 找出奖金高于工资60%的员工。

5. 找出部门编号为10中所有经理，和部门编号为20中所有销售员的详细资料。

6. 找出部门编号为10中所有经理，部门编号为20中所有销售员，还有即不是经理又不是销售员但其工资大或等于20000的所有员工详细资料。

<img src="img/wpsv1Kf4w.jpg" alt="img" style="zoom:15%;" /> 

8. 无奖金或奖金低于1000的员工。

9. 查询名字由三个字组成的员工。

10.查询2000年入职的员工。

<img src="img/wpsYCvqE3.jpg" alt="img" style="zoom:10%;" /> 

11. 查询所有员工详细信息，用编号升序排序

12. 查询所有员工详细信息，用工资降序排序，如果工资相同使用入职日期升序排序

13. 查询每个部门的平均工资

<img src="img/wpsLB0l5a.jpg" alt="img" style="zoom:10%;" /> 

14. 查询每个部门的雇员数量。 

15. 查询每种工作的最高工资、最低工资、人数

<img src="img/wpslCrlgn.jpg" alt="img" style="zoom:10%;" /> 

  

16. 显示非销售人员工作名称以及从事同一工作雇员的月工资的总和，并且要满足从事同一工作的雇员的月工资合计大于50000，输出结果按月工资的合计升序排列

17. 有奖金的工种。