---
title: SQL学习小记
date: 2019-09-12 22:45:00
category: 分享境
tags:
  - SQL
---
最近在学习数据库这方面的知识，学数据库的话就离不开SQL了。

所以想整理一下知识点，写成笔记咯~
## SQL 语法

顺便提一下，在 SQL 中，语句不分大小写。

所以我们用 `SELECT` 与 `select` 效果是一样的。

分号是在数据库系统中分隔每条 SQL 语句的标准方法，这样就可以在对服务器的相同请求中执行一条以上的 SQL 语句。

SQL 使用单引号来环绕文本值（大部分数据库系统也接受双引号）。如果是数值字段，请不要使用引号。

所以要记得加分号，这是个好习惯

## 基础语句

SQL的基础语句还是蛮多的= =！
### SELECT 语句

#### 定义

> SELECT 语句用于从数据库中选取数据。  
> 结果被存储在一个结果表中，称为结果集。

#### 语法

```sql
SELECT column_name,column_name
FROM table_name;
```

#### 与

```sql
SELECT * FROM table_name;
```

#### 例子

```sql
--下面的 SQL 语句从 "Students" 表中选取 "StuID" 和 "StuName" 列：
SELECT StuID,StuName
FROM Students;

--下面的 SQL 语句是选择 "Students" 表中所有的列：
SELECT * FROM Students;
```

### SELECT DISTINCT 语句

#### 定义

> 在表中，一个列可能会包含多个重复值，有时您也许希望仅仅列出不同（distinct）的值。  
> DISTINCT 关键词用于返回唯一不同的值。

#### 语法

```sql
SELECT DISTINCT column_name,column_name
FROM table_name;
```

#### 例子

```sql
--下面的 SQL 语句仅从 "Students" 表的 "StuID" 列中选取唯一不同的值,也就是去掉 "StuID" 列重复值：
SELECT DISTINCT StuID FROM Students;
```

### WHERE 语句

#### 定义

> 在表中，WHERE 子句用于提取那些满足指定条件的记录。

#### 语法

```sql
SELECT column_name,column_name
FROM table_name
WHERE column_name operator value;
```

#### WHERE 子句中的运算符

|运算符|描述|
|---|---|
|`=`|等于|
|`!=`|不等于|
|`<>`|不等于|
|`>`|大于|
|`<`|小于|
|`>=`|大于等于|
|`<=`|小于等于|
|`BETWEEN`|在某个范围内|
|`LIKE`|搜索某种模式|
|`IN`|指定针对某个列的多个可能值|
#### 例子

```sql
--下面的 SQL 语句从 "Students" 表中选取班级为 "软件工程" 的所有学生：
SELECT * FROM Students WHERE class='软件工程';

--下面的 SQL 语句从 "Students" 表中选取 "StuID"为 1 的学生：
SELECT * FROM Students WHERE StuID=1;
```