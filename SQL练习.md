# 题目来源
https://leetcode-cn.com/problemset/database/

## 595. 大的国家
### 题目描述
这里有张 `World` 表

```
+-----------------+------------+------------+--------------+---------------+
| name            | continent  | area       | population   | gdp           |
+-----------------+------------+------------+--------------+---------------+
| Afghanistan     | Asia       | 652230     | 25500100     | 20343000      |
| Albania         | Europe     | 28748      | 2831741      | 12960000      |
| Algeria         | Africa     | 2381741    | 37100000     | 188681000     |
| Andorra         | Europe     | 468        | 78115        | 3712000       |
| Angola          | Africa     | 1246700    | 20609294     | 100990000     |
+-----------------+------------+------------+--------------+---------------+
```

如果一个国家的面积超过300万平方公里，或者人口超过2500万，那么这个国家就是大国家。

编写一个SQL查询，输出表中所有大国家的名称、人口和面积。

例如，根据上表，我们应该输出:

```
+--------------+-------------+--------------+
| name         | population  | area         |
+--------------+-------------+--------------+
| Afghanistan  | 25500100    | 652230       |
| Algeria      | 37100000    | 2381741      |
+--------------+-------------+--------------+
```

### 贴出代码

```sql
select name,population,area
from World
where area > 3000000 or population > 25000000;
```

## 596. 超过5名学生的课

### 题目描述

有一个`courses` 表 ，有: **student (学生)** 和 **class (课程)**。

请列出所有超过或等于5名学生的课。

例如,表:

```
+---------+------------+
| student | class      |
+---------+------------+
| A       | Math       |
| B       | English    |
| C       | Math       |
| D       | Biology    |
| E       | Math       |
| F       | Computer   |
| G       | Math       |
| H       | Math       |
| I       | Math       |
+---------+------------+
```

应该输出:

```
+---------+
| class   |
+---------+
| Math    |
+---------+
```

**Note:**
学生在每个课中不应被重复计算。

### 贴出代码

```sql
select class
from courses
group by class having count(distinct student) >= 5
```

SELECT DISTINCT语句

在表中，可能会包含重复值。这并不成问题，不过，有时您也许希望仅仅列出不同（distinct）的值。
## 182. 查找重复的电子邮箱
### 题目描述

编写一个 SQL 查询，查找 `Person` 表中所有重复的电子邮箱。

**示例：**

```
+----+---------+
| Id | Email   |
+----+---------+
| 1  | a@b.com |
| 2  | c@d.com |
| 3  | a@b.com |
+----+---------+
```

根据以上输入，你的查询应返回以下结果：

```
+---------+
| Email   |
+---------+
| a@b.com |
+---------+
```

**说明：**所有电子邮箱都是小写字母。

### 贴出代码

```sql
select Email
from Person
group by Email 
having count(Email) > 1
```

## 620. 有趣的电影
### 题目描述
某城市开了一家新的电影院，吸引了很多人过来看电影。该电影院特别注意用户体验，专门有个 LED显示板做电影推荐，上面公布着影评和相关电影描述。

作为该电影院的信息部主管，您需要编写一个 SQL查询，找出所有影片描述为非 boring (不无聊) 的并且 id 为奇数 的影片，结果请按等级 rating 排列。

 

例如，下表 cinema:
```
+---------+-----------+--------------+-----------+
|   id    | movie     |  description |  rating   |
+---------+-----------+--------------+-----------+
|   1     | War       |   great 3D   |   8.9     |
|   2     | Science   |   fiction    |   8.5     |
|   3     | irish     |   boring     |   6.2     |
|   4     | Ice song  |   Fantacy    |   8.6     |
|   5     | House card|   Interesting|   9.1     |
+---------+-----------+--------------+-----------+
```
对于上面的例子，则正确的输出是为：
```
+---------+-----------+--------------+-----------+
|   id    | movie     |  description |  rating   |
+---------+-----------+--------------+-----------+
|   5     | House card|   Interesting|   9.1     |
|   1     | War       |   great 3D   |   8.9     |
+---------+-----------+--------------+-----------+
```
### 贴出代码
```sql
select * 
from cinema
where description != 'boring' and id % 2 != 0 
order by rating desc
```

## 175. 组合两个表
### 题目描述
SQL架构
表1: `Person`
```
+-------------+---------+
| 列名         | 类型     |
+-------------+---------+
| PersonId    | int     |
| FirstName   | varchar |
| LastName    | varchar |
+-------------+---------+
PersonId 是上表主键
```
表2: `Address`
```
+-------------+---------+
| 列名         | 类型    |
+-------------+---------+
| AddressId   | int     |
| PersonId    | int     |
| City        | varchar |
| State       | varchar |
+-------------+---------+
AddressId 是上表主键
```
编写一个 SQL 查询，满足条件：无论 person 是否有地址信息，都需要基于上述两表提供 person 的以下信息：
> FirstName, LastName, City, State
### 分析
数据库在通过连接两张或多张表来返回记录时，都会生成一张中间的临时表，然后再将这张临时表返回给用户。 在使用left jion时，on和where条件的区别如下：

1、on条件是在生成临时表时使用的条件，它不管on中的条件是否为真，都会返回左边表中的记录。

2、where条件是在临时表生成好后，再对临时表进行过滤的条件。这时已经没有left join的含义（必须返回左边表的记录）了，条件不为真的就全部过滤掉。

via: https://www.cnblogs.com/toSeeMyDream/p/6843984.html
### 贴出代码
```sql
select p.FirstName, p.LastName,a.City,a.State
from Person as p left join Address a 
on p.PersonId = a.PersonId
```

## 627. 交换工资
### 题目描述
给定一个 salary表，如下所示，有m=男性 和 f=女性的值 。交换所有的 f 和 m 值(例如，将所有 f 值更改为 m，反之亦然)。要求使用一个更新查询，并且没有中间临时表。

例如:
```
| id | name | sex | salary |
|----|------|-----|--------|
| 1  | A    | m   | 2500   |
| 2  | B    | f   | 1500   |
| 3  | C    | m   | 5500   |
| 4  | D    | f   | 500    |
```
运行你所编写的查询语句之后，将会得到以下表:
```
| id | name | sex | salary |
|----|------|-----|--------|
| 1  | A    | f   | 2500   |
| 2  | B    | m   | 1500   |
| 3  | C    | f   | 5500   |
| 4  | D    | m   | 500    |
```
### 贴出代码
```sql
UPDATE salary
SET
    sex = CASE sex
        WHEN 'm' THEN 'f'
        ELSE 'm'
    END;
```

## 181. 超过经理收入的员工
### 题目描述
Employee 表包含所有员工，他们的经理也属于员工。每个员工都有一个 Id，此外还有一列对应员工的经理的 Id。
```
+----+-------+--------+-----------+
| Id | Name  | Salary | ManagerId |
+----+-------+--------+-----------+
| 1  | Joe   | 70000  | 3         |
| 2  | Henry | 80000  | 4         |
| 3  | Sam   | 60000  | NULL      |
| 4  | Max   | 90000  | NULL      |
+----+-------+--------+-----------+
```
给定 Employee 表，编写一个 SQL 查询，该查询可以获取收入超过他们经理的员工的姓名。在上面的表格中，Joe 是唯一一个收入超过他的经理的员工。
```
+----------+
| Employee |
+----------+
| Joe      |
+----------+
```
### 分析

### 贴出代码
```sql
SELECT 
    Name Employee
FROM
    Employee AS a
WHERE
    Salary > (SELECT 
            Salary
        FROM
            Employee
        WHERE
            Id = a.Managerid)
```

## 183. 从不订购的客户
### 题目描述
某网站包含两个表，Customers 表和 Orders 表。编写一个 SQL 查询，找出所有从不订购任何东西的客户。

Customers 表：
```
+----+-------+
| Id | Name  |
+----+-------+
| 1  | Joe   |
| 2  | Henry |
| 3  | Sam   |
| 4  | Max   |
+----+-------+
```
Orders 表：
```
+----+------------+
| Id | CustomerId |
+----+------------+
| 1  | 3          |
| 2  | 1          |
+----+------------+
```
例如给定上述表格，你的查询应返回：
```
+-----------+
| Customers |
+-----------+
| Henry     |
| Max       |
+-----------+
```
### 分析

### 贴出代码
```sql
select Name Customers
from Customers 
left join Orders
on Customers.Id = Orders.CustomerId
where Orders.CustomerId is null
```

## 196. 删除重复的电子邮箱
### 题目描述
编写一个 SQL 查询，来删除 Person 表中所有重复的电子邮箱，重复的邮箱里只保留 Id 最小 的那个。
```
+----+------------------+
| Id | Email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
| 3  | john@example.com |
+----+------------------+
```
`Id`是这个表的主键。
例如，在运行你的查询语句之后，上面的 Person 表应返回以下几行:
```
+----+------------------+
| Id | Email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
+----+------------------+
```
### 分析

### 贴出代码
```sql
DELETE p1 FROM Person p1,
    Person p2
WHERE
    p1.Email = p2.Email AND p1.Id > p2.Id
```
## 197. 上升的温度
### 题目描述
给定一个 Weather 表，编写一个 SQL 查询，来查找与之前（昨天的）日期相比温度更高的所有日期的 Id。
```
+---------+------------------+------------------+
| Id(INT) | RecordDate(DATE) | Temperature(INT) |
+---------+------------------+------------------+
|       1 |       2015-01-01 |               10 |
|       2 |       2015-01-02 |               25 |
|       3 |       2015-01-03 |               20 |
|       4 |       2015-01-04 |               30 |
+---------+------------------+------------------+
```
例如，根据上述给定的 Weather 表格，返回如下 Id:
```
+----+
| Id |
+----+
|  2 |
|  4 |
+----+
```
### 分析
### 贴出代码
```sql
select b.Id
from Weather a
left join
Weather b
on
date_add(a.RecordDate,interval 1 day) = b.RecordDate
where
a.Temperature < b.Temperature;
```
## 176. 第二高的薪水
### 题目描述
编写一个 SQL 查询，获取 Employee 表中第二高的薪水（Salary） 。
```
+----+--------+
| Id | Salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
```
例如上述 Employee 表，SQL查询应该返回 200 作为第二高的薪水。如果不存在第二高的薪水，那么查询应返回 null。
```
+---------------------+
| SecondHighestSalary |
+---------------------+
| 200                 |
+---------------------+
```
### 分析
select distinct salary from Employee order by salary desc limit 1,1 就可以，但是输不出null，所以外面再加一层 select (select distinct salary from Employee order by salary desc limit 1,1) as SecondHighestSalary ;
当Employee表里只有一条数据时，内层SQL语句查询不到数据，其返回结果是空，而外层SQL的作用是把内层的查询结果赋值给SecondHighestSalary.
### 贴出代码
```sql
select 
(select distinct salary 
from Employee 
 order by salary 
 desc limit 1,1) 
 as SecondHighestSalary ;
```

## 626. 换座位
### 题目描述
小美是一所中学的信息科技老师，她有一张 seat 座位表，平时用来储存学生名字和与他们相对应的座位 id。
其中纵列的 id 是连续递增的
小美想改变相邻俩学生的座位。
你能不能帮她写一个 SQL query 来输出小美想要的结果呢？
**示例：**
```
+---------+---------+
|    id   | student |
+---------+---------+
|    1    | Abbot   |
|    2    | Doris   |
|    3    | Emerson |
|    4    | Green   |
|    5    | Jeames  |
+---------+---------+
```
假如数据输入的是上表，则输出结果如下：
```
+---------+---------+
|    id   | student |
+---------+---------+
|    1    | Doris   |
|    2    | Abbot   |
|    3    | Green   |
|    4    | Emerson |
|    5    | Jeames  |
+---------+---------+
```
**注意：**
如果学生人数是奇数，则不需要改变最后一个同学的座位。
### 贴出代码
```sql

```